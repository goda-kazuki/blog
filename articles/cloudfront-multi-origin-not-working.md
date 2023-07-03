---
title: "CloudFrontでマルチオリジン(multi origin)を設定しても反映されない時"
published_at: "2022-01-26"
type: "tech"
published: true
coverImage: "CloudFrontのマルチオリジンが反映されないとき.jpg"
---

## はじめに

S3の静的ウェブサイトホスティングでWebアプリを作成しました。 ユーザがファイルをアップし、閲覧できる機能があるのですが、アプリが格納されているバケットとは分けたかったので、 CloudFrontのマルチオリジンを使って解決しようとしたところ、いつまで経っても反映されないので調査しました。

## 前提情報

| バケット名 | 概要 |
| --- | --- |
| sample-app | 静的ウェブホスティングでアプリが格納されている |
| sample-file | ユーザがアップしたファイルが格納されている |

CloudFrontにドメインを紐付けていない場合、過去記事参照ください。 [https://gdtypk.com/cloudfront-original-dmain-route53-acm/](https://gdtypk.com/cloudfront-original-dmain-route53-acm/)

## ゴール

| URL | 遷移先 |
| --- | --- |
| `https://sample.com` | `sample-app`バケットへアクセス |
| `https://sample.com/upload_file` | `sample-file`バケットへアクセス |

## オリジンの設定状況

`sample-app`と`sample-file`のオリジンを追加する ![CloudFrontオリジンの設定状況](/images/CloudFrontオリジンの設定状況-1024x292.png)

## ビヘイビアの設定状況

`https://sample.com/upload_file/*`にアクセスがあれば、`sample-file`を参照し、 その他は`sample-app`を参照するように設定。

![CloudFrontビヘイビアの設定状況](/images/CloudFrontビヘイビアの設定状況-1024x319.png)

## 問題発生

`https://sample.com/upload_file/*`にアクセスしてもファイルが表示されない。 バケットポリシーや公開範囲についての問題は無い。

CloudFrontの反映に時間がかかるのかもと思い、1時間ほど待ったが解決せず。

## 原因

`sample-file`バケットの構成に問題があった。

処理フロー

1. `https://sample.com/upload_file/sample.txt`へアクセス
2. `s3://sample-file/upload_file/sample.txt`を参照
3. そんなファイルない！ERROR！

こんな感じでした。

## 対応

`sample-file`の階層を以下のように変更しました。

```
sample.txt
```

↓

```
upload_file
└sample.txt
```

## おわり

ビヘイビアにパスパターンを設定しても、それを無視してバケットを参照しにいくと勝手に思っていたのが悪かったようです。

## 参考

[https://dev.classmethod.jp/articles/cloudfront-multioriginbehavior/](https://dev.classmethod.jp/articles/cloudfront-multioriginbehavior/) [https://qiita.com/uramotot/items/592b43cf564060e8a365](https://qiita.com/uramotot/items/592b43cf564060e8a365)
