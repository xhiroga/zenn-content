---
title: "HerokuのOAuthトークン流出で、やっておくといいことリスト（コメント大歓迎）"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Heroku", "Travis-CI", "GitHub"]
published: true
---

2022年4月16日(日本時間)にアナウンスがあった、Heroku/Travis-CIのOAuthトークンの流出および悪用を受けて、ユーザーとしてやっておくといいことをまとめました。  
GitHubのOrganizationのオーナー向けと個人向けで分けてあります。

> 追記: 複数の[補足のコメント](https://zenn.dev/link/comments/5639edd17311a9)を頂き、記事にも取り込んでいます。ありがとうございます！


## 注意

- 執筆者はGitHub, Heroku, Travis-CIの専門家ではありません。この記事は誤っている可能性があります。
- この記事は現在調査中の問題について書かれています。最新情報は必ず公式サイトをご確認ください。
    - [GitHub](https://github.blog/2022-04-15-security-alert-stolen-oauth-user-tokens/)
    - [Heroku](https://status.heroku.com/incidents/2413)
    - [Travis CI](https://blog.travis-ci.com/2022-04-17-securitybulletin)


## インシデントの概要

GitHubがHerokuとTravis-CIのOAuthアプリケーションに発行したトークンが流出・悪用したことで、それらの連携が有効だった多くのOrganizationからプライベートリポジトリが閲覧・ダウンロードされる自体が発生しました。npmも被害に遭ったOrganizationの一つです。

時系列順にイベントをまとめました。

| 日付（UTC） | イベント | Source |
| --- | --- | --- |
| 日付不明 | 攻撃者がTravis-CIのOAuthトークンを用いた攻撃を行う |  |
| 2022-04-09 | 攻撃者がHerokuのOAuthトークンを用いてプライベートリポジトリを大量にダウンロード（それ以外の日付での攻撃の有無は不明） | [Incident 2413 \| Heroku Status](https://status.heroku.com/incidents/2413) |
| 2022-04-12 | GitHubのセキュリティチームが調査を開始 | [Security alert \| The GitHub Blog](https://github.blog/2022-04-15-security-alert-stolen-oauth-user-tokens/) |
| 2022-04-13, 14 | GitHubからHeroku, Travis-CIに対して通知 |  [Security alert \| The GitHub Blog](https://github.blog/2022-04-15-security-alert-stolen-oauth-user-tokens/) |
| 2022-04-13~ | Herokuが攻撃に用いられたOAuthトークンを無効化 | [Incident 2413 \| Heroku Status](https://status.heroku.com/incidents/2413) |
| 2022-04-15 | GItHubのセキュリティチームがブログを公開 | [Security alert \| The GitHub Blog](https://github.blog/2022-04-15-security-alert-stolen-oauth-user-tokens/) |
| 2022-04-15 | Herokuがブログを公開 | [Incident 2413 \| Heroku Status](https://status.heroku.com/incidents/2413) |
| 2022-04-15 | Travis-CIの従業員らしきユーザーがOAuthトークンを無効化している報告がツイッターに上げられる | [Twitter](https://twitter.com/sf_tristanb/status/1515117868484071425) |
| 2022-04-16 | SalesforceのBotがHeroku Dasuboardの全てのOAuthトークンの無効化を始めたと思われる報告がツイッターに上げられる | [Twitter](https://twitter.com/kn1cht/status/1515354344522399744) |
| 2022-04-17 | 全てのHeroku DashboardのOAuthトークンの無効化が完了 | [Incident 2413 \| Heroku Status](https://status.heroku.com/incidents/2413) |
| 2022-04-18 | Travis-CIがブログを公開 | [The Travis CI Blog](https://blog.travis-ci.com/2022-04-17-securitybulletin) |
| 2022-04-18 | GitHubが被害を受けたユーザーに対しての通知を実施（日本時間の4/19 5:30AM） | [Security alert \| The GitHub Blog](https://github.blog/2022-04-15-security-alert-stolen-oauth-user-tokens/) |

## インシデントの影響


### 所属するOrganizationまたは自分自身が攻撃を受けた場合の影響

プライベートリポジトリ内にクレデンシャルなどがコミットされていた場合、悪用される恐れがあります。アカウントの無効化やクレデンシャルのローテーションなどが必要になるでしょう。


### npmが攻撃を受けたことによる影響

npmのOrganizationのプライベートリポジトリにコミットされてたAWSクレデンシャルを用いた、npmのS3へのアクセスが行われました。2022-04-17時点ではパッケージの変更やユーザーアカウント・クレデンシャルへのアクセスは確認されていないそうです。また、プライベートパッケージへのアクセスの有無は調査中です。詳細は[GitHubのブログ](https://github.blog/2022-04-15-security-alert-stolen-oauth-user-tokens/)を参照下さい。


### OAuthトークンの無効化による影響

Heroku DashboardのGitHub統合の全てのOAuthトークンが失効させられる方針となったため、Heroku Dashboardを用いたHerokuへのアプリケーションのデプロイはできなくなります。  
GitHubのリポジトリをHerokuにデプロイするための方法は他にもあるので、[公式の案内](https://status.heroku.com/incidents/2413)を参考にしてください。

Travis-CIでも類似の影響があるかもしれません。[公式の案内](https://blog.travis-ci.com/2022-04-17-securitybulletin)を参考にして下さい。


## やったほうがいいこと

GitHubによる攻撃対象者への連絡とHeroku/Travis-CIによるOAuthトークンの無効化が進められていますが、私達自身でやったほうがいいこともあるのでご紹介します。

- メールの確認
- (まだの場合)Heroku Dashboard/Travis-CIのOAuthトークンの無効化
- 攻撃を受けたどうかの確認
- 関係者への連絡


## Organizationのオーナーがやったほうがいいこと

1. GitHub/Heroku/Travis-CIからのメールの確認
2. Third-party application access policyの確認
3. (まだの場合)Heroku Dashboard/Travis-CIのOAuthトークンの無効化
4. Audit logの調査
5. 関係者への連絡

### 1. GitHub/Heroku/Travis-CIからのメールの確認

自身が攻撃対象であったか、必要なアクションなどの連絡が来ている場合、確認して従いましょう。

### 2. Third-party application access policyの確認

以下URL(`org_id`を自身のOrganizationで置換)にアクセスし、`Third-party application access policy`が `Access restricted` になっていることを確認します。

`https://github.com/organizations/{org_id}/settings/oauth_application_policy`

![](/images/2022-04-16_heroku-incident-2413-checklist_7.png)

`Access restricted` になっていない場合、組織のメンバーがインストールしたアプリが組織のプライベートリポジトリに制限なくアクセスできることを表します。この機会に制限したほうがいいかもしれません。

詳細は次のドキュメントを参考にして下さい。  
[About OAuth App access restrictions \- GitHub Docs](https://docs.github.com/en/organizations/restricting-access-to-your-organizations-data/about-oauth-app-access-restrictions)

> [@azuさんのコメント](https://zenn.dev/link/comments/9eba952cb95414)を元に加筆しました。ありがとうございました！


### 3. (まだの場合)Heroku Dashboard/Travis-CIのOAuthトークンの無効化

以下URL(`org_id`を自身のOrganizationで置換)にアクセスし、`Third-party application access policy` を確認します。

`https://github.com/organizations/{org_id}/settings/oauth_application_policy`

![](/images/2022-04-16_heroku-incident-2413-checklist_1.png)

この中に Heroku Dashboard/Travis-CIがなければ、現時点で無効化が必要なアプリケーションはないと私は判断しています。

### 4. Audit logの調査

以下URL(`org_id`を自身のOrganizationで置換)にアクセスし、`Audit log` を確認します。

`https://github.com/organizations/{org_id}/settings/audit-log?q=action%3Aorg.oauth_app_access_approved`

github.com 上で簡単に検索した後、ログをExportして詳細な調査を行いましょう。


#### github.com 上で検索

> 注意: 経験上、github.com 上では昔のイベントが表示されない可能性があります。詳細な調査はログをExportして行いましょう。

Filter欄に `action:org.oauth_app_access_approved` と入力します。すると、過去にOrganizationでアクセスがApproveされたアプリケーションが表示されます。

![](/images/2022-04-16_heroku-incident-2413-checklist_2.png)

この中にHeroku Dashboard/Travis-CIがなければ、少なくとも直近の攻撃の対象ではなさそうだ、と言えるかもしれません。

また、Filter欄に `action:repo.download_zip` と入力することで、リポジトリのダウンロードに関わるイベントが表示されます。このイベントは正常なアプリケーションを利用していても発生するので、以下の点に注意して不審なイベントを見極めると良いと思います。  

- 国や時間、頻度
- 連携しているアプリケーション側のログと整合するか


#### Exportによる調査

`Audit log` の画面から、JSONまたはCSVとしてログをExportすることができます。

![](/images/2022-04-16_heroku-incident-2413-checklist_3.png)

上述の通り、国や時間、頻度、連携しているアプリケーションのログとの整合性などを見ると良いと思います。`action:org.oauth_app_access_approved` と `action:repo.download_zip` 以外でも、重要なイベントは確認すると良さそうです。

その他のactionは[こちら](https://docs.github.com/ja/github-ae@latest/admin/monitoring-activity-in-your-enterprise/reviewing-audit-logs-for-your-enterprise/searching-the-audit-log-for-your-enterprise)からご覧いただけます。

### 5. 関係者への連絡

攻撃の証拠を見つけた場合、[Herokuであればサポートに問い合わせるよう案内が出ています](https://status.heroku.com/incidents/2413)。


## 個人がやったほうがいいこと

1. GitHub/Heroku/Travis-CIからのメールの確認
2. (まだの場合)Heroku Dashboard/Travis-CIのOAuthトークンの無効化
3. Security logの調査
4. 関係者への連絡


### 1. GitHub/Heroku/Travis-CIからのメールの確認

自身が攻撃対象であったか、必要なアクションなどの連絡が来ている場合、確認して従いましょう。

### 2. (まだの場合)Heroku Dashboard/Travis-CIのOAuthトークンの無効化

[`Applications` > `Authorized OAuth Apps`](https://github.com/settings/applications)からOAuthアプリケーションを確認します。

![](/images/2022-04-16_heroku-incident-2413-checklist_4.png)

Heroku Dashboard/Travis-CIがあったら、クリックして開きます。
`Organization Access`が許可されていた場合、Organizationのオーナーに報告しましょう。

![](/images/2022-04-16_heroku-incident-2413-checklist_5.png)

まだアクセスがApproveされている場合 `Revoke Access`から取り消すことができます。  

![](/images/2022-04-16_heroku-incident-2413-checklist_6.png)


### 3. Security logの調査

[Security log](https://github.com/settings/security-log) を確認します。  
github.com 上で簡単に検索した後、ログをExportして詳細な調査を行いましょう。

#### github.com 上で検索

> 注意: 経験上、github.com 上では昔のイベントが表示されない可能性があります。詳細な調査はログをExportして行いましょう。

[Security Log で Filtersに `action:oauth_authorization.create` と入力すると](https://github.com/settings/security-log?q=action%3Aoauth_authorization.create)、過去にアクセスを許可したOAuth Applicationが表示できます。

また、[同様にSecurity Log で Filtersに `action:oauth_authorization.destroy` と入力すると](https://github.com/settings/security-log?q=action%3Aoauth_authorization.destroy)、アクセス権限を取り消したOAuth Applicationを表示できます。

それ以外の種類のactionは[こちら](https://docs.github.com/ja/enterprise-server@3.4/authentication/keeping-your-account-and-data-secure/reviewing-your-security-log)からご覧いただけます。

#### Exportによる調査

[`Security Log`](https://github.com/settings/security-log)から監査ログをExportします。  
国や時間、頻度などから、不審なイベントがないかを見ると良いと思います。


### 4. 関係者への連絡

個人の方でOrganizationに所属している方は、必要に応じて以下を管理者に報告すると良いと思います。

1. Heroku Dashboard/Travis-CIのアクセスが許可されていたか
2. Heroku Dashboard/Travis-CIのアクセスが許可されていた場合、対象のOrganizationへのアクセスは許可されていたか
3. Heroku Dashboard/Travis-CIのアクセスが許可されていた場合、アクセスを取り消したか
4. 過去にHeroku Dashboard/Travis-CIにアクセスを許可したり、削除した履歴があるか（ある場合はその期間）

また、攻撃の証拠を見つけた場合、[Herokuであればサポートに問い合わせるよう案内が出ています](https://status.heroku.com/incidents/2413)。

## まとめ

GitHubのOrganizationのオーナーと個人向けに、今回のインシデントで行うべきことをまとめました。  
繰り返しますが、筆者は専門家ではなく単なるWebアプリケーション開発者です。よりよい知見をお持ちの方のコメントをお待ちしております。

## 参考

- [Security alert: Attack campaign involving stolen OAuth user tokens issued to two third\-party integrators \| The GitHub Blog](https://github.blog/2022-04-15-security-alert-stolen-oauth-user-tokens/)
- [Incident 2413 \| Heroku Status](https://status.heroku.com/incidents/2413)
- [The Travis CI Blog: SECURITY BULLETIN; Certain private customer repositories may have been accessed](https://blog.travis-ci.com/2022-04-17-securitybulletin)
- [Searching the audit log for your enterprise \- GitHub Docs](https://docs.github.com/ja/github-ae@latest/admin/monitoring-activity-in-your-enterprise/reviewing-audit-logs-for-your-enterprise/searching-the-audit-log-for-your-enterprise)
- [セキュリティログをレビューする \- GitHub Docs](https://docs.github.com/ja/enterprise-server@3.4/authentication/keeping-your-account-and-data-secure/reviewing-your-security-log)
