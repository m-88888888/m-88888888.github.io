---
Title: 【Rails】scopeの引数がtrueならば、scopeを有効化する
Date: 2020-01-12T19:44:20+09:00
---

# はじめに
検索条件を複数選択することができる絞り込み検索の機能で、選択された条件のみで検索させたい。

最初はcase文かなにかでパターンを網羅しようと思ってたけど検索条件が増えるほど記述が増えてしまうので、なにか他にやり方がないか調べてみた。

# 結論

結論から述べると、scopeに渡した引数に値が有効であれば、scopeが有効化させればいい。

そのために後置きif文を使用する。

次のようなscope `narrow_down_name`にnameパラメータが渡されていたならばscopeを有効にしたい。

```ruby
  scope :narrow_down_name, -> (name) { where("name LIKE ?", "%#{name}%") }

```

後置きif文をつけて、もうひとつscopeを用意してあげればいい。

```ruby
  scope :narrow_down_name, -> (name) { where("name LIKE ?", "%#{name}%") if name.present? }
  
  scope :search, -> (search_params) do
    return if search_params.blank?

    narrow_down_name(search_params[:name])
  end
```

こうすればnarrow_down_nameの引数nameがtrueの場合、narrow_down_nameが有効化される。
あとは検索条件の数だけscopeを用意してあげればOK。

