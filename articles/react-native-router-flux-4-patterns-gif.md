---
title: "react-native-router-flux の画面遷移4パターン@GIF画像"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react-native","react-native-router-flux"]
published: true
---
`Actions`で提供されているアニメーションから4パターンをGIF画像でまとめました。

# Actions.pop
`Actions.popTo`も同様です。
![20181027_155107.GIF](https://qiita-image-store.s3.amazonaws.com/0/96286/1007fa70-96d5-0057-dd94-3e20b24b62ef.gif)

現在のページは右手に退きつつ、新しいページが現在のページの左側から登場します。

# Actions.push
key名をメソッドに指定した場合もこれ。
![20181027_155558.GIF](https://qiita-image-store.s3.amazonaws.com/0/96286/ce7c2311-61d0-937f-d009-9d15f23c6bf6.gif)
`Actions.jump`と同様ですね。  
現在のページが左手に退きつつ、新しいページが右から重なってきます。

# Actions.replace
![20181027_155634.GIF](https://qiita-image-store.s3.amazonaws.com/0/96286/c50ebfaa-940b-40df-b8e5-45c191358a84.gif)
アニメーションなしでいきなりページが切り替わります。

# Actions.reset
![20181027_155700.GIF](https://qiita-image-store.s3.amazonaws.com/0/96286/a6a3e04f-46d9-47db-7585-42158a2a5830.gif)
`Actions.replace`とアニメーションは変わりません。違いはルーティングスタックに対する処理だけなので、画面を見ている人からすると変わらないです。


以上、お役に立てば幸いです。

