---
title: "API Gatewayとバックエンドの疎通確認で時間を無駄にしないための手順書"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","APIGateway"]
published: true
---

API Gateway(HTTP API)の疎通確認は、段階ごとに進めないと混乱します。
リクエストがバックエンドに到達するまでと、各段階の調査方法をまとめました。

# API Gateway → バックエンドのリクエスト解決順

1. カスタムドメインをStageごとのInvoke URLに解決する。
2. ステージ内の指定したリソースに対するアクセス権が存在するかを確認する。
3. メソッドリクエスト実行のための認証を行う
4. メソッドリクエストを受け付けた後、API Gatewayからバックエンド（例えばAWS Lambda）に統合リクエストを行う（終了）


# 1. カスタムドメインをStageごとのInvoke URLに解決する

普通に`dig`してCNAMEレコードが返ってくるか確認してください。
ここで失敗していると、当然ですがそもそもAPIからレスポンスが返ってきません。

# 2. ステージ内の指定したリソースに対するアクセス権が存在するかを確認する

疎通確認のため、API GatewayのStageから`Enable CloudWatch Logs` を必ず有効にしてください。
CloudFormationで設定している場合は `TracingEnabled: true` で有効になります。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/4b53a52c-b1c7-5769-8f48-a81281c21a35.png)

その後、普通に `curl` や `Postman` でAPIにアクセスしてください。
私がよく遭遇するエラーは以下のとおりです。  

### Missing Authentication Token

API Gatewayの指定したリソースが存在しない場合に表示されます。
要するにURLやメソッドが間違ってるか、API Gatewayのリソースポリシーが間違っています。

```
{
    "message": "Missing Authentication Token"
}
```

# 3. メソッドリクエスト実行のための認証を行う

メソッドリクエストの実行に失敗した時によく遭遇するエラーは以下の通りです。

### Forbidden

例えばAPI_KEYが間違っています。

```
{
    "message": "Forbidden"
}
```

### The security token included in the request is invalid

メソッドリクエストの AuthorizerをIAMに指定している場合で、かつAWSのセッションが切れている場合です。

```
{
    "message": "The security token included in the request is invalid"
}
```


# 4. メソッドリクエストを受け付けた後、API Gatewayからバックエンド（例えばAWS Lambda）に統合リクエストを行う

ここまでくれば、後は API Gateway自体の CloudWatch Logsでデバッグできるでしょう。

よくやらかすのはAPI GatewayにLambdaを実行する権限がない場合などです。



