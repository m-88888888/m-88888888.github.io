---
Title: MySQL8→MySQL5.7にダウングレード
Date: 2019-10-12T23:15:26+09:00
---

MySQL8を5.7にダウングレードする際に盛大にハマったのでその対処を備忘録として残します。

HomebrewでMySQLをインストールしていたのでbrewコマンド使ってMySQL本体をアンインストール

```
brew uninstall mysql
```

MySQLに関連するファイルをすべて削除

この削除するべきファイルを探し出すのに時間がかかった…

```
sudo rm -rf /usr/local/mysql
sudo rm -rf /Library/StartupItems/MYSQL
sudo rm -rf /Library/PreferencePanes/MySQL.prefPane
sudo rm -rf /Library/Receipts/mysql-.pkg
sudo rm -rf /usr/local/Cellar/mysql*
sudo rm -rf /usr/local/bin/mysql*
sudo rm -rf /usr/local/var/mysql*
sudo rm -rf /usr/local/etc/my.cnf
sudo rm -rf /usr/local/share/mysql*
sudo rm -rf /usr/local/opt/mysql*
```

削除後、Finderの検索欄にmysqlと入力して関連ファイルがないことを確認

brew info mysqlでアンインストールされたことを確認してインストール

```
brew install mysql@5.7
```

==> Summaryと表示されたらインストール完了！

