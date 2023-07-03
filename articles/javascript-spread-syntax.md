---
title: "JavaScript,TypeScriptのスプレッド構文の使いどころについて"
published_at: "2022-08-25"
type: "tech"
published: true
coverImage: "JavaScriptTypeScriptのスプレッド構文の使いどころについて.jpg"
---

## はじめに

「プロを目指す人のためのTypeScript入門」を読んでいる時に、 スプレッド構文(...構文)について記載がありました。

コードレビューの時に見かけることがあるので、挙動は何となく分かりますが あまり理解していなかったので、まとめることにしました。

## 使いどころ

### 関数の引数に渡すとき

```typescript
const numbers: [number, number, number,] = [1, 2, 3];

console.log(sumOfThreeNumbers(...numbers));

function sumOfThreeNumbers(number1: number, number2: number, number3: number) {
    return number1 + number2 + number3;
}
```

sumOfThreeNumbersの引数に...numbersと指定することで、 配列が展開され、3つの引数として処理されます。

### 新しい配列の一部として展開するとき

```typescript
const numbers: [number, number, number,] = [1, 2, 3];
const newNumbers = [...numbers, 4];

console.log(newNumbers); // [ 1, 2, 3, 4 ]
```

newNumbersはnumbersの内容を展開しつつ、変数定義することができる。

また、この方法の場合はnumberを変更しても参照先が異なるため、newNumbersには影響が無い。

```typescript
const numbers: [number, number, number,] = [1, 2, 3];
const newNumbers = [...numbers, 4];

numbers[0] = 10;

console.log(newNumbers); // [ 1, 2, 3, 4 ]
console.log(numbers); // [ 10, 2, 3 ]
```

### 新しいオブジェクトの一部として展開するとき

配列時と同様に新しいオブジェクトとして定義される。

```typescript
const obj = {foo: "foo", bar: "bar"};
const obj2 = {...obj, foo2: "foo2"};
const obj3 = {...obj, foo: "foo3"};

obj.foo = "new foo";

console.log(obj); // { foo: 'new foo', bar: 'bar' }
console.log(obj2);// { foo: 'foo', bar: 'bar', foo2: 'foo2' }
console.log(obj3);// { foo: 'foo3', bar: 'bar' }
```

ただし、以下のようにプロパティ名が被っていて、スプレッド構文より先に定義しようとするとコンパイルエラーになる。

```typescript
// エラー TS2783: 'foo' is specified more than once, so this usage will be overwritten.
const obj4 = {foo: "foo2",...obj};
```

恐らく、どうせ上書きされるんだから、定義しても無駄。バグの元だからエラーにします。 ということでしょうか。

### ネストしたオブジェクトの時の注意

さきほど、新しいオブジェクトの一部として展開されると書きましたが、 ネストされているオブジェクトに関しては、対象外となります。

```typescript
const obj = {
    foo: "foo",
    bar: "bar",
    nest: {nest_foo: "nest_foo"}
};
const obj2 = {...obj};

obj.nest.nest_foo = "aaaa"
obj.foo = "bbb"
obj.bar = "ccc"

console.log(obj); //{ foo: 'bbb', bar: 'ccc', nest: { nest_foo: 'aaaa' } }
console.log(obj2); //{ foo: 'foo', bar: 'bar', nest: { nest_foo: 'aaaa' } }
```

obj.nest.nest\_fooを書き換えると、obj2.nest.nest\_fooも書き換わっています。

これは「各プロパティが同じ値を持つ新しいオブジェクトを作る」というものであり、 プロパティの中にオブジェクトが入っていた場合はそれは相変わらず同じオブジェクトのままになります。 (シャローコピーというみたいです)

もし、ネストしたものも含めて新しいオブジェクトを作りたい時は、以下のようにJSONに変換してから対応します。

```typescript
const obj = {
    foo: "foo",
    bar: "bar",
    nest: {nest_foo: "nest_foo"}
};

const obj2 = JSON.parse(JSON.stringify(obj));

obj.nest.nest_foo = "aaaa"
obj.foo = "bbb"
obj.bar = "ccc"

console.log(obj); //{ foo: 'bbb', bar: 'ccc', nest: { nest_foo: 'aaaa' } }
console.log(obj2); //{ foo: 'foo', bar: 'bar', nest: { nest_foo: 'nest_foo' } }
```

## おわりに

関数の引数に渡すっていうのは、現実的に使うことが無さそうな気がしましたが、 見かけたら追記しようと思います。

展開する時には新しい配列,オブジェクトとして定義されるのは知らなかったので勉強になりました。
