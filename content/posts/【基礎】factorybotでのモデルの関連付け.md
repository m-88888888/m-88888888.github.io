---
Title: 【基礎】factorybotでのモデルの関連付け
Date: 2019-12-17T00:39:06+09:00
---

# factorybotでのモデルの関連付け

メモ代わりにもならないけど...一応アウトプット

簡素だけど以下のモデルのような1対Nの関係の場合

```ruby:article.rb
class Article < ApplicationRecord
  belong_to :user
end
```

```ruby:user.rb
class User < ApplicationRecord
  has_many :articles
end

```

1のモデルは特に関連付けを宣言する必要なし

```ruby:users.rb
factory :user do
  email { 'test@test.com'}
  password { 'password' }
  password_confirmation { 'password'}
  name { Faker::Name.unique.name }
  gender { 1 }
  height { 170 }
end
```

Nのモデルはassociationを宣言する必要あり

```ruby:articles.rb
FactoryBot.define do
  factory :article, class: Article do
    photo { File.new("#{Rails.root}/spec/fixtures/nofile.jpg") }
    comment { "テストコメント" }
    association :user #Userモデルと関連があることを宣言
  end
end
```

あとはSystem SpecなりModel Specで使用するarticleのfactoryを使用するときに、引数にUserモデルを渡してやるだけ

```ruby:article_spec.rb
require 'rails_helper'

RSpec.describe Article, type: :model do

  before do
    @user = create(:user)
  end

  describe "#create" do
    it 'コーディネート画像、コーディネート紹介文があれば有効な状態であること' do
      article = create(:article, user: @user)　#ここでUserモデルを渡さないとnilだって怒られる
      article.valid?
      expect(article).to be_valid
    end
.
.
.
end
```

超簡単
