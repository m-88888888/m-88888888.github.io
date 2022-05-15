---
Title: bash→fishへの移行
Date: 2020-02-01T23:26:53+09:00
---

## はじめに
ターミナルの画面が味気なくて物足りなさを感じる。

テーマの導入がてら、ログインシェルをbashからfishに乗り換えてみる！

## Homebrewでfishのインストール

```
brew update
brew install fish
```

## fishをshellに追加する
シェルファイルにfishの情報を書き込む
```
sudo -s
echo /usr/local/bin/fish >> /etc/shells
exit
```

~~ログインシェルをfishに変更する~~  
ここで一つ問題が。fishは`POSIX非互換`のシェルらしい。

※`POSIX`とは、`シェルスクリプトの標準仕様`のこと。

なので`sh`で書かれたシェルスクリプトには対応していないらしく（別途Bash互換のスクリプトを動かすプラグインがあるらしい？？）

`chsh`でログインシェルをfishに変えてしまうと、トラブルシューティングに苦労するかもとのこと。

# iTerm2
ログインシェルを変えずにターミナルの設定で起動時に読み込むシェルを指定する

<figure class="figure-image figure-image-fotolife" title="読み込むシェルを指定">[f:id:hachi88888888:20200201232138p:plain]<figcaption>読み込むシェルを指定</figcaption></figure>

## PATH設定
`~/.config/fish/config.fish`を作る

```
touch ~/.config/fish/config.fish
```

PATHを書き込む  
あとaliasも設定しておく

```
# rbenv
set -x PATH $HOME/.rbenv/bin:$PATH
set -x PATH $HOME/.rbenv/shims $PATH
status --is-interactive; and source (rbenv init -|psub

# MySQL
set -x PATH /usr/local/opt/mysql@5.7/bin:$PATH)

# alias
alias be='bundle exec'
alias gs='git status'
alias ga='git add'
alias gc='git commit'
```

# fishプラグイン

## プラグイン管理ツールfisheをインストール
```
curl https://git.io/fisher --create-dirs -sLo ~/.config/fish/functions/fisher.fish
```

## bobthefishをインストール
gitのブランチやらステータスを表示してくれる優秀なやつ
```
fisher add oh-my-fish/theme-bobthefish
```

## Power-line-fontsをインストール
デフォルトでtheme-bobthefishを使うと、文字化けしてしまう

```
git clone https://github.com/powerline/fonts.git
cd fonts
./install.sh
cd ../
rm -rf ./fonts
```

iTerm2 > Preferences > Profiles > Text > Change Fontから、Literation Mono Powerlineを選択

これでOK

## fisherのconfig
とりあえずの設定。

```
# theme-bobthefish
set -g fish_prompt_pwd_dir_length 0  # ディレクトリ省略しない
set -g theme_display_git_master_branch yes # git branch名を表示
set -g theme_display_date no  # 時刻を表示しないように設定
set -g theme_display_cmd_duration no  # コマンド実行時間の非表示
```

最終的にこんな感じになった。

<figure class="figure-image figure-image-fotolife" title="完成">[f:id:hachi88888888:20200201232449p:plain]<figcaption>完成</figcaption></figure>

今日はここまで。
