---
title: "KtorをAWS Lambdaで動かしてみる"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","Kotlin","lambda","Ktor"]
published: true
---
# 事前知識
* Ktor
* AWS Lambda

---
## Ktor
JetBrains製、ピュアKotlinのマイクロWebアプリケーションフレームワーク。

参考資料: [Komparing Kotlin
Server Frameworks](https://resources.jetbrains.com/storage/products/kotlinconf2018/slides/1_Komparing%20Kotlin%20Server-Side%20Frameworks.pdf)

---
## AWS Lambda
AWSのマネージドFaaS。  
ソースコードをWebコンソールで貼り付ければ、APIやバッチ処理が簡単に作成可能。  

---
# WebフレームワークをFaaSで動かす？？？

---
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/87aa8e87-f9ca-d98c-6b18-ec11ecf7e0d7.png)

https://ktor.io/servers/lifecycle.html

---
# テスト用の機能を使えば、KtorのApplicationにLambdaの入力を渡せるのでは...🤔

---
![2019-04-25 08_29_47 AM からスキャン.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/53921ad2-ce99-05d6-fcbb-6676c033c2b3.jpeg)

---
# デモ

---
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/04260ff8-2d1f-9225-783f-01d363d02fdf.png)

---
# 解説
https://github.com/hiroga-cc/aws_lambda_gradle_kotlin

---
# 何の役に立つの？

---
# 分からん...😭

---
# むしろ私が聞きたい、けど...

---
# 一応考えられるユースケース
* コスト面を考慮しAPIをLambdaで構築したいが、テストが大変※1なのでローカルではWebフレームワークとして運用したい
* バッチ処理をLambdaで書きたいが、チームメンバーがKtorに慣れすぎていて学習コスト的にKtorで開発できた方が有利

※1: 感想には個人差があります。

---
# まとめ
* フレームワークの中身を使いまわせて面白い
* Ktorの仕組みを理解する上ではよかったかも...?

---

