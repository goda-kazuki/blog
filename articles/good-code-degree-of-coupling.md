---
title: "良いコードとは何か 結合度についてまとめ"
published_at: "2022-08-15"
type: "tech"
published: true
coverImage: "良いコードとは何か結合度についてまとめ.jpg"
---

## はじめに

前回の記事では凝集度についてまとめたため、 今回は結合度について整理しようと思います。

前回の記事 [https://gdtypk.com/good-code-cohesion/](https://gdtypk.com/good-code-cohesion/)

## 結合度

関数の独立性を表現する尺度のこと。 高ければ悪く、低いと良いとされています。

結合度が高い状態の場合、ある関数が持つデータを変更すると意図せず他の関数に影響あり、不具合につながる恐れがあります。

7種類あるので高い順に説明します。

### 内容結合

宣言されていない変数での結合のこと。 ポインタを指定して処理するもの。 現代の多くのプログラミング言語はメモリ安全であるため、書くことができないことが多いみたいです。

```
function main(){
int *a = 0x00000008;
*a = 10;
twice();
echo *a;
}

function twice(){
int *num = 0x00000008;
*num = *num * 2;
}
```

コードの挙動予測が難しくなってくる。 改善方法は以下のように引数を受け取り返すようにする。

```
function twice(int a){
return a*2;
}
```

### 共通結合

共通の複数のグローバル変数(構造体やクラス)で結合のこと。 userIdAdultの呼ばれるタイミングを把握してなければ再利用が難しい。

```
type User struct{
    Name string
    Age int
    IsAdult bool
}

var user *User
function main(){
    user = User{Name:"山田",Age:20}
    userIdAdult()
}

function userIdAdult(){
    user.IsAdult = user.Age > 19
}
```

改善方法は内部結合と同じように引数を受け取り返すようにする。

### 外部結合

共通のスカラ値のグローバル変数で結合すること。 共通結合と同じように見えるが、スカラ値なので結合度が少し低い。

```
var number;
function main(){
    number = 10
    twice()
}

function twice(){
    number = number * 2;
}
```

改善方法は内部結合と同じように引数を受け取り返すようにする。

### 制御結合

制御フラグで結合すること。 [論理的凝集](https://gdtypk.com/good-code-cohesion/#toc4)と似ている。 呼び出し側が実装の中身を理解して制御フラグを渡す必要があったり、 不必要な引数が増えてしまう問題がある。

### スタンプ結合

構造体やクラスで結合すること。 良い悪いは特になく、適切であればOK。 ただ、例でいうprintUserで使わない構造が多い場合は検討した方が良い。(NameとAgeだけ渡すとか)

```
type User struct{
    Name string
    Age int
    IsAdult bool
}

function main(){
    var user = User{Name:"山田",Age:20}
    printUser(user)
}

function printUser(user User){
    echo "Name:" + user.Name + "Age:" + user.Age
}
```

### データ結合

スカラ型(単一の値を表したり格納する型)の値で結合 良い悪いは特になく、適切であればOK。 スタンプ結合と比較すると、受け渡す数が少ないため結合度が低いと言える。

```
type User struct{
    Name string
    Age int
    IsAdult bool
}

function main(){
    var user = User{Name:"山田",Age:20}
    printName(user.Name)
}

function printUser(user User){
    echo "Name:" + user.Name
}
```

### メッセージ結合

引数のない関数の呼び出しで結合のこと。 関数の実行でしか結合していないため、最も結合度が低い。

```
function main(){
    printHello()
}

function printHello(){
    echo "Hello"
}
```

## まとめ

| 名前 | 対応 |
| --- | --- |
| 共通結合 | 絶対避けよう |
| 外部結合 | 避けよう |
| 制御結合 | できれば避けよう |
| スタンプ結合 | 場合によってOK |
| データ結合 | OK |
| メッセージ結合 | OK |

## おわりに

7種類学んでみましたが、基本は守れているかなといった感じでした。 新人の方などのレビューを担当する時の説明に使えそうなので良かったです！
