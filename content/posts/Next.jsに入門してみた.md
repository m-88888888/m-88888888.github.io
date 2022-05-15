---
title: "Next.jsに入門してみた"
date: 2020-11-04T00:54:53+09:00
draft: false
tags: ["next.js"]
---

最近入った現場でReact扱っているんですけど、フロントエンドを担当するのは初めてということもあって苦労してます。。。まあ楽しいのでいいんですが

そのReactでNext.jsっていうフレームワーク使ってるんだけどこれまたよくわからない所がちょくちょく出てきて何度か詰まったりした点があったので、一度体系的に調べてみようと思い、公式のチュートリアルやってみることにしました。

<!--more-->

# Next.jsの特徴
[公式によるとこんなこと書いてあった](https://nextjs.org/learn/basics/create-nextjs-app)
> Next.js has the best-in-class "Developer Experience" and many built-in features; a sample of them are:
> - An intuitive page-based routing system (with support for dynamic routes)
>- Pre-rendering, both static generation (SSG) and server-side rendering (SSR) are supported on a per-page basis
>- Automatic code splitting for faster page loads
>- Client-side routing with optimized prefetching
>- Built-in CSS and Sass support, and support for any CSS-in-JS library
>- Development environment with Fast Refresh support
>- API routes to build API endpoints with Serverless Functions
>- Fully extendable

サーバーサイドレンダリングの機能がデフォルトであったり、ページごとのルーティングが作られたり、Sassサポートしてたりいろいろと多機能なフレームワークみたいですね。

# そもそもサーバーサイドレンダリング（SSR)って何よ
フロントエンド開発でよく耳にする単語でSSRってあるけど、その内容については全然知らなかったのでついでに自分なりに調べてみました。
SSRだけでなくCSRってのもあるらしい。

## Server-Side Rendering
- サーバーサイドでHTMLレンダリングすること
- サーバ側でデータを整形したHTMLを返すため、初期ページの画面レンダリング時間が短縮される
- 整形済みのHTMLをブラウザが読み込むからSEO対策になるらしい。SEO対策ってどんなものかあまりわかっていない。。。

## Client-Side Rendering
- ブラウザ側でHTMLレンダリングすること。
- サーバとはデータ（JSON）のみを通信して、そのデータに基づいてページの更新を行う
- 画面全体ではなく、部分的なレンダリングができるため高度なUIを構築できる
- SPAの実装に向いている

## 参考
- [Rendering on the Web](https://developers.google.com/web/updates/2019/02/rendering-on-the-web?hl=ja)

# [Next.js公式チュートリアル](https://nextjs.org/learn/basics/create-nextjs-app)
内容はかなりシンプルなので個人的に重要だと思った部分のみを整理してみる。
※2020年11月時点の内容のため、今後内容が変化する可能性アリ
## Navigate Between Pages
- Next.jsにおいて、`pages`ディレクトリ配下に置いたJavaScript,Typescriptファイルがブラウザ上に表示されるページの本体となる。
- ページコンポーネントは`default`としてexportされたReactコンポーネントである必要がある。
- `pages`配下の階層がルーティングとなるから、専用のルーティングライブラリは不要。
- pagesの階層とファイル名が実際のURLパスになる。
  - `pages/posts/first-post.js`というファイルの場合、`posts/first-post.js`でアクセスできる。
- クライアントサイド内のページ遷移には`next/link`という`<a>タグ`をラップしたコンポーネントを使うこと。
  - next/linkの`Link`は`client-side navigation`を可能とするから、フロントエンド内のページ遷移には極力この`Link`を使うほうがいい。
  - `client-side navigation`とは、ブラウザの機能ではなくJavaScriptを使ったページ遷移のこと。ブラウザによる遷移よりも高速でページ遷移が可能となる。
  - 公式チュートリアルにあるようにDeveloper toolで変えたCSSスタイルがページ遷移後にも適用されているのは、ブラウザがページ全体を完全にリロードしていないから。
- コード分割や本番環境限定のプリフェッチ機能によって、アプリケーションを最適化してパフォーマンスを向上されてくれる。

## Assets, Metadata, and CSS
CSSは省略

## Pre-rendering and Data Fetching
- Next.jsはデフォルトでSSRに対応していて、`プリレンダリング`されることで各ページのHTMLは最小限のJavaScriptコードを残して紐付いた状態で生成される。
  - ブラウザが実際にページを読み込むときに、残されたJavaScriptのコードを実行される。この工程のことを`ハイドレーション`と呼ぶ。
- JavaScriptをオフにした状態でNext.jsによって作られたサイトにアクセスすると、アプリのUIが正しく構築された状態で表示されていることが確認できる。これは、Next.jsによって`プリレンダリング`されたことで静的なHTMLが生成されているから。
- `プリレンダリング`には次の2つの形式がある。
  - static generation：ビルド時にレンダリングされる
  - SSR：リクエスト毎にレンダリングされる
- つまり、レンダリングされるタイミングが異なるというだけ。なのでページごとにプリレンダリングの形式を選択することができる。
- static genarationに外部データを取り込みたい（fetch）とき、`getStaticProps`という関数を呼ぶことで外部データを取り込んで遷移先にpropsとして送ることができる。
  - getStaticPropsはサーバサイドで実行されるためデータベースにクエリを投げることも可能
  - ビルド時に実行される
- リクエストごとにfetchしたい時には`getServerSideProps`を使う。

## Dynamic Routes
- static generate pagesにも`動的なルーティング`を設定することができる。
- `/posts/<id>`というURLで`<id>`を動的にしたいという場合、`pages/posts/[id].js`が対応するページとなる。`[]`で囲った部分が動的な値となる。
  - 今回の場合だとページコンポーネントでは`getStaticPaths`で`id`の値（ファイル名）を返し、`getStaticProps`で必要なデータ（ファイル内のデータ）をフェッチする必要がある。

## API Routes
- アプリ内にAPIのエンドポイントを作成することもできる。
- APIは関数として定義することができて、サーバレス関数としてデプロイすることができるからAWS Lambdaに似た動きを実現できる。
- getStaticPathsとgetStaticPropsからAPI Routesをフェッチしてはいけない。
  - getStaticPathsとgetStaticPropsかクライアントサイドではなく、サーバサイドでのみ実行可能だから。
  - 代わりに、サーバサイドに直接getStaticPathsとgetStaticPropsを書く、あるいはヘルパー関数を呼び出す必要がある。

---

# getInitialProps
`Next.js 9.3`以降では`getStaticPathsとgetStaticProps`を使うことを推奨されている。
既存のコードも書き直すべき、というほどではないが、今後新しく書き始める場合に特別な理由がなければ使う必要はないというニュアンス？

`getInitialProps`はレンダリングされる前に実行されるAPI。リクエストが送られるとページに必要なデータを`props`として渡してくれる。
CSRとSSRの両方に対応している特徴がある。
- 通常のアクセス：サーバー側で実行（SSR)
- `next/link`でルーティング：クライアント側で実行（CSR)

# getStaticProps
getInitialPropsが行っていた処理（ページに必要なデータをpropsに渡す処理）をビルド時に行って静的なファイルを生成するためのAPI。
クライアント側で実行されることはなく、サーバー側でのみ実行される。→いつアクセスしても同じ結果が表示される
必須パラメーターとして`paths`と`fallback`があり、`paths`はビルドするパス対象、`fallback`は事前ビルドしたパス以外にアクセスした時の動作を決めるもの。
- falseの場合
  - 404 pageとなる
- trueの場合
  - 静的なファイルをサーバー側で生成して返す処理を書く必要がある

# getStaticPaths
`Dynamic Routes`使用時にも静的なファイルを生成するためのAPI。

# getServerSideProps
SSRのためのAPI。`next/link`でルーティングを行った場合でも、サーバーサイドで実行される。

---

なんか内容にまとまりがなくなってきた...また後日修正しよう。

# 参考
- https://nextjs.org/docs/getting-started
- https://qiita.com/amakawa_/items/e7d0720e1ab8632769bf#spasingle-page-application
- https://qiita.com/matamatanot/items/1735984f40540b8bdf91%E3%80%80#%E5%89%8D%E6%8F%90%E7%9F%A5%E8%AD%98-ssg-%E3%81%A8-getinitialprops-%E3%81%A8-next-export