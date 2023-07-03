---
title: "Python における便利な文法の使い方と使い所"
published_at: "2023-03-16"
type: "tech"
published: true
coverImage: "Python-における便利な文法の使い方と使い所.jpg"
---

## はじめに

仕事でPythonを書き始めて2,3週間経ちました。 ここまでで学んだ文法系のことをまとめてみました。

今度ライブラリ系についても触れたいなと思います。

### 想定する読者

- 他の言語は経験あるけど初めてPythonを書き初めた人
- Pythonっぽい文法を知りたい人

## 内容

### inによる比較

対象の文字列や数値が配列に含まれているかを調べたいときってよくあると思いますが、 Pythonだとinを使うことでスマートに書くことができます。

```
target_word='たろう'
target_word in ['たろう','じろう','さぶろう']
→True

# 辞書はキーに対して検索を行います
target_word in {'a':'たろう','b':'じろう','c':'さぶろう'}
→False
target_word in {"たろう": "a", "じろう": "b", "さぶろう": "c"}
→True
```

### セイウチ演算子

ifで比較した要素をそれ以降も使うことってよくありますよね。 そのために、ifの前に変数宣言していたと思いますが、 Python3.8から追加されたセイウチ演算子を使うと少しスッキリします。

```
# これまでは事前に定義しておく必要があった。
bool_sample1 = True
if bool:
    print("ターゲットは", bool_sample1, "です")
        →ターゲットは True です

# セイウチ演算子を使うと短く書けます。
if bool_sample2 := True:
    print("ターゲットは", bool_sample2, "です")
        →ターゲットは True です
print("ifを抜けたあとも使えます", bool_sample2)
→ifを抜けたあとも使えます True
```

### スライスを使って文字列から範囲指定して取り出す

ある文字列の先頭何文字かを取得したい時に使える書き方です。

このような形で表現します。 始点：終点:ステップ

注意点としては、始点はその数値を含みますが、終点は含みません。 以下のサンプルにも書いていますが、 始点：1,終点:3とした時は、234と出力されそうな気がしますが、 実際は23になります。

```
target = "123456789"
print(target[5:])
→6789
print(target[1:3])
→23
print(target[-3:])
→789
print(target[0:5:2])
→135
```

### joinによる配列の結合

配列の中の文字列を全て結合したい時にはjoinを使います。 個人的には、配列.join(",")の形の方が直感的な気はしました笑

```
list = ["a", "b", "c"]
print(",".join(list))
→a,b,c
print("a,b,c" == ",".join(list))
→True
```

### 文字列のフォーマット(f文字列)

文字列の中に変数を埋め込みたい時はf文字列を使用します。

```
name = "taro"
print(f"watasi namae {name} desu")
→watasi namae taro desu
```

### forやwhileが正常終了した時にのみ実行させる

forやwhileが正常終了した1回のみ実行したい時にはelseを使います。

ループで作った変数を最後に加工するみたいなのは、よく出てきそうですね。

```
list = [1, 2, 3, 4, 5]

for i in list:
    if i == 6:
        raise Exception
else:
    print("正常")
```

この例だと、リストに6はないので、"正常"のみ出力されます。

### 2つのタプルや配列を辞書に変換する

日本語と英語の配列やタプルがあったとき、辞書に変換したい時はzipとdictを使います。

```
japan = "月曜", "火曜", "水曜"
english = "monday", "tuesday", "wednesday"

print(zip(japan, english))
print(dict(zip(japan, english)))

japan = japan + ("木曜",)
print(dict(zip(japan, english)))
```

ちなみに、zipの返り値はリストでも辞書でもなく、イテラブルな値になっています。 そのため、配列にしたければlistを使うと解決します。

### 辞書内包表記

リストを要素毎にカウントした辞書を作りたい時などあると思います。 for文でループして作成もできますが、辞書内包表記で作ると早く処理できます。

ただ、私は内包表記に慣れないのか、読み手の負荷が高いように感じます・・。

```
day_of_weeks = ["月曜", "火曜", "水曜", "水曜"]

# 辞書内包表記
count = {day_of_week: day_of_weeks.count(day_of_week) for day_of_week in day_of_weeks}
print(count)
→{'月曜': 1, '火曜': 1, '水曜': 2}

# リスト内包表記
list = [day_of_week + "日" for day_of_week in day_of_weeks]
print(list)
→['月曜日', '火曜日', '水曜日', '水曜日']
```

for文で書くとこんな感じになります

```
count = {}
for day_of_week in day_of_weeks:
    count[day_of_week] = day_of_weeks.count(day_of_week)
```

まあ、長いような気がしなくもない・・。

### 集合について

これまで集合を使ったことはなかったので、初めて知りました。

辞書はキーとバリューがあって、キーは重複すると上書きされてしまうと思います。 それと同じ特性を持ち、バリューを排除したものが集合です。

集合演算を使うことができます。

```
set1 = set()
set1 = {1, 2, 3, 4, 5}

set2 = set()
set2 = {5, 6, 7, 8, 9}

set1.add(3)

print(set1)
→{1, 2, 3, 4, 5}

print(set1 & set2)
→{5}

print(set1 | set2)
→{1, 2, 3, 4, 5, 6, 7, 8, 9}

print(set1 - set2)
→{1, 2, 3, 4}
```

できることは理解しやすかったけど、そんなに登場する機会は無い気がしています。

### デフォルト引数に空の配列を使う時は注意しよう

関数の引数にデフォルト値を書くことができます。

ただ、デフォルト値は関数が実行されるたびに格納されるのではなく、 定義された時に格納されます。

そのため、以下のような思わぬバグを生む可能性があるので注意が必要です。

```
def add_list(arg, result=[]):
    result.append(arg)
    return result

print(add_list(1))
→[1]

print(add_list(2))
→[1,2] # [2]が出力されそうな気がしてしまうので注意
```

### 無名関数(ラムダ関数)

引数に関数を受け取るような関数があるとき、 小さい処理を関数として定義して渡すこともできますが、 関数の数が多くなってしまい、メンテが大変になります。

その時にラムダ関数を使用することができます。

```
str_list = ["a", "b", "c"]

print(list(map(lambda value: value.upper(), str_list)))
→["A","B","C"]
```

複雑なものや他の箇所でも使うような場合は、 素直に関数として定義した方が可読性が上がると思います。

### デコレータ

関数を処理する時に、コードを変更せずに、変更を加えたいときがあるが、 その時に使用するのがデコレータである。

説明を見ても、使い所が分からなかったが、 以下のように、引数の情報をデバッグしたりするときに使えるんじゃないかなあと思います。

```
def deco(func):
    def wrapper(*args, **kwards):
        print(func.__name__)
        print(args, kwards)
        return func(*args, **kwards)

    return wrapper

@deco
def print_a(
    arg1,
    arg2,
):
    print("a")

print_a("a", arg2="b")

# 出力結果
print_a
('a',) {'arg2': 'b'}
a
```

### 少しずつ値を返すyield

他の言語でも見かけるyieldがPythonにもあります。 大きすぎるデータを一気に返却すると、その分だけメモリを消費してしまうので、 細かく返却していくために使用するのがyeildです。

以下は入れ子になった配列を平坦化する例です。

```
def flat(items):
    for item in items:
        if isinstance(item, list):
            for i in flat(item):
                yield i
        else:
            yield item

print(list(flat([1, 2, 3, [4], [5, [6, [7]]], 8, 9])))
→[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

実務で見かける機会はそこまで多くないかもしれません。

### クラスメソッド(classmethod)

こちらも使い道が思いつかないのですが、 クラス全体に影響を与えるメソッドになっており、インスタンス化する必要無く呼び出せます。

以下はクラスが何回インスタンス化されたのかをプリントする例になります。

```
class User:
    count = 0

    def __init__(self):
        User.count += 1

    @classmethod
    def print_user_number(cls):
        print(User.count)
        print(cls.count)

a = User()
b = User()
c = User()

User.print_user_number()
```

### 静的メソッド(staticmethod)

クラスにもオブジェクトにも影響を与えないメソッドです。 独立させてもいいけど、意味的にそのクラスに関連するから紐づけておこうみたいな イメージで使うことがあるかなと思います。

```
class User:
    count = 0

    def __init__(self):
        User.count += 1

    @staticmethod
    def ad():
        print("Userクラスを使いませんかー？")

User.ad()
```
