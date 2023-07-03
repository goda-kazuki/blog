---
title: "AppSyncの認証をCognito指定した時のPostman設定方法"
published_at: "2022-02-08"
type: "tech"
published: true
coverImage: "AppSyncの認証をCognito指定した時のPostman設定方法.jpg"
---

## 前提

AppSyncとCognitoユーザプールの設定は済ませておく必要があります。 まだの方は過去記事参照ください。 [https://gdtypk.com/how-appsync-auth-cognito-user-pool/](https://gdtypk.com/how-appsync-auth-cognito-user-pool/)

## 認証系の設定を行う

1. Authorizationタブを選択し、Oauth2.0を選択する
2. 下記パラメータ入力

| 項目 | 値 |
| --- | --- |
| Token Name | 任意の名前 |
| Callback URL | Cognitoで指定しているコールバックURL |
| Auth URL | Cognitoで指定しているドメイン(アプリの結合->ドメイン名) |
| Client ID | CognitoのアプリID(全般設定->アプリクライアント) |
| Grant Type | Implicitを選択 |

![Postman Authorizationタブの内容](/images/Postman-Authorizationタブの内容-1024x782.png)

3. Get New Access Tokenでトークンを取得する Postmanでログイン画面が表示されるので、ログインするとリダイレクトされます。 ![Postmanログイン画面](/images/Postmanログイン画面.png)
    
4. id\_tokenの値をコピーし、HeadersタブでAuthorizationをキーにして、貼り付け。 ログインが完了すると、下記のようなレスポンスがあるので、id\_tokenの値をコピーしておきます。 ![Get New Access Tokenの結果](/images/Get-New-Access-Tokenの結果-1024x577.png)
    

## その他、必要なパラメータの入力

| 項目 | 値 |
| --- | --- |
| URL | AppSyncの管理画面で「設定->API Details->API URL」の値を取得し格納 |
| リクエストメソッド | デフォルトではGETになってるので、POSTに変更 |
| Bodyタブ | GraphQLのリクエストを記入 |

![GraphQLのリクエストを記入](/images/GraphQLのリクエストを記入-1024x291.png)

## 動作確認

正常なレスポンスがあることを確認

![PostmanでGraphQLのレスポンスがあることを確認](/images/PostmanでGraphQLのレスポンスがあることを確認-1024x215.png)

## 参考情報

Postmanの参考記事 [https://qiita.com/gaku3601/items/2ce42ef4b5a7a61a6a96](https://qiita.com/gaku3601/items/2ce42ef4b5a7a61a6a96) [https://qiita.com/t-kigi/items/4b5076b217ca0c871d4b](https://qiita.com/t-kigi/items/4b5076b217ca0c871d4b)
