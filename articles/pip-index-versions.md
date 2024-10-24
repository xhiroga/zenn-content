---
title: "pip index versionsコマンドを知っているか？"
emoji: "📦"
type: "tech"
topics: ["python", "pip", "パッケージ管理"]
published: false
---

## TL;DR

- `pip index versions <package_name>` コマンドで、パッケージの利用可能なバージョンを確認できます。
- このコマンドは実験的な機能で、将来変更される可能性があります。

## 動機

機械学習プロジェクトなどでパッケージをインストールする際、再現性を確保するために依存関係を厳密に指定したいことがあります。そのためには、パッケージの利用可能なバージョンを調べる必要があります。

しかし、既存の方法にはいくつかの問題がありました：

- `pip search`: 用途が違います（キーワードを含むパッケージ名を探す）[^pip_reference_search]
- `pip install torch==`: バージョン名を指定しないことで利用可能なバージョンを示すテクニックですが、エラーになります。
- `pip install torch --dry-run`: 最新バージョンしかわかりません（ただし、関連する依存関係を教えてくれるので、これはこれで便利です）。
- `pip-search`や`yolk3k`を使う: 追加のパッケージインストールが必要です。

[^pip_reference_search]: <https://kurozumi.github.io/pip/reference/pip_search.html>

:::details なぜpip searchが使えなくなったのか？
PyPIがXML-RPCのAPIサポートを終了したことにより、従来の `pip search` コマンドが使用できなくなりました。そのため、パッケージの情報を取得する新しい方法が必要となりました。
<!-- TODO: 資料が必要 -->
:::

## pip index versionsとは？

`pip index versions` は、パッケージの利用可能なバージョンを表示するコマンドです。このコマンドは、2021年に実装された比較的新しい機能です。[^pip_changelog_v21_2]

[^pip_changelog_v21_2]: <https://pip.pypa.io/en/stable/news/#v21-2>

このコマンドの背景には、次のような問題意識がありました（翻訳はClaudeによる）[^pip_issue_7975]：

> 現在、pipにはPackageFinderを呼び出して"best match"を選択せずに利用する方法がありません。しかし、ユーザーにとっては、バージョン範囲をどのように指定するかを決めるために、利用可能なバージョンのリストを見ることがよくある操作です。

[^pip_issue_7975]: <https://github.com/pypa/pip/issues/7975>

`pip index versions` は、この問題を解決するために導入されました。

## 使い方

`pip index versions` の基本的な使い方は次の通りです：

```shell
$ pip index versions torch
WARNING: pip index is currently an experimental command. It may be removed/changed in a future release without prior warning.
torch (2.5.0)
Available versions: 2.5.0, 2.4.1, 2.4.0, 2.3.1, 2.3.0, 2.2.2, 2.2.1, 2.2.0, 2.1.2, 2.1.1, 2.1.0, 2.0.1, 2.0.0
  INSTALLED: 2.2.0
  LATEST:    2.5.0

# PyPI以外のインデックスサーバーを利用する場合
$ pip index versions torch --index "https://download.pytorch.org/whl/cu124"
WARNING: pip index is currently an experimental command. It may be removed/changed in a future release without prior warning.
torch (2.5.0+cu124)
Available versions: 2.5.0+cu124, 2.4.1+cu124, 2.4.0+cu124
  INSTALLED: 2.2.0
  LATEST:    2.5.0+cu124
```

このコマンドは、指定したパッケージの利用可能なバージョンとインストール済みのバージョン、最新バージョンを表示します。

:::message alert
`pip index versions` は現在実験的な機能です。将来のリリースで予告なく削除または変更される可能性があります。
:::

## uvでの対応

（あまりいないと思いますが）2024年10月現在、`uv pip`は`index versions`に対応していません。しかし、`pip index versions` に相当する機能の実装を期待するIssueがあります。[^uv_issue_2111]。

[^uv_issue_2111]: <https://github.com/astral-sh/uv/issues/2111>

## まとめ

`pip index versions` は、パッケージの利用可能なバージョンを簡単に確認できる便利なコマンドです。特に、再現性が重要な機械学習プロジェクトなどで役立ちます。

ただし、実験的な機能であることに注意が必要です。今後の変更に備えて、最新の情報をチェックすることをおすすめします。
