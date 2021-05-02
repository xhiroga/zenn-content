---
title: "IBM Watson Assistantにアクセスできない！権限設定のハマりどころ5つ"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Watson","iBM","WatsonAssistant"]
published: true
---

![image.png](https://qiita-image-store.s3.amazonaws.com/0/96286/3e196d00-7cca-48d9-3280-26cd0f222aaa.png)

# TL;DR
普段IBM Cloudを使い慣れていない私がWatsonAssistantを使ってハマったポイントは...
* 一つのメニューにたどり着くための方法が多い
* 権限の名前とできることの繋がりがちょっと分かりにくい
* コンソールのUIに難があり、設定変更が保存されたかどうか分かりづらい

# Watson Assistantに新規ユーザーを追加する手順
## 1. ユーザーを招待する
ハマり1. `IAM`やユーザー一覧などの設定あh、左上のハンバーガーメニューにありません。
ハマり2. Userの一覧を表示するためのメニューが2つあります。
* Manage >  Account > Users
* Manage >  Security > Identity and access 

以下のように、 `Manage`メニューからユーザー追加&権限の設定を行います。
<img width="1279" alt="スクリーンショット 2019-01-12 12.25.01.png" src="https://qiita-image-store.s3.amazonaws.com/0/96286/05d5228c-9552-b842-a147-d7b26acb1823.png">

## 2. ユーザーに Cloud Foundry Organizations の権限を追加する。
ハマり3. Cloud Foundry Orgの表示方法も２通りあります。
* Users > Cloud Foundry access > View Organization Details
* Manage > Account > Cloud Foundary Orgs

`Users > Cloud Foundry access > View Organization Details` で表示する場合
![image.png](https://qiita-image-store.s3.amazonaws.com/0/96286/ae6ba217-e354-8d75-ab2d-60d75891b4f9.png)

`Manage > Account > Cloud Foundary Orgs` で表示する場合
![image.png](https://qiita-image-store.s3.amazonaws.com/0/96286/1837a05e-89fe-6dfd-6ebb-ef41072c5046.png)

ハマり4. Managerの権限ではWatson Assistantは参照できません。 Developer権限が必要
※ Managerは他の人にCloud Foundary Orgの権限を渡す権限のようです。

ハマり5. Save と表示されているが、この `Save`は実はボタン！ 押さないと保存されない!
<img width="1278" alt="スクリーンショット 2019-01-12 12.35.22.png" src="https://qiita-image-store.s3.amazonaws.com/0/96286/d08404a9-d18d-e394-c1fa-4f017830de08.png">

# まとめ
普段AWSを使い慣れていることもあって、他のクラウドは戸惑うことも多いです。
まとめてみて、IBM Cloudのこれはこれで便利かも、と思いました（例えばメニューにはコンピューターリーソース以外の設定は入れない）
他の困った人の参考になれば幸いです。 

