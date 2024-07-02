---
title: "【Python開発に欠かせない！】pydanticの使い方とメリット"
published_at: "2023-03-24"
type: "tech"
published: true
coverImage: "Python開発に欠かせない！】pydanticの使い方とメリット.jpg"
---

## はじめに

pythonで堅牢な開発をしたい時、pydanticが候補に上がってくると思います。 基本的な使い方、よく使いそうなことをまとめてみました。

## 想定読者

Pythonを最近触り始めて、型がある開発をしたいと思って、pydanticの存在を知った人 pydanticでできることをざっくり知りたい人

## pydanticとは

[pydantic](https://docs.pydantic.dev/)はデータのバリデーションや型ヒントを提供します。 これにより、Pythonで安全な開発を行うことができます。

逆にPythonの書きやすさみたいなものは少し犠牲にするかもしれませんが、 仕事で使うなら、安全さを優先したいかなと思います。

なお、Pydanticは入力ではなく、出力モデルの型と制約を保証するものになります。 (intの定義のところにfloatを突っ込むとintに変換されてしまう)

## dataclassじゃだめなのか

Pythonにデフォルトでdataclassがあります。 アノテーションはされるんですが、厳密ではありません。

intの定義のところにfloatを突っ込むとfloatが登録されてしまう

```Python
@dataclass
class User2:
    name: str
    age: int

user2 = User2(name=1.23, age="aa")
# User2(name=1.23, age='aa')
print(user2)
```

## 準備

Pythonのバージョンやpipenvは揃える必要はありません。 pydanticのみインストールしてください。

```shell
$pipenv --python 3.9.16
$pipenv shell
$pip install pydantic
```

## モデル

[https://docs.pydantic.dev/usage/models/](https://docs.pydantic.dev/usage/models/)

### dictとparse\_objについて

dictでモデルを辞書に変換することができます。 parse\_objで辞書からクラスを作成することができます。

```Python
user = User(id=123, age=27)
user_dict = user.dict()
user_dict["test"] = "aaa"
user2 = User.parse_obj(user_dict)
```

### バリデーションのエラーについて

バリデーションのエラーに引っかかった場合は、ValidationErrorがraiseされるので、 try,exceptで補足してあげることで管理することが可能です。

```Python
class User(BaseModel):
    id: int
    age: int

try:
    user = User(id="aaa", age=27)
except ValidationError as e:
    print(e.json())
    # [{"loc": ["id"],"msg": "value is not a valid integer","type": "type_error.integer"}]
```

### 自作のバリデーション作成

デフォルトの型だけでは不十分な場合、自分でバリデーションを作成することができます。 また、raiseするエラーも自由なので、エラーによって後の処理を管理することが可能です。

```Python
class User(BaseModel):
    id: int
    age: int

    @validator("age")
    def age_size(cls, v):
        if 0 <= v <= 100:
            return v
        else:
            raise ValueError("範囲外のageが入力されました。")

try:
    user = User(id=123, age=999)
except ValueError as e:
    print(e.json())
    # [{"loc": ["age"],"msg": "範囲外のageが入力されました。","type": "value_error"}]
```

### Fieldを使った一般的なバリデーション

先ほどの自作バリデーション、自由度は高いのですが よくあるパターンに対しても書かないといけないのは大変ですよね。

Field関数を使えば、よくあるパターンに関しては対応することができます。 [https://docs.pydantic.dev/usage/schema/#field-customization](https://docs.pydantic.dev/usage/schema/#field-customization)

```Python
class User(BaseModel):
    age: int = Field(..., ge=0, le=100)

try:
    User(age=101)
except ValidationError as e:
    print(e.json())
    # [{"loc": ["age"],"msg": "ensure this value is less than or equal to 100","type": "value_error.number.not_le","ctx": {"limit_value": 100}}]
```

自作のバリデーションを作る前にFieldで補えないか確認するのが良さそうですね。

### 必須と必須ではないフィールドを分ける

それぞれ分けたい時は以下のような記述をします。

```Python
class User(BaseModel):
    id: int  # 必須
    age: Optional[int]  # 必須ではない

user = User(id=123)
print(user)
# id=123 age=None
```

### 動的なデフォルト値を持つフィールドの作り方

日付やuuidなんかは動的に生成したい時があると思いますが、 Field関数とdefault\_factoryを使うことで叶えられます。

```Python
class User(BaseModel):
    now: datetime = Field(default_factory=datetime.now)

user = User()
print(user)
# now=datetime.datetime(2023, 3, 23, 11, 48, 53, 594133)
```

### 暗黙的な型変換を避ける

以下の例ではidはintが正しいのに、モデルをインスタンス化する際にfloatの型を渡しています。 pydanticでは出力結果の正当性を保証するので、暗黙的にintに変換されてしまいます。

```Python
class User(BaseModel):
    id: int

user = User(id=1.23)
print(user)
# id=1
```

これを防ぐために、いくつか厳密な型定義が提供されています。 [https://docs.pydantic.dev/usage/types/#strict-types](https://docs.pydantic.dev/usage/types/#strict-types)

以下は先ほどの意図しない変換を防ぐ例になります。

```Python
class User(BaseModel):
    id: StrictInt

try:
    user = User(id=1.23)
    print(user)
except ValidationError as e:
    print(e.json())
    #[{"loc": ["id"],"msg": "value is not a valid integer","type": "type_error.integer"}]
```

## フィールドの種類について

### リテラル値のみを受けるようなフィールドを定義する

決まった値しか入力されたくないフィールドってあると思います。 Booleanで解決できればいいですが、 もう少し種類が多い時、Literal型を使えば解決できます。

```Python
class User(BaseModel):
    kind: Literal[1, 2, 3]

try:
    User(kind=1)
    User(kind=3)
    User(kind=5)
except ValidationError as e:
    print(e.json())
    # [{"loc": ["kind"],"msg": "unexpected value; permitted: 1, 2, 3","type": "value_error.const","ctx": {"given": 5,"permitted": [1,2,3]}}]
```

カスタムバリデータを宣言する必要が無いので、便利です。

### よくある条件に一致するフィールドを宣言する

何文字以上、何文字以下にしたいだったり、 配列に格納する個数に制限を持たせたい時があると思います。

便利な型が用意されています。 [https://docs.pydantic.dev/usage/types/#constrained-types](https://docs.pydantic.dev/usage/types/#constrained-types)

以下は2文字以上、10文字以下しか受け付けないようなフィールドを宣言しています。

```Python
class User(BaseModel):
    id: constr(min_length=2, max_length=10)

try:
    User(id="1")
except ValidationError as e:
    print(e.json())
    # [{"loc": ["id"],"msg": "ensure this value has at least 2 characters","type": "value_error.any_str.min_length","ctx": {"limit_value": 2}}]
```

## バリデーター

[https://docs.pydantic.dev/usage/validators/](https://docs.pydantic.dev/usage/validators/)

### 独自のバリデーターを作成したい時

constrなどで基本の形のバリデーションはできますが、 全てに対応することは難しく、独自のバリデーターを作成したい時があると思います。 基本の形は以下になります。

```Python
class User(BaseModel):
    id: str

    @validator("id")
    def int_is_prefix_a(cls, v):
        if not v[0] == "a":
            raise ValidationError
        return v

try:
    User(id="1")
except ValidationError as e:
    print(e.json())
    # [{"loc": ["id"],"msg": "__init__() takes exactly 3 positional arguments (1 given)","type": "type_error"}]
```

### バリデーターを再利用したい時

名前に一定の入力規則があり、複数のクラスにそれが存在する時、 それぞれのクラスでバリデーターを書くのは大変だと思います。

そんな時はバリデーターを再利用することができます。

```Python
def id_is_prefix_a(id: str):
    if not id[0] == "a":
        raise ValidationError
    return id

class User(BaseModel):
    id: str

    _id_is_prefix_a = validator("id", allow_reuse=True)(id_is_prefix_a)

try:
    User(id="1")
except ValidationError as e:
    print(e.json())
    # [{"loc": ["id"],"msg": "__init__() takes exactly 3 positional arguments (1 given)","type": "type_error"}]
```

## モデルの構成(Config)

### バリデーションエラーメッセージを変えたい時

フォームなどでユーザーが入力した時にその値がバリデーションエラーにかかった場合、 適切なメッセージを出したいと思いますが、デフォルトでは英語で返却されます。 以下のような形で定義すると、自由に変えることができます。

```Python
class User(BaseModel):
    id: str

    class Config:
        max_anystr_length = 5
        error_msg_templates = {
            "value_error.any_str.max_length": "max_length:{limit_value}",
        }

try:
    User(id="123456")
except ValidationError as e:
    print(e.json())
    # [{"loc": ["id"],"msg": "max_length:10","type": "value_error.any_str.max_length","ctx": {"limit_value": 10}}]
```

## モデルのエクスポート

### 辞書形式でエクスポートしたい時

モデルを別で利用する時に、辞書形式の方が扱いやすいことがあると思います。 そんな時は、dict関数を使用することで叶えることができます。

```Python
class User(BaseModel):
    id: str
    name: str

try:
    hanako = User(id="123456", name="hanako")
    print(hanako.dict())
    # {'id': '123456', 'name': 'hanako'}
```

## その他

### イミュータブルにしたい

frozenをTrueにしてあげると、イミュータブルになる。 ただし、ミュータブルなフィールド(リストなど)がイミュータブルになるわけではないので注意

```Python
class User(BaseModel):
    id: int
    age: int
    name = "taro"

    class Config:
        frozen = True
```

## おわりに

途中参加した案件でpydanticを使っていたのですが、コピペで使っていたのを 改めて学習すると改善できるところも見つかったのでよかったです。
