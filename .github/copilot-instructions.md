# DIRECTION

@hirogaのZenn.devの記事を書く上での注意事項です。

## 前提

- 記事内で指定がない限り、一般的なソフトウェアエンジニアが対象となる読者です。

## Page Structure

ページ構成の一例を次の通り示します。

### トラブルシュート

```md

## TL;DR

## 動機

## すること

## やり方

## まとめ
```

### 失敗談

```md
## TL;DR

## 経緯

## 再現手順

## 再発防止

## まとめ
```

## Sentence Structure

### ハルシネーションの予防

主張を裏付ける品質の高い資料がない場合、その旨をコメントしてください。

#### Bad

```md
Pythonのパッケージのフォーマットであるwheelは、モンティパイソンのコントが由来になっています。
```

#### Good

```md
Pythonのパッケージのフォーマットであるwheelは、モンティパイソンのコントが由来になっています。
<!-- TODO: 資料が必要 -->
```

#### Very Good

```md
Pythonのパッケージのフォーマットであるwheelは、モンティパイソンのコントが由来になっています。[^python_2021]
[^python_2021]: <https://discuss.python.org/t/where-the-name-wheel-comes-from/6708>
```

### 曖昧より狭くても具体的に

#### Bad

```md
pip indexコマンドは比較的新しい機能で、2021年に実装されました。そのため、一部の環境では利用できない可能性があります。
```

#### Good

```md
pip indexコマンドは比較的新しい機能で、2021年に実装されました。x.y.z未満のバージョンでは利用できません。
<!-- TODO: 調査が必要 -->
```

#### Very Good

```md
pip indexコマンドは比較的新しい機能で、2021年に実装されました。21.2未満のバージョンでは利用できません。[^pip_changelog_v21_2]
[^pip_changelog_v21_2]: <https://pip.pypa.io/en/stable/news/#v21-2>
```

### 百聞は一見にしかず

#### Bad

```md
pip index versionsは、インデックスサーバー上のパッケージ情報を表示するコマンド・サブコマンドです。
```

#### Good

```md
pip index versionsは、インデックスサーバー上のパッケージ情報を表示するコマンド・サブコマンドです。
例えば、次のコマンドを実行してみてください。

pip index versions torch
```

### 文体

敬体（です・ます）・常体（だ・である）は統一します。特にDraftで指示がない場合は敬体を使います。

### 文の長さ

文が短くなるよう、分けられる文は分けます。

#### Bad

```md
私はProxmoxを運用していますが、容量が気になるので、バックアップをDiskstationに保存しています。
```

#### Good

```md
私はProxmoxを運用しています。容量が気になるので、バックアップをDiskstationに保存しています。
```

### 漢語と和語

親しみやすい文章になるよう、漢語と同じ意味の和語があればそちらを使います。

#### Bad

- 遅延を測定します。
- 同様の問題に直面した方々の助けになればと思います。

#### Good

- 遅延を測ります。
- 同じ問題で困っている人の助けになればと思います。

### カギ括弧

書籍・単行本・雑誌・新聞・Webサイト・映画・CDアルバム等の名前・タイトルの固有名詞には、「」ではなく『』を用いてください。

## Zenn Markdown Syntax

### Frontmatter / Header 1

FrontmatterがあるのでHeader 1は不要です。

### 太字

Zenn.devでは、太字の**に続けて空白が必要です（ただし引用は続けて良い）

#### Bad

```md
続けて、**PyTorch**をインストールします。
```

#### Good

```md
続けて、**PyTorch** をインストールします。
```

### コメントアウト

Zenn.devは複数行のコメントアウトに対応していません。1つの行にまとめましょう。

#### Bad*

```md
<!-- 
コメント1
コメント2
 -->
```

#### Good

```md
<!-- コメント1, コメント2 -->
```

### コードブロック

- ターミナルのコードブロックのLanguage Identifierには`shell`を利用します。また、プロンプトの記号は`$`に統一します。
- ターミナルの出力に個人情報やクレデンシャルが残っている場合、その部分を元の文字数に関わらず`******`(*6つ)に置換します。
- ただし、ブログ執筆者のユーザー名が`hiroga`であることは公開情報なので、隠さなくてよいです。

#### Bad

```terminal
hiroga@hiroga-air zenn-content % pwd
/Users/hiroga/Documents/GitHub/zenn-content
```

#### Good

```shell
$ pwd
/Users/hiroga/Documents/GitHub/zenn-content
```

### Log

ターミナルの出力などのログは、アコーディオンで残してください。

#### Good

:::details pip index versions torchの出力

```shell
pip index versions torch
WARNING: pip index is currently an experimental command. It may be removed/changed in a future release without prior warning.
torch (2.5.0)
Available versions: 2.5.0, 2.4.1, 2.4.0, 2.3.1, 2.3.0, 2.2.2, 2.2.1, 2.2.0, 2.1.2, 2.1.1, 2.1.0, 2.0.1, 2.0.0
  INSTALLED: 2.2.0
  LATEST:    2.5.0
```

:::

## Meta notes

Reviewer向けのnotesをFrontmatterに書いてください。

### Topics

Zenn.dev において、特に機械学習の分野では次のようなタグが使われています。表記揺れ防止のため、予め次の通り示しておきます。

#### Good

- machinelearning
- deeplearning
- computervision
- cg
- AI
- LLM
- 論文

#### Bad

- 機械学習: machinelearning に統一
- 深層学習: deeplearning に統一
- ディープラーニング: deeplearning に統一
- CV: computervision とは別Topicとして扱われてしまう

### Self Review

書いていて不足を感じる点はself reviewとして残して下さい。

### Slug

slugの候補を提案してください。slugとは、URLの次の部分です。

`https://zenn.dev/ユーザー名/articles/{slug}`

条件は次の通り。

- 半角英小文字（a-z）、半角数字（0-9）、ハイフン（-）、アンダースコア（_）
- 12〜50字
- ただし、他の記事はkebab-caseで統一

### Good

```md
---
title: "Pythonのパッケージ管理ツールuvの便利な使い方999選"
emoji: "🔖"
type: "tech" # or "idea"
topics: [...]
published: false
self_review: # HERE
  - ...
  - ...
slug_idea: # HERE
  - python-uv-package-manager-999-tips
  - useful-python-uv-tips-999
  - 999-ways-to-use-python-uv
  - python-uv-package-management-guide
  - mastering-python-uv-tips
---
```

### 因果関係

#### Example

- "PyPIがXML-RPCのAPIサポートを終了したことにより、従来の `pip search` コマンドが使用できなくなりました。そのため、パッケージの情報を取得する新しい方法が必要となりました。" とあるが、直接的に言及しているIssueやPRの議論が引用されていない。

## 参考URL

この指示にあたり、以下のURLを参考にした。

- [X | コミュニティノートガイド](https://communitynotes.x.com/guide/ja/contributing/examples)
- <https://ja.wikipedia.org/wiki/Wikipedia:秀逸な記事>
- [日本語の文章等で使われる約物の使い分けルール](https://jp-college.com/cat_japanese05/20220224/)
- [詭弁のカタログ](https://togetter.com/li/1355346)
