---
title: "【Go言語】AWS Lambdaを使ってオウム返しするLINE Botを作ってみた"
date: 2020-09-30T22:20:19+09:00
draft: false
tags: ["golang", "aws", "linebot"]
---

最近学習中のGo言語とLambdaで何か作ってみたかったので、Lambdaを使ったLINE　Botに挑戦してみた。

<!--more-->

## Bot概要
- LINEでメッセージをオウムがえしして返答するチャットボット


## 構成図
![AWS Icons](https://user-images.githubusercontent.com/51853475/94694865-3f690600-0370-11eb-988f-f182d42f6ff0.png)

## AWS設定
Lambdaの設定についてはぐぐれば出てくるので省略。

## 処理の流れ
1. API Gatewayからイベントリクエストを受け取る
2. リクエスト内のBody部にあるJSON文字列をデコードする
3. LINE SDKを用いてBotを定義する（トークンなどの情報は環境変数として保持）
4. 応答メッセージを生成、ここではオウム返しするため送信されたメッセージをそのまま応答メッセージとして設定する
5. ステータスコードを返却して処理を終了

## コード

```go
package main

import (
	"encoding/json"
	"fmt"
	"github.com/aws/aws-lambda-go/events"
	"github.com/aws/aws-lambda-go/lambda"
	"github.com/line/line-bot-sdk-go/linebot"
	"log"
	"os"
)

func main() {
	// Lambda関数のエントリポイント
	lambda.Start(HandleRequest)
}

func HandleRequest(request events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {

	// LINEプラットフォームで生成されたWebhookオブジェクト
	fmt.Println("=== Body ===")
	fmt.Println(request.Body)

	// JSONをデコード
	fmt.Println("=== JSON decode ===")
	myLineRequest, err := UnmarshalLineRequest([]byte(request.Body))
	if err != nil {
		log.Fatal(err)
	}

	// ボットの定義
	fmt.Println("=== linebot new ===")
	bot, err := linebot.New(
		os.Getenv("LINE_CHANNEL_SECRET"),
		os.Getenv("LINE_CHANNEL_TOKEN"),
	)
	if err != nil {
		log.Fatal(err)
	}

	// 応答メッセージの生成
	fmt.Println("=== reply ===")
	replyMessage := myLineRequest.Events[0].Message.Text
	if _, err = bot.ReplyMessage(myLineRequest.Events[0].ReplyToken, linebot.NewTextMessage(replyMessage)).Do(); err != nil {
		log.Fatal(err)
	}

	// 終了
	fmt.Println("=== end ===")
	return events.APIGatewayProxyResponse{Body: request.Body, StatusCode: 200}, nil
}

// API Gatewayから受け取ったevents.APIGatewayProxyResponseのBody（JSON）をParseする
func UnmarshalLineRequest(data []byte) (LineRequest, error) {
	var r LineRequest
	err := json.Unmarshal(data, &r)
	return r, err
}

func (r *LineRequest) Marshal() ([]byte, error) {
	return json.Marshal(r)
}

type LineRequest struct {
	Destination string  `json:"destination"`
	Events      []Event `json:"events"`
}

type Event struct {
	ReplyToken string   `json:"replyToken"`
	Type       string   `json:"type"`
	Mode       string   `json:"mode"`
	Timestamp  int64    `json:"timestamp"`
	Source     Source   `json:"source"`
	Message    *Message `json:"message,omitempty"`
}

type Message struct {
	ID   string `json:"id"`
	Type string `json:"type"`
	Text string `json:"text"`
}

type Source struct {
	Type   string `json:"type"`
	UserID string `json:"userId"`
}
```

### ポイント
- API GatewayのイベントリクエストをLINE SDKを使用せずに自分でパースしてる。
  - LINE SDKで用意しているパースメソッド`ParseRequest()`は`*http.Request`を引数に渡す必要があるけど、lambdaがAPI Gatewayから受け取るイベントの型が対応していないから。

## ハマった点
- ポイントでも紹介した通り、リクエストをパースする処理で詰まってた。`ParseRequest()`をオーバーライドするか、自分でパース処理を書くか。今回は初めてだったので自分でパース処理を書いた。

## 今後の課題
- チャット以外送信したときの処理がないｌ
  - スタンプや画像、動画などを送信された時にも応答メッセージを送る
- lambdaから他のAWSサービスを利用する処理を追加したい。
  - 例>画像が送信されたらS3に保存する