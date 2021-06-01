---
title: "@aws-cdk/aws-amplify » App の addDomainを使ってはいけない"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "amplify"]
published: true
---

## TL;DR

- Amplifyではドメイン指定時、その前に接続先のブランチが作成されている必要がある
- にも関わらずCloudFormationの AWS:Amplify:Domain リソースでは、ブランチ名を文字列で取る
- したがってCloudFormationでAmplifyにドメイン付与する際はBranchに対する明示的なDependencyを指定する必要がある
- CDKはそんなこと考慮してくれない。

## トラブルシュート

このようなエラーが発生します。しかし `main` ブランチは存在するので、はぁ？となります。

```
A sub-domain can not point to a non existing branch: main
```

ここでの branch とは、リポジトリのブランチではなくAmplify AppのBranchリソースを指しています。  
したがって、先に AWS:Amplify:Branchリソースができている必要があるんですね。


## 所感

CloudFormation の AWS:Amplify:Domain リソースをデザインしたAWSのエンジニアには反省を求めたい。  
もしくは私の想像のつかない深い事情があるのかもしれない...
