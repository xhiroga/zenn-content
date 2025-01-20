---
title: "AWSマネジメントコンソールのマルチセッションの仕様深堀り"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS"]
published: true
---

2025-01-16[^release-day]に、AWSマネジメントコンソールで複数アカウントによる同時サインインが可能になりました！
[^release-day]: 2025-01-10から段階的にリリースされていたようです。

従来はブラウザのプロファイルを使い分けるなどの工夫が必要だったので、とてもありがたいです。

## 公式ドキュメント

https://docs.aws.amazon.com/awsconsolehelpdocs/latest/gsg/multisession.html

## 仕様深堀り

### AWS SSOとの併用

アカウントメニューコンテンツ（アカウントメニューボタンをクリックすると表示されるメニュー）では表示されないものの、AWS SSOとも普通に併用できます。さらに、Organizationを跨いだ同時ログインも可能です。

![AWS Multi Session Buttons](/images/aws-multi-session-fig1.png)

SSOとの併用方法は[Serverworksさんのブログ](https://blog.serverworks.co.jp/sign-in-for-multiple-AWS-accounts)にスクショがあって分かりやすいです。

### サブドメインの仕様

マネジメントコンソールのサブドメインのURLは次の形式になっているようです。

`https://{アカウントID}-英数8文字.us-east-1.console.aws.amazon.com/...`

なお、クロスアカウントでスイッチロールした場合、`アカウントID`はスイッチ先のアカウントIDでした。

また、同じロールでセッションを出入りしても、`英数8文字`は不変でした。`Federated user名`→`英数8文字`に変換する何らかの関数があると思われます。

### アカウントエイリアスの表示

マルチセッションを有効化した場合、アカウントメニューボタンにはアカウントIDに加えてアカウントエイリアスが表示されます。紛らわしいのですが、アカウント名ではなくアカウントエイリアスです。違いは次の通り。

- アカウントエイリアス:
  - サインイン−ページのデフォルトURLのサブドメインとして用いられる英数字
  - [IAMから設定可能](https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/console-account-alias.html)
- アカウント名:
  - ルートユーザーでサインインした場合にアカウントメニューボタンに表示される名前
  - コストレポートの連結アカウントで表示される名前でもある
  - [Billing and Cost Managementから設定可能](https://repost.aws/ja/knowledge-center/change-organizations-name)

一方で、アカウントエイリアスはグローバルでユニークである必要があったり、アカウント作成時に一括で設定するものではなかったりするので、アカウント名に比べると利便性は落ちる印象です。

アカウント名や任意の文字列を表示したいという要望は根強いと思うので、今後のアップデートに期待したいですね。

### セッション数制限の画面をいつでも開く

同時ログインしているセッション数が5つを超えそうな場合は[Session limit reached](https://signin.aws.amazon.com/sessions/limit)にリダイレクトされます。実はこのページは、セッション数が5つ以内の場合も普通に開くことができます。

![Session limit (not) reached](/images/aws-multi-session-fig3.png)

何分前にログインしたセッションか？はここでしか見られない情報なので、URLを記録しておくと便利な場合が稀にあるかもしれないですね。

### その他

マルチセッションサポートを有効にすると、`aws-prism-opt-in`というCookieの値が`true`になるようです。

![Cookie](/images/aws-multi-session-fig2.png)

## 告知

複数のAWSのセッションを使い分ける上で、アカウントやロールを間違えないためのChrome拡張を開発しています。ヘッダーの色を変えられる他、アカウント名を表示することが可能です（マルチセッションサポートを現在対応中です）

![AWS Peacock Management Console](https://github.com/xhiroga/aws-peacock-management-console/blob/43709a5b0f1acb8354721a8acb9d3e224387dd8e/images/aws-peacock-mc.png?raw=true)

[Chrome Web Store](https://chromewebstore.google.com/detail/aws-peacock-management-co/bknjjajglapfhbdcfgmhgkgfomkkaidj?utm_source=zenn-dev_aws-multi-session)と[Firefox ADD-ONS](https://addons.mozilla.org/ja/firefox/addon/aws-peacock-management-console/)からご利用いただけます。

ソースコードもGitHubで公開しています。スターをいただけると励みになります！

https://github.com/xhiroga/aws-peacock-management-console
