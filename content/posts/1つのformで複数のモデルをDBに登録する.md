---
Title: 1つのformで複数のモデルをDBに登録する
Date: 2019-10-11T00:26:34+09:00
---

今まで色々な教材で取り組んできましたけど、大半は一つの画面でフォームに入力されたパラメータを一つのモデルに登録する、というケースでした。

ポートフォリオを作成中に複数のモデルにパラメータを登録させたい場面が出たので、そのときにやったことを備忘録代わりに書き連ねていきます。

# 関連付けがされているモデルをDBに保存するためには何が必要か？

結論から言うと、`accepts_nested_attributes_for`というメソッドを使用しました。

このメソッドを使うと、関連付けられているモデルをネストさせて一纏めにしてまとめてDBに更新することができるみたいですね。

では実際にその実装を見ていきます。

***

まず、前提としてモデルは関連付けされていることですね。
今回は`article`という親モデルと`gear`という子モデルの１：Nの関係のモデルのケースで使用します。

## 1.親モデルにネストされた子モデルの更新させることを許可させる。

```ruby
class Article < ApplicationRecord
  has_many :gears

  accepts_nested_attributes_for :gears, allow_destroy: true
end
```

`allow_destroy: true`はarticleモデルを削除したときにgearモデルも一緒に削除することを許可するというオプションですね。

## 2.formにインスタンスを親モデルと子モデル両方を渡す

```ruby
class ArticlesController < ApplicationController
.
.

  def new
    @article = Article.new
    @article.gears.build
  end
.
.
```

これ実装する時はコピペでやってしまったんですけど、この記事書くときに改めて調べたら
モデルオブジェクトを生成する方法って`new`と`build`と２種類あるんですね。

最近のRailsのバージョンでは機能的に両者に違いはないらしいですけど、昔のバージョンでは関連付けられたモデルを生成する時に使われたメソッドらしいです。
その名残で、関連付けられたモデルを生成するときはbuildを使われる慣習があるみたいですね〜

## 3.formから受け取ったパラメータを保存する

```ruby
Parameters: 
    {"utf8"=>"✓",              
    "authenticity_token"=>"kuNbqST7Jb8DU3ggnC4jOTkNS68tdUMBn3tabSDkTxyirsRNK6ZpcbAaBpYApChoIhofrpfkMR9AFmLzW/zdLg==",
    "article"=>
        {"photo"=>#<ActionDispatch::Http::UploadedFile:*************
        @tempfile=#<Tempfile:/var/folders/**************,
        @original_filename="IMG_7968.JPG", 
        @content_type="image/jpeg", 
        @headers="Content-Disposition: form-data; name=\"article[photo]\"; filename=\"IMG_7968.JPG\"\r\nContent-Type: image/jpeg\r\n">, 
        "comment"=>"テストコメント", 
        "gears_attributes"=>{"0"=>
            {"gear_image"=>#<ActionDispatch::Http::UploadedFile:********,       
                @original_filename="20150205174601179_1000.jpg", 
                @content_type="image/jpeg", 
                @headers="Content-Disposition: form-data; name=\"article[gears_attributes][0][gear_image]\"; filename=\"20150205174601179_1000.jpg\"\r\nContent-Type: image/jpeg\r\n">, 
            "url"=>"http://localhost:3000/articles/new"}}}, "commit"=>"登録する"}
```

実際のパラメータです。`gears_attributes`が入っていますね！

あとはコントローラーで処理するだけです。

```ruby
class ArticlesController < ApplicationController
.
.

  def create
    @article = Article.new(article_params)
    @article.user_id = current_user.id
    if @article.save
      redirect_to articles_path, notice: "記事を登録しました。"
    else
      render :new
    end

  end
.
.
```

ログを確認してみると。。。

```ruby
  User Load (0.5ms)  SELECT  `users`.* FROM `users` WHERE `users`.`id` = 9 LIMIT 1
  ↳ app/controllers/articles_controller.rb:21
  Article Create (0.4ms)  INSERT INTO `articles` (`photo`, `comment`, `created_at`, `updated_at`, `user_id`) VALUES ('IMG_7968.JPG', 'テストコメント', '2019-10-10 15:06:55', '2019-10-10 15:06:55', 9)
  ↳ app/controllers/articles_controller.rb:21
  Gear Create (0.4ms)  INSERT INTO `gears` (`url`, `created_at`, `updated_at`, `article_id`, `gear_image`) VALUES ('http://localhost:3000/articles/new', '2019-10-10 15:06:55', '2019-10-10 15:06:55', 14, '20150205174601179_1000.jpg')
  ↳ app/controllers/articles_controller.rb:21
   (1.5ms)  COMMIT
```

無事登録できました〜

このメソッド便利なんですけど、どうやらActive Recordを汚染する原因になりやすいだとかなんとか？らしいのであまり使われないらしいですね（実装してから知った）
何で汚染の原因となるのかがよくわからないので、今後色々な機能を追加していく中で身を持って経験していきたいと思います！

参考にさせて頂いた記事：


[https://qiita.com/yuta-ushijima/items/22b823bfa3e2dd358fa0:embed:cite]



[https://www.ryotaku.com/entry/2019/03/28/new%E3%81%A8build%E3%81%AE%E9%81%95%E3%81%84%EF%BC%88Ruby_on_Rails%EF%BC%89:embed:cite]

