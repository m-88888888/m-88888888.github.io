---
title: "今更ながらDockerについて学習した"
date: 2020-10-25T17:37:57+09:00
draft: true
tags: ["docker"]
---

いよいよ今の案件でDocker触ることになりそうなのでいい加減Dockerについて簡単なキャッチアップをしたのでまとめてみた。

<!--more-->

- そもそもコンテナとは？
- コンテナとは、プロセスの実行空間を隔離するための技術
  - 仮想化
    - ホストOS→仮想化ソフトウェア→ゲストOS→プロセス
  - コンテナ
    - ホストOS→プロセス（コンテナ）
- 従来の仮想化と比較して、余計なものがなくなった
- namespace
  - 同じホストOS上で複数のアプリケーションを同時に動かすための技術
  - namespaceで区切った空間＝コンテナ
- プロセスをコンテナ上で稼働させるためにはOS動作に必要なコマンド、ライブラリなどが必要
- このような、namespaceの作成、コマンド・ライブラリの取得、ネットワーク空間の分離、ファイルシステム空間の分離etx...
- これらの複雑なコマンドをラップして一つのコマンドで簡単にコンテナを作れるようにしたものがDocker

# Dockerってなんなの？
- `コンテナ`を簡単に作るための技術。
  - コンテナって何？
    - プロセスの実行空間を隔離するための技術
      - プロセスって何？隔離してどんなメリットがあるの？
        - プロセス＝アプリケーションのこと。複数のアプリケーションを`同時に`実行させることができる。

このコンテナを作るためにはファイルやネットワーク空間などを`namespace`と呼ばれる空間で区切ったり
プロセスの実行に必要なコマンドやライブラリ郡を取得する必要がある。

これらの過程を一つのコマンドにまとめて実行できるようにしたものが`Docker`

# コマンド解説
```docker
# Dockerイメージからコンテナを起動する
docker run
# 現在のコンテナの状態を見る(「-a」で停止中のコンテナも表示)
docker ps -a
# イメージ一覧
docker images
# コンテナを削除
docker rm <コンテナ名orコンテナID>
# イメージを削除
docker rmi <イメージ名orイメージID>
# ホスト側のファイルをコンテナ内にコピー
docker cp <ホスト側のファイル> <コンテナ名>:<コンテナ内のコピー先ディレクトリ>
# コンテナ起動、停止
docker start <コンテナ名>
docker stop <コンテナ名>
```

# Dockerfile
- Dockerfileとは、Dockerイメージをコードで記述したファイルのこと

サンプル：TomcatのDockerイメージを作成するためにDockerfileを作成
```docker
FROM centos:7
RUN yum install -y java
ADD files/apache-tomcat-9.0.39.tar.gz /opt/
CMD [ "/opt/apache-tomcat-9.0.39/bin/catalina.sh", "run" ]
```

Dockerfileの各コマンドは毎回コンテナを起動して実行している
だから一つの流れに沿ってコマンドを実行させたい場合は&&などを使って連結させる必要がある

# コンテナ間の通信
Dockerネットワークを構築して、コンテナ名で接続

# コンテナイメージ
`alpine linux`というLinuxディストリビューションがあって、これをベースイメージとすることでDockerイメージのサイズを小さくできる。

# Docker compose

## コマンド
```docker
# Dockerイメージの作成
docker-compose build
# コンテナの起動
docker-compose up -d
# サービスのコマンド実行
docker-compose exec api go run main.go
```

# 参考
https://qiita.com/wasanx25/items/d47caf37b79e855af95f#version
https://qiita.com/zembutsu/items/9e9d80e05e36e882caaa#build
https://www.w2solution.co.jp/tech/2020/03/25/dockercompose%E3%81%A7go%E3%81%AEgin%E3%81%A8mysql%E3%81%AE%E7%92%B0%E5%A2%83%E3%82%92%E4%BD%9C%E3%81%A3%E3%81%A6%E3%81%BF%E3%81%9F-%E7%AC%AC1%E5%BC%BE/
https://docs.docker.jp/compose/toc.html
https://docs.docker.jp/engine/toc.html
http://www.tohoho-web.com/docker/compose.html
https://hub.docker.com/_/golang
https://astaxie.gitbooks.io/build-web-application-with-golang/content/ja/
https://qiita.com/tenntenn/items/0e33a4959250d1a55045
