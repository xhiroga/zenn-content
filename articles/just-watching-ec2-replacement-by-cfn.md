---
title: "CloudFormationで EC2 の`Update requires: Replacement` プロパティを更新した時の流れを追っただけ"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["EC2"]
published: true
---

CluodFormationで管理しているAmazon Linuxのバージョンを上げたかったので、ついでに挙動をチェックしました。



# CloudFormation Event
新しいインスタンスの作成を伴う更新の場合、まず新規にインスタンスを作成し、それが問題なければ古いインスタンスを削除する、という順番で動作するようですね。

```
| 2020-02-24 17:11:56 UTC+0900 | TestLinux | DELETE_COMPLETE | - |
| 2020-02-24 17:11:08 UTC+0900 | TestLinux | DELETE_IN_PROGRESS | - |
| 2020-02-24 17:11:03 UTC+0900 | TestLinux | UPDATE_COMPLETE | - |
| 2020-02-24 17:10:47 UTC+0900 | TestLinux | UPDATE_IN_PROGRESS | Resource creation Initiated |
| 2020-02-24 17:10:46 UTC+0900 | TestLinux | UPDATE_IN_PROGRESS | Requested update requires the creation of a new physical resource; hence creating one. |
```

# Resource

* Instance ID が新しくなったインスタンスが作成されました。
* 古い Instance ID のインスタンスはterminated状態になっています。


# Amazon Linux の中身

当たり前ですが、初期状態になっていました。

