---
title: "【CDK】AWS CDKについてイベントに参加したので、まとめてみた【AWS Tech talk Night】"
published_at: "2022-12-14"
type: "idea"
published: true
coverImage: "【CDK】AWS-CDKについてイベントに参加したので、まとめてみた【AWS-Tech-talk-Night】.jpg"
---

AWS Tech talk Night#4 ~アーキテクチャ道場＆ライブコーディング！に参加しました。 [https://techplay.jp/event/880530](https://techplay.jp/event/880530)

IaCってハードル高いなあと思ってたのが、だいぶ気が楽になりました。

感想と簡単な構成を試してみたので、そちらをまとめようと思います。

## 感動したところや感想

### ベストプラクティスに則った最低権限のロールの設定を良い感じにやってくれる

Lambdaに対してDynamoDBに適切なアクセス権限を付与するようなIAMロールを作成するのって、 結構面倒くさくて、DynamoDB Full Accessとか渡しちゃってたんですが、 IAMロールを作成するような関数があるため、それを使えば ベストプラクティスに則ったIAMロールをいい感じに作ってくれます。

```typescript
boyakiTable.grantReadWriteData(backend);
```

[https://github.com/aws-samples/cdk-nuxtjs3-sample/blob/main/lib/cdk-sns-app-sample-stack.ts#L67](https://github.com/aws-samples/cdk-nuxtjs3-sample/blob/main/lib/cdk-sns-app-sample-stack.ts#L67)

### リソース名については、CDKに任せるのがベストプラクティス

CDKに任せると、プロジェクト固定以外の部分は乱数っぽいリソース名になります。

直感的には、分かりにくいんじゃない・・？と思っていたんですが 以下のような理由から、そうしているのかなと感じました。

- 構成はCDKを見れば分かること
- 作成、更新、削除の際に名前が被っていて事故る可能性がある。
- リソース名考えるのに時間がかかる

公式ページにもありました。 [https://aws.amazon.com/jp/blogs/news/best-practices-for-developing-cloud-applications-with-aws-cdk/](https://aws.amazon.com/jp/blogs/news/best-practices-for-developing-cloud-applications-with-aws-cdk/) ※「自動で生成されるリソース名を使用し、物理的な名前を使用しない」の箇所

### CloudFormationより便利なこと

CDKは各種プログラミングコードで構成を定義した後、CloudFormationに変換されて実行します。

CloudFormationに変換するタイミングでチェックが入るので、フィードバックが早く、素早く進められます。 CloudFormationだと10分待ってエラーでコケるとか結構あるそうです。

### construct Hubが便利

CloudFrontとS3の連携とかってよくあると思うんですが、そういったパターンのものは、 construct Hubで公開されているので、それらを使うと早く構築可能です。 [https://constructs.dev/](https://constructs.dev/) [https://constructs.dev/packages/@aws-solutions-constructs/aws-cloudfront-s3/v/2.27.0?lang=typescript](https://constructs.dev/packages/@aws-solutions-constructs/aws-cloudfront-s3/v/2.27.0?lang=typescript)

### その他

IaC全般に言えることですが、 マネジメントコンソールだと構成把握が難しいが、CDKならコードに全て書いてるので把握が容易に行えます。

CloudFormationの設定と1対1なので、CDKが対応してなくて実装できない・・。なんてことは無いようです。

IDEがサジェストしてくれるので、慣れたら早くなりそう。

中の実装はTypeScriptなので、TypeScriptが1番ドキュメント充実してるそうです。

これから学習するなら以下をオススメされてました [https://cdkworkshop.com/ja/](https://cdkworkshop.com/ja/)

#### 本題と関係ないところ

- Nuxt.jsはnitroと組み合わせてLambdaのコードを出力可能
- DBの選定は目的によって使い分けることがベストプラクティス
    - [https://aws.amazon.com/jp/products/databases/](https://aws.amazon.com/jp/products/databases/)

## 実践

こちらのワークショップを実践してみました。 [https://cdkworkshop.com/ja/20-typescript.html](https://cdkworkshop.com/ja/20-typescript.html)

### 書いたコード

/lib/aws-cdk-stack.ts

```typescript
import * as cdk from 'aws-cdk-lib';
import * as lambda from 'aws-cdk-lib/aws-lambda'
import * as apigw from 'aws-cdk-lib/aws-apigateway'

export class AwsCdkStack extends cdk.Stack {
    constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
        super(scope, id, props);

        console.log(lambda.Code.fromAsset('lambda'));

        const helloLambda = new lambda.Function(this, 'HelloHandler', {
            runtime: lambda.Runtime.NODEJS_14_X,
            code: lambda.Code.fromAsset('lambda'),
            handler: 'hello.handler'
        });
        const sampleApiGateway = new apigw.LambdaRestApi(this, 'Endpoint', {
            handler: helloLambda
        })
        console.log(sampleApiGateway);
    }
}
```

/lambda/hello.js

```javascript
exports.handler = async function (event) {
    console.log("request:", JSON.stringify(event, undefined, 2));
    return {
        statusCode: 200,
        headers: {"Content-Type": "text/plain"},
        body: `Hello, CDK! You've hit ${event.path}\n`
    };
};
```

### コマンドまとめ

CDKからCloudFormationテンプレートを生成(合成)する。

```shell
npx cdk synth
```

Bootstrapスタックをインストールする必要がある。

```shell
npx cdk synth
```

デプロイ

```shell
npx cdk deploy
```

差分チェック

```shell
npx cdk diff
```

ホットスワップデプロイ(危険なので開発時のみ使用) Lambdaのコード部分をさっとデプロイしたい時に使用。(通常のデプロイ30秒。ホットスワップ3秒)

```shell
npx cdk deploy --hotswap
```

コードの変更を見て、ホットスワップデプロイを自動で行う

```shell
npx cdk watch
```

ホットスワップではなく、CloudFormationを通したデプロイを自動で行う。(もちろん長時間)

```shell
npx cdk watch --no-hotswap
```

## おわりに

CDK触ってて、再現性が高いのとコードで表現されているのでマニュアル作る必要無いのが良いなと思いました。
