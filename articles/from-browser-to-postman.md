---
title: "WebアプリのリクエストをPostmanで再現する"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## TL;DR

ブラウザの検証モードから API リクエストを cURL 形式でコピーし、Postman に RawText として Import すると便利。

## How to

1. ブラウザの Inspector(検証モード)を起動し、Network タブを開きます。
2. 記録したい API リクエストを実行し、そのリクエストを右クリック > Copy > Copy as cURL
3. Postman で Import > Raw Text を選択し、cURL 形式のテキストを貼りつけ。
