---
title: "API Gatewayをクロスアカウントで参照する場合のトラブルシューティング"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","APIGateway"]
published: true
---
# 前提
* API Gatewayのリソースポリシーは、APIをデプロイしないと反映されません。
* API Gatewayをデプロイしてから、リソースポリシーが反映されるまで10秒くらいかかりました。
* IAM RoleにAssume Roleしている場合(Federated Identityを利用している場合を含む)、ポリシーの更新が適用されるのはセッション更新後のようです（?）(これは自信ないですが...）

# The client is not authorized to perform this operation.
API Gateway自体のリソースポリシーで、指定したオペレーション（＝特定のエンドポイントへのリクエスト）が制限されています。
API Gatewayのリソースポリシー（sam拡張のCFnテンプレートの場合は`x-amazon-apigateway-policy`）を確認し、目的のResource（＝エンドポイント）への　`execute-api:Invoke` がAllow担っていることを確認しましょう。

# Credential should be scoped to a valid region, not ...

```
{
    "message": "Credential should be scoped to a valid region, not 'ap-northeast-1'. "
}
```

認証タイプが `AWS_IAM` の場合、リクエスト時のREGIONは指定必須のようです。また、API GatewayのあるREGIONを指定する必要があります。

# The security token included in the request is invalid.

```
{
    "message": "The security token included in the request is invalid."
}
```

## 考えられる原因
* 認証タイプが `AWS_IAM` の場合、AccessKeyかSessionTokenが間違っています。
* そもそもリソースが存在しないケースでも表示されます。

# Missing Authentication Token

```
{
    "message": "Missing Authentication Token"
}
```

## 考えられる原因
* そもそもリソースが存在しないケースでも表示される気がします。


# is not authorized to perform

```
{
    "Message": "User: arn:aws:sts::********5355:assumed-role/****VY71/CognitoIdentityCredentials is not authorized to perform: execute-api:Invoke on resource: arn:aws:execute-api:us-west-2:********1790:****bkva/v1/GET/test"
}
```

## 考えられる原因
* IAM側のポリシーの誤り
* API Gateway側のリソースポリシーの誤り
* 存在しないAPI Gatewayのリソースにアクセスしている

# 404 ACCESS DENIED

## 考えられる原因
* リソースポリシーの設定漏れ・誤り
* IAM Role/ User側のポリシーの設定漏れ・誤り

# OPTIONSに対するリクエストで403 Forbiddenが返ってくる

## 考えられる原因
* そもそもoptionsメソッドが設定されていない
* optionsメソッドの設定に誤りがある。


# レスポンスが返ってこない

## 考えられる原因
* カスタムドメインのBasePathMappingの設定が完了していない
* `https://` ではなく `http://` でアクセスしている
* Route53の設定が完了していない

参考: [ドメイン名を使用してトラフィックを Amazon API Gateway API にルーティングする](https://docs.aws.amazon.com/ja_jp/Route53/latest/DeveloperGuide/routing-to-api-gateway.html)


# 参考
[Amazon API Gateway での REST API の作成、デプロイ、および呼び出し » API Gateway での REST API へのアクセスの制御と管理 » API Gateway Lambda オーソライザーの使用
](https://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/apigateway-use-lambda-authorizer.html)


