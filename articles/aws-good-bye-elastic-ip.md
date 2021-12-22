---
title: "多分CDKかCloudFormationが消しそこねたElastic IPを憂いなく消す"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS"]
published: true
---

CloudFormationのデプロイでこんなエラーメッセージを受け取ったことはありませんか？

```
The maximum number of addresses has been reached. (Service: AmazonEC2; Status Code: 400; Error Code: AddressLimitExceeded; Request ID: ******; Proxy: null)
```

そんなにたくさんElastic IPを取得した覚えはないのに...おかしいな...?と思って見てみると使途不明のアドレスがぎっしり。誰がいつ作ったかを確認して、安心して消しましょう！

## TL;DR

1. [CloudTrail](https://ap-northeast-1.console.aws.amazon.com/cloudtrail/home) のEventHistoryを開きます。  
2. Lookup Attributes を Resource Name にした上で、検索条件にIPアドレスを打ち込みます。
3. AllocateAddress イベントが見つかります。

User Name（AWS SSOを利用している場合はメールアドレス）やSourte IP Address（CloudFormationを使っている場合は cloudformation.amazonaws.com ）が分かるので、社内で確認しやすいですね！
