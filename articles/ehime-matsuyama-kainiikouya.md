---
title: "愛媛県松山市 買いにいこうやキャンペーンの検索ページを作成しました"
published_at: "2021-10-28"
type: "tech"
published: true
coverImage: "main.png"
---

## 作成したページ

[https://gdtypk.com/matsuyama.html](https://gdtypk.com/matsuyama.html) CSSがよく分からず、スマホだと見づらく感じるかもしれません・・。 PC版のChrome,FireFox,Safariで動作確認しました。

## なぜ作成したか

以下の理由から作成しました。

- 公式が配布している店舗リストがPDFで検索しづらいこと
- ざっくりしたエリアしか掲載されていないこと

## 愛媛県松山市買いにいこうやキャンペーンについて

> コロナ禍の飲食店への営業時間短縮要請や外出自粛などで影響を受けた市内の飲食店や宿泊施設、小売、サービスなど、幅広い業種の事業者を支援するため、「買いにいこうや！キャンペーン」を開始します。 消費を松山市独自で喚起し、本市経済を活性化させます。

[引用元はこちら](http://www.city.matsuyama.ehime.jp/smph/kurashi/sangyo/shokogyo/syouhinken.html) [特設サイト](https://www.all-matsuyama.com/buy/store/entry.html)

食品だけでなく、家電なども対象となっており、私もこれを期に何か購入しようと思っています。

## どう作成したか

キャンペーン期間が2ヶ月程度なので、泥臭く手動の部分が多いです😅

### 対象店舗一覧のPDFを公式サイトからダウンロード

[https://www.all-matsuyama.com/buy/user/](https://www.all-matsuyama.com/buy/user/)

### PDFをエクセルに変換する

[https://www.adobe.com/jp/acrobat/online/pdf-to-excel.html](https://www.adobe.com/jp/acrobat/online/pdf-to-excel.html)

### エクセルをスプレッドシートにコピペ

GASを使いたかったので、スプレッドシートにコピペする必要があります。

### 不要な行の削除

GASで以下のプログラムを用意し、不要行を削除しました。

<script src="https://gist.github.com/goda-kazuki/d12c6d8079cc90e31a7428e819e28953.js"></script>

[https://github.com/goda-kazuki/matsuyama/blob/main/removeRow.js](https://github.com/goda-kazuki/matsuyama/blob/main/removeRow.js)

### 住所と町域の作成

Geocoding APIを使用し、店舗名から住所を取得しています。

全行に対して、処理しているだけの何でもないプログラムですが、 店舗名だけでは検索に引っかからないものがあったので、地名を事前に入れました。

<script src="https://gist.github.com/goda-kazuki/e1fcf385e239968952b4fdc496cc823e.js"></script>

[https://github.com/goda-kazuki/matsuyama/blob/main/geocoder.js](https://github.com/goda-kazuki/matsuyama/blob/main/geocoder.js)

### csv出力

住所と町域が反映されたシートをcsvで出力します。

### csvをjsonに変換

CSVのままだと、javaScriptで扱いにくいので、以下のサイトで変換しました。 [https://www.site24x7.com/ja/tools/csv-to-json.html](https://www.site24x7.com/ja/tools/csv-to-json.html)

### HTMLとjavaScriptで表示

jsonをjavaScriptから読み込み、1行ずつテーブルに追加していきます。 [https://github.com/goda-kazuki/matsuyama/blob/main/matsuyama.html](https://github.com/goda-kazuki/matsuyama/blob/main/matsuyama.html)

bootstrapとCSSが分からんすぎて見た目に関しては途中で諦めました・・。

## おわりに

パソコンでないとちょっと見づらいですが、検索できるようになって私は満足しています。

仕事でCSS書くことは無いけど、bootstrapは使えるようにしとかないとなー思いました・・。
