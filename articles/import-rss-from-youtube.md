---
title: "Youtubeのチャンネル登録リストをRSSでFeedly等に取り込む方法"
published_at: "2022-05-06"
type: "idea"
published: true
coverImage: "import-rss-from-youtube.jpg"
---

Youtubeでチャンネル登録しているチャンネルをFeedlyで管理したかったのですが、 Youtube公式ではRSSのリストを出力するような機能がなかったため、やり方を調べました。

## はじめに

Youtubeのチャンネルリストでコードを実行すると、URLのリストが取得できるので

それをxml形式に変換することで、RSSのリストが出力できます。

以降は具体的な方法を記載します。

## 取得方法

### Youtubeのチャンネルリストへ遷移

以下のURLで遷移します。 [https://www.youtube.com/feed/channels](https://www.youtube.com/feed/channels)

チャンネル数が多いと、下までいかないと表示されないので、一番下までスクロールしてください。

![Youtubeのチャンネルリストへ遷移](/images/Youtubeのチャンネルリストへ遷移.jpg)

### デベロッパーツールでコードを実行する

デベロッパーツールを開きます。

| Windows | Mac |
| --- | --- |
| Ctrl+Shift+I | Command + Option + I |

もしマウスで行うなら、以下画像の箇所になります。

![デベロッパーツールでコードを実行する](/images/デベロッパーツールでコードを実行する.png)

以下のURLに書かれているコードをコピーします。

[https://gist.githubusercontent.com/jeb5/da22862e469dea21e873acabb562f638/raw/b2f4139c2b0ad33c8872a11e88f2c8067a2c25d7/bookmarklet](https://gist.githubusercontent.com/jeb5/da22862e469dea21e873acabb562f638/raw/b2f4139c2b0ad33c8872a11e88f2c8067a2c25d7/bookmarklet)

以下の手順でコード実行していきます。

1. Consoleタブをクリックします
2. テキストエリアにコピーしたコードを貼り付け、エンターで実行します。 ![テキストエリアにコピーしたコードを貼り付け](/images/テキストエリアにコピーしたコードを貼り付け.png)
3. 出力されたURLのリストをコピーしてください。 ![出力されたURLのリストをコピー](/images/出力されたURLのリストをコピー.jpg)

### RSS用のファイルを取得する(xmlファイル)

[opml-gen](https://opml-gen.ovh/)というサービスで、RSS用のファイルを生成します。

[https://opml-gen.ovh/](https://opml-gen.ovh/)

※テキストエリアにURLのリストをペーストして、Generateを押すだけです

### Feedly等のRSSリーダーに取り込む

設定ボタンを選択します。 ![Feedlyに取り込む1](/images/Feedlyに取り込む1.png)

IMPORT OPMLから先程作成したファイルを選択することで取り込みが完了します。 ![Feedlyに取り込む2](/images/Feedlyに取り込む2-300x33.png)

## おわりに

昔はYoutube単体で出力できていたようなのですが、 今は無いので方法知らなければ 手動で1個ずつ登録する必要があり、大変なので良かったです。
