---
title: "【Go言語】新規プロジェクトの始め方について整理してみる"
date: 2020-10-03T16:01:46+09:00
draft: false
tags: ["golang"]
---

今まで適当な場所でコード書いていたけど、本来は`$GOPATH/src`配下に置くのが正しいらしい。

ただ、`Go Modules`を使えば上記の場所に置かなくてもいいとか？
そこらへんが曖昧なのでちょっと整理して、新規プロジェクトの始め方をまとめてみた。

<!--more-->

# そもそもGOPATHって何？
Go言語では`GOPATH`というパスにコードをすべて置くというルールが存在する。
一般的には1つのディレクトリに1つのプロジェクトが割り当てられる。
例えば、`$GOPATH/src/github.com/username/mycat`は`mycat`パッケージが実行可能アプリケーションとなる。
このパッケージに`main関数`が含まれているかどうかで実行可能アプリケーションかアプリケーションパッケージか判別される。

## GOPATHの構成
GOPATH配下には3つのディレクトリが存在する。
自分で書いたコードは`src`配下に置く。`src/ProjectName`みたいな感じ。
```bash
├── go # $GOPATH
│   ├── bin # バイナリファイルが保存される
│   ├── pkg # コンパイル後に生成されるファイル（*.a)
│   └── src # ソースコード(.go)
```

`go get`したパッケージもすべて`src`配下に置かれることになるから、プロジェクトごとに特定のパッケージのバージョンを分けたいというような場合に困る。

# Go Modules
そこでこのパッケージ依存関係を管理する仕組みとして`Go Modules`が`Go 1.11`から追加された。
他言語のパッケージ管理で例えるとRubyの`Bundle`、Node.jsの`npm`にあたる機能。

ダウンロード先は共通だけど、使うモジュールのバージョンは各プロジェクトの`go.mod`ファイルが特定してくれる。

つまり、パッケージ依存関係の情報を`go.mod`ファイルが保持してくれるから、`$GOPATH`の外で作業できるようになったということ！

# 結局、どうやってはじめればいいの？
完全新規のGoプロジェクトを作成するときは下記コマンドを実行すればいい
```fish
$ cd プロジェクトを作成したい場所
$ mkdir PROJECT_NAME
% cd PROJECT_NAME
$ go mod init github.com/username/repository名
# どんどんファイルを作成
$ touch main.go
$ go build # 依存モジュールを自動インストール
```

`go build`すると依存モジュールを`$GOPATH/pkg/mod`に自動インストールされて、`go.mod`に依存モジュールの情報が記載されていく。

## 参考記事
- https://blog.golang.org/using-go-modules
- https://blog.mmmcorp.co.jp/blog/2019/10/10/go-mod/
- https://qiita.com/yoshinori_hisakawa/items/268ba201611401ca7935