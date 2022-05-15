---
Title: 【Rails】URLヘルパーメソッドを使わないURLの生成方法について
Date: 2020-04-18T10:57:59+09:00
---

# URLを得る方法

RailsでルーティングのURLを得る方法はURLヘルパーメソッドを使うしかないと思っていたんだけど
他にも方法があったみたいなので、サンプルとして下記のルーティングを使って、その方法を試してみた。

```ruby
# routes.rb
  namespace :admin do
    resources :users
  end

```

```ruby
# rails routes
Prefix Verb   URI Pattern                                                                              Controller#Action
admin_user GET    /admin/users(.:format)                                                                   admin/users#show
```

## 1.URLヘルパーメソッド
rails routesで`Prefix`に表示されるパスのこと。

`admin_user_path(user)`で呼び出せる。

```slim
h1 ユーザー一覧
.
.
.
    tbody
      - @users.each do |user|
        tr
          td= link_to user.name, admin_user_path(user)
.
.
.
```

## 2.ハッシュを利用する
`{ controller: :users, action: :show, id: user.naem }`のように呼び出すアクションをハッシュを使って直接指定する。

## 3.文字列を直接記述する
`"/admin/users/#{user.id}"`のように文字列を直接渡す。
しかし、このやり方はURL変更時に修正箇所が増えるため避けるべき。

## 使い分け
基本的に3.の書き方が使われることはまずないので、1.と2.のどちらかを使うことになる。

1.は一番ベーシックな書き方なので、特別な事情がない限りはこの方法を採る。
2.を使う場面があまりよくわからない。`method: :delete`とかはよく使うんだけど…
<s>分かり次第また更新します。</s>

同じ書き方で別々のコントローラーやアクションを呼び出したいという時に使えるみたい。

```ruby
<% %i[users posts].each do |controller| %>
  <% %i[index new].each do |action| %>
    <%= link_to "#{controller}##{action}", controller: controller, action: action %>
  <% end %>
<% end %>
```

管理者画面とかのViewで使えるみたいなので、こういう書き方があることを覚えておいて損はないのかな？
