---
title: "arm64のLambdaでpydantic_coreモジュールが見つからなかった"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Python","Lambda","Arm","Pydantic","FastAPI"]
published: true
---

# はじめに

LambdaのOSアーキテクチャをArmに設定した状態でデプロイすると、エラーになったので解決方法を残します。

# 問題

Lambdaにコードをデプロイすると、以下のエラーが表示されました。

```sh
No module named 'pydantic_core._pydantic_core'
```

手元(Amazon Linux2023 X86)では動いていたのに、何でだろうなーと思いました。

arm64とX86で互換性が無いからかと思って、ビルドとデプロイする環境をM1 Macに切り替えたのですが、状況変わらずでした。

# 解決方法

以下のリンクを見つけ、Amazon Linux2023 arm64のインスタンスからデプロイするとうまくいきました。
<https://github.com/pydantic/pydantic/issues/6557>

試していませんが、Amazon Linux arm64のDockerイメージが存在するようなので、
それを経由してもデプロイすることが可能だと思います。
<https://hub.docker.com/_/amazonlinux/tags>

# おわりに

新しくarm64の環境を作成するのが大変だったのですが、
今後arm64でデプロイすることが増えると思うので、徐々に開発環境を移行しようと思います。

# 参考

<https://github.com/pydantic/pydantic/issues/6557>
<https://hub.docker.com/_/amazonlinux/tags>
