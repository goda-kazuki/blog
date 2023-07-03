---
title: "Route53に登録済みのドメインとCloudFrontに紐付ける AWS"
published_at: "2022-01-20"
type: "tech"
published: true
coverImage: "CloudFrontにRoute53を使って独自ドメインを設定する.jpg"
---

## はじめに

CloudFrontとS3を使って静的なウェブサイトを構築することはよくあると思いますが、 そこに独自ドメインを設定を設定する必要がでてきたので、その対応メモになります。

## 前提

- Route53に対象のドメインがホストゾーンに登録済みであること
- CloudFrontのディストリビューションドメインが発行済みであること

Route53にドメインを紐付けるていない方はこちらを参考にしてください。 [https://gdtypk.com/sakura-domain-route53-hostz-one/](https://gdtypk.com/sakura-domain-route53-hostz-one/)

CloudFrontでS3静的ウェブサイトを構築する手順はこちらになります。 [https://dev.classmethod.jp/articles/cloudfront-s3-web/](https://dev.classmethod.jp/articles/cloudfront-s3-web/)

## 対応内容

### ACM(AWS Certificate Manager)でパブリック証明書を作成

ACMのページでリクエストを選択します。 ![ACMのページでリクエストを選択する](/images/ACMのページでリクエストを選択する-1024x117.png)

パブリック証明書をリクエストを選択します。 ![パブリック証明書をリクエストを選択する](/images/パブリック証明書をリクエストを選択する-1024x349.png)

証明書リクエストに必要な情報を入力します。 DNS認証であれば、自動更新が有効になるので可能であればDNS認証が良いです。(Route53にホストゾーン設定済みであれば、可能) ![証明書に必要な情報を入力する](/images/証明書に必要な情報を入力する-1024x807.png)

検証が保留されているので、選択して詳細ページに遷移します。 ![保留の証明書を選択する](/images/保留の証明書を選択する-1-1024x127.png)

Route53でレコードを作成を選択します ![Route53でレコードを作成を選択](/images/Route53でレコードを作成を選択-1024x472.png)

Route53のホストゾーンにドメインがあれば、レコードを作成できるので選択します。 ![レコードを作成を選択](/images/レコードを作成を選択-1024x290.png)

無事にステータスが変更されたことを確認します ![無事にステータスが変更されたことを確認](/images/無事にステータスが変更されたことを確認-1-1024x134.png)

念の為Route53にCNAMEレコードが追加されていることを確認します。 ![Route53にCNAMEレコードが追加されていることを確認](/images/Route53にCNAMEレコードが追加されていることを確認-1024x68.png)

### CloudFrontでACMを利用してドメインを紐付ける

設定したいCloudFrontのディストリビューションを選択し、編集を選択します。 ![ディストリビューションを編集する](/images/ディストリビューションを編集する-1024x242.png)

ディストリビューションに必要な情報を入力をします。 ![ディストリビューションに必要な情報を入力](/images/ディストリビューションに必要な情報を入力-685x1024.png)

設定したドメインが表示されることを確認します。 ![設定したドメインが表示されることを確認](/images/設定したドメインが表示されることを確認-1-1024x552.png)

ディストリビューションをメモしておく。 ![ディストリビューションをメモ](/images/ディストリビューションをメモ.png)

### Route53にAレコードを設定する

対象のドメインにアクセスがあった時に、CloudFrontにアクセスするようにする必要があります。

Route53のホストゾーンの詳細ページでレコード作成を選択します。 ![Route53のホストゾーンの詳細ページでレコード作成を選択](/images/Route53のホストゾーンの詳細ページでレコード作成を選択-1024x262.png)

必要な情報を入力します ![Route53レコード作成に必要な情報を入力](/images/Route53レコード作成に必要な情報を入力-1024x502.png)

Aレコードが設定されているか確認する

```shell
dig  対象のドメイン
対象のドメイン.    60  IN  A   IPアドレス(CloudFront側で自動的に振られたもの)
対象のドメイン.    60  IN  A   IPアドレス(CloudFront側で自動的に振られたもの)
対象のドメイン.    60  IN  A   IPアドレス(CloudFront側で自動的に振られたもの)
対象のドメイン.    60  IN  A   IPアドレス(CloudFront側で自動的に振られたもの)
```

実際にドメインでアクセスできるか確認する

## おわりに

よく発生することだと思いますが、やったことなかったので勉強になりました。

## 参考

[https://gdtypk.com/sakura-domain-route53-hostz-one/](https://gdtypk.com/sakura-domain-route53-hostz-one/) [https://dev.classmethod.jp/articles/cloudfront-s3-web/](https://dev.classmethod.jp/articles/cloudfront-s3-web/)
