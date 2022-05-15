---
title: "AWS_CLIをインストールしてみる"
date: 2020-09-26T22:01:43+09:00
draft: false
tags: ["aws"]
---

AWS Lambdaを使ってLINE Botを作ろうとしたところ、LambdaにコードをデプロイするためにAWS CLIがあると便利みたいなのでインストール〜設定作業をまとめてみた。

<!--more-->

## そもそもAWS CLIって何？

公式より
[AWS Command Line Interface とは - AWS Command Line Interface](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-chap-welcome.html)
>AWS Command Line Interface (AWS CLI) は、コマンドラインシェルでコマンドを使用して AWS サービスとやり取りするためのオープンソースツールです。AWS CLI は、最小限の設定で、任意のターミナルプログラムのコマンドプロンプトから、ブラウザベースの AWS マネジメントコンソール で提供される機能と同等の機能を実装するコマンドの実行を開始できます。

AWSをコマンドラインで設定、操作するためのツールみたい。
ソースをLambdaにデプロイするときに毎回圧縮したソースをマネジメントコンソールでアップロードするという作業が要らなくなる！

## インストール
とりあえず最新バージョンをインストール
```bash
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```

バージョン確認できればインストール成功
```bash
$ which aws
/usr/local/bin/aws
$ aws --version
aws-cli/2.0.52 Python/3.7.4 Darwin/18.7.0 exe/x86_64
```

## 認証設定
AWSのアカウント情報とかの設定をする。
`aws configure`で基本設定ができるっぽい。

```bash
$ aws configure
AWS Access Key ID [None]: アクセスキー
AWS Secret Access Key [None]: シークレットアクセスキー
Default region name [None]: リージョン
Default output format [None]: 出力形式
```

この認証設定を切り替えたりするために`プロファイル`ってものがあるらしい。
複数アカウント使用するときに使いそう。今回は割愛。

## CLIでLambdaにコードをデプロイしてみる
Lambda関数自体は既に作成済みなので関数を更新するコマンドを叩けばいい。
コード自体は圧縮してアップロードするだけなので単純。
```bash
$ zip -r lambda zip main.go
$ aws lambda update-function-code --function-name hogehoge --zip-file fileb://lambda.zip
{
  .
  .
  .
  "LastUpdateStatus": "Successful"
}
```
コマンドの結果がJSON形式で返って、更新結果が`Successful`となっているのでデプロイに成功。

## まとめ
- いちいちマネジメントコンソール開く手間がなくなるのはかなり大きい。
- CLIを使用する上での最低限の設定はかなり単純なので使用する上での敷居は低い
- けど初見のサービスをCLIで操作するのは大変そう
- ある程度操作になれたらCLIでも操作してみる、って手順を踏んだほうが良さそう