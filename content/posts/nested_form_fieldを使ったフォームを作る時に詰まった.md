---
Title: nested form fieldを使ったフォームを作る時に詰まった
Date: 2019-12-01T17:02:02+09:00
---

前回の投稿から2ヶ月も空きましたが、再度アウトプットしていきます。



[https://github.com/ncri/nested_form_fields:embed:cite]


nested field formとは親子関係にあるモデルの子モデルのフォームを動的に追加・削除することができるgemです。  
githubのスター数から見てあまりメジャーなgemとは言えないのかなと。

今回はこのgemで追加したフォームにcarrierwaveを用いた画像アップローダーを複製し、その画像プレビューを表示させる時に詰まった点を紹介したいと思います。

# 詰まった点その１
READMEを見るとわかる通り、導入は至ってシンプルです。  

1対多にある関係の親モデルに`accepts_nested_attributes_for :子モデル`という宣言を加えます。  

`allow_destroy: true`とはフォームを削除した時にモデルも共に削除させることを許可するための宣言です。  
これが抜けているとPOSTした時にフォームは削除していても複製されたモデルが削除されずにPOSTされてしまい、エラーの原因となります。

```ruby
class Article < ApplicationRecord
  mount_uploader :photo, PhotoUploader
  has_many :gears
  accepts_nested_attributes_for :gears, allow_destroy: true
.
.
```

```ruby
class Gear < ApplicationRecord
  mount_uploader :gear_image, GearImageUploader
  belongs_to :article, optional: true
.
.
```

この時に、必ずStrong Parameterに`:_destroy`というパラメータを追加させることです。

私はこれを追加しそびれたせいでフォームを追加しても削除することができず、詰まりました。
公式にはちゃんと書いてあるのに見逃してましたね。公式ちゃんと読みましょう

```ruby
.
.
  private

    def article_params
      params.require(:article).permit(
                                      :photo,
                                      :comment,
                                        gears_attributes:[
                                                          :id,
                                                          :gear_image,
                                                          :name,
                                                          :brand,
                                                          :kind,
                                                          :model_year,
                                                          :_destroy]
                                      )
        end

.
.
```

# 詰まった点その２

次にViewです。  
追加するformについてはREADMEを参照してください。

```ruby
.
.
= f.nested_fields_for :gears, wrapper_tag: :div do |g|
  .row
    .col-6
      .form-group
        = g.label :gear_image
        - if g.object.gear_image?
          = image_tag(g.object.gear_image.url, id: 'preview__nested_field_for_replace_with_index__', class: 'gear_image')
        - else
          = image_tag('nofile.jpg', id: 'preview__nested_field_for_replace_with_index__', class: 'gear_image')
        = g.file_field :gear_image, class: "form-contro-file", onChange: "imgPreView(event, 'pre__nested_field_for_replace_with_index__')"
    .col-6
      .form-group
        = g.label :name
        = g.text_field :name, class: "form-control"
      .form-group
        = g.label :brand
        = g.text_field :brand, class: "form-control"
      .form-group
        = g.label :kind
        = g.select :kind, gear_type_list, { include_blank: true }, class: "form-control"
      .form-group
        = g.label :model_year
        = g.text_field :model_year, class: "form-control"
      = g.remove_nested_fields_link '削除', class: 'btn btn-danger', role: 'button'
= f.add_nested_fields_link :gears, '＋', class: 'btn btn-primary', role: 'button', id: 'add-form'
.
.
```

ここで詰まった事が、editアクションの時に画像を表示させるためのimage_tagにnested objectのattributeにアクセスする方法です。  

単純に`g.gear_image.url`'でアクセスできるかと思いきや、`undefined method `gear_image' for...`というエラーに。

そこでissueを覗いてみたら答え載ってました。（もう↑に正解のコードのせていますが）
`g.object.gear_image.url`という風にobjectを噛ませてあげればいいだけでした。

***
この記事書くのに40分ぐらい掛かってしまった。10分ぐらいで1記事書けるようにしたいですね



