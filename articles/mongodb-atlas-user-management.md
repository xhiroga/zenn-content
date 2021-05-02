---
title: "MongoDB Atlasのユーザー管理"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["MongoDB","MongoDBAtlas"]
published: true
---

MongoDB Atlasのユーザー管理に対する現時点の理解をまとめました。

# TL;DR

MongoDB Atlasには、 `Database User` と `Atlas User` の2種類があります。

# MongoDB Atlasについて

MongoDB, Inc. が管理するパブリッククラウドのアカウントでマネージドなMongoDBクラスターを利用できるサービスです。

MongoDBクラスターの作成や設定変更（オートスケーリング・バックアップ有効化・ストレージサイズ変更など）、ネットワーク設定、アラート設定等に加えて、コンソール上から `Collection` の中身に対してCRUD操作が可能な `Data Explorer`と、コンソール上での `Database User` の管理をサポートしています。

# Database User と Atlas User

Database User と Atlas Userの違いを表にまとめました。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/091d2bda-55d9-7524-3ada-7bfc72482bc9.png)

# 現在試しているユーザー設定

上記を踏まえ、私が現在試しているユーザー設定です。
ご意見ご感想お待ちしております。

## Database User

- アプリケーションごとにユーザーを作成しています（例: app, data-ware-house）
- オペレーションで人間がデータベースにアクセスする場合、MongoDBクライアントを利用しています。その際、都度期限つきのDatabase Userを発行します。
- 開発用プロジェクトでは誰でもユーザーを発行してよい設定です（後述）

## Atlas User

AWSのようにスイッチロールの概念がないため、個人ユーザーと管理用ユーザーで分けています。

- 個人ユーザー: `Organization ReadOnly` かつ 開発用プロジェクトの `ProjectOwner`
- 管理用ユーザー: `Organization Owner`（ログインごとに通知する)

※ `Organiztion Member` と　`Organization ReadOnly` の違いですが、前者の場合自分が所属していないプロジェクトの存在を認識できなくなります。

上記設定により、以下のそれぞれの操作に制約を設けることができました。

- 開発用プロジェクトのAtlasの操作: 個人ユーザーが可能
- 開発用プロジェクトのMongoDBの操作: 個人ユーザーが可能
- 本番プロジェクトのMongoDBの操作: 管理用ユーザーのみ可能
- 本番プロジェクトのMongoDBの操作: 管理用ユーザーのみ払い出した `Database User` が可能（有効期限をつける）

