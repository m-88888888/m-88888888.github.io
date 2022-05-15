---
Title: HerokuのDBをMySQLに設定する
Date: 2020-01-15T23:12:59+09:00
---

# はじめに

Herokuに設定するデータベース情報について、勘違いしていて詰まったので備忘録として残します。

以下の項目について設定する必要があるのですが
```
heroku config:add DB_NAME='<データベース名>'
heroku config:add DB_USERNAME='<ユーザー名>'
heroku config:add DB_PASSWORD='<パスワード>'
heroku config:add DB_HOSTNAME='<ホスト名>'
heroku config:add DB_PORT='3306'
heroku config:add DATABASE_URL='mysql2://<ユーザー名>:<パスワード>@<ホスト名>/<データベース名>?reconnect=true'
```

本来は、Herokuにデプロイした際に自動的に作成されるCLEARDB_DATABASE_URLの情報を設定しなければならないのですが
どの値を設定すれば良いのかわからず、自分で適当に決めた値を設定してしまっていました。

# 正しい設定方法

CLEARDBアドオン追加までの流れは大体同じなので省略します。

1.`heroku config`で`CLEARDB_DATABASE_URL`を確認します。
```
$ heroku config
CLEARDB_DATABASE_URL:mysql://<ユーザー名>:<パスワード>@<ホスト名>/<データベース名>?reconnect=true
```

2.このURLからデータベース名などをそれぞれ設定していきます。
```
$ heroku config:add DB_NAME='<データベース名>'
$ heroku config:add DB_USERNAME='<ユーザー名>'
$ heroku config:add DB_PASSWORD='<パスワード>'
$ heroku config:add DB_HOSTNAME='<ホスト名>'
$ heroku config:add DB_PORT='3306'
$ heroku config:add DATABASE_URL='mysql2://<ユーザー名>:<パスワード>@<ホスト名>/<データベース名>?reconnect=true'
```

3.database.ymlへの設定も忘れずに！

```ruby
production:
  <<: *default
  adapter: mysql2
  encoding: utf8
  database: <%= ENV['DB_NAME'] %>
  username: <%= ENV['DB_USERNAME'] %>
  password: <%= ENV['DB_PASSWORD'] %>
  host: <%= ENV['DB_HOST'] %>
```

4.最終的に`CLEARDB_DATABASE_URL`と`DATABASE_URL`が同じ値になる。

※注意
`DATABASE_URL`は`mysql2`であること。GemのDBのバージョンと同じにする必要があるため。

```
$ heroku config
CLEARDB_DATABASE_URL:     mysql://<ユーザー名>:<パスワード>@<ホスト名>/<データベース名>?reconnect=true
DATABASE_URL:             mysql2://<ユーザー名>:<パスワード>@<ホスト名>/<データベース名>?reconnect=true
.
.
.
```

あとはpushして`heroku run rails db:migrate`すれば成功するはず！

今回はここまで。
