---
Title: describeとcontextの使い分け？
Date: 2019-12-19T01:27:26+09:00
---

## はじめに
RSpecのdescribeとcontextの使い分けについてイマイチ理解していなかったので備忘録として残します。

結論から言うと、describeが機能についての説明、contextがその機能を実行中の状況をそれぞれ表してます。

単語そのままの意味でしたね。深く考えすぎでした

## 具体例

次のようなdescribeとcontextがない場合のRSpecの実行結果を見てみましょう

before

```bash
$ bin/rspec
Running via Spring preloader in process 2242

Note
  return notes that match the search term
  returns an empry collection when no results are dound

Finished in 0.47442 seconds (files took 0.48964 seconds to load)
2 examples, 0 failures
```

うーん。もうひと押し説明が欲しいよね

ということでdescribeとcontextを使ってアウトラインを作成すると

after
```bash
$ bin/rspec
Running via Spring preloader in process 2608

Note
  search message for a term
    when a match is found
      return notes that match the search term
      when no match is foumd
        returns an empry collection when no results are dound

Finished in 0.52984 seconds (files took 0.41586 seconds to load)
2 examples, 0 failures
```

もはやドキュメントですね。文章として読める。

これこそRSpecの醍醐味なのでしょう。

文章になるように意識して組み立てていけばいい感じになりますね〜

## まとめ

describe・・・説明「文字列に一致するメッセージを検索する」  
context・・・状況「一致するデータが見つかるとき」or「見つからないとき」

おしまい
