---
title: "CloudWatchのLogGroupをpecoで選択してcliで見る、awslogsの拡張シェル書いた"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","CloudWatch","awslogs"]
published: true
---

# TL;DR

AWSのCloudWatchのログをPECOで絞り込んで参照できます。

![e6506d60438a230b7c7f8fffce0c6fdb.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/215524c7-0d87-cf16-2171-aa418d1ad03a.gif)


# どうなっているか？

[awslogs](https://github.com/jorgebastida/awslogs)のラッパーになっています。

```sh
# 抜粋
awslogs get $(awslogs groups $AWS_PROFILE_OPTION | peco) $AWSLOGS_PURE_OPTIONS
```

業務で使うことを想定しているので、 `--profile`オプションにも対応しています。
（もともと `awslogs` が対応している、というだけの話ですが）

ソースはこちら。スターください！
https://github.com/hiroga-cc/cli/blob/master/awslogx/awslogx

