---
title: "Amazon Athenaのハンズオンをやってみました。"
published_at: "2023-01-12"
type: "tech"
published: true
coverImage: "Amazon-Athenaのハンズオンをやってみました.jpg"
---

AthenaはS3のファイルに対して、検索かけれるもの。という認識で、 触ることはなかったのですが、ハンズオンを通して雰囲気を学びました。 [https://aws.amazon.com/jp/blogs/news/amazon-athena-workshop-publishment-in-japanese/](https://aws.amazon.com/jp/blogs/news/amazon-athena-workshop-publishment-in-japanese/)

## 気づき

### どんなファイル(オブジェクト)でも動くわけではない。

勝手にS3にあるテキストファイルに対して検索できるのかと思ってたんですが 決まったフォーマットのファイルが対象のようです [https://docs.aws.amazon.com/ja\_jp/athena/latest/ug/supported-serdes.html](https://docs.aws.amazon.com/ja_jp/athena/latest/ug/supported-serdes.html)

### TSV形式とParquet形式について

ハンズオンではTSV形式とParquet形式のファイルを扱いました。 どちらも初めて知った形式でしたので、調べました。

#### TSV形式について

こちらは分かりやすく 項目同士がタブで区切られている形式のこと。 CSVはカンマで区切られているが、それがタブになっただけのもの。

正式名称は「Tab Separated Values」 CSVは「Comma Separated Value」 そのまんまですね。

#### Parquet形式

効率的なデータ保存や検索ができるように設計された形式。 ファイルを開いてみましたが、人間が読めるものではなかったです。

特化しているので、TSV形式より高速に動きます。 [https://www.databricks.com/jp/glossary/what-is-parquet](https://www.databricks.com/jp/glossary/what-is-parquet) 実際にハンズオンでも20倍近いパフォーマンスが出ていました。

### TSV形式をParquet形式に変換することも可能

これは、TSV形式で保存されているものからParquet形式のもの生成するクエリです。

```
CREATE TABLE 〇〇
WITH ( format='PARQUET', parquet_compression = 'SNAPPY', partitioned_by = ARRAY['〇〇'], 
external_location =   's3://〇〇/') AS
SELECT 〇〇,
〇〇,
FROM 〇〇
WHERE "$path" LIKE '%tsv.gz';
```

### AWS Glueについて

ハンズオンの途中でGlueが出てきました。 こちらもほとんど知識がなかったので、少しまとめます。

ETLツールとして、大量のデータを処理,変換するサービス。

今回だと、Parquetのデータを一度扱いやすい形式に変換し、テーブルへ保存するような処理を担当しています。

## おわり

データソースはS3に限らず、RDBやDynamoDBもLambdaを経由することが可能だそう。 決まった形式で大量に生成されるものって、今回のような購入履歴や行動履歴、ログなんかを扱う感じなのかなー。
