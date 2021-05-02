---
title: "Cognito UserPoolとFederated Identityのいろいろな呼び方のまとめ"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","cognito"]
published: true
---
CognitoのUserPoolとFederated Identityには、いろいろな呼び方があります。
とっても迷いやすいポイントだと思うので、整理しました。


# まとめ
| 名前が登場する場所       |       UserPool |    Federated Identity    |
|:-----------------|------------------:|:------------------:|
| [製品紹介](https://aws.amazon.com/jp/cognito/dev-resources/)             |              UserPool |        Federated Identity        |
| API Reference           |            [Identity Provider](https://docs.aws.amazon.com/ja_jp/cognito-user-identity-pools/latest/APIReference/Welcome.html) |       [Federated Identities](https://docs.aws.amazon.com/ja_jp/cognitoidentity/latest/APIReference/Welcome.html)       |
| API Reference (URL)             |              cognito-user-identity-pools |        cognitoidentity        |
| cli               |                [cognito-idp](https://docs.aws.amazon.com/ja_jp/cli/latest/reference/cognito-idp/index.html) |         [cognito-identity](https://docs.aws.amazon.com/ja_jp/cli/latest/reference/cognito-identity/index.html)         |
| SDK (JavaScript)             |             [CognitoIdentityServiceProvider](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/CognitoIdentityServiceProvider.html) |       [CognitoIdentity](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/CognitoIdentity.html)       |
| SDK (Python)             |             [CognitoIdentityProvider](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/cognito-idp.html) |       [CognitoIdentity](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/cognito-identity.html)       |


