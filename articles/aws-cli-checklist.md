---
title: "aws cliが動きません😭 と言われたときに試してもらうリスト【随時更新】"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS"]
published: true
---
# キャッシュと環境変数のセッショントークンを削除してもらう

```
rm -f ~/.aws/cli/cache/*
unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN
```


# どこに設定されている認証情報でAWSのAPIを叩いているかをチェックする

```
aws configure list

# スイッチロールありの場合
aws configure list --profile ${PROFILE}
```

参考: https://dev.classmethod.jp/cloud/aws/how-to-configure-aws-cli/

# どのユーザーとして認証情報を叩いているかチェックする

```
aws sts get-caller-identity

# スイッチロールありの場合
aws sts get-caller-identity --profile ${PROFILE}
```

[awscli-alias](https://github.com/awslabs/awscli-aliases) を設定すると `aws whoami`で呼び出せるようになります。
参考: https://dev.classmethod.jp/cloud/aws/aws-cli-alias/


# 設定されいているIAMポリシーをチェックする

```
aws iam list-attached-user-policies --user-name ${USERNAME} --query "AttachedPolicies[*].PolicyArn" --output text | xargs -n1 -I {} aws iam get-policy --policy-arn {}

aws iam get-policy-version \
  --policy-arn <↑で取得したIAMポリシーのARN> \
  --version-id <↑で取得したバージョンID>

# スイッチロールする場合
PROFILE=YOUR_PROFILE
ROLE_NAME=YOUR_ROLE_NAME
aws iam list-attached-role-policies --role-name ${ROLE_NAME} --profile ${PROFILE} --query "AttachedPolicies[*].PolicyArn" --output text | xargs -n1 -I {} aws iam get-policy --policy-arn {} --profile ${PROFILE}

aws iam get-policy-version \
  --policy-arn <↑で取得したIAMポリシーのARN> \
  --version-id <↑で取得したバージョンID> \
  --profile=${PROFILE}
```




