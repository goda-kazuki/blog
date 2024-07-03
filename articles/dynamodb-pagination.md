---
title: "DynamoDBの大量データ取得を簡単にするget_paginatorの使用例"
emoji: "😺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: true
---

## はじめに

Amazon DynamoDB を使ってデータを取得する時、対象のデータ量が多すぎると一度では取得することができません。
<https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/Scan.html#Scan.Pagination>

公式ページには、 LastEvaluatedKey があればループして再度データを取得するように案内されています。

その案内通りループしてもいいのですが、boto3にはget_paginatorという便利な関数があるので、それを使ってみようと思います。
<https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb/client/get_paginator.html>

## 普通にLastEvaluatedKeyの有無でループする

```Python
def scan_without_paginate():
    dynamodb = boto3.client("dynamodb")

    kwarg = {"TableName": "sample_paginator"}
    data = []

    while True:
        response = dynamodb.scan(**kwarg)
        data.extend(response["Items"])

        if "LastEvaluatedKey" not in response:
            break
        kwarg["ExclusiveStartKey"] = response["LastEvaluatedKey"]

    print(len(data))
```

まあ、単純といえば単純な実装ですね。

## get_paginatorを使って取得する

```Python
def scan_with_paginate():
    dynamodb = boto3.client("dynamodb")
    paginator = dynamodb.get_paginator("scan")

    data = []
    for page in paginator.paginate(TableName="sample_paginator"):
        data.extend(page["Items"])

    print(len(data))
```

whileがforになっただけで、ループしてることは変わらないのでリクエスト回数自体は変わりません。
ただ、かなりシンプルになったので理解しやすいと思います。

page変数には、scanした結果が格納されているので加工することも可能です。

## おまけ

scanだけでなく、queryについてもget_paginatorを使用することが可能です。

```Python
def query_with_paginate():
    dynamodb = boto3.client("dynamodb")
    paginator = dynamodb.get_paginator("query")

    data = []
    for page in paginator.paginate(
        TableName="sample_paginator",
        KeyConditionExpression="#partition = :partition",
        ExpressionAttributeNames={"#partition": "partition"},
        ExpressionAttributeValues={":partition": {"S": "1"}},
    ):
        data.extend(page["Items"])

    print(len(data))
```

## おわり

get_paginatorはDynamoDBだけでなく、S3やTimestreamなど他のサービスにも存在する関数です。

1回に取得できるデータ量はどれも制限があるので、これを使って綺麗に書いていきましょう。
