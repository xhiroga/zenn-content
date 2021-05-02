---
title: "mongoDB AtlasにCLIで接続するまで"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["MongoDB"]
published: true
---

# 手順
1. CLIのインストール
2. MongoDB Userの追加
3. IPホワイトリストの編集
4. 疎通確認

# mongo CLIのインストール

    brew tap mongodb/brew
    brew install mongodb-community

参考: https://github.com/mongodb/homebrew-brew

# MongoDB Userの追加
プロジェクトのメニュー`Database Access`からユーザーを追加します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/bb21adf6-28a4-ca57-2c0c-78c8bdfbf7d5.png)

# IPホワイトリストの編集
アクセス可能なIPアドレスを追加します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/c8caab09-a1b7-ae15-aa7e-53ad8293cecf.png)

# 疎通確認

Clusters > Connectから接続先のホスト名を取得し...

```
mongo "mongodb+srv://${HOST}"  --username ${USERNAME}
```

これで接続可能なはずです。

