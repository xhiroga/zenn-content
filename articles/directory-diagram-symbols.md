---
title: "ディレクトリ構成図を描くときに便利な記号（┐┘┌└） 入力方法3選"
emoji: "🔖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["macOS","unicode"]
published: true
---

以下のような図を描きたい。

```md
/
├ bin
├ boot
...
└ var
```

すでにQiitaに[ディレクトリ構成図を書くときに便利な記号](https://qiita.com/paty-fakename/items/c82ed27b4070feeceff6)という素晴らしい記事がありますが、より自分好みの方法を考えたので記事にした次第です。


また、macOSの場合です。

## TL;DR

- 英語入力なら Ctrl + Cmd + Space で Emoji and Symbols Viewer を表示し、 box と入力して候補に出すのが最も早い
- かな入力の場合、ショートカットを登録するかGoogle日本語入力で変換する
- Unicode Hex Inputでも入力できる

## 記号について

正式名称を[罫線素片](https://ja.wikipedia.org/wiki/%E7%BD%AB%E7%B7%9A%E7%B4%A0%E7%89%87)といい、文字のみで罫線のあるアプリケーションをデザインしたかった時代に登場した記号とのこと。

## Emoji and Symbols Viewer による入力

Cmd + Ctrl + Space で 絵文字入力のためのウィンドウを開き、boxで表示できます（英語で利用している場合）

![Input box drawing characters from emoji and symbols viewer](/images/box-drawing-characters.gif)


## ショートカットの設定またはGoogle日本語入力

Macの標準日本語入力・Google日本語入力のいずれでも、以下の通り変換することが出来ます。

| 読み | 文字 |
| --- | --- |
| みぎした／けいせん | ┘ |
| みぎうえ／けいせん | ┐ |
| ひだりうえ／けいせん | ┌ |
| ひだりした／けいせん | └ |

そのほかは [ディレクトリ構成図を書くときに便利な記号](https://qiita.com/paty-fakename/items/c82ed27b4070feeceff6) をご覧ください。

## Unicode Hex Input

macOSのシステム環境設定のキーボードから設定できる、Unicodeの入力用キーボードです。

![Unicode Hex Input](2021-12-25-unicode-hex-input.png)

以下のYouTubeの動画がわかりやすいので、ご参考まで。

[Typing Special Characters Using Unicode On Your Mac \- YouTube](https://www.youtube.com/watch?v=ShOJYrZaMJU)


## まとめ

日本語入力で変換するのが覚えやすいが、ディレクトリ構成図を描いているときは英語入力のはず。Emoji and Symbols Viewerから入力すると効率が上がるかもしれません。
