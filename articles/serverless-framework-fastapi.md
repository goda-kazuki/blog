---
title: "ServerlessFrameworkとFastAPIを使ってみた"
emoji: "🦄"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","ServerlessFramework","FastAPI","Mangum"]
published: true
---

## はじめに
これまでLambdaでAPIを開発する際に特にフレームワークを使用していませんでした。
入力値のチェックなど、いろいろな部分でフレームワークに乗っかって開発した方が効率的だと思って、
Serverless FrameworkとFastAPIで開発できないか試します。

## ポイント
### Mangumについて
Mangum
https://github.com/jordaneremieff/mangum

Lambdaで受け取ったリクエストをそのままFastAPIなどのフレームワークに渡すことはできません。
Mangumがリクエストを中継することで、Lambda上でFastAPIを動かすことができるようになります。

### ローカルで開発が可能に
これまでLambdaをローカル実行しようとすると、以下のようなスクリプトを書いてやる必要がありました。

```python
if __name__ == "__main__":
    import base64
    import json

    class LambdaContext:
        function_name: str = "lambda-function"
        memory_limit_in_mb: int = 256
        invoked_function_arn: str = "arn:aws:lambda:ap-northeast-1:123456789012:function:lambda-function"

    event = {
        "httpMethod": "GET",
        "body": body,
        "isBase64Encoded": True,
    }
    result = handler(event, LambdaContext())
    print(result)
```

それが、Webフレームワークに乗っかることでローカル実行できるようになります。

uvicorn
https://www.uvicorn.org/
ASGI Webサーバーで動作するアプリケーションをローカルで実行するためのツールです。

起動コマンド
```shellScript
uvicorn app.app:app --reload
```

## 気になる点
### CloudWatchのログが1つに貯まってしまう
どのURLでアクセスしたとしても、1つのLambdaにアクセスが集中するためログが1ヶ所に貯まることになります。
検索やログの情報次第で回避できますが、1API1Lambdaの頃と比べると見にくいかもしれません。

または、serverless.ymlを以下のように変えると、複数Lambda関数ができるので回避できそうです。
(同じファイルが複数できるので、どうなのかなとは思います)

```yml:serverless.yml
  HelloFunction1:
    handler: app.lambda_handler.handler
    name: ${param:resourceNamePrefix}-hello1
    description: hello function
    memorySize: 512
    timeout: 3
    role: HelloFunctionRole
    # environment:
    events:
      - http:
          path: /
          method: ANY
    layers:
      - Ref: PythonRequirementsLambdaLayer

  HelloFunction2:
    handler: app.lambda_handler.handler
    name: ${param:resourceNamePrefix}-hello2
    description: hello function
    memorySize: 512
    timeout: 3
    role: HelloFunctionRole
    # environment:
    events:
      - http:
          path: /{proxy+}
          method: ANY
    layers:
      - Ref: PythonRequirementsLambdaLayer
```

### Lambdaのロールがデカくなりがち
CloudWatchのログと同じですが、
S3を操作するAPIやDynamoDBにアクセスするAPIがあった時、
それらを可能にするポリシーをロールにアタッチすると思います。

関数が1つだと、APIに対して不必要なポリシーが渡ってしまうのでベストとは言えません。
もし、細かく制御したいのであれば、複数Lambda関数を作成するのが良いのかなと思います。

## 参考にさせていただいた記事
https://zenn.dev/nameless_sn/articles/fastapi_tutorial_for_rest
https://zenn.dev/hayata_yamamoto/articles/781efca1687272
https://techblog.raksul.com/entry/2023/06/30/081048
https://aws.amazon.com/jp/builders-flash/202304/api-development-sam-fastapi-mangum/

## 成果物
見たい人がいれば

### serverless.yml
```yml:serverless.yml
service: sls-fastapi
frameworkVersion: "3"
useDotenv: true

provider:
  name: aws
  runtime: python3.10
  stage: ${opt:stage, "dev"}
  region: ${opt:region, "ap-northeast-1"}
  environment:
    LOG_LEVEL: ${env:LOG_LEVEL}
  versionFunctions: false

plugins:
  - serverless-python-requirements

custom:
  pythonRequirements:
    layer: true

params:
  default:
    resourceNamePrefix: ${self:service}-${self:provider.stage}
    resourceNameSuffix: "-${aws:accountId}"
  dev:
  stg:
  prd:

package:
  patterns:
    - "!*/**"
    - "!*"
    - "app/**"

functions:
  HelloFunction:
    handler: app.lambda_handler.handler
    name: ${param:resourceNamePrefix}-hello
    description: hello function
    memorySize: 512
    timeout: 3
    role: HelloFunctionRole
    # environment:
    events:
      - http:
          path: /{proxy+}
          method: ANY
      - http:
          path: /
          method: ANY
    layers:
      - Ref: PythonRequirementsLambdaLayer

resources:
  Resources:
    HelloFunctionRole:
      Type: AWS::IAM::Role
      Properties:
        AssumeRolePolicyDocument:
          Version: "2012-10-17"
          Statement:
            - Effect: Allow
              Principal:
                Service:
                  - lambda.amazonaws.com
              Action:
                - "sts:AssumeRole"
        Description: HelloFunctionRole
        RoleName: ${param:resourceNamePrefix}-role-hello-function
        Path: /
    HelloFunctionPolicy:
      Type: AWS::IAM::Policy
      Properties:
        PolicyDocument:
          Version: "2012-10-17"
          Statement:
            - Effect: Allow
              Action:
                - "logs:CreateLogGroup"
              Resource:
                - !Sub arn:${AWS::Partition}:logs:${AWS::Region}:${AWS::AccountId}:log-group:/aws/lambda/${param:resourceNamePrefix}-hello
            - Effect: Allow
              Action:
                - "logs:CreateLogStream"
                - "logs:PutLogEvents"
              Resource:
                - !Sub arn:${AWS::Partition}:logs:${AWS::Region}:${AWS::AccountId}:log-group:/aws/lambda/${param:resourceNamePrefix}-hello:*
        PolicyName: ${param:resourceNamePrefix}-policy-hello-function
        Roles:
          - !Ref HelloFunctionRole
```

### Pipfile

```text:Pipfile
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]
fastapi = "*"
uvicorn = "*"
mangum = "*"
aws-lambda-powertools = "*"

[requires]
python_version = "3.10"
```

### Pythonファイル

```python:app/lambda_handler.py
from mangum import Mangum
from app.app import app

handler = Mangum(app)
```

```python:app/app.py
from fastapi import FastAPI
from starlette.requests import Request
from aws_lambda_powertools.utilities.data_classes import APIGatewayProxyEvent

app = FastAPI()

@app.get("/")
def get_hello(request:Request):
    from app.functions.hello import main
    return main(request.scope.get("aws.event",generate_default_event("GET")),request.scope.get("aws.context"))

@app.get("/{item_id}")
def get_hello(request:Request,item_id:int):
    from app.functions.item import main
    return main(request.scope.get("aws.event",generate_default_event("GET")),request.scope.get("aws.context"),item_id)


def generate_default_event(http_method:str,query_string_parameters:dict={},body:dict={}):
    return APIGatewayProxyEvent(
        data={
            "httpMethod":http_method,
            "headers":{'Content-Type': 'application/json'},
            "queryStringParameters":query_string_parameters,
            "body":body
        }
    )
```

```python:app/functions/hello.py
from aws_lambda_powertools.utilities.data_classes import APIGatewayProxyEvent

def main(event:APIGatewayProxyEvent, context):
    body = {
        "message": "Go Serverless v1.0! Your function executed successfully!",
        "event":event,
    }

    return body
```

```python:app/functions/item.py
import json
from aws_lambda_powertools.utilities.data_classes import APIGatewayProxyEvent

def main(event:APIGatewayProxyEvent, context,item_id):
    body = {
        "message": "Go Serverless v1.0! Your function executed successfully!",
        "event":event,
        "item_id":item_id
    }

    return body
```