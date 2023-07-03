---
title: "M1 Macでhomebrewが使えなかったときの対処法 (command not found: brew)"
published_at: "2022-06-01"
type: "tech"
published: true
coverImage: "M1-Mac-homebrew-install.jpg"
---

M1 Macを購入し、homebrewをインストールしたところ、 使えなかったので、その対応を残します。

## 手順

公式サイトで紹介されているコマンドでインストール

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

[https://brew.sh/index\_ja](https://brew.sh/index_ja)

brewコマンドが使用できるのか確認

```
[~]$ brew -v
command not found: brew
```

コマンドが無いとのこと・・。

M1の場合、opt/homebrewディレクトリにインストールされているようなので、 パスを通す

```
vi ~/.zshrc
export PATH="/opt/homebrew/bin:$PATH"
source ~/.zshrc
```

再度、コマンドが使用できるか確認

```
[~]$ brew -v
Homebrew 3.4.11
Homebrew/homebrew-core (git revision 17d4c381377; last commit 2022-06-01)
```

無事、使用する準備ができました。
