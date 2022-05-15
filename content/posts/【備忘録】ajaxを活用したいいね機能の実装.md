---
Title: 【備忘録】ajaxを活用したいいね機能の実装
Date: 2019-10-26T15:44:37+09:00
Draft: true
---

作成中のポートフォリオにいいね機能を非同期処理で実装しました。

## いいね機能

Ajax・・・Webブラウザ上で動作するJavaScriptでサーバからXMLデータを取得し、取得したデータをコンテンツに動的に反映するという手法

ユーザーモデルと記事モデルの中間テーブルを用意して、この中間テーブルを使って制御する。

中間テーブルの作成

```ruby
class CreateLikesModel < ActiveRecord::Migration[5.2]
  def change
    create_table :likes_models do |t|
      t.integer     :user_id
      t.integer     :article_id
      t.timestamps
    end
  end
end
```

Likeモデルの生成
`counter_cashe`は関連付けされているテーブルのレコード数を集計するためのRailsの機能
ここでは`likes_count`という変数にいいねの数を代入させるという意味。

```ruby
class Like < ApplicationRecord
  belongs_to :user
  belongs_to :article, counter_cashe: :likes_count
end
```

モデルの関連付け

```ruby
# like.rb
class Like < ApplicationRecord
  belongs_to :user
  belongs_to :article, counter_cashe: :likes_count
end
```

```ruby
# user.rb
class User < ApplicationRecord
.
.
  has_many :likes, dependent: :destroy
.
.
end
```

```ruby
# article.rb
class Article < ApplicationRecord
.
.
  has_many :likes, dependent: :destroy
.
.
end
```

ルーティングの設定

いいねする対象のarticleモデルのIDをパラメータとして送信する

```ruby
# routes.rb
Rails.application.routes.draw do
.
.
  post   '/like/:article_id' => 'likes#like',   as: 'like'
  delete '/like/:article_id' => 'likes#unlike', as: 'unlike'
end
```

コントローラーの生成

```ruby
#like_controller.rb
class LikesController < ApplicationController
  before_action :set_valiables

  def like
    @like = current_user.likes.new(article_id: params[:article_id])
    @like.save
  end

  def unlike
    @like = current_user.likes.find_by(article_id: params[:article_id])
    @like.destroy
  end

  private

    def set_valiables
      @article = Article.find(params[:article_id])
      @id_name = "#like-link-#{@article.id}"
    end
end
```

ビューの生成

`remote: true`
リクエストがHTML形式ではなくJS形式となる。
