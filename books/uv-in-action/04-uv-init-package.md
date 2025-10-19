---
title: "uvでパッケージを開発する"
references:
- https://packaging.python.org/ja/latest/overview/
- https://peps.python.org/pep-0508/ # パッケージの指定方法について
- https://chatgpt.com/g/g-p-68e9f86e76e48191bb2f1c486ee5a527/project    PEP508など
- pip拡張 https://pip.pypa.io/en/stable/topics/vcs-support/
---

（執筆中です...）

## GitHub上のリポジトリをuvでインストールする

## 5分ください！GitHubでプライベートなパッケージをホストできます

（執筆中...）

パッケージを作成したらビルドを試します。`uv build` を実行してください。

:::message
`git+https://...` の形式ってなんなのでしょうか？定義は次のとおりです。

```
<scheme>+<url>@<revision>#egg=<project_name>&subdirectory=<path>
````

実は、この文法はGitなどバージョン管理システムで管理されたパッケージを指定するためのPython独自の文法です。かつてPythonのパッケージ管理システムの事実上の標準だった`EasyInstall`が採用したフォーマットが、`pip`や`uv`にも受け継がれているんですね。
:::


## パッケージの高速な改善のためのテクニック

```sh
uv sync --reinstall
```
