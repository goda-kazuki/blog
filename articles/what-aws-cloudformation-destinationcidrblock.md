---
title: "AWS CloudFormationのDestinationCidrBlockとは何なのか調べてみた"
published_at: "2022-06-15"
type: "tech"
published: true
coverImage: "AWS-CloudFormationのDestinationCidrBlockとは何なのか調べてみた.jpg"
---

## 内容

クラスメソッドさんのCloudFormation入門を実践していました。 [https://dev.classmethod.jp/articles/cloudformation-beginner01/](https://dev.classmethod.jp/articles/cloudformation-beginner01/)

そのときに、以下の設定情報があります。

```
  FrontendRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref FrontendRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway
```

これ、書いてるけどどこに反映されるのか分からず、悩んでいました。

やっていることは 全てのトラフィックをインターネットゲートウェイに向けるようにしているのだと思います。

ただ、ルートテーブルを見ても、0.0.0.0/0の設定はありません。 ![DestinationCidrBlockに0.0.0.0/0を設定した時](/images/DestinationCidrBlockに0.0.0.00を設定した時.png)

明確に説明されているページは見つかりませんでしたが、 インターネットゲートウェイの場合は、デフォルトでは表示されないようになっているのではないかと思います。

そう思った理由は以下のように設定してデプロイしたとき

```
      DestinationCidrBlock: 54.11.22.33/32
```

![DestinationCidrBlockに54.11.22.33/32を設定した時](/images/DestinationCidrBlockに54.11.22.3332を設定した時.png)

明示的に送信先のルートが記載されていました。

上記の理由から、0.0.0.0/0のときはスキップされるのではないかと思います。

## 参考

[https://stackoverflow.com/questions/52004474/destinationcidrblock-property-of-awsec2route](https://stackoverflow.com/questions/52004474/destinationcidrblock-property-of-awsec2route)
