---
title: "AWSの料金引き落としがコケたときのMyチェック手順"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS"]
published: true
---

クレカの利用上限額に達するなどでAWSの利用料金の支払に失敗していたときのチェックリストです。

## 本当に支払いに失敗しているか確認

注文と請求書画面を確認しましょう。
https://console.aws.amazon.com/billing/home?#/paymenthistory

**お支払い期限(Payments Due)**に表示されているものが未払いの請求書、**注文と請求書の履歴(Order and invoice history)**に表示されているものが支払い済みの請求書です（そういう日本語訳にしてほしかった...!）

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/e5d0ed5c-c780-9de6-1fb7-befff92f583b.png)

## 支払免除になっていないか確認

決済に失敗した場合に即時支払免除（キャンセル）になる請求があります。ドメインの更新です。
その場合、請求書(Bills) > 合計(Estimated Total)の中にキャンセルされた支払いが表示されています。

以上です。

