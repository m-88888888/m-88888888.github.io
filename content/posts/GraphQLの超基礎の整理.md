---
title: "GraphQL整理"
date: 2021-02-07T00:00:10+09:00
draft: false
tags: ["graphql"]
---
マイクロサービス案件に参画して4ヶ月が経過しで身につけたことを整理したくなってきたので
キャッチアップしやすかったGraphQLから整理していこうと思う。

<!--more-->

# GraphQLって何？

公式によると
> GraphQL is a query language for APIs and a runtime for fulfilling those queries with your existing data. GraphQL provides a complete and understandable description of the data in your API, gives clients the power to ask for exactly what they need and nothing more, makes it easier to evolve APIs over time, and enables powerful developer tools.

Google翻訳すると
>GraphQLは、APIのクエリ言語であり、既存のデータでこれらのクエリを実行するためのランタイムです。 GraphQLは、API内のデータの完全で理解しやすい説明を提供し、クライアントが必要なものだけを正確に要求できるようにし、時間の経過とともにAPIを進化させやすくし、強力な開発者ツールを有効にします。

自分なりにまとめると
- クエリ言語
  - SQLはデータベースに対するクエリ言語
  - GraphQLはAPIに対するクエリ言語
- クライアントアプリケーションが、自分にとって必要な情報だけを要求することができる
- エンドポイントが1つだけ

# GraphQLのクエリの定義

GraphQLはエンティティを次のような型で定義できる。

```graphql
type User {
    id Int!
    name String!
}
```

この定義された型を`Query型`を使って参照することができる。

```graphql
type Query {
    users: [User!]!
    user(id: Int!): User!
}
```

これで参照するポイントは定義されたので、次は実際にDBからデータを取得する処理がまだ実装する。
GraphQLが提供する`resolvers`と呼ばれる関数を定義することで実現することができる。
クライアントがリクエストしたプロパティだけを返す処理についてはGraphQLがよしなにしてくれる。すごい。

```javascript
// 実際はもっと色々な情報が必要。
function users() {
    return usersArray;
}

function user({id}) {
    return usersArray.find((user) => user.id == id)
}

```

こうすることによって、次のようなQueryを実行できるようになる。

```graphql
{
    # userのコレクションを取得
    users {
        id
        name
    }

    # id=001のユーザーのnameを取得
    user(id: 001) {
        name
    }
}
```

その他の詳細なQueryやMutation、他の機能については追々まとめるつもり...
