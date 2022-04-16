---
title: "HerokuのOAuthユーザートークン流出で、やっておくといいことリスト（コメント大歓迎）"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Heroku", "GitHub"]
published: true
---

日本時間の2022年4月16日にアナウンスがあった、Heroku/Travis-CIのOAuthユーザートークンの流出および悪用を受けて、ユーザーとしてやっておくといいことをまとめました。  
GitHubのOrganizationのオーナー向けと個人向けとに分けてあります。

> 追記: 複数の[補足のコメント](https://zenn.dev/link/comments/5639edd17311a9)を頂き、記事にも取り込んでいます。ありがとうございます！


## 注意

- 執筆者はGitHub, Heroku, Travis-CIの専門家ではありません。この記事は誤っている可能性があります。
- この記事は現在調査中の問題について書かれています。最新情報は必ず公式サイトをご確認ください。
    - [GitHub](https://github.blog/2022-04-15-security-alert-stolen-oauth-user-tokens/)
    - [Heroku](https://status.heroku.com/incidents/2413)
    - 2022-04-16T19:00:00JST 時点ではTravis-CIからの公式声明はなし。


## インシデントの概要

GitHubがHerokuとTravis-CIのOAuthアプリケーションに発行したトークンが流出・悪用したことで、それらの連携が有効だった多くのOrganizationからプライベートリポジトリが閲覧・ダウンロードされる自体が発生しました。npmも被害に遭ったOrganizationの一つです。  
2022年4月12日にGitHub社のセキュリティチームが調査を開始し、4月13,14日にはGitHubからHeroku/Travis-CIに対して通達が行われました。
[日本時間の4月16日にはHerokuから声明が出され](https://status.heroku.com/incidents/2413)、Heroku側でOAUthアプリケーションのトークンを無効化することが発表されています（声明どおりなら4月16日中に無効化が行われます。）  
また、Travis-CIのアプリケーションのトークンも、[同社従業員と思われるユーザーによって無効化が進められていると思われます](https://twitter.com/sf_tristanb/status/1515117868484071425)。

GitHubは現在、影響を受けたユーザーと組織の特定と通知を行っているようです。もし被害に遭っていた場合、4/19の朝8時頃までに通知メール等を受信すると思われます。


## この記事での対応方針

上記の通りHeroku/Travis-CI側でも対応が進んでおりますが、ユーザーとしての最低限の自衛を方針にしています。具体的には以下の通りです。

- Heroku DashboardとGitHubを接続している場合、権限を取り消す
- 過去にHeroku DashboardとGitHubを接続していないかを確認する

なお、上記対応は[Herokuのブログ](https://status.heroku.com/incidents/2413)を参考に作成いたしました。

## Organizationのオーナー向け

以下の3つのアクションを行います。
1. 現在Organizationのリポジトリにアクセス可能なOAuthアプリケーションの検索
2. 過去にOrganizationでアクセス権限がApproveされたOAuthアプリケーションの検索
3. 監査ログの調査

### 1. 現在Organizationのリポジトリにアクセス可能なOAuthアプリケーションの検索

以下のURLの{organization_id}をご自身の管理するOrganizationの値に置き換えてアクセスし、`Third-party application access policy` を確認します。

```
https://github.com/organizations/{organization_id}/settings/oauth_application_policy
```

![](/images/2022-04-16_heroku-incident-2413-checklist_1.png)

この中に Heroku Dashboard/Travis-CIがなければ、現時点で無効化が必要なアプリケーションはないと私は判断しています。

### 2. 過去にOrganizationでアクセス権限がApproveされたOAuthアプリケーションの検索

以下のURLの{organization_id}をご自身の管理するOrganizationの値に置き換えてアクセスし、`Audit log` を確認します。

```
https://github.com/organizations/{organization_id}/settings/audit-log?q=action%3Aorg.oauth_app_access_approved
```

Filter欄に `action:org.oauth_app_access_approved` と入力します。すると、過去にOrganizationでアクセス権限がApproveされたアプリケーションが表示されます。

![](/images/2022-04-16_heroku-incident-2413-checklist_2.png)

この中にHeroku Dashboard/Travis-CIがなければ、Organizationは一応今回のインシデントの対象ではないと私は判断しています。ただし個人側のSecurity Logでは恐らく古いイベントが検索結果に表示されなかったこともあるので、監査ログと併用したりメンバー側にも確認を依頼するのが確実かもしれません。

### 3. 監査ログの調査

`Audit log` の画面から監査ログをExportします。

![](/images/2022-04-16_heroku-incident-2413-checklist_3.png)

監査ログには アクション, 作業者, 作業者のアクセス元の国, 日時などの情報が含まれています。  
不審なアクティビティがないか見てみてもよいのではないでしょうか。


## 個人向け

以下の4つのアクションを行います。
1. 現在リポジトリにアクセス可能なOAuthアプリケーションの検索
2. 過去にアクセス権限がApproveされたOAuthアプリケーションの検索
3. 監査ログの調査
4. 管理者への報告


### 1. 現在リポジトリにアクセス可能なOAuthアプリケーションの検索

個人の方の場合、[`Applications` > `Authorized OAuth Apps`](https://github.com/settings/applications)から確認します。

![](/images/2022-04-16_heroku-incident-2413-checklist_4.png)

Heroku Dashboard/Travis-CIがあったら、クリックして開きます。

![](/images/2022-04-16_heroku-incident-2413-checklist_5.png)

`Organization Access`でアクセスが許可されていた場合、Organizationの管理者に報告すると良いと思います。

アクセスは `Revoke Access`から取り消すことができます。

![](/images/2022-04-16_heroku-incident-2413-checklist_6.png)


### 2. 過去にアクセス権限がApproveされたOAuthアプリケーションの検索

[Security Log で Filtersに `action:oauth_authorization.create` と入力すると](https://github.com/settings/security-log?q=action%3Aoauth_authorization.create)、過去にアクセスを許可したOAuth Applicationが表示できるようです。ただし、昔すぎるイベントは結果に出ないかもしれません（この辺詳しくありません）。

また、[同様にSecurity Log で Filtersに `action:oauth_authorization.destroy` と入力すると](https://github.com/settings/security-log?q=action%3Aoauth_authorization.destroy)、アクセス権限を取り消したOAuth Applicationを表示できます。手順1. でHeroku Dashboardのアクセス権限を取り消している場合、ここに表示されると思います。

### 3. 監査ログの調査

[`Security Log`](https://github.com/settings/security-log)から監査ログをExportします。  
アクセス元の国なども分かるので、覚えのないアクセスがないか見てみても良いのではないでしょうか。


### 4. 管理者への報告

個人の方でOrganizationに所属している方は、必要に応じて以下を管理者に報告すると良いと思います。

1. Heroku Dashboard/Travis-CIのアクセスが許可されていたか
2. Heroku Dashboard/Travis-CIのアクセスが許可されていた場合、対象のOrganizationへのアクセスは許可されていたか
3. Heroku Dashboard/Travis-CIのアクセスが許可されていた場合、アクセスを取り消したか
4. 過去にHeroku Dashboard/Travis-CIにアクセスを許可したり、削除した履歴があるか（ある場合はその期間）


## まとめ

GitHubのOrganization管理者と個人向けに、今回のインシデントで行うべきことをまとめました。  
繰り返しますが、筆者は専門家ではなく単なるWebアプリケーション開発者です。よりよい知見をお持ちの方のコメントをお待ちしております。

## 参考

- [Security alert: Attack campaign involving stolen OAuth user tokens issued to two third\-party integrators \| The GitHub Blog](https://github.blog/2022-04-15-security-alert-stolen-oauth-user-tokens/)
- [Incident 2413 \| Heroku Status](https://status.heroku.com/incidents/2413)
