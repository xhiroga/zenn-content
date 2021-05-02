---
title: "STOPPEDになってから1時間以上経過したECSタスクの起動ログを確認する"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","ECS"]
published: true
---
# TL;DR
CloudTrailイベント履歴から、RunTask APIの実行結果を参照します。

# 概要

ECSのコンソールで確認できるECSのタスク実行結果は、1時間以内のものに限られています
（それ以上昔の実行結果の表示は保証されない）  
夜間のバッチ処理が失敗していたときなど、すぐに気がつけなかったときは困ります。CloudTrailから証跡を確認しましょう。

# 手順

基本的にCluodTrailの公式ドキュメントに沿う形で進めます。  
https://docs.aws.amazon.com/ja_jp/awscloudtrail/latest/userguide/view-cloudtrail-events-console.html

1. CloudTrailのコンソールを開きます。  
https://ap-northeast-1.console.aws.amazon.com/cloudtrail/home

2. イベント名 `RunTask` で絞り込みます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/fad4e8af-ed86-b0fa-c0e9-b3a0afba7e09.png)

3. 目当てのイベントを見つけたら `イベントの表示` でAPIの実行結果を参照します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/b899925b-4adf-9ffe-f98d-70882b354cd9.png)

以上です。

