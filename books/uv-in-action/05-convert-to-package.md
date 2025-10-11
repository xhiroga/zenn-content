---
title: "既存のリポジトリをパッケージ化する"
references:
- https://chatgpt.com/c/68ea0126-7fa0-8323-85d3-01eaa38bb493
- https://chatgpt.com/c/68ea02e8-1a94-8322-bddb-8cff16357125    # 軽量なPythonリポジトリ
- https://github.com/crazyguitar/pysheeet   # チートシート。パッケージ追加の失敗例として例示しようと思ったが lfs の問題があるらしくimportに失敗。
- https://github.com/faif/python-patterns   # パッケージ化のやり方を誤っている実例になるかも...
---

## はじめに

## パッケージとしてインストール可能にする

`pyproject.toml` を追加すれば、リポジトリをパッケージとしてインストール可能にできます。

逆にいうと、Pythonで書かれたリポジトリであっても、`pyproject.toml` または `setup.py` が存在しない場合はインストールができません。

```console
% uv add git+https://github.com/JeongHun0716/zero-avsr
    Updated https://github.com/JeongHun0716/zero-avsr (6206c9e5c80a05a602471740e48323536ecd7c01)
```

したがって、フォークして `pyproject.toml` を追加する必要があるんですね。

:::message `pyproject.toml` さえあれば何でもインストールできるってこと？

そうです。次の例では、ハッシュ計算のライブラリ `mmh3` をインストールします。

```console
% # 適当なプロジェクトを uv init してから...
% uv add mmh3
Resolved 3 packages in 16ms
Audited 2 packages in 1ms
% ls .venv/lib/python3.13/site-packages/mmh3
__init__.pyi  hashlib.h  mmh3module.c  murmurhash3.c  murmurhash3.h  py.typed
```

ご覧の通り、`*.py` ファイルが1つも入っていません。バイナリだけでも、`pyproject.toml` があるからPythonのパッケージとして認識されているんですね。
:::

## パッケージを通してコードを利用可能にする




