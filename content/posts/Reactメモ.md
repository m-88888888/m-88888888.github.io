---
title: "React+Typescriptいろいろメモ"
date: 2020-11-08T23:01:22+09:00
draft: false
tags: ["React", "Typescript"]
---
React+Typescriptのプロジェクトの始め方、React hooksなど自分なりに整理してみる。

# React + Typescriptの始め方

- Reactアプリの作成には`create-react-app`を使う。（ゼロから構築するスキルがまだない)
- パッケージ管理には`npm`ではなく`yarn`を使うほうがベターっぽい
  - パッケージのインストール速度やコマンドの簡潔さで`yarn`が優れている
  - `yarn.lock`ファイルのおかげで依存パッケージをケアしてくれる

```bash
yarn create react-app <アプリ名> --template typescript
```

まだまだ本格的に設定からカスタマイズできるレベルに達していないので`create-react-app`を使っているが、`lint`の独自ルールがデフォルトで組み込まれているので、今後いろいろとカスタマイズする必要性が出てきたら`create-react-app`に頼らずにゼロからreact-appを構築できるようになりたい。

# 参考
- https://qiita.com/Hai-dozo/items/90b852ac29b79a7ea02b
- https://qiita.com/jonakp/items/7d9f47c613c16cbf95aa

# コンポーネントの定義

## 関数コンポーネント

`React.FC<Props>`で定義する。FC=FunctionComponent

```typescript
type AppProps = {
  hoge: String;
};

const App: React.FC<AppProps> = (props) => {
  return <h1>Hello, {props.hoge}</h1>;
};

export default App;
```

## クラスコンポーネント
```typescript
type AppProps = {
  hoge: String;
};

class App extends React.Component<AppProps, {}> {
  render() {
    return <h1>Hello, {this.props.hoge}</h1>;
  }
}

export default App;
```

## 両者の違い
`hooks`導入前までは、状態管理やライフサイクルイベントの機能を使うことができるのはクラスコンポーネントのみだった。
しかし、`hooks`の登場で関数コンポーネントでもそれらの機能を使うことができるようになったため、クラスコンポーネントよりも簡潔に書くことができるようになった。

# propsとstate
Reactのコンポーネント間でデータをやり取りするための仕組みには2種類ある。

## props
親コンポーネントから子コンポーネントに値を渡すための仕組み。
propsは読み取り専用で、値を更新してはいけない。また、子から親のコンポーネントに渡すことはできない。
更新したかったり、子から親へ値を渡したいときはもう一つの仕組みのstateを利用すること。

propsとして渡せるものに制限はなくて、何でも渡せる。
コンポーネントの再利用性を上げてくれるすごいやつ。
## state
コンポーネントの状態(state)を管理するための仕組み。
値とその値を更新するための関数を持つ。

データは下方向に伝わる。
propsが下方向へ流れる滝だとすると、stateは任意の場所で合流する水源。
