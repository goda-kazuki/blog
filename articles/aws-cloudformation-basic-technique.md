---
title: "CloudFormationの勉強中に見つけた基礎的なテクニック"
published_at: "2022-06-20"
type: "tech"
published: true
coverImage: "CloudFormationの勉強中に見つけた基礎的なテクニック.jpg"
---

## はじめに

CloudFormationを勉強していた時のまとめです。

## テンプレートファイル

```
AWSTemplateFormatVersion: '2010-09-09'

Parameters:
  EnvironmentType:
    Type: String
    AllowedValues:
      - master
      - staging

  VpcCIDR:
    Type: String
    Default: 10.0.0.0/16

  RepositoryName:
    Type: String
    Default: gdtypk-repo

Mappings:
  EnvironmentToParams:
    master:
      Name: master

    staging:
      Name: staging

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCIDR
      Tags:
        - Key: Name
          Value: !Sub
            - "${env_name}-VPC"
            - env_name: !FindInMap
                - EnvironmentToParams
                - !Ref EnvironmentType
                - Name

  S3:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub
        - "${env_name}-${repository_name}"
        - env_name: !FindInMap [ EnvironmentToParams,!Ref EnvironmentType,Name ]
          repository_name: !Ref RepositoryName
      AccessControl: PublicRead
```

## ポイント

### Sub 変数を用意して、文字列をつなげたいとき

Sub関数を使うと変数(env\_name)を用意して、そこに代入するような形で記述することができる。

下記の例だと、Valueにはsample-VPCが格納される。

```
      Tags:
        - Key: Name
          Value: !Sub
            - "${env_name}-VPC"
            - env_name: sample
```

### FinfinMapとRefを使って、MappingsとParametersを使う時

FindinMapの中で、Refを使うことで、Parametersの値を取得することができる

下記の例だと Mappings->EnvironmentToParams->○○->Name を参照しようとしている。

○○の部分は、CloudFormationを実行する時のParametersに使用する値を参照している

```
      Tags:
        - Key: Name
          Value: !FindInMap
                - EnvironmentToParams
                - !Ref EnvironmentType
                - Name
```

### CLIでParametersに値を渡す時

parametersオプションで渡すことができる。 ParameterKeyとParameterValueはカンマ区切りにして、セットで渡す必要がある

```
aws cloudformation create-stack --template-body file://front/cloudformation.yaml --stack-name dgtypk-sample --parameters  ParameterKey=EnvironmentType,ParameterValue=staging
```

## おわり

CloudFormationを勉強していると、途中でCDKの存在を知りました。 圧倒的にそっちの方が良さそうだったので、CDKを使おうと思います。

ただ、そもそもIaCをして得られるメリットが今やっている案件ではあまり無さそうなので 見送ることになりそう・・。
