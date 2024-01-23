---
title: "defaultdictを使って楽に配列カウントをする"
emoji: "🕌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python","defaultdict"]
published: true
---

# はじめに

配列で出てきたものを種類ごとにカウントしたいことってあると思います。
Pythonでは標準でcollectionsライブラリが用意されていて、
その中のdefaultdictが良い感じだったので紹介します。

defaultdictの公式ドキュメント
<https://docs.python.org/ja/3.10/library/collections.html#collections.defaultdict>

# やりたいこと

以下のような配列があった時、種類ごとに何回配列に出てきたかカウントしたいです。

```:python
fruits = ["りんご", "ばなな", "みかん", "りんご", "りんご", "みかん"]
```

# 実現方法

## dict

dictを使って素直に実装すると以下のようになります。

```:python
fruits = ["りんご", "ばなな", "みかん", "りんご", "りんご", "みかん"]

count_dict = dict()
for i in fruits:
    if i in count_dict:
        count_dict[i] += 1
    else:
        count_dict[i] = 1

print(count_dict)
# >{'りんご': 3, 'ばなな': 1, 'みかん': 2}
```

## defaultdict

defaultdictを使って素直に実装すると以下のようになります。

```:python
from collections import defaultdict

fruits = ["りんご", "ばなな", "みかん", "りんご", "りんご", "みかん"]

count_defaultdict = defaultdict(int)

for i in fruits:
    count_defaultdict[i] += 1

print(count_defaultdict)
# >defaultdict(<class 'int'>, {'りんご': 3, 'ばなな': 1, 'みかん': 2})
```

着目してほしい点は以下になります。

```:python
for i in fruits:
    count_defaultdict[i] += 1
```

dictと比較して存在しないキーに対して1をセットしていません。
これは、defaultdictの引数にintを指定しているため、
存在しないキーに対しては0が自動的にセットされるからです。

## その他挙動の違い

dictの場合、存在しないキーを取得しようとするとKeyErrorになりますが
defaultdictの場合、初期値が取得できます。

```:python
print(count_dict["ぱいなっぷる"])
# >KeyError: 'ぱいなっぷる'

print(count_defaultdict["ぱいなっぷる"])
# >0
```

dictで同じようなことをしようとすると、こんな感じです。

```:python
print(count_dict.get("ぱいなっぷる", int()))
```

お分かりの通り、defaultdictの引数で指定した関数を実行し、それを初期値としてくれます。便利ですね。

## 最初はdefaultdictで作って、参照する際はdictにしたい時

以下のようにすることで、dictとして扱うことができます。

```:python
print(dict(count_defaultdict)["ぱいなっぷる"])
# > KeyError: 'ぱいなっぷる'
```

ただ、dict変換前に参照していると、初期値がセットされてしまうのでエラーになりません。

```:python
print(count_defaultdict["ぱいなっぷる"])
# >0

print(dict(count_defaultdict)["ぱいなっぷる"])
# >0

```

# おわりに

条件分岐が1つ減って、単純に便利だなと思いました。
他の言語でもあるんですかねー。

# 参考

<https://qiita.com/xza/items/72a1b07fcf64d1f4bdb7>
