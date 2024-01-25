---
title: "AWS SESのサプレッションリストを全表示・サマリー・削除するCLIを作った"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "go"]
published: true
---

AWS SESのアカウントレベルのサプレッションリストを全表示・サマリー・削除するCLIを作りました。

## TL;DR

- AWS SESのアカウントレベルのサプレッションリストは、AWSのコンソールからは10件づつしか表示できない
- AWS CLIには、SESのアカウントレベルのサプレッションリストを全削除する機能はない
- AWS NukeにもSESのアカウントレベルのサプレッションリストを削除する機能はない
- GoでCLIを自作した

## AWS SESのアカウントレベルのサプレッションリストについて

アカウントレベルのサプレッションリストとは、AWS SESからメールを送信する際、過去にハードバウンスや苦情を受けたメールアドレスを登録しておくことで、そのメールアドレスに対してメールを送信しないようにする機能です。

今回、AWSアカウントから個人情報を全て削除する要望があり、その一環としてサプレッションリストに登録されたメールアドレスの全削除を行いました。サプレッションリストはアカウントのメール送信機能を一時停止すれば90日後に自動的に削除しますが、今回は期限が定められていたため、手動での削除になりました。

## サプレッションリストの全削除

AWSの管理コンソールですが、サプレッションリストの削除は一度に10件しかできません。

また、AWS CLIには、SESのアカウントレベルのサプレッションリストを全削除する機能はありません。[^sesv2]

[^sesv2]: [sesv2](https://docs.aws.amazon.com/cli/latest/reference/sesv2/delete-suppressed-destination.html)

さらに、AWS NukeにもSESのアカウントレベルのサプレッションリストを削除する機能はありませんでした。[^aws-nuke]

[^aws-nuke]: [rebuy-de/aws-nuke](https://github.com/rebuy-de/aws-nuke)

## [aws-ses-suppression-list](https://github.com/xhiroga/aws-ses-suppression-list)の使い方

サプレッションリストの全件表示、サマリー、削除を行うCLIをGo言語で実装しました。

https://github.com/xhiroga/aws-ses-suppression-list

使用時のログは次のとおりです（イメージ）

```terminal
% go run main.go summary
Reason, Count
BOUNCE, 987
TOTAL, 987

% go run main.go deleteAll
You are about to delete the following suppressed email destinations:
Reason, Count
BOUNCE, 987
TOTAL, 987
Do you really want to delete all suppressed email destinations? This action cannot be undone.
If you want to continue, type 'delete' to proceed: delete
Deleting suppressed email destinations:
Progress: 654/987
```

実装した感想ですが、`Cursor`でコーディングしたのが非常に楽でした。特にPythonの`tqdm`のような挙動をGo言語で簡単に実装できるのを知らなかったのですが、ダメ元で`Cursor`にお願いしたら数行でやってくれて驚きました。

CLIを利用した感想としては、AWSの管理コンソールで10件づつ削除するよりも、一度実行すれば待つだけでよいので楽ですね。

## まとめ

AWS SESのアカウントレベルのサプレッションリストを全表示・サマリー・削除するCLIを作りました。ぜひ使ってみてください。
