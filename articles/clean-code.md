---
title: "クリーンコードで学習したことのまとめ(分割代入,デフォルト引数,参照透過性など)"
published_at: "2022-12-12"
type: "tech"
published: true
coverImage: "クリーンコードまとめ.jpg"
---

クリーンコードについて学習したため、その中で重要だと思ったことをまとめていきます。

## 前提

特にバージョンは関係無いと思いますが、Nodeのバージョンは18.3.0です。

## 変数

### Classではプライベートの変数を使いましょう

```javascript
class Person {
    // _の場合は、プライベートフィールドであることは分かるけど、呼び出すことが可能
    _privateAge
    // #の場合は、呼び出しが制限されるため、基本こちらを使うと良い
    #privateAge

    constructor(age) {
        this._privateAge = age;
        this.#privateAge = age;
    }
}

const person = new Person(123);
console.log(person._privateAge);
// → 123

console.log(person.#privateAge);
// → SyntaxError: Private field '#privateAge' must be declared in an enclosing class
```

### 分割代入を使いましょう

#### 配列の引数を順番に入れる

```javascript
const nowDate = new Date("2022-12-12").toLocaleDateString();
const [year, month, day] = nowDate.split("/");
console.log(`${year}年${month}月${day}日`);
```

#### オブジェクトから必要なものだけ入れる

```javascript
// NG例
const user = fetchLoginUser();
const userName = user.userName;
console.log(userName);

// OK例
// userNameだけを使ってることが分かる。
const {userName} = fetchLoginUser();
console.log(userName);

function fetchLoginUser() {
    return {userName: "sample", age: 27}
}
```

#### 最初とその他で分けたい時

```javascript
const skills = ["ruby", "js", "python"];
const [firstSkills, ...otherSkills] = skills;
console.log(firstSkills);
// → ruby
console.log(otherSkills);
// → [ 'js', 'python' ]
```

### レキシカルスコープ(静的スコープ、外部スコープ)は使わないようにしよう

関数は参照透過性がある方が良い。 レキシカルスコープを参照していると、関数を呼び出す前に変数が変更されていると参照透過性が担保できない。

回避方法としては、関数の引数に値を渡すことで解決するようにする。

```javascript
// NG例
const user = {name: "tanaka"};

function greeteng() {
    console.log(user.name);
}

// OK例
// 引数を介してやりとりする。コードも追いやすくなる。userを使ってることが明確。
function greeting(user) {
    console.log(user.name);
}
```

## 関数

### デフォルト引数を使いましょう

#### 一般的なやつ

```javascript
function createUser(name = "sample", age = 27) {
    console.log(name, age);
}

createUser();
// → sample 27
createUser("taro", 5);
// → taro 5
```

#### オブジェクトを渡す関数でもデフォルト引数は使える

```javascript
function createUser({name = "sample", age = 27} = {}) {
    console.log(name, age);
}

// オブジェクト自体が渡ってこなければ、デフォルト引数で{}が渡る。
createUser();
// → sample 27

// // オブジェクトに値が足りなければ、初期値が使われる。
createUser({name: "taro"});
// → taro 27
```

### 関数にフラグを渡すのをやめましょう

これは過去記事にも書いた論理的凝集にあたる。 [https://gdtypk.com/good-code-cohesion/#toc4](https://gdtypk.com/good-code-cohesion/#toc4)

#### NG例

```javascript
function getUser(userId, isAdmin = false) {
    if (isAdmin) {
        console.log({type: "Admin", name: "管理者"});
    } else {
        console.log({type: "General", name: "一般"});
    }
}

getUser(100, false);
// { type: 'General', name: '一般' }
getUser(100, true);
// { type: 'Admin', name: '管理者' }
```

#### OK例

条件分岐は呼び出す側で行うようにすると、再利用性も高まる。

```javascript
function getAdmindUser(userId) {
    console.log({type: "Admin", name: "管理者"});
}

function getGeneralUser(userId) {
    console.log({type: "General", name: "一般"});
}

const isAdmin = true;
if (isAdmin) {
    getAdmindUser(100);
} else {
    getGeneralUser(100);
}
```

### 関数の副作用について気をつけよう

関数の副作用とは、関数の実行の戻り値以外にも影響を与えていること。基本的には無い方が良い 例えば、以下のようなもの

- console.logの出力
- 引数で渡ってきた中身を書き換えること

## クラス

### クラス継承

DogはAnimalの一種という、is-aの関係で定義する。 継承元は1つしか定義できないため、ガチガチに設計を作り込まないと変更が大変

```javascript
class Animal {
    eat() {
        console.log("たべる");
    }
}

class Dog extends Animal {
    fly() {
        console.log("飛ぶ")
    }
}
```

### コンポジション

TVはSwitchを持っているという、has-aの関係で定義する。 後からつけることができるため、設計が楽。 ただし、複数コンポジションを定義すると、1つのクラスで複数呼ばないといけないので、管理が煩雑になってしまう。

```javascript
class Switch {
    changeChannel() {
        console.log("チャンネル変更")
    }
}

class TV {
    #switch

    constructor() {
        this.#switch = new Switch();
    }
}
```
