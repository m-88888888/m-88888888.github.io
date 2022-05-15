---
title: "ディレクトリをブックマークするCLI作ってみた"
date: 2021-02-18T00:13:57+09:00
draft: false
tags: ["golang", "cli", "個人開発"]
---

自分が日常的に一番多く叩いているであろう`cd`を叩く回数を減らすために、よく使うディレクトリをブックマークできるCLIとかあると便利じゃないかなと思って作ってみました。

# 完成形

リポジトリ
https://github.com/m-88888888/bk

CLIライブラリにはメジャーで情報量の多い`cobra`を使用しました。
https://github.com/spf13/cobra
kubectlとかこのブログを作るのに使用しているhugoもこのcobraを使っていたことも採用の理由です。

![bk-show](https://user-images.githubusercontent.com/51853475/108219876-7d2cc680-7179-11eb-86e1-4b425b6feff2.gif)


# 機能
- ブックマークの一覧・登録・削除
- ブックマークしたディレクトリに`cd`

## `bk show`・・・一覧
![スクリーンショット 2021-02-17 23 47 01](https://user-images.githubusercontent.com/51853475/108220803-794d7400-717a-11eb-8535-b041172e9027.png)

## `bk save`・・・登録
![スクリーンショット 2021-02-17 23 48 17](https://user-images.githubusercontent.com/51853475/108220972-a26e0480-717a-11eb-876f-05ef1d17b695.png)

## `bk delete`・・・削除
![bk-delete](https://user-images.githubusercontent.com/51853475/108221281-f5e05280-717a-11eb-919f-1804d21d8304.gif)

## `jp`・・・特定ディレクトリにcd
golangで`cd`コマンドを叩く方法がわからず、このコマンドだけシェルスクリプト...

![jp](https://user-images.githubusercontent.com/51853475/108221661-625b5180-717b-11eb-9d4a-e81d5ea583a7.gif)

# 感想
今回始めてCLIを作ってみましたが、思った以上に簡単に作れて楽しかったです。
課題としては

- `jp`がただのシェルスクリプト
- Makefileの整備ができておらず自動インストールができていない
- テストコードが書けていない

などがありますが、ひとまずやりたかったことはできたので良かったです。

今後はkubectlやhugoのコードを読んだり、CLIを作る上でのお作法をインプットしたりして
`コマンドをブックマークするCLI`や`git周りを便利にするCLI`とかを作ってみたいと思います。
