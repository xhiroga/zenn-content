---
title: "AWSのSavings Planを理解して使う"
emoji: "🐘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws", "cost-explorer"]
published: true
---

## 免責

AWSの利用状況ごとに適切な割引があるはずです。正確な情報は最寄りのSAの方にお尋ねください。  
この記事はEC2, ECS, Lambdaをバランス良く使っている前提で書いています。

## TL;DR

- Savings Plansは前払い。よく理解して払おう。
- 前払いをした後は、払いすぎではなかったか、逆に不足ではなかったかを毎月検証しよう。

## 前提

Savings Plansって何？という場合、[公式ドキュメント](https://docs.aws.amazon.com/ja_jp/savingsplans/latest/userguide/what-is-savings-plans.html)をご参照ください。

## Steps

Savings Plansとの付き合いは、前払いと毎月の検証からなります。

1. 現状のコストの確認
2. Saving Plansの設定
3. 毎月の削減額の検証

### 現状のコストの確認

Savings Plansの対象となるEC2, Fargate, Lambdaのオンデマンド利用金額は、Cost Explorerから以下のフィルターを適用することで確かめられるようです。

- 使用タイプ - BoxUsage で検索して表示されるすべての使用タイプ
- 使用タイプ - Fargate で検索して表示されるすべての使用タイプ
- 使用タイプ - LAMBDA で検索して表示されるすべての使用タイプ

ただし、BoxUsage, Fargate, LAMBDAの使用タイプにも種類があり、もしかしたらすべてが対象というわけではないかもしれません。  
サポートにお尋ねした回答は「使用タイプのBoxUsage項目、Fargate (ECS、EKS) 、Lambdaが含まれます」とのことで、ざっくりしたご回答でした。

Cost Explorerから金額を確認したら、次は Savings Plans > 推奨事項を確認します。  
数字が3つ並んでいるので、それぞれの読み方を解説します。

#### 現在の月別オンデマンド使用量

計算ロジックは内部情報のため非公開ですが、公開情報によれば `[現状のコストの確認](###現状のコストの確認) で算出したオンデマンド使用量 - 一時的なSpike` で算出されているようです。  

#### 推定月別削減額

`([現在の月別オンデマンド使用量](###現在の月別オンデマンド使用量) - オンデマンドのまま残しておく支出) * 割引率` のように見えます。

オンデマンドのまま残しておく支出とは、使用金額が低くなる可能性を見越して全部を前払いしないためのバッファのようなものです。私の環境だと[現在の月別オンデマンド使用量](###現在の月別オンデマンド使用量)の3%程度でした。  
割引率とは、Savings Plans期間・支払いオプションによって定まると思われる値です。私の環境では、1年間全額前払いで22%でした。

#### 推定月別支出額

`[コミットメント](####コミットメント) + オンデマンドのまま残しておく支出` のように見えます。  
言い換えると、前払い分を対象期間の各月に配分した実質的な支払金額ですね。

#### コミットメント

`([現在の月別オンデマンド使用量](###現在の月別オンデマンド使用量) - オンデマンドのまま残しておく支出) * (1 - 割引率))` のように見えます。

カッコつけた物言いですが、要するに前払いですね。  
ちなみに、前払い金額の合計はカートに追加しないと見えません（ここなんとかしてほしい）

### Savings Plansの設定

Savings Plansをカートに追加し、Savings Plans > カート から注文書を送信することで前払いが行われます。  
前払いの合計金額はカート画面でしか表示されないので、必ずいくら払うことになるのか確認してください。  
金額によってはクレジットカードの支払額上限を超えるので、チーム開発の場合は経営陣・財務と相談が必要なこともあります。

### 毎月の削減額の検証

Savings Plansを購入したら、毎月コストを確認するタイミングで削減額もチェックしましょう。  
以下の金額を毎月チェックするといいでしょう。

 - [当月に実際にクレジットカードから引き落とされる金額](#当月に実際にクレジットカードから引き落とされる金額)
 - [Savings Plansの前払い分を対象付きに配分した、当月にAWSを動かすために実質的に支払った金額](#savings-plansの前払い分を対象付きに配分した当月にawsを動かすために実質的に支払った金額)
 - [当月の前払い分、Savings Plans対象使用量、当月のEC2, Fargate, Lambdaの使用量の比較](#当月の前払い分savings-plans対象使用量当月のec2-fargate-lambdaの使用量の比較)

#### 当月に実際にクレジットカードから引き落とされる金額

単に[Cost Explorer](https://console.aws.amazon.com/cost-management/home#/custom)から確認すればOKです。

#### Savings Plansの前払い分を対象付きに配分した、当月にAWSを動かすために実質的に支払った金額

`料金タイプから Savings Plan の取り消し, Saving Plan の対象使用量 を除いた金額 + Savings Plan のコミットメント($X/時間) * 24 * 当月の日数` です。  
料金タイプは 詳細 / その他のフィルターの中に隠れていることがあるので、注意してください。  
また、Savings Plan のコミットメント($X/時間)は、 Savings Plans > インベントリ から確認することができます。

#### 利用率, 当月の前払い分、Savings Plans対象使用量、当月のEC2, Fargate, Lambdaの使用量の比較

前払いし過ぎではないか？逆に、追加で払ったほうがよいか？を、毎月検証します。  
金額はそれぞれ以下の通り算出します。

- 利用率 = Cost Explorer > Savings Plan > 使用状況レポート
- 当月の前払い分 = `Savings Plan のコミットメント($X/時間) * 24 * 当月の日数`
- Savings Plans対象使用量 = `料金タイプから Savings Plan の対象使用量 のみを選択した金額`
- 当月のEC2, Fargate, Lambdaの使用量 = `[現状のコストの確認](###現状のコストの確認)のやり方で算出した金額`

このとき、`当月の前払い分`が`Savings Plans対象使用量`を上回っているようなら、前払いをしすぎだったのかもしれません（自分の環境で前払いのしすぎだったことがないので、本当にこの比較で確かめられるかは自信がないです）  
また逆に、`当月のEC2, Fargate, Lambdaの使用量` が `Savings Plans対象使用量` を大幅に上回っているようならSavings Planの追加適用が必要かもしれません。

## まとめ

とっつきづらい印象のSavings Planですが、一歩一歩理解するのが大事だと実感しました。  
もし誤りや不明な点があればぜひコメントお願いします。
