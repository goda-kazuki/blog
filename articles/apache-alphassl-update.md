---
title: "ApacheとAlphaSSLの更新作業してみた"
published_at: "2022-11-30"
type: "tech"
published: true
coverImage: "ApacheとAlphaSSLの更新作業してみた.jpg"
---

## はじめに

先日、業務でSSL更新作業を行いました。 このブログのSSLは自動更新なので、証明書の更新など行ったことがありませんでした。

SSLの仕組みは何となく理解しているつもりでしたが、振り返りがてらまとめます。 (最近はSSL自動更新のサービスが増えてきているので、できるなら自動化した方が良いです)

## 前提

サーバー:Apache SSL発行会社:アルファSSL(Alpha SSL)

## 手順

1. SSL発行会社にドメイン更新の依頼を行う
2. 発行されたドメイン審査コードをサーバー内に設置する
3. 承認処理を実施する
4. サーバー内で秘密鍵を生成する
5. サーバー内でCSRを生成する
6. CSRをSSL発行会社に提出し、SSL証明書をメールで受信
7. SSLサーバ証明書+中間CA証明書+ルート証明書を結合したファイルをサーバー内に設置
8. Apacheの設定を変更
9. Apacheの再起動

重要なところだけ詳細を書きます。

### 2\. ドメイン審査コードをサーバー内に設置する

[参考サイト](https://jp.globalsign.com/introduce/document/result.html?q1=3&q2=1)

ユーザーから見た以下のパスに、SSL発行会社から発行された審査コードを設置します。 [http://example.com/.well-known/pki-validation/gsdv.txt](http://example.com/.well-known/pki-validation/gsdv.txt)

### 4\. サーバー内で秘密鍵を生成する

```shell
# 秘密鍵の生成
openssl genrsa -des3 -out example.com.2022.key 2048
# パスフレーズ無しの秘密鍵の生成
openssl rsa -in example.com.2022.key -out example.com.2022.nopass.key
```

### 5\. CSRの生成

```shell
openssl req -new  -key example.com.2022.nopass.key -out example.com.2022.csr
```

コマンドラインで質問されていきますので、識別名を順に入力していきます。

### 7\. SSLサーバ証明書+中間CA証明書+ルート証明書を結合したファイルをサーバー内に設置

catコマンドで結合します

```shell
\cat example.com.2022.crt DigiCertCA.crt TrustedRoot.crt > example.com.2022.cer
```

※バッククォートを付けている理由は、catコマンドのデフォルトオプションをユーザーが設定していた場合に無視するためです。

### その他

秘密鍵とSSLの証明書はチェックサムが同じになります。

```
openssl rsa -in example.com.2022.nopass.key -modulus -noout | openssl md5
→AABBCC

openssl x509 -in example.com.2022.cer -modulus -noout | openssl md5
→AABBCC
```

これによって、この証明書はサーバー管理者が発行したことを証明しています。

## おわりに

catコマンドのオプション設定をしていて危なかったので、生成されたファイルの中身は確認した方が良いなと思いました。
