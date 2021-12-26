---
title: "RSSフィードを他の人が一括登録可能な状態で公開した"
emoji: "🐘"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["rss", "opml"]
published: true
---

## TL;DR

クラスメソッドの濱田さんの記事で[導入3分！RSSフィード29種一括登録でAWS最新情報を漏れなくチェック \| DevelopersIO](https://dev.classmethod.jp/articles/aws-rss-feeds/)というのがあります。これのマルチクラウド版がやりたかった。作りました。

[xhiroga/rss: RSS feeds in OPML format](https://github.com/xhiroga/rss)

## ポイント

### AWS

基本的にはWhat's Newが抑えられていれば十分なので、（もしかしたら3年前には存在しなかった）リリースノートのRSSフィードを購読。それ以外にも、読み物として日本語ブログを購読。

[オリジナルのリポジトリ](https://github.com/HamadaKoji/aws-public-rss-feeds/blob/master/aws-blog-youtube-podcast.opml)には大量のRSSがありますが、私には読みきれないと判断しました。


### JPCERT/CC

脆弱性情報を確実に捉えるため、JPCERT/CCのRSSを購読。

Log4Jの脆弱性も、TwitterのMinecraft界隈で騒がれていたのが2021/12/10 → RSS登録が 2021/12/11 なので、Twitterで見逃したニュースを追いかけるには十分と判断。


### その他

GCPやDatadog, GitHubなどのブログを収録しました。

ポッドキャストと各社の技術ブログは充実化を検討中。あまり多くなると読みきれないんですよね...


## まとめ

GitHubでRSSフィードの管理をはじめました。今後はInoreaderへの登録も自動化できたらいいな。
