---
title: "業務でDBeaverを利用する上でのオススメ設定"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["DBeaver"]
published: true
---
業務でDBeaverを利用しはじめて半年程度です。便利に使っています。
設定を見直したメモです。


# Edit Connection
Connectionごとの設定です。

## Auto-commitを無効化
Auto-commitを無効化
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/3f2676a4-0161-7862-afa5-d8cdab73ded6.png)

## Isolation Levelを適切に設定
私の場合、普段は `Read Commit` に設定しています。手動でDBを更新することがないため、常にアプリケーションの一連の処理が終わった状態を参照したいからです。
メンテナンスなどで手動で更新する必要がある場合のみ、 `Serializable` に設定します。メンテナンスでは全件取得 → WHERE句が同じ状態で UPDARE のような手順を取ることがあり、その間に行数が増減するのを防ぐためです。

## Connection type
環境ごとに適切に設定します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/dd579289-4855-0941-9078-824a7536ddc1.png)

## Security
影響の大きい環境では `Read-only connection` に設定します。こうすることで、UPDATEやDELETEの誤爆を防げます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/382a4ccd-6254-06d1-2295-0670d2373c24.png)


# Preference
全体で共通の設定です。

## Preference > DBeaver > Connection Types
普段からトランザクションに慣れておきたいので、Development環境でも `Auto-commit by default` はOFFにしています。
その代わり、`Confirm data changes` はDevelopmentではオフにします。どちらもオンだと、DevelopmentとProductionの間違えに気がつくきっかけが減ってしまうと考えているためです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/ffa72128-b040-fe5c-5ec4-7a98f0d78bfe.png)

## Preference > DBeaver > Editors > SQL Editor 
`Connect on editor activation` をオフにして、 `.sql` ファイルを開いた際の接続先を明示的に指定するようにします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/7d41dc43-b0c4-1e4d-d2c7-a8eeeb9ca0b9.png)


## Preference > DBeaver > Transactions
`Auto Commit` をオフにしていれば無意味なのですが、万一 Auto Commitがデフォルトでオンになっていた場合に備えて `Return to auto-commit on transaction end` を無効化します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/44df405a-00f2-61ab-6688-04914f870d1c.png)

# JVM起動時のオプション

## Timezone を UTCに変更

`/Applications/DBeaver.app/Contents/Eclipse/dbeaver.ini` に `-Duser.timezone=GMT` オプションを追加します。時刻が勝手にJSTで表示されるのはハマるのでおすすめできません。

```dbeaver.ini
-vm
../Eclipse/jre/Contents/Home/bin/java
(省略)
-Xms64m
-Xmx1024m
-XstartOnFirstThread
-Duser.timezone=GMT

```



# その他

ショートカット読み込んでおくと業務効率が上がりそうですね。
素晴らしいまとめがあったのでリンクをご紹介。
https://wonwon-eater.com/dbeaver-shortcut


