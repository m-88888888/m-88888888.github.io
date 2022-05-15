---
Title: '【rbenv】mkdir: /usr/local/bin/../version_cache: Permission denied【Mac】'
Date: 2020-02-22T23:24:09+09:00
---

# はじめに

Railsコマンドを叩くたびに発生していた以下のエラーが解決できたので、記事に残しておきます

# エラー

```
 $ rails s
mkdir: /usr/local/bin/../version_cache: Permission denied
/usr/local/bin/rbenv-communal-gem-home: line 21: /usr/local/bin/../version_cache/2.7.0: No such file or directory
=> Booting Puma
=> Rails 6.0.2.1 application starting in development
=> Run `rails server --help` for more startup options
..
```

`rbenv-communal-gem-home`というファイルのキャッシュPATHが間違っているとか怒られているみたいですね

# 対応

`rbenv-communal-gem-home`を開く

```
#!/usr/bin/env bash
#
# Summary: Show the communal gem home if communal gems are enabled
#
# Usage: rbenv communal-gem-home [<version>]

# Provide rbenv completions
if [ "$1" = "--complete" ]; then
  exec rbenv-versions --bare
fi

rbenv_version="${1:-$(rbenv-version-name)}"
if [ -L "$RBENV_ROOT/versions/$rbenv_version/lib/ruby/gems" ]; then
  cachedir="${BASH_SOURCE%/*}/../version_cache"
  cachefile="$cachedir/$rbenv_version"
  if [ -f "$cachefile" ]; then
    communal_version="$(cat $cachefile)"
  else
    mkdir -p "$cachedir"
    communal_version="$("$RBENV_ROOT/versions/$rbenv_version/bin/ruby" -rrbconfig -e 'puts RbConfig::CONFIG["ruby_version"]')"
    echo "$communal_version" > "$cachefile"
  fi

  echo "$RBENV_ROOT/gems/$communal_version"
else
  exit 1
fi
```

14~15行目あたりのPATHを変更します
```
- cachedir="${BASH_SOURCE%/*}/../version_cache"
+ cachedir="$RBENV_ROOT/.communal-gem-version-cache"
```

# 結果

Railsコマンド叩いてエラーが発生しなくなったかを確認してみる

```
$ rails s
=> Booting Puma
=> Rails 6.0.2.1 application starting in development
=> Run `rails server --help` for more startup options
..
```

でなくなったっぽい！

念の為`rbenv doctor`コマンドでも確認しておく
```
$ curl -fsSL https://github.com/rbenv/rbenv-installer/raw/master/bin/rbenv-doctor | bash
Checking for `rbenv' in PATH: /usr/local/bin/rbenv
Checking for rbenv shims in PATH: OK
Checking `rbenv install' support: /usr/local/bin/rbenv-install (ruby-build 20200214)
Counting installed Ruby versions: 3 versions
Checking RubyGems settings: OK
Auditing installed plugins: OK
```

気持ち悪いログがでなくなってスッキリ！

今日はここまで

## 参考
- [[Mac]Rails コマンドでエラー /usr/local/bin/rbenv-communal-gem-home Permission denied - Qiita](https://qiita.com/sachiotomita/items/3a298bae37835b55ae40)
- [Installing via homebrew breaks communal-gem-home command · Issue #12 · tpope/rbenv-communal-gems · GitHub](https://github.com/tpope/rbenv-communal-gems/issues/12)
