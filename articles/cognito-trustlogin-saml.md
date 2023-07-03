---
title: "Cognitoとトラストログイン(TrustLogin)でSAML認証したい時"
published_at: "2022-02-15"
type: "tech"
published: true
coverImage: "CognitoとトラストログインTrustLoginでSAML認証したい時.jpg"
---

CognitoとトラストログインをSAML連携したい時に、設定情報などが見つからず、苦労したので情報を残します。

AppSyncと連携したい時は過去記事参照ください

- [AppSyncの認証をCognito指定した時のPostman設定方法](https://gdtypk.com/postman-setting-appsync-auth-cognito/ "AppSyncの認証をCognito指定した時のPostman設定方法")
- [AppSyncの認証にCognitoユーザープールを設定する方法まとめ](https://gdtypk.com/how-appsync-auth-cognito-user-pool/ "AppSyncの認証にCognitoユーザープールを設定する方法まとめ")

## TrustLoginメタデータダウンロードまで

トラストログインのアプリを新規作成する ![trustlogin SAMLアプリ登録](/images/trustlogin-SAMLアプリ登録.png)

トラストログインのユーザを追加する ![trustlogin メンバー追加](/images/trustlogin-メンバー追加-1.png)

メタデータのダウンロードを行う ![trustloginメタデータのダウンロード](/images/trustloginメタデータのダウンロード.png)

## Cognitoアプリクライアントの設定まで

Cognitoユーザプールをデフォルト設定で作成します。 ![Cognitoユーザプール作成](/images/Cognitoユーザプール作成-300x117.png)

IDプロバイダーを登録します。(フェデレーション->IDプロバイダー)

- メタデータドキュメント：トラストログインでダウンロードしたもの
- プロバイダ名:任意
- 識別子:任意
- IdPサインアウトフローの有効化:チェックなし

![Cognito IDプロバイダーの追加](/images/Cognito-IDプロバイダーの追加-300x274.png)

ドメイン名の設定を行います。(アプリの統合->ドメイン名) ![Cognitoドメイン名登録](/images/Cognitoドメイン名登録-300x133.png)

アプリクライアントの追加をします。(全般設定->アプリクライアント) ![Cognitoアプリクライアントの追加](/images/Cognitoアプリクライアントの追加-243x300.png)

アプリクライアントの設定を行います。(アプリの統合->アプリクライアントの設定) ![Cognitoアプリクライアントの設定](/images/Cognitoアプリクライアントの設定-1-300x274.png)

## トラストログインサービスプロバイダーの設定

- ログインURL:Cognitoでアプリの統合->アプリクライアントの設定画面からホストされたUIのURL
    - ネームID用値:メンバー、email
- エンティティID:CognitoのプールID(Cognito->全般設定)
- ネームIDフォーマット:unspecified
- サービスへのACS URL:[https://{ドメイン}.auth.ap-northeast-1.amazoncognito.com/saml2/idpresponse](https://{ドメイン}.auth.ap-northeast-1.amazoncognito.com/saml2/idpresponse)
    - SAML属性の設定 :
- 属性指定名:[http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress](http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress)
    - 属性種類:Email 属性値:メンバー、メンバーメールアドレス

![trustlogin サービスプロバイダーの設定](/images/trustlogin-サービスプロバイダーの設定.png)

## Cognito属性マッピングの設定

Cognito属性マッピングの設定を行います。 (Cognitoユーザプール->フェデレーション->属性マッピング)

- SAML属性:[http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress](http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress)
    - ユーザープール属性:Email ![Cognito属性マッピングの設定](/images/Cognito属性マッピングの設定-300x125.png)

## Cognito動作確認

Cognito動作確認をします。 (アプリの統合->アプリクライアントの設定)

ホストされた UIを起動します。 ![CognitoホストされたUIを起動](/images/CognitoホストされたUIを起動-300x158.png)

表示されているSAMLのログインボタンを選択し、トラストログインのログインを済ませてください。

トークン情報が付与された状態でアプリクライントのコールバックURLで指定したページ(今回はhttps://www.amazon.co.jp)に遷移します。 ![Cognito トークン情報が付与されたページ](/images/Cognito-トークン情報が付与されたページ.png)

## 参考

[https://www.youtube.com/watch?v=FKU3Ai1AzNM](https://www.youtube.com/watch?v=FKU3Ai1AzNM)

[https://www.youtube.com/watch?v=E3VDZvPuS44](https://www.youtube.com/watch?v=E3VDZvPuS44)

```
https://aws.amazon.com/jp/premiumsupport/knowledge-center/cognito-saml-onelogin/
```
