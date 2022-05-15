---
title: "CircleCIのJobの結果をいい感じにSlackに通知するOrbsを作ってみた"
date: 2022-04-09T21:11:47+09:00
draft: false
---

## はじめに

所属している会社のプロジェクトは CI/CD に CircleCI を利用しており、ビルドやデプロイの結果を Slack に通知させています。  
CircleCI Official の Orbs を利用しており、ある程度メッセージのカスタマイズはできるのですが、通知にコミットメッセージを含ませたり、コミットの Author に対してメンションを飛ばすというようなことができません。
https://circleci.com/developer/ja/orbs/orb/circleci/slack

そこで、この Orbs をベースにカスタマイズした Orbs を作ってみました。

## 成果物

repository

- [m-88888888/slack-notify-orb](https://github.com/m-88888888/slack-notify-orb)

### ビルド成功

![スクリーンショット 2022-04-09 22 23 24](https://user-images.githubusercontent.com/51853475/162575988-8c623d39-a43f-430a-ac29-5f52d5105b17.png)

### ビルド失敗

![スクリーンショット 2022-04-09 22 23 33](https://user-images.githubusercontent.com/51853475/162575989-368b087d-d289-47a9-8cc3-228355ce5a6e.png)

### デプロイ成功

![スクリーンショット 2022-04-09 22 28 13](https://user-images.githubusercontent.com/51853475/162576137-3248c42b-0779-4f69-90b0-635249b44dac.png)

### デプロイ失敗

![スクリーンショット 2022-04-09 22 23 40](https://user-images.githubusercontent.com/51853475/162575990-1407411f-5d4a-484d-be35-c42d046b9ec9.png)

## how to use

CircleCI version

```yml
version: 2.1
```

orbs の読み込み

```yml
orbs:
  slack: m-88888888/slack-notify-orb@1.0.3
```

steps への組み込み

```yml
- slack/notify:
    slack_name_mappings: '{ "<@Uxxxxxxxx>": "m-88888888" }'
    deploy_flag: true
```

### 説明

| PARAMETER           | DESCRIPTION                                                                                                                                      | REQUIRED | DEFAULT | TYPE    |
| :------------------ | :----------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------- | ------- |
| slack_name_mappings | 通知先の Slack のメンバー ID と Github の username の対応関係。コミットの Author に対応する Slack 上のメンバーにメンションが飛ぶようになります。 | No       | -       | string  |
| deploy_flag         | 送信するメッセージの種類（build or deploy）                                                                                                      | No       | false   | boolean |

## 感想

Orbs の作成自体は非常に簡単でした。公式 Docs を読みつつ、`CircleCI Orbs 入門`あたりワードでググって出てきた記事を参考にすれば特に詰まることもなく作業できました。  

メンションを飛ばすために、Author と Slack 上のメンバー ID を紐付ける部分が厄介でした。単純にシェルスクリプト力が足りておらず、やりたいロジックをどう書けばよいか知らなかっただけでしたが...

JSON 形式のパラメータを String として受け取るようにして、シェルと jq を駆使してパラメータからメンバー ID を取り出すことで対処することができました。  

また何らかの課題を見つけたら作って公開してみたいと思います。

## 参考リンク

- [Orb の概要](https://circleci.com/docs/ja/2.0/orb-intro/)
- [CircleCI Orbs 入門 - 生産性向上ブログ](https://www.kaizenprogrammer.com/entry/2018/12/01/111145#Orb-%E3%81%AE%E5%85%AC%E9%96%8B)
- [circleci/slack@4.9.3](https://circleci.com/developer/ja/orbs/orb/circleci/slack)
