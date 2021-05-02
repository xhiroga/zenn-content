---
title: "退職した社員が取得したRoute53管理のドメインを削除する"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws", "route53"]
published: true
---

# TL;DR

ドメインの削除申請をすると、そのドメインの管理者メールアドレスに削除の是非を問うメールが飛ぶ。そこで確認しないとドメインは削除できない。

## 何が起きたか？

Route 53 の Registered domains から、既に使っていないドメインを削除しようとした。
数日待つがドメインは削除されず、代わりに AWS からのメールが。

```
Dear AWS customer,

We recently received an online request from your AWS account to delete a domain name:

(ドメイン名)

We weren't able to delete the domain name.

We apologize for the inconvenience. For more information, contact Amazon Web Services Customer Support.
Domain Name: (ドメイン名)
```

## なぜ起きたか？

ドメイン管理者がかつて在籍したメンバーの名前になっていたため。
（現在の運用ではシステム管理者のグループアドレスで管理しているので、創業期ゆえの過ちという感じ）

## どう対処したか？

サポートにお問い合わせ → 理由を確認。
Registered domains から管理者メールアドレスを変更し、再度削除リクエストで OK.
