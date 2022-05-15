---
Title: ' RSpecでletを使う理由'
Date: 2019-12-17T23:25:21+09:00
---

## はじめに

RSpecのexampleでデータのcreateをDRYにするためにbeforeメソッドでデータ作成の処理をしていたんだけど

そういえばletとかいうメソッドあったよな〜と思い出して

beforeと役割被ってね？letってなんのために使うの？と疑問に思ったので調べてみました。



たとえば、次のようなModel Specはletを使ってリファクタリングできる

```ruby
RSpec.describe Article, type: :model do

  before do
    @user = create(:user)
  end

  describe 'validationのテスト' do
    it 'コーディネート画像、コーディネート紹介文があれば有効な状態であること' do
      article = create(:article, user: @user)
      article.valid?
      expect(article).to be_valid
    end

    it 'コーディネート画像がなければ無効な状態であること' do
      article = build(:article, photo: nil)
      article.valid?
      expect(article.errors[:photo]).to include("を入力してください")
    end
.
.
.
```

before中に書いたコードはdescribeの内部のテストすべてのexampleで実行されてしまっている  
つまり、使う必要のないデータを作成してしまっているというわけで、これはテストが遅くなったり予期しない影響を及ぼす場合もある  

そんな時に使うのが`let`。これは`遅延読み込み`を実現するメソッドで、必要になったときだけ実行される。  
また、beforeブロックで作成したデータを使用するためにインスタンス変数に格納していたが、letはその必要はない  
リファクタリング前のコードでは@userを使っていたが、リファクタリング後のコードではuserで呼び出している。

```ruby
RSpec.describe Article, type: :model do

  let(:user) { create(:user) }

  describe 'validationのテスト' do
    it 'コーディネート画像、コーディネート紹介文があれば有効な状態であること' do
      article = create(:article, user: user)
      article.valid?
      expect(article).to be_valid
    end

    it 'コーディネート画像がなければ無効な状態であること' do
      article = build(:article, photo: nil)
      article.valid?
      expect(article.errors[:photo]).to include("を入力してください")
    end
.
.
.
```

## まとめ

### 1. 読みやすい
単純にbeforeでは３行あったコードがletでは１行になったから読みやすくなるよね

### 2. 無駄な初期化を省略できる
「〜であれば無効な状態であること」とかのexampleでuserモデル作成する必要ないからいらない処理はしない。  
こういう細かな無駄な処理が積み重なるとテストも遅くなるのかな？

### 3. typo対策
beforeで作成したデータをインスタンス変数に格納しちゃうと、typo起こった時に気づきにくいよねってこと。  
letだと`メソッドを作るからtypoするとNameErrorが発生する`から便利〜

```ruby
NameError:
  undefined local variable or method `user' for #<RSpec::ExampleGroups::Article::Validation:0x00007f8dfdd09250>
  Did you mean?  uer
  # ./spec/models/article_spec.rb:26:in `block (3 levels) in <top (required)>'
```

## 参考資料

[https://qiita.com/jnchito/items/cdd9eef2ed193267c651#3-%E3%83%AD%E3%83%BC%E3%82%AB%E3%83%AB%E5%A4%89%E6%95%B0%E3%82%92%E3%81%9D%E3%81%AE%E3%81%BE%E3%81%BElet%E3%81%AB%E7%BD%AE%E3%81%8D%E6%8F%9B%E3%81%88%E3%82%89%E3%82%8C%E3%82%8B:embed:cite]

