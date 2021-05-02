---
title: "怪奇！ECSのタスクをFargateで実行するとたまに起動失敗する！（ヒント：SubnetSelection）"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","CloudFormation","ECS","Fargate","CDK"]
published: true
---

# TL;DR
CloudFormationにおける `AWS::Events::Rule` - `AwsVpcConfiguration` - `Subnets ` は必須プロパティだが、CDKでは必須ではありません。
CDKで指定しなかった場合、VPC内から適当なサブネットが選択される(私の場合8つ)が、NAT Gatewayがアタッチされているかは考慮されないようです。
ECRのイメージをプルするには、PrivateLinkを使わない場合はインターネットへの接続が必要となるため、イメージが取得できずに起動に失敗していました。

# 事象
CloudWatch Rulesで定期実行に指定しているバッチ処理が3回に1回起動しませんでした。
単に起動しないだけではなく、

 * CloudWatchのメトリクス（TriggeredRules および Invocations）は記録されている
 * CloudTrailで ECSのRunTaskイベントのログを見ると、container のlastStatusは PENDING
 * CloudWatch Logsへのアプリケーションログは記録されていない

と、**CloudWatchからの呼び出しは成功しているのに、アプリケーションは立ち上がらない** 、という状況でした。


# どうやって原因究明したか

ECSタスクのイベントの変更をLambdaでCloudWatchに記録した上で、**証拠が出るまでバッチ処理を走らせました**笑

ECSタスクの記録方法は以下のチュートリアルに従いました。
https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/ecs_cwet.html

結果、以下のようなログが得られました。

```
...省略
        "containers": [
            {
                "containerArn": "arn:aws:ecs:ap-northeast-1:******:container/******",
                "lastStatus": "STOPPED",
                "name": "web",
                "image": "******.dkr.ecr.ap-northeast-1.amazonaws.com/******:******",
                "reason": "CannotPullContainerError: Error response from daemon: Get https://******.dkr.ecr.ap-northeast-1.amazonaws.com/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)",
                "taskArn": "arn:aws:ecs:ap-northeast-1:******:task/******",
                "networkInterfaces": [
                    {
                        "attachmentId": "******",
                        "privateIpv4Address": "******"
                    }
                ],
                "cpu": "256"
            }
        ],
...
```

はい。ECRからイメージをプルできていませんね。

対応としては、CDKのタスク起動時の定義でSubnetSelectionを明示的に指定しました。

# 反省

* CloudFormation → CDKに移行した環境では、CDKがよしなに処理してくれないこともある
* CDKで作成したリソースのCloudFormationテンプレートは確認しよう






