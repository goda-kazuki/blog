---
title: "AWS CodePipeline CodeDeploy CodeDeploy触ってみた"
published_at: "2022-11-09"
type: "tech"
published: true
coverImage: "AWS-CodePipeline-CodeDeploy-CodeDeploy触ってみた.jpg"
---

## はじめに

AWS Certified DevOps Engineer - Professionalの勉強中にCodeシリーズが出てきたので、 AWS公式のチュートリアルに沿って学習しました。 その手順を記録しておきます。

GitHub [https://github.com/goda-kazuki/codepipeline-serverless](https://github.com/goda-kazuki/codepipeline-serverless)

## AWS SAM

### 準備

AWS SAM CLIのインストール [https://docs.aws.amazon.com/ja\_jp/serverless-application-model/latest/developerguide/serverless-sam-cli-install-mac.html](https://docs.aws.amazon.com/ja_jp/serverless-application-model/latest/developerguide/serverless-sam-cli-install-mac.html)

初期テンプレのアプリを用意 [https://docs.aws.amazon.com/ja\_jp/serverless-application-model/latest/developerguide/serverless-getting-started-hello-world.html](https://docs.aws.amazon.com/ja_jp/serverless-application-model/latest/developerguide/serverless-getting-started-hello-world.html)

### 重要なファイル

- template.yaml
    - SAMリソースを定義
- hello\_world/app.py
    - Lambdaハンドラーロジック
- hello\_world/requirements.txt
    - 依存関係の定義。
    - `sam build`時に利用

### メモ

- `sam build`
    - .aws-samディレクトリに依存関係を解消したものを配置
    - 要らないディレクトリも生成されている(Python経験が無いだけで、必要なのかも)
- `sam deploy --guided`
    - guidedはCLIのプロンプトガイドを表示するオプション
    - 裏ではCloudFormationが動いてるっぽい
- `aws cloudformation delete-stack --stack-name sam-app --region region`
    - クリーンアップ

## CodePipeline

使用した資料 [https://docs.aws.amazon.com/ja\_jp/codepipeline/latest/userguide/tutorials-serverlessrepo-auto-publish.html](https://docs.aws.amazon.com/ja_jp/codepipeline/latest/userguide/tutorials-serverlessrepo-auto-publish.html)

### やったこと

- パイプラインの作成
- ソースステージをGitHubにして接続
- CodeBuildの設定
    - S3にzipファイルが出力されていることを確認
- CodeDeployの設定
- Lambdaにデプロイされることを確認
- おまけ
    - SNSでデプロイされたら通知してみる

### メモ

資料にちゃんと書いてくれてるのに、勝手に詰まってたポイント

- CodeBuild
    
    - buildspec.ymlに`template.yml`と指定してたが、正しくは`template.yaml`だったので、エラーでコケてた。
    - ビルドする際に、S3に出力されるが、IAMロールに権限を追加する必要がある
- CodeDeploy
    
    - 既存のLambdaにデプロイしようとすると、デプロイが完了しなかった。
    - メタデータをtemplate.yamlに記入しないとデプロイできないようなので、以下を記載
        
        ```yaml
        Metadata:
        AWS::ServerlessRepo::Application:
        Name: codepipeline-serverless
        Description: hello world
        Author: goda
        ```
