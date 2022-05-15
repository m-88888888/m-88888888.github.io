---
Title: Railsで表示中のページを判定する
Date: 2019-12-27T15:44:24+09:00
---

# はじめに
タブ表示を実現するために、どのタブがactiveかどうかを判定する方法に少し手間取ったので備忘録として残します。

# 結論

おあつらえ向きのヘルパーメソッドがありました

```ruby
current_page('指定のpath')
```

実際の使い方としてはこんな感じ

```ruby
ul.nav.nav-tabs.nav-fill
  li.nav-item
    = link_to  "ALL", root_path, class: "nav-link #{('active' if current_page?(root_path))}"
  li.nav-item
    = link_to  "MEN", gender_index_articles_path(gender: 1), class: "nav-link #{('active' if current_page?('/articles/gender/1'))}"
  li.nav-item
    = link_to  "WOMEN", gender_index_articles_path(gender: 2), class: "nav-link #{('active' if current_page?('/articles/gender/2'))}"
```

わかってしまえばどういうことはない
