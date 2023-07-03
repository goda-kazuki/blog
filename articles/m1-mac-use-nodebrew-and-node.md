---
title: "M1 Macにhomebrew経由でnodebrewとnodeをインストールする"
published_at: "2022-06-07"
type: "tech"
published: true
coverImage: "M1Macにhomebrew経由でnodeをインストールする.jpg"
---

M1のMacでnodeをインストールするときに、Intel Macと少し違う箇所があったので、まとめます。

## homebrew導入まで

homebrewの準備については、過去記事を参照してください。 [https://gdtypk.com/m1-mac-use-homebrew/](https://gdtypk.com/m1-mac-use-homebrew/)

## nodebrewのインストール

```
brew install nodebrew
```

この後、nodeをインストールするのでディレクトリを用意しておきます。

```
mkdir ~/.nodebrew
mkdir ~/.nodebrew/src
```

## nodeインストール

```
# どのようなバージョンがあるか確認
nodebrew ls-remote
# 安定版をインストールするならこちら
nodebrew install-binary stable
```

v16未満のnodeをインストールしたい場合、M1Mac用のバイナリが存在しないため、自身でコンパイルする必要があります。

```
nodebrew compile v15.0
```

## nodeを使用する

```
# インストールしているnodeのリストを出力
nodebrew ls
# 使用するnodeのバージョンを選択
nodebrew use v18.3.0
# パスを通す
export PATH=$HOME/.nodebrew/current/bin:$PATH
# ターミナル再起動か設定を再読み込み

# バージョン確認
node -v
```

## おわり

無事、nodeコマンドが使用できるようになりました。 homebrewでアプリを用意するようになってから、とても便利で助かっています。
