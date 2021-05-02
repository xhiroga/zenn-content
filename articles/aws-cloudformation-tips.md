---
title: "AWS CloudFormationの細かすぎるTIPS集 【最終更新: 2019-06-02】"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","CloudFormation"]
published: true
---
# Templatesの仕様

## リソースが存在しないスタックを作成することはできない

## 空のディレクティブがあると怒られる
```
Outputs: 
    #何も記載しない
```

## DescriptionにはExportされる値の例を書いておくと嬉しい
```
Outputs:
  ECSCluster:
    Description: A reference to ECS cluster name. ex) dev-Cluster
    Value: !Ref TheECSCluster
    Export:
      Name: !Sub ${Env}-Cluster
```

リソースの戻り値の例はこちら  
[!Ref](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html)
[!GetAtt](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html)

## Export Nameの頭にも環境名をつけちゃう
同一リージョンに同じスタックを複数作成するときにExportの名前がかぶるからですね。
環境名じゃなくてスタック名とかでもOKです。

```
# 例１
Name: !Sub ${Env}-Cluster

# 例２ （こんな書き方もできる）
Name: !Join [ "-", [ "Ref":"AWS::StackName", "vpcid"]]
```

## cfn-lintで一括lint
`find ./cfn/templates -type f | xargs --no-run-if-empty cfn-lint`

## S3からincludeする場合は、JSON/YAMLとして正しい形式でないと駄目。
!Ref 拡張関数などはYAMLのフォーマットの拡張なので、S3からincludeする場合は使うのをやめましょう。Ref: にしておけば問題ないです。

```
Status reason
Transform AWS::Include failed with: The specified S3 object's content should be valid Yaml/JSON
```

## Type: AWS::Serverless::FunctionのCodeUriのパスの基準
`aws cloudformation` コマンドを実行したディレクトリ**ではなく**、そのテンプレート自体を基準としたパス


# Stackの仕様
## ParametersのDefaut値は更新されない
TemplateのParameterでDefaultを指定できますが、この値がスタックに読み取られるのはスタックの新規作成時だけです。  
もしもDefault値を更新したいなら、Parameterの名前を変えるなど工夫する必要があります。


## リソースに変更がなくても強制的にupdateしたい場合は、毎回タグを更新する
```
aws cloudformation deploy \
  ......
  --tags date="$(date '+%Y%m%d%H%M%S')"
```
## CloudFomationで扱うとちょっと面倒臭いリソース一覧
* DynamoDBは一度に１０個以上作成・更新・削除できないので、一つのテンプレートにまとめて記述すると後から困ることがある
* CloudFrontはリソース作成・削除に15分くらいかかる
* API GatewayのVPC Linkは作成に10分くらいかかる
* `AWS::ApiGateway::Deployment`は、リソース自体を更新してもAPI Gatewayのデプロイが再発生するわけではない

## 削除しづらいリソース一覧
* ECSはサービスが動いているとクラスタを削除できない。
* ECRのリポジトリはイメージが保存されていると削除できない
* API Gatewayのステージはカスタムドメイン名にリンクされていると削除できない
* API GatewayのVPC Linkはメソッドに紐ついていると削除できない
* ELBはVPC Linkに紐ついていると削除できない。

## タイミングが重要なリソース一覧
* スタック削除を実行したのち、そのスタックのリソースを利用する手動作成したRDSを削除を実行すると、タイミングによってはスタックで管理されたDBSubnetGroupが削除されないことがある
* UserPoolのラムダトリガーは、逆にUserPoolから参照されていても削除することができる。

# リソース

## ECS
fargateの場合、EC2モードとは異なる注意が必要。
* Cpuはコンテナではなくタスクレベルで定義する必要がある
* タスク実行ロールの指定が必須


```
Fargate requires task definition to have execution role ARN to support log driver awslogs. 
```


# AWS CLI
## --debugオプションのAPIのRequest/Responseが参照可能

Terraformの `TF_LOG=DEBUG` のようなものですね。  
ちなみに他のCLIのコマンドでも共通のようです。

```
# SAMPLE
$ aws cloudformation create-change-set ... --debug

2019-05-08 15:20:42,783 - MainThread - botocore.endpoint - DEBUG - Sending http request: <PreparedRequest [POST]>
2019-05-08 15:20:42,784 - MainThread - botocore.vendored.requests.packages.urllib3.connectionpool - INFO - Starting new HTTPS connection (1): cloudformation.us-west-2.amazonaws.com
2019-05-08 15:20:44,671 - MainThread - botocore.vendored.requests.packages.urllib3.connectionpool - DEBUG - "POST / HTTP/1.1" 200 525
2019-05-08 15:20:44,671 - MainThread - botocore.parsers - DEBUG - Response headers: {'x-amzn-requestid': '689a47e0-7159-11e9-b146-1b67a791f534', 'content-type': 'text/xml', 'content-length': '525', 'date': 'Wed, 08 May 2019 06:20:43 GMT'}
2019-05-08 15:20:44,672 - MainThread - botocore.parsers - DEBUG - Response body:
b'<CreateChangeSetResponse xmlns="http://cloudformation.amazonaws.com/doc/2010-05-15/">\n  <CreateChangeSetResult>\n    <StackId>arn:aws:cloudformation:us-west-2:061260995355:stack/Base/02955f00-12f2-11e9-9754-0660270eccea</StackId>\n    <Id>arn:aws:cloudformation:us-west-2:061260995355:changeSet/master-update-20190508152032/6e709df7-925c-4d50-b0ed-612e89256b3b</Id>\n  </CreateChangeSetResult>\n  <ResponseMetadata>\n    <RequestId>689a47e0-7159-11e9-b146-1b67a791f534</RequestId>\n  </ResponseMetadata>\n</CreateChangeSetResponse>\n'
...
```

## create/update-stack と deploy で微妙に挙動が違う
* テンプレートファイルを指定するときのパスの書き方が違う

```
# create/update-stack
--template-file "file://cfn/banana.yaml" \

# deploy
--template-file "cfn/banana.yaml" \
```

* deployではcli-input-jsonが利用できない
パラメータを外出ししたいときに、オプションから指定することしかできないのでちょっと不便です。

# SAM
## DefinitionBodyと組み込み関数の両立には工夫が必要
```
```

# その他
## コマンドごとのS3のアクセス方法まとめ

`Fn::Transform:` の `Location:` で指定する場合: s3スキーム

参考: 
aws s3 mb の場合は s3:// スキームでアクセスする必要がある。
https:// スキームの場合は、received non 200 status code of 404と出て失敗する



