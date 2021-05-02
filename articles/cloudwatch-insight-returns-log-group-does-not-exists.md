---
title: "CloudWatch Logs Insightsで、存在するはずのLogGroupへのリクエストで Log group 'XXX' does not exist for account ID 'ZZZ' と返ってくる"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","CloudWatch"]
published: true
---
# tl;dr

[公式ドキュメント](https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/logs/AnalyzingLogData.html)の記述の通り、2018年11月５日以降にLogEventが発生したLogGroupしか検出できないようです。

抜粋：  
> CloudWatch Logs Insights では、2018 年 11 月 5 日以降に CloudWatch Logs に送信されたログデータを検索できます。

# なんでこんな仕様なんだろう...？
完全に推測ですが、LogEventの作成時にDBの索引化の処理が走っているのではないでしょうか？
で、それが開始したのが2018年 11月 5日なのではないかと。
たしかに、もう２度と見られないログにまでindexを作成するのは、AWSからしたらコストが嵩んでたまったものじゃないと思われます。

