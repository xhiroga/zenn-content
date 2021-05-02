---
title: "CloudWatch Logs Insightsで、存在するはずのLogGroupへのリクエストが失敗する。"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "CloudWatch"]
published: true
---

## tl;dr

[公式ドキュメント](https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/logs/AnalyzingLogData.html)の記述の通り、2018 年 11 月５日以降に LogEvent が発生した LogGroup しか検出できないようです。

抜粋：

> CloudWatch Logs Insights では、2018 年 11 月 5 日以降に CloudWatch Logs に送信されたログデータを検索できます。

## 実際のエラーメッセージ

Log group 'XXX' does not exist for account ID 'ZZZ'

## なんでこんな仕様なんだろう...？

完全に推測ですが、LogEvent の作成時に DB の索引化の処理が走っているのではないでしょうか？
で、それが開始したのが 2018 年 11 月 5 日なのではないかと。
たしかに、もう２度と見られないログにまで index を作成するのは、AWS からしたらコストが嵩んでたまったものじゃないと思われます。
