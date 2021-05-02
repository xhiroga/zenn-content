---
title: "AWS Route53で無停止サブドメイン委任（すでにサブドメイン以下にCNAME等のレコードが存在するケース）"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","route53"]
published: true
---

免責: DNSの専門家ではないので、よりよい方法があればぜひ教えてください！

# 背景
親ドメイン（例: hiroga.cc）のHosted Zone内でサブドメイン（例: sub.hiroga.cc）に関するレコードを管理していましたが、Hosted Zone内の見通しが悪くなってしまったので、サブドメインを分割します。
すでにレコードがあるケースの手順が調べても見当たらなかったため、備忘録とします。

# 手順

1. サブドメイン用のHosted Zoneの作成
2. サブドメイン用のHosted Zone内へ、親ドメインのHosted Zone内のレコードをコピー
3. サブドメインのHosted Zoneに対するNSレコードの作成
4. 親ドメインのHosted ZoneのNSレコードのTTLが過ぎるまで待つ。
5. 親ドメインのHosted Zone内の、2.でサブドメイン側にコピーしたレコードの削除

# 注意点

- 手順2.の段階で親ドメイン内のレコードを削除してはいけません。名前解決ができない時間が発生しますし、最悪悪意ある第三者が同様のレコードを公開するリスクがあると思われます。
- 手順3.が完了した段階で疎通チェックをします。

```
dig NS sub.hiroga.cc   # サブドメイン側Hosted Zoneに対応するレコードが表示されること
dig CNAME some-web.sub.hiroga.cc   # AUTHORITY SECTIONがサブドメイン側Hosted ZoneのNSレコードの値であること
```


# 参考資料:

[クラスメソッド - Route 53でサブドメインを別のHosted Zoneに権限委譲する](https://dev.classmethod.jp/cloud/aws/route53-transfer-hostedzones/)


