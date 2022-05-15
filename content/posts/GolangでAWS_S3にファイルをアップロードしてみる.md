---
title: "GolangでAWS_S3にファイルをアップロードしてみる"
date: 2020-09-23T10:50:58+09:00
draft: false
tags: ["golang", "aws"]
---

AWSのサービスに接続するための`aws-sdk-go`で試しにS3にファイルをアップロード/削除してみた。

<!--more-->
## AWS S3のマネジメントコンソール
1. 適当なバケットを作成
2. アクセスレベルを`オブジェクトは公開可能`に設定
3. リージョンは`アジアパシフィック（東京)`

## アップロードの処理
```go
package main

import (
	"log"
	"os"

	// SDKのインポート
	"github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/aws/credentials"
	"github.com/aws/aws-sdk-go/aws/session"
	"github.com/aws/aws-sdk-go/service/s3/s3manager"
)

func main() {

	// 処理対象のファイル
	file, errr := os.Open("hoge.jpg")
	if errr != nil {
		log.Println(errr)
	}

	// AWSセッションインスタンスの作成
	sess, err := session.NewSession(&aws.Config{
		// 東京リージョン
		Region:      aws.String("ap-northeast-1"),
		// credential
		// 本来はENVなどの環境変数として設定する
		Credentials: credentials.NewStaticCredentials("AWS_ACCESS_KEY", "AWS_SEACRET_ACCESS_KEY", ""),
	})
	if err != nil {
		log.Fatal("session error: ", err)
	}

	// s3のアップローダー
	uploader := s3manager.NewUploader(sess)

	res, err := uploader.Upload(&s3manager.UploadInput{
		// アップロード先のバケット
		Bucket: aws.String("backet_name"),
		// ファイル名
		Key:    aws.String(file.Name()),
		// ファイル本体
		Body:   file,
	})
	if err != nil {
		log.Fatal("Unable to uploaded. reason: ", err)
	}

	// res.LocationでURLを取得でき、ファイルを参照できる
	log.Println("Sucessfully uploaded! file path: " + res.Location)
}
```

## 削除の処理
```go
package main

import (
	"log"
	"os"

	"github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/aws/credentials"
	"github.com/aws/aws-sdk-go/aws/session"
	"github.com/aws/aws-sdk-go/service/s3"
)

func main() {

	// 処理対象のファイル
	file, errr := os.Open("hoge.jpg")
	if errr != nil {
		log.Println(errr)
	}

	// AWSセッションインスタンスの作成
	sess, err := session.NewSession(&aws.Config{
		// 東京リージョン
		Region:      aws.String("ap-northeast-1"),
		// credential
		// 本来はENVなどの環境変数として設定する
		Credentials: credentials.NewStaticCredentials("AWS_ACCESS_KEY", "AWS_SEACRET_ACCESS_KEY", ""),
	})
	if err != nil {
		log.Fatal("session error: ", err)
	}

	svc := s3.New(sess)

	_, err = svc.DeleteObject(&s3.DeleteObjectInput{
		Bucket: aws.String("image-uploader-golang"),
		Key:    aws.String(file.Name()),
	})
	if err != nil {
		log.Fatal(err)
	}

	err = svc.WaitUntilObjectNotExists(&s3.HeadObjectInput{
		Bucket: aws.String("image-uploader-golang"),
		Key:    aws.String(file.Name()),
	})
	if err != nil {
		log.Fatal(err)
	}
	log.Println("Object is deleted.")
}
```