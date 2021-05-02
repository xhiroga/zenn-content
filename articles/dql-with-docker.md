---
title: "DQL使ってみた with Docker & マルチプロファイル"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","DynamoDB"]
published: true
---

DynamoDBをSQLライクに操作できる[DQL](https://github.com/stevearc/dql)。
仕事で使うことになったものの、環境設定から始めるとメンバーへの展開に時間がかかるし、マルチプロファイルじゃないと大規模な現場では使えないですよね！どちらも設定してみました。

# Docker

```Dockerfile
FROM python:2.7.18-buster

RUN pip install --upgrade pip;\
    pip install --upgrade dql

ENTRYPOINT [ "dql" ]
```

# プロファイル

スイッチロールにMFAが必要だったりするとdqlでは対処できないので、`aws-vault` にお世話になります。

`aws-vault`ではパスに実行可能ファイルが必要なので、こんなファイルを$PATHの通る場所に置いておき...

```sh
#!/bin/sh
docker run --env AWS_DEFAULT_REGION --env AWS_REGION --env AWS_ACCESS_KEY_ID --env AWS_SECRET_ACCESS_KEY --env AWS_SESSION_TOKEN --env AWS_SECURITY_TOKEN --env AWS_SESSION_EXPIRATION --rm -it -v "$HOME/.aws:/root/.aws" hiroga/dql:latest "$@"
```

以下のように実行します。

```terminal
aws-vault exec my-profile -- dql -r us-west-2
us-west-2>
```


実際にDynamoDBを操作してみましょう。

```terminal
aws-vault exec my-profile -- dql -r us-west-2
us-west-2> scan * from users;
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------
   user_id : 'c06b0c58-5aeb-49a3-87a4-2671cfca0694'
      name : 'hiroga'
```

いい感じですね！


