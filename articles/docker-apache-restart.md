---
title: "DockerコンテナでApacheを再起動したい時"
published_at: "2022-12-05"
type: "tech"
published: true
coverImage: "DockerコンテナでApacheを再起動したい時.jpg"
---

## はじめに

DockerコンテナでApacheを使っていて、設定ファイルを書き換えたので再起動しようとすると、 以下のような権限エラーが表示されました。

```shell
$ systemctl restart httpd
Failed to get D-Bus connection: Operation not permitted
```

これを回避する手段として、 privilegedを有効にし、/sbin/initを起動する方法があるようです。 [https://www.suzu6.net/posts/324-docker-compose-centos-privileged/](https://www.suzu6.net/posts/324-docker-compose-centos-privileged/)

ただ、既に運用されている環境に対して、理解できていない変更を加えるのは怖いため、代替手段が無いか検討していました。

## 解決方法

まずは対象のコンテナに接続します。

```shell
例
docker-compose exec server /bin/bash
```

### 設定ファイルが正しいか確認

```shell
$ httpd -t
```

### apacheを起動する

```shell
$ httpd -k start
```

### apacheを停止する

```shell
$ httpd -k stop
```

### apacheを再起動する

```shell
$ httpd -k restart
```
