---
title: "AWS IoT Core モノをCLIで楽して作りたい"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","IoT","awsiotcore","awscli"]
published: true
---


# 問題

IoT Coreのモノを複数台作る必要があったのですが、CloudFormationで実現することができないことが分かり、
CLIで実現したので、その手順を残します。

# 結論

```sh
# モノを作成
aws iot  create-thing --thing-name Test-Device

# 証明書を作成(証明書のArnは控えておいてください)
aws iot create-keys-and-certificate --set-as-active --certificate-pem-outfile "certificate.pem" --public-key-outfile "publicKey.pem" --private-key-outfile "privateKey.pem"

# 証明書にポリシーをアタッチ
aws iot attach-policy --policy-name {ポリシー名} --target {証明書のArn}

# モノに証明書をアタッチ
aws iot attach-thing-principal --thing-name Test-Device --principal {証明書のArn}
```

# 前提

IoT Core モノに関してはCloudFormationで作成することができませんが、
IoT CoreのポリシーはCloudFormationで作成できるので作っておきましょう。

```yml
Resources:
  IotPolicy:
    Type: AWS::IoT::Policy
    Properties:
      PolicyName: sample-policy-name
      PolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Action: iot:*
            Resource: "*"
```

# おわりに

何回もマネジメントコンソールをポチポチするのは苦痛なので、それが改善できてよかったです。

# 参考

モノ側から送信する際に、手元に出力した証明書に加えAmazon Root CAが必要になることがあります。
以下のリンクからダウンロードすることが可能です。
<https://docs.aws.amazon.com/ja_jp/iot/latest/developerguide/server-authentication.html#server-authentication-certs>

AWS::IoT::Policy CloudFormationドキュメント
<https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-resource-iot-policy.html>
