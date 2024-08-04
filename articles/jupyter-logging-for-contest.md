---
title: "競プロJupyter Notebook勢のためのlogging設定考察"
emoji: "🐍"
type: "tech" # tech: 技術記事 / idea: アイデア
published: true
topics: ["python","jupyternotebook", "logging"]
---

## TL;DR

```python
from logging import basicConfig, root, DEBUG, WARNING

def some_algorithm():
    basicConfig(level=DEBUG if 'get_ipython' in globals() else WARNING)
    root.debug('hello')

some_algorithm()
```

## 動機

テストの書きやすさや、単純に好みから、Jupyter Notebookで競技プログラミングに出ています。

しかし、ロギングは競プロではただでさえ気を使うのに、Jupyter Notebook固有の問題が加わってきます。

- AtCoderでは標準出力を判定するので、`print`文を消し忘れると誤答になる
- 代わりに標準エラー出力を使おうとするも、Jupyter Notebookの標準エラー出力は真っ赤なのでデバッグには向かない
- `logging`ライブラリを使う場合、何度も実行するセル内で`StreamHandler`を定義していると、実行の度にハンドラが増える

上記の問題に加えて、コピペする行数を最小限（2行）に抑えたのが冒頭のコードです。

## 解説

コードは冒頭と同じです。コメントで解説します。

```python
from logging import basicConfig, root, DEBUG, WARNING

def some_algorithm():

    # basicConfig は、ルートロガーに StreamHandler がない場合のみ設定する
    # ルートロガーの利用を認めることになるが、子ロガーで冪等にハンドラを設定する方法がデフォルトで用意されていない
    basicConfig(
        # 一般的には get_ipython() を呼び出すことが多いようだが、Jupyter Notebook以外の環境でエラーハンドリングが必要になる
        # Jupyter Notebook 固有の変数として __IPYTHON__ があるが、これは globals() には定義されていないようだ
        # その他、環境変数やモジュールを見る案もあったが、os, sysのimportが必要なので見送った。
        level=DEBUG if 'get_ipython' in globals() else WARNING
    )

    # logging.debug() でも構わない
    # しかし、root.debug() の方が明示的にルートロガーを使っていることが伝わる
    root.debug('hello')

some_algorithm()
```

## まとめ

- `basicConfig`を用いることで、セルの再実行に対しても冪等に`StreamHandler`を設定できました。
- `get_ipython`がグローバル変数にあるかを見ることで、ライブラリのimportやエラーハンドリング無しに実行環境を判定することができました。
- `root`を使うことで、ルートロガーの利用を明示しました。
