---
title: "Docker環境構築 TypeScript × Prisma × MySQL"
published_at: "2022-09-14"
type: "tech"
published: true
coverImage: "Docker環境構築-TypeScript-×-Prisma-×-MySQL.jpg"
---

## はじめに

以前、javaScriptと[sequelize](https://github.com/sequelize/sequelize)を使った開発を行ったことがあるのですが sequelizeと肩を並べる（？）Prismaを使ってみたかったのでDockerで環境構築しました。

Prisma [https://www.prisma.io/](https://www.prisma.io/)

## ファイル構成

```
.
├── README.md
├── docker
│   ├── db
│   │   ├── Dockerfile
│   │   ├── data
│   │   ├── my.cnf
│   │   └── sql
│   └── node
│       └── Dockerfile
├── docker-compose.yml
└── server
    ├── package-lock.json
    ├── package.json
    ├── prisma
    │   └── schema.prisma
    ├── src
    │   └── script.ts
    └── tsconfig.json
```

[https://github.com/goda-kazuki/serverless-framework-Prisma](https://github.com/goda-kazuki/serverless-framework-Prisma)

## 解説

今回はMigrate機能は使用せず、既存のDBに対してORMとしてPrismaを採用したという体で環境構築しました。 全体のコードはGitHubに公開しているので興味があれば確認してください。

### 手順

1. Docker環境の用意(docker-compose.yml,DockerFile,sql)
2. nodeコンテナ内に入り、各種モジュールのインストール(package.json)
3. TypeScriptの設定ファイルの用意(tsconfig.json)
4. prismaの初期設定(npx prisma init --datasource-provider mysql)
5. server/.envのDATABASE\_URL変更(mysql://docker:docker@db:3306/docker?connect\_timeout=30)
6. 既存のDBからスキーマの生成(npx prisma db pull)
7. クライアントコードの自動生成(npx prisma generate,server/node\_modules/.prisma/client/index.d.ts)
8. プログラム実行(npx ts-node src/script.ts)

### ポイント

#### Dockerコンテナ同士の接続が不安定 P1001 Can't reach database server

Dockerコンテナで接続がうまくいかない問題がありました。 [https://github.com/prisma/prisma/issues/14971](https://github.com/prisma/prisma/issues/14971)

コンテナ間で接続する時は、コンテナ名を指定するみたいなことは知っていたのですが、 成功したり失敗したり不安定な感じでした。

私のPCの問題だと思いますので、タイムアウトの時間を伸ばすことで接続が安定するようになりました。

```
DATABASE_URL="mysql://docker:docker@db:3306/docker?connect_timeout=30"
```

#### クライアントコードがスゴイ

スキーマを自動定義した後、以下のコマンドでクライアントコードが生成されます。

```shell
npx prisma generate
```

ファイルはnode\_modules内に展開されます。(server/node\_modules/.prisma/client/index.d.ts)

自動でテーブル情報に合わせた型が生成されています。

```typescript
export type users = {
    id: number
    name: string
}
```

また、各関数に対して返り値の型などがこんな感じで定義されています。

```typescript
    findMany < T
extends
usersFindManyArgs > (
    args ? : SelectSubset<T, usersFindManyArgs>
)
:
CheckSelect < T, PrismaPromise < Array < users >>, PrismaPromise < Array < usersGetPayload < T >>> >

export type usersGetPayload<S extends boolean | null | undefined | usersArgs,
    U = keyof S> = S extends true
    ? users
    : S extends undefined
        ? never
        : S extends usersArgs | usersFindManyArgs
            ? 'include' extends U
                ? users & {
                [P in TrueKeys<S['include']>]:
                P extends 'cars' ? Array<carsGetPayload<Exclude<S['include'], undefined | null>[P]>> :
                    P extends '_count' ? UsersCountOutputTypeGetPayload<Exclude<S['include'], undefined | null>[P]> : never
            }
                : 'select' extends U
                    ? {
                        [P in TrueKeys<S['select']>]:
                        P extends 'cars' ? Array<carsGetPayload<Exclude<S['select'], undefined | null>[P]>> :
                            P extends '_count' ? UsersCountOutputTypeGetPayload<Exclude<S['select'], undefined | null>[P]> : P extends keyof users ? users[P] : never
                    }
                    : users
            : users
```

細かく見れば、読めなくは無いが難しい・・。 Prismaエンジンがこのような型定義を細かくしてくれるため、TypeScriptの恩恵を受けられます！

#### ちゃんと型情報が付いている

型情報がついているため、for文の中でconsole.log(user.hoge)とかってすると コンパイルエラーでちゃんと弾いてくれます。

```typescript
async function main() {
    // 型推論の結果は以下のようになっている
    // const users: (users & {cars: cars[]})[]
    const users = await prisma.users.findMany({
            include: {cars: true}
        }
    )

    // let user: users & {cars: cars[]}
    for (let user of users) {
        console.log(user);
    }
}
```

#### FatModelにならない

findMany等で取得した結果はモデルクラスではなく、オブジェクトになります。 なので、何かしたくてもオブジェクトに対して操作することになります。

そのため、モデルクラスは存在しないため、FatModelにすることができません。

## おわり

テーブル定義をちゃんとしてれば、自動でスキーマ、リレーションを定義してくれるのは良いですね。 sequelizeはモデル1つ1つに定義してて大変でした。(設計次第なのかもしれませんが)

TypeScriptとかなり相性が良さそうなので、どんどん使っていきたいです
