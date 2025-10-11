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

したがって、フォークして `pyproject.toml` を追加する必要があるんですね。既存プロジェクトをuvに移行したのと同じやり方で大丈夫です。

:::message
`pyproject.toml` さえあれば何でもインストールできるってこと？

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

パッケージを利用可能にするには、`pyproject.toml` の追加だけでなく、ビルドの設定が必要です。

### srcレイアウトとフラットレイアウト

パッケージとして利用したいコードがプロジェクトルートにある場合、リポジトリの構造の変更が必要な可能性が高いです。いわゆる **srcレイアウト** と **フラットレイアウト** のどちらかを確認しますが、それだけではありません。

はじめに、**srcレイアウト** と **フラットレイアウト** について、Python公式ドキュメントの [src レイアウト対フラットレイアウト](https://packaging.python.org/ja/latest/discussions/src-layout-vs-flat-layout/)から引用します。

「フラットレイアウト」とは、さまざまな設定ファイルや インポートパッケージ をすべてトップレベルのディレクトリに置くようなやり方で、プロジェクトのファイル群をひとつのフォルダまたはリポジトリに配置することです。

```
.
├── README.md
├── noxfile.py
├── pyproject.toml
├── setup.py
├── awesome_package/
│   ├── __init__.py
│   └── module.py
└── tools/
    ├── generate_awesomeness.py
    └── decrease_world_suck.py
```

「src レイアウト」は、インポート可能 (すなわち import awesome_package 、別名 インポートパッケージ) にするつもりのソースコードをサブディレクトリに置く点でフラットレイアウトとは異なります。このサブディレクトリは、典型的には src/ と命名されるので、「src レイアウト」と呼ばれるのです。

```
.
├── README.md
├── noxfile.py
├── pyproject.toml
├── setup.py
├── src/
│    └── awesome_package/
│       ├── __init__.py
│       └── module.py
└── tools/
    ├── generate_awesomeness.py
    └── decrease_world_suck.py
```

リポジトリが srcレイアウト であれば、パッケージ化は簡単です。

フラットレイアウトの場合、もしリポジトリルートにインポート予定のコードがあれば、srcレイアウト に直してしまった方が良いかもしれません。フラットレイアウトであっても、インポート予定のコードがディレクトリに入っている場合は、それらのディレクトリをパッケージとして扱うことでレイアウトを直さずにパッケージ化することができます。

### リポジトリの状況別！おすすめパッケージ化手順

基本的にはsrcレイアウトでパッケージ化するのがおすすめですが、前方互換性を保ちたい場合や、フォーク元との差分を抑えたい場合もあると思います。そこで選択チャートを用意しました。

- リポジトリがsrcレイアウトである
  - パターン1: `pyproject.toml` を編集してパッケージ化完了
- フラットレイアウトである
  - 自分の裁量で大規模な変更をしてOK
    - パターン2: srcレイアウトに移行してパッケージ化
  - なるべく変更量を抑えたい...
    - import予定のコードがリポジトリルートにある
      - リポジトリのコード間でのimportは絶対importである
        - importの親パッケージの書き換えは必要だが、おすすめ通りsrcレイアウトにする
          - パターン2: srcレイアウトに移行してパッケージ化
        - レイアウトを変えないのが絶対なので、相対import化する
          - パターン3: フラットレイアウトでパッケージ化（制限あり）
      - コード間のimportを書き換える覚悟はない
        - パターン2: srcレイアウトに移行してパッケージ化
    - import予定のコードはディレクトリに格納されている
      - `from {リポジトリ名}` ではなく `from {ディレクトリ名}` でも問題ない
        - パターン3: フラットレイアウトでパッケージ化（制限あり）
      - `from {リポジトリ名}` でimportしたい
        - 以降、import予定のコードがリポジトリルートにある と同様に分岐

### 事例1: [musubi-tuner](https://github.com/kohya-ss/musubi-tuner)

kohya-ssさんが開発・運営する、画像・動画生成AIの追加学習・推論リポジトリです。アプリケーションに組み込んで利用しやすいようパッケージ化し、本家にもPRをmergeいただきました。非常に感謝しています。

https://github.com/kohya-ss/musubi-tuner/pull/319

（執筆中...）

### 事例2: [zero-avsr](https://github.com/JeongHun0716/zero-avsr)

Jeong Hun Yeoさんが開発した、音声・視覚から発話内容を文字起こしする研究の公式実装です。私が個人的にフォークし、研究のためにパッケージ化させていただきました。

https://github.com/xhiroga/zero-avsr/tree/packaged

zero-avsrは、importしたいコードが全てディレクトリに格納されています。また、初めから相対importを使って実装されていました。そこで、フラットレイアウトのままパッケージ化する判断をしました。

