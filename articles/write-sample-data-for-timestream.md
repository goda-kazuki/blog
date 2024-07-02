---
title: "Amazon Timestreamへのデータ投入をPythonで簡単に実現！"
emoji: "👏"
type: "tech"
topics: ["AWS","Timestream","Python","boto3"]
published: true
---

## はじめに

Amazon Timestream(以降はTimestreamと書きます)を使う機会が少しずつ増えてきました。
開発中はモックしたりすることもありますが、テストのタイミングではAWS上のTimestreamに対してリクエストしたいことがあると思います。

DynamoDBであれば、S3バケットにCSVやJSONファイルを格納し、インポートすることができますが、Timestreamにはそういった機能がありません。

今回は、Timestreamに対してboto3を使ってサンプルデータを登録したいと思います。

## ファイル内容

### Python

```Python
import boto3
import json
from datetime import datetime, timedelta

# Timestreamのクライアントを作成
my_session = boto3.Session(profile_name="default")
timestream_write_client = my_session.client("timestream-write")

# JSONファイルを読み込み
file_path = "write_timestream.json"
with open(file_path, "r") as file:
    data = json.load(file)

# Timestreamへの書き込み
database_name = "sample-db-name"
table_name = "sample-table-name"


def write_records_to_timestream(records):
    try:
        result = timestream_write_client.write_records(
            DatabaseName=database_name,
            TableName=table_name,
            Records=records,
            CommonAttributes={},
        )
        print(
            "WriteRecords Status: [%s]" % result["ResponseMetadata"]["HTTPStatusCode"]
        )
    except Exception as err:
        print("Error:", err)


# JSONデータをそのままTimestreamのレコード形式に変換して書き込み
records = []
for record_entry in data["records"]:
    dimensions = [
        {"Name": k, "Value": str(v)} for k, v in record_entry["dimensions"].items()
    ]
    for measure in record_entry["measures"]:
        # 入力されたタイムスタンプを解析
        dt = datetime.strptime(measure["timestamp"], "%Y-%m-%d %H:%M:%S")

        # 9時間足す
        dt_adjusted = dt + timedelta(hours=9)

        record = {
            "Dimensions": dimensions,
            "MeasureName": measure["measure_name"],
            "MeasureValue": str(measure["measure_value"]),
            "Time": str(int(dt_adjusted.timestamp() * 1000)),
            # 常に上書きするように設定
            "Version": int(datetime.now().timestamp()),
        }
        records.append(record)

write_records_to_timestream(records)
```

以下の箇所で実行したいアカウントを指定できるようにしました。

```Python
my_session = boto3.Session(profile_name="default")
```

試験中、値を変えたくなったりすることもあると思うので、Versionを常に更新することで毎回新しいレコードが反映されるようにしています。

```Python
# 常に上書きするように設定
"Version": int(datetime.now().timestamp()),
```

### JSON

```json
{
    "records": [
        {
            "dimensions": {
                "prefecture": "愛媛",
                "municipalities": "松山"
            },
            "measures": [
                {
                    "measure_name": "Temperature",
                    "measure_value": 15,
                    "timestamp": "2024-04-01 00:00:00"
                }
            ]
        },
        {
            "dimensions": {
                "prefecture": "沖縄",
                "municipalities": "那覇"
            },
            "measures": [
                {
                    "measure_name": "Temperature",
                    "measure_value": 20,
                    "timestamp": "2024-04-01 00:00:05"
                }
            ]
        }
    ]
}
```

## 結果

無事、2レコード投入されていることを確認しました。

![Amazon Timestreamにデータが書き込まれていること](/images/writed_data_for_AmazonTimestream.png)

## おわりに

Timestreamに投入したデータは、基本的に消せないため再現性ある形(JSON)で管理しておくことでプロジェクトが上手く進むと良いなと思いました。
