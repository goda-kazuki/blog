---
title: "Amazon SNSでHTTPSとEmailをサブスクライブする"
published_at: "2023-01-12"
type: "tech"
published: true
coverImage: "Amazon-SNSでHTTPSとEmailをサブスクライブする.jpg"
---

Amazon SNSではHTTPSエンドポイントやEmailに対して、通知を送ることができますが、 実際に使用したことがなかったので、今回試してみました。

## 流れ

### トピックの作成

タイプはスタンダードを選択しましょう。 FIFOにすると、サブスクリプションプロトコルをSQSしか選択できなくなります。

![トピックの作成](/images/トピックの作成.png)

### HTTPSのサブスクリプション

今回、webhook.siteさんを使用させていただきます。 以下のURLにアクセスすると、自動でHTTPSエンドポイントを発行してくれます。 [https://webhook.site/](https://webhook.site/) ![webhook.siteのエンドポイント](/images/webhook.siteのエンドポイント.png)

そのURLをエンドポイントに設定してください。 ![サブスクプションのURL設定](/images/サブスクプションのURL設定.png)

作成すると、webhook.siteの方にリクエストが飛びます。 SubscribeURLに承認用のURLがあるので、そちらに遷移して許可します。 ![Subscribe URL](/images/Subscribe-URL.png)

サブスクリプションの設定が承認済みになりました。

### Emailのサブスクプション

基本、HTTPSのときと同じです。

メールアドレスを入力し、作成する。

入力したメールアドレス宛に承認用のURLが送られるため、許可します。

### 疎通確認

メッセージの発行を行うと、それぞれに対して通知がされます。

![HTTPS疎通確認](/images/HTTPS疎通確認.png)

![Email疎通確認](/images/Email疎通確認.png)

## おわりに

機能は有名なので知っていましたが、実際に試してみるとかなり簡単に通知ができて驚きました。 Slackへの通知などは、Lambdaを経由する必要がありますが手軽で良い感じですね。

## 参考にした記事

[https://dev.classmethod.jp/articles/subscribe-to-http-s-endpoints-in-amazon-sns/](https://dev.classmethod.jp/articles/subscribe-to-http-s-endpoints-in-amazon-sns/)
