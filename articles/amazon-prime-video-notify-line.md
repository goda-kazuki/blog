---
title: "Amazon Prime Videoのエピソード更新通知をLINEで受け取る方法(GASで実装)"
published_at: "2022-08-26"
type: "tech"
published: true
topics: ["GAS"]
coverImage: "Amazon-Prime-Videoのエピソード更新通知をLINEで受け取る方法GASで実装.jpg"
---

## はじめに

Amazon Prime Videoをよく利用しており、いろんな番組を見ているのですが 更新通知が無いのが不便に感じていました。

今回は上記の問題を解決するためにGAS(Google Apps Script)からスクレイピングし、LINEで通知する仕組みを作成しました。

なお、スクレイピングする際は、サービス側に負荷をかけないように、 短時間に大量のリクエストを送るのはやめましょう。

## 実装手順

### LINE Notify準備

まず、LINEで通知するためにLINE Notifyを用意します。 こちらのサイトを参考に、「LINE Notify」をトークルームに入れるところまで行ってください。 トークンは後ほど使用するので、どこかにコピペしておいてください。 [https://www.smilevision.co.jp/blog/tsukatte01/](https://www.smilevision.co.jp/blog/tsukatte01/)

### スプレッドシートの準備

まず、以下のようなスプレッドシートを用意してください。

| タイトル | URL | 話数 |
| --- | --- | --- |
| リコリコ | [https://www.amazon.co.jp/リコリス・リコイル/dp/B0B5JMSQKS](https://www.amazon.co.jp/リコリス・リコイル/dp/B0B5JMSQKS) | 0 |
| アオアシ | [https://www.amazon.co.jp/アオアシシーズン1/dp/B09RMKJHXG](https://www.amazon.co.jp/アオアシシーズン1/dp/B09RMKJHXG) | 0 |

↓実際のスプレッドシートの画面 ![スプレッドシートの準備](/images/スプレッドシートの準備.png)

### コード貼り付け

スプレッドシートのヘッダー部分の拡張機能->Apps Scriptを選択してください。 エディタが開いたら、以下のコードを貼り付けてください。

```javascript
function myFunction() {
    const sheet = SpreadsheetApp.getActiveSheet();
    const lastRow = sheet.getLastRow();
    const range = sheet.getRange(`A2:C${lastRow}`);
    const targetContents = range.getValues();
    let message = "\n";

    targetContents.forEach((content, index) => {
        const response = UrlFetchApp.fetch(content[1]);
        const text = response.getContentText("utf-8");
        const filteredText = text.match(/<h1.*エピソード.*\/h1>/);
        const numberOfEpisode = filteredText[0].match(/<!-- -->\d+<!-- -->/)[0].replaceAll("<!-- -->", "");

        const regex=new RegExp(/<span.*で見る<\/span>/);
        const isOtherThanPrimeUser=regex.test(text);

        const countEpisodeNumber=Number(numberOfEpisode) - (isOtherThanPrimeUser?1:0);

        // 前回通知した時の話数より進んでなければスキップ(PrimeVideo以外はカウントから弾く)
        if (countEpisodeNumber <= content[2]) return;

        // 1行目を除く+indexは0から始まるため、2を加算する
        const updateTargetRosNumber = index + 2;
        sheet.getRange(updateTargetRosNumber, 3).setValue(countEpisodeNumber);

        message += content[0] + " " + countEpisodeNumber + "話\n";
    });
    if (message === "\n") return;

    // 最後の改行が不要なので削除
    message = message.slice(0, -1);

    // LINEで通知
    sendLine(message);
}

function sendLine(message) {
    // LINEから取得したトークン
    const token = "evuqNFQnQfkhSv6sF7BoMYHOU4DyDFBGC1rD9zc0BMq"
    const options = {
        "method": "post",
        "headers": {"Authorization": "Bearer " + token},
        "payload": {"message": message}
    }
    const lineNotifyUrl = "https://notify-api.line.me/api/notify"
    UrlFetchApp.fetch(lineNotifyUrl, options)
}
```

### トークンを入力

コードの以下の箇所にトークンをコピペしてください。

```javascript
const token = "ここにLINE Notifyのトークンを入力"
```

### トリガーの設定

定期的に自動実行する必要があるので、 GASのトリガーを設定してください。 設定方法は以下が参考になると思います。 [https://blog.synnex.co.jp/google/gas-trigger/](https://blog.synnex.co.jp/google/gas-trigger/)

## ポイント

### UrlFetchApp.fetch(URL)

以下の箇所でコンテンツのURLに対してリクエストを行っています。

```javascript
const response = UrlFetchApp.fetch(content[1]);
```

HTMLが返却されるので、そのテキストを置換などすることにより、最新の話数を求めています。

### Amazon Prime以外に契約が必要なエピソードは弾く処理

dアニメなどのサービスが別途契約な場合はカウントしないようにしたかったです。

HTMLを見ていると、以下の処理で存在チェックができたので反映しています。

```
        const regex=new RegExp(/<span.*で見る<\/span>/);
        const isOtherThanPrimeUser=regex.test(text);
```

## おわりに

GASからスクレイピングするのは、大変なのかと思っていましたが 用意されている関数を使用することで簡単にできました。

GASは自分で環境を用意する必要もなく、メンテも不要なので嬉しいですね。
