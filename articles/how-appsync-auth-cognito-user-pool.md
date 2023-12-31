---
title: "AppSyncの認証にCognitoユーザープールを設定する方法まとめ"
published_at: "2022-02-08"
type: "tech"
published: true
coverImage: "サンプル.jpg"
---

## はじめに

AppSyncの認証にCognitoを使用することになったので設定手順をまとめます。

後日 SAML連携やPostmanから使用する方法を別記事でまとめようと思います。

## Cognito

### ユーザープールの作成

デフォルトのまま変更せずに作成する ![Cognitoユーザプール新規作成](/images/Cognitoユーザプール新規作成.png)

### アプリクライアントの作成

全般設定->アプリクライアント->アプリクライアントの追加 ![Cognitoアプリクライアント作成](/images/Cognitoアプリクライアント作成-817x1024.png) ※クライアントシークレットを作成はチェック外してください チェックが入っていると、AppSyncのコンソールでログインできません。

### アプリクライアントの設定

アプリの統合->アプリクライアントの設定 さっき作成したアプリクライアントを選択し、下記の画像のように設定します。 ![Cognitoアプリクライアントの設定](/images/Cognitoアプリクライアントの設定.png)

### ドメインの設定

アプリの統合->ドメイン名 好きなドメインを設定してください。 ![Cognitoドメインの設定](/images/Cognitoドメインの設定.png)

### ユーザーの作成

アプリの統合->アプリクライアントの設定 下の方に「ホストされたUIを起動」というリンクがあるので選択します。

![Cognitoサインイン画面](/images/Cognitoサインイン画面.png)

サインインの画面が表示されるので、Sign upを選択しユーザーを作成します。

全般設定->ユーザーとグループ 作成したユーザが表示されていることを確認します。 ![Cognitoユーザー作成完了](/images/Cognitoユーザー作成完了.png)

## AppSyncの設定

### 新規作成

デフォルトから特に変更なしでポチポチ作成する ![AppSync新規作成](/images/AppSync新規作成.png) ![AppSyncモデル作成](/images/AppSyncモデル作成.png) ![AppSyncリソース作成](/images/AppSyncリソース作成.png)

### 認証方法の変更

設定->デフォルトの認証モード デフォルトでAPIキーになっているので、AmazonCognitoユーザープールに変更します。

![AppSyncデフォルト認証モード変更](/images/AppSyncデフォルト認証モード変更.png)

### クエリ叩いてみる

クエリの画面に行くと、ログインしていないので実行できないようになっています。

では、ユーザープールでログインしてみます。 ![AppSyncユーザープールでログインする](/images/AppSyncユーザープールでログインする.png)

ログインが完了したのでクエリを実行することができます。 ![AppSyncログインし、クエリ実行確認](/images/AppSyncログインし、クエリ実行確認.png)
