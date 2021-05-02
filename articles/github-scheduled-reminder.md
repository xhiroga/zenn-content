---
title: "GitHubの scheduled reminders を設定して Pull Requestのレビュワー指定/ 再レビュー依頼/ Conflict等々をSlack通知する"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GitHub","Slack"]
published: true
---

みなさん、**[Scheduled Reminders](https://help.github.com/en/github/setting-up-and-managing-organizations-and-teams/managing-scheduled-reminders-for-pull-requests)** 使ってますか？

Scheduled Reminderとは, GitHubが Pull Pandaを買収して実現したGitHubのPR通知機能です。
[GitHub acquires Pull Panda—a better way to collaborate on code reviews](https://github.blog/2019-06-17-github-acquires-pull-panda/)

レビュワーのアサイン通知、approveやrequest change、conflictの発生などを**DMで**受け取れます。
（その点で、リポジトリを subscribeしてチャンネルに投稿するGitHubのSlack Botとは異なります）

# 手順

（私の環境では既にインストール済みなので、初めてインストールする人と画面上の差異があります）

1. [Settings > Reminders](https://github.com/settings/reminders) から、リマインダーを受け取りたいOrganizationを選択します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/9c864354-353d-9049-8a4b-39825258ee8a.png)

2. Slackへのアクセスを承認します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/de9d8667-e8fc-dbcc-21b1-5a03945643f9.png)

3. リマインダーを設定します。このとき、`Enable real-time alerts` を設定すると、レビュワー指定を受けた瞬間に通知が来るようになります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/0ecd553f-90f3-5d32-2a17-7e46bc2fde98.png)

ちなみに、 `You are assigned a review` は レビューの再リクエスト♻️にも対応していました。

以上です。


