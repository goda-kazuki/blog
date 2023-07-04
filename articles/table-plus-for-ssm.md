---
title: "AWSのSystemsManager経由でTablePlusを使う時の設定"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","SSM","SystemsManager","TablePlus"]
published: false
---

## はじめに
開発環境を手元のMacからEC2に移行しようとしています。

EC2に構築したMySQLサーバーに対してSSMを経由してTablePlusで接続しようとすると詰まったので残しておきます。

## 結論
以下のように設定することで接続することができました。

```text:~/.ssh/config
host ec2
    HostName "インスタンスID"
    Port 22
    User ec2-user
    IdentityFile "/Users/ユーザー名/.ssh/personal-development.pem"     
    ProxyCommand sh -c "PATH=/usr/local/bin:$PATH && PATH=/usr/local/bin:/opt/homebrew/bin  && aws ssm start-session --target %h --document-name AWS-StartSSHSession --parameters 'portNumber=%p' --profile default"
```

## 過程
こちらの記事を参考にSSM経由でSSH接続できるようにはなりました。
https://dev.classmethod.jp/articles/how-to-use-vscode-remote-ssh-with-aws-systems-manager/

ただ、TablePlusをAlfredやアプリケーションから開くと以下のようにSSH接続できませんでした。

![](/images/2023-07-05_8.44.57.png)

下記Issueでも同じ問題が報告されています。
https://github.com/TablePlus/TablePlus/issues/2531

こちらのコメントを参考に変更したのですが、それでもうまくいきませんでした。
https://github.com/TablePlus/TablePlus/issues/2531#issuecomment-1071949787

## 原因
私はTablePlusをHomebrew経由でインストールしていることが原因のようでした。

そのため、ProxyCommandで通すパスに`PATH=/usr/local/bin:/opt/homebrew/bin`を指定してあげることで解決しました。

## おわり
セキュリティグループでSSHを公開したり、VPN経由しなくてよいSSMは便利で良いですね！