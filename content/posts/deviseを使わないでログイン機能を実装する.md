---
Title: deviseを使わないでログイン機能を実装する
Date: 2019-10-01T00:03:15+09:00
---

ポートフォリオの作成時にログイン機能を簡単に実装できる`devise`というgemを利用してみました。
コマンドを叩くだけで簡単なログイン機能を実装できるという、scaffoldに近い部分があるので、裏の動きを理解するためにRailsチュートリアルの内容を参考に、deviseが担っている処理を学習し直しました。

# やること

## 1. サインアップ機能

ユーザー登録時に最低限必要な情報は次の３つ。

- ユーザー名
- メールアドレス
- パスワード
 
サインアップ機能で重要なのは、パスワードを平文の状態で触れないこと。
そこでパスワードをハッシュ化（規則性のない文字列）に変換してからDBに保存させるようにする。

`BCrypt`というgemを使ってパスワードのハッシュ化に関する処理を行う。
主に以下の２つのメソッドが使える様になる。

1. has_secure_password
  セキュアにハッシュ化した不可逆なパスワードをDBのpassword_digestカラムに保存することができる。
  また、passwordとpassword_confirmationを使える様になる

2. authenticate
   引数の文字列が保存したパスワードと一致するとそれに紐づくUserオブジェクトを、一致しなければfalseを返す

### Userモデルとテーブルの作成

```ruby
# create_table.rb
class CreateUsers < ActiveRecord::Migration[5.2]
　　def change
　　　　create_table :users do |t|
　　　　　　t.string :username, null: false
　　　　　　t.string :email, null: false
　　　　　　t.string :password_digest
 
　　　　　　t.timestamps
　　　　end
　　end
end
```

```ruby
# User.rb
class User < ApplicationRecord
　　before_save { self.email = email.downcase }　#DB保存前に小文字に変換
 
　　validates :username,presence: true
　　validates :email,
　　　  　　　　 presence: true,
  　  　　　    uniqueness: { case_sensitive: false },　#大文字・小文字の区別なし
 　　　　       format: { with: /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i } #「/i」は大文字・小文字を区別せずにマッチングさせる正規表現
 
 　　has_secure_password
end
```

メールアドレスの大文字・小文字の区別は、メールサーバーによって変わるので小文字でフォーマットを統一させる。

### Userコントローラーの作成

ひとまずはサインアップとサインインの画面を作るだけなのでnewとcreateアクションを作成する

```ruby
class UsersController < ApplicationController
　　def new
　　  @user = User.new
　  end
 
  def create
    @user = User.new(user_params)
    if @user.save
      redirect_to root_path
　　 else
      render action: "new"
　　 end
 　end
 
　　privatef
 
　　def user_params
　　  params.require(:user).permit(:username, :email, :password, :password_confirmation)
 　 end
end
```

### サインアップ画面の作成

```ruby
h1 Login

= form_with model: @user do |f|
  .form-group
    = f.label :name, 'Name'
    = f.text_field :name, class: 'form-control'
  .form-group
    = f.label :email, 'Email'
    = f.text_field :email, class: 'form-control'
  .form-group
    = f.label :password, 'Password'
    = f.password_field :password, class: 'form-control'
  .form-group
    = f.label :password_confirmation, 'Password_Confirmation'
    = f.password_field :password_confirmation, class: 'form-control'
  = f.submit

```

サインアップの機能完了！

## 2.ログイン機能
sessionコントローラーを作成

- new・・・ログインフォームの作成
- create・・・ログイン処理
- destroy・・・ログアウト処理

```ruby
# sessions_controller.rb
class SessionsController < ApplicationController
  def new
  end

  def create
    user = User.find_by(email: params[:session][:email].downcase)
    #セッション情報に含まれているメールアドレスをもとに、Usersテーブルに保存されているUserオブジェクトを抽出

    if user && user.authenticate(params[:session][:password])
      # Userオブジェクトが抽出できる（つまり、Usersテーブルに存在している）かつ
　　　 # paramsで送られてきたセッションの中のパスワード（ハッシュ化されている）が
      # DB内のハッシュ化されたpassword_digestカラムの値と一致しているかどうか検証することで、ユーザーを識別
      log_in user
      redirect_to # ログイン後の遷移先_path
    else
      flash.now[:danger] = 'Invalid email/password combination'
      render 'new'
    end
  end
 
  def destroy
    log_out
    redirect_to root_url
  end
end
```

ヘルパーメソッドに処理を追加

```ruby
module SessionsHelper
    # ユーザーのブラウザ内にある一時cookieに暗号化済みのユーザーIDを作成する
    def log_in(user)
        session[:user_id] = user.id
    end
     
      # 現在ログインしているユーザーを返す (ユーザーがログイン中の場合のみ)
    def current_user
        @current_user ||= User.find_by(id: session[:user_id])
    end
     
     
    def logged_in?
        !current_user.nil?
        # ログイン中の状態＝セッションにユーザーが存在する＝current_userがnilでない状態。
    end
     
    def log_out
        session.delete(:user_id)　#セッションからユーザーIDを削除
        @current_user = nil　　　　#現在のユーザーをnilに設定
    end
end
```

ログイン機能の実装完了！

# まとめ
本当に必要最低限なユーザー認証を実装してみましたが、これらをコマンドひとつで解決してくれるdeviseは本当に便利

ただ、自分で機能をカスタマイズしたい時はその裏の動きを理解しなければならないので
gemを使わずに実装してみることの重要さも理解できました！
