---
title: "Windowsからクライアント証明書をexportしてスクレイピングする"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["openssl"]
published: true
---
[justInCase](https://www.wantedly.com/companies/justincase)では保険業務の自動化を進めており、その中にはWindows端末が必須とされている業務もありました。
そこで、クライアント証明書&秘密鍵をexportしてプログラムからでもアクセスできるようにしてみたので共有します。

# TL;DR
* クライアント証明書&秘密鍵をexportし、pemファイルに変換する
* export不可の場合もjailbreak用のツールがある**（自己責任）**
* 想定と異なるクライアントを利用する点についてサーバー側の許可を得ること。
* スクレイピングは`robot.txt`の指示に従うこと

# 1. 秘密鍵をexport可能な場合
Windowsのcertmgrから証明書を選択し、全てのタスク > エクスポート、と進め、証明書と一緒に秘密鍵をエクスポートしますか？で"はい"を選択します。
ここで"はい"がグレーアウトしていたら、 certmgrを無視して秘密鍵をエクスポートできるツールを利用するしかないと思います。

証明書は暗号化された形式でエクスポートされるので、これを.pemファイルに変換する手間が必要です。
`openssl` コマンドで可能なので、詳しく知りたい方はリンク先をどうぞ。
[@IT ー Windows上で、証明書や秘密鍵をPEM形式に変換してエクスポートする (1/2)](http://www.atmarkit.co.jp/ait/articles/1602/05/news039.html)


# 2. 秘密鍵をexport不可能な場合
![image.png](https://qiita-image-store.s3.amazonaws.com/0/96286/0e58d27d-7416-fab4-c51c-4e3dd4bb3e8f.png)

cermgrで秘密鍵のエクスポートが不可能な場合は、アメリカのセキュリティ関連サービス企業が公開している秘密鍵のエクスポートツールがあるので、それを利用してエクスポートすることができます。
https://github.com/iSECPartners/jailbreak

jailbreak後は.pfx形式のファイルが得られるので、後は`openssl`を使って.pem形式に変換すればOKです。

# 3. クライアント証明書を用いたアクセス
業務でPythonを使っているので、HTTPリクエストには`requests`という定番ライブラリを用いています。ドキュメント読んだら想像以上にクライアント証明書を使ったアクセスが簡単でした。

```access.py
import requests
res = requests.post(url, cert="./certification.pem")
```

# まとめ
Windows端末が必須と思われる定常業務でも、その理由がわかれば自動化の対策が立てられます。
今回はSSLクライアント認証の設定ファイルを外だしする方法でした！

[justInCase](https://www.wantedly.com/companies/justincase)では自動化が大好きなあなたを求めてますので、ご興味があれば是非ご応募くださいね！以上、お役に立てば幸いです。

