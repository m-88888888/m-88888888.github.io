---
title: "Next.js9.3で追加されたAPIについて"
date: 2021-02-27T23:11:26+09:00
draft: false
tags: ["next.js"]
---

現在の案件ではNext.js9.6を使用しているのですが、ページのAPIに`getInitialProps`を使用しており
これを新規追加されたAPIに置き換えた方が良いのではないかという話が出ていたので
ページのAPIに関して少し整理してみました。

# getInitialPropsとは
ページがレンダリングされる前に実行されるAPI。
該当ページへのパスにリクエストが送られると実行されて、ページに必要なデータをpropsとして渡す。

![とりあえず整理する用 P1](https://user-images.githubusercontent.com/51853475/109381342-3da26f00-791d-11eb-9d23-7b8be32a965b.png)

このページが「外部のデータをビルド時に取得する静的なページ」の場合、`next export`というコマンドを使うことで、静的なページを生成することができた。
しかし、このページに`next/link`を使用してアクセスした場合、静的なページではなく`getInitialProps`が実行されてしまうという問題があった。

# getStaticProps
この問題を解決するために、`getStaticProps`が導入された。
このAPIを使用したページは、必ず静的ファイルを使用することになる。
つまり、再ビルドしない限り、必ず同じ結果のページが表示されるということ。

# getStaticPaths
Dynamic Routes使用時にも静的ファイルを使用するためにAPI。`getStaticProps`とセットで使用される。

# getServerSideProps
`next/link`でルーティングしても必ずサーバ側でgetInitialPropsと同じ処理が実行されるというAPI。

# まとめ
今後作成するページは
- getStaticProps
- getServerSideProps

いずれかのAPIを使用すること
使い分けとしては、静的なファイルかどうか
つまりSSRではなく、SSGでも実現できるかどうかを考慮する必要がある

# 参考
- https://nextjs.org/docs/api-reference/data-fetching/getInitialProps
