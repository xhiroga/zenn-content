---
title: "IBM CloudでParentになっているユーザーを削除する方法"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ibmcloud"]
published: true
---
Parentになっているユーザーを削除しようとすると、次のエラーが表示されます。

```
Remove User:
User cloud not be removed. User cloud not be removed - You must disable or delete users first: ......
```

しかし、削除したいユーザーの子になっているユーザーを削除しなくても、子のユーザーのParentを別の親ユーザーに付け替えればOKです。
注意: 自分の親を変更することはできません。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/5d07f937-3acd-f866-66b0-096d51220b80.png)

