---
title: "uv add torchで2度とNo solution foundと言わせない"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "uv", "rye", "pytorch"]
published: false
---

## TL;DR

uvでPyTorchをインストールする際に私が遭遇したエラーと、その対処法は次のとおりです。

1. "No solution found when resolving dependencies"
   1. パッケージのバージョン識別子がpublic+localの形式になっているか確認
   2. オプションに`index`（0.4.22までは`extra-index-url`）を加える。
2. "it doesn't have a source distribution or wheel for the current platform"
   1. uv 0.4.4以上を用いて、`uv init`時に意図せず最新バージョンが用いられないようにする

## 動機

uvでPyTorchをインストールしようとした際、「No solution found」を含む複数のエラーに遭遇しました。同じ問題で困っている人の助けになれば幸いです。

## No solution found when resolving dependencies

次のような状況で発生するエラーです。

- 最も優先度の高いインデックスサーバーに存在するパッケージの中に、指定したバージョンがない
- すべてのインデックスサーバーを検索するpip互換のオプションが有効だが、どのサーバーにも指定したバージョンがない

### すること

1. Pythonパッケージのバージョンの仕様を正しく指定する
2. インデックスサーバーを正しく指定する

### Pythonパッケージのバージョンの仕様

私は調べるまで知らなかったのですが、`torch-2.1.2+cu118-cp310-cp310-win_amd64.whl`のうちバージョン番号なのは`torch-2.1.2+cu118`の部分だけです。したがって、次のようなパッケージの指定は誤りということになります。

```shell
uv add torch==2.1.2+cu118-cp310 --index https://download.pytorch.org/whl/cu118 # 失敗します
# Because there is no version of torch==2.1.2+cu118.cp310...
```

Pythonのパッケージを共有する特別な形式のZipファイルをwheel（ホイール）と呼び、その形式は次のようになっています。[^python_binary_file]
[^python_binary_file]: <https://packaging.python.org/en/latest/specifications/binary-distribution-format/#file-format>

`{distribution}-{version}(-{build tag})?-{python tag}-{abi tag}-{platform tag}.whl`

更に、バージョンは次のような内訳になっています。[^python_version]
[^python_version]: <https://packaging.python.org/en/latest/specifications/version-specifiers/#local-version-identifiers>

`<public version identifier>[+<local version label>]`

なので、次のように指定する必要があったんですね。

```shell
uv add torch==2.1.2 --index https://download.pytorch.org/whl/cu118
または
uv add torch==2.1.2+cu118 --index https://download.pytorch.org/whl/cu118
```

### インデックスサーバー

pipがパッケージを探す際は、すべてのインデックスサーバー（デフォルトまたは--index-urlで指定されたサーバー + --extra-index-urlで指定されたサーバー）から最新のバージョンを探します。一方で、uvはデフォルトで、最初にそのパッケージが見つかったインデックスサーバーの中の最新のバージョンを探します。

:::pipとuvのインデックス戦略の違い

```shell
hiroga@hiroga-air .local % mkdir pip-spec
hiroga@hiroga-air .local % cd pip-spec 
hiroga@hiroga-air pip-spec % python -m venv .venv
hiroga@hiroga-air pip-spec % source ./.venv/bin/activate                            
(.venv) hiroga@hiroga-air pip-spec % pip install tqdm --extra-index-url https://download.pytorch.org/whl/cu118  
Looking in indexes: https://pypi.org/simple, https://download.pytorch.org/whl/cu118
Collecting tqdm
  Using cached tqdm-4.66.5-py3-none-any.whl.metadata (57 kB)
Using cached tqdm-4.66.5-py3-none-any.whl (78 kB)
Installing collected packages: tqdm
Successfully installed tqdm-4.66.5

hiroga@hiroga-air uv-spec % uv --version
uv 0.4.24 (Homebrew 2024-10-17)
hiroga@hiroga-air .local % mkdir uv-spec
hiroga@hiroga-air .local % cd uv-spec 
hiroga@hiroga-air uv-spec % uv init
Initialized project `uv-spec`
(.venv) hiroga@hiroga-air uv-spec % uv add tqdm --extra-index-url https://download.pytorch.org/whl/cu118
warning: Indexes specified via `--extra-index-url` will not be persisted to the `pyproject.toml` file; use `--index` instead.
warning: `VIRTUAL_ENV=/Users/hiroga/Documents/GitHub/zenn-content/.local/pip-spec/.venv` does not match the project environment path `.venv` and will be ignored
Using CPython 3.12.7 interpreter at: /opt/homebrew/opt/python@3.12/bin/python3.12
Creating virtual environment at: .venv
Resolved 3 packages in 1.25s
Prepared 1 package in 29ms
Installed 1 package in 3ms
 + tqdm==4.64.1
```

:::

なお、これまで断りなく使ってきましたが、uv 0.4.24以降は`--extra-index-url`に代わって`--index`を利用することが推奨されています。[^uv_7481]
[^uv_7481]: <https://github.com/astral-sh/uv/pull/7481>

したがって`--index`オプションでPyPI以外のインデックスサーバーを指定すればよいわけです。ただし1点だけ注意事項があり、uvはインデックスサーバーのURLが不正な場合でも警告を出してくれません。

```shell
uv add tqdm --index https://download.pytorch.org/whl/cu999 # 明らかに存在しないインデックスサーバー
Resolved 3 packages in 1.07s
Installed 1 package in 2ms
 + tqdm==4.66.5
```

おかしいな？と思ったら`--verbose`オプションで確認しましょう。

## it doesn't have a source distribution or wheel for the current platform

指定されたバージョンのパッケージは存在するものの、PythonのバージョンやCPUのアーキテクチャが異なるなどで利用できないケースです。

### すること

1. Pythonバージョンやプラットフォームが意図通りであるかを確認する

### Pythonバージョンの確認

実は、`uv init`は元々バージョンを固定する仕様ではありませんでした。uv 0.4.4以降、`uv init`と同時に`.python-version`が作成されるようになったんですね。[^uv_6869]
[^uv_6869]: https://github.com/astral-sh/uv/pull/6869

なので、もしuvを使っていて、かつインデックスサーバーに存在するバージョンがaddできない場合、ランタイムのバージョンが誤っている可能性があります。次の通り確認できます。

```shell
uv run python --version
```

## まとめ

uvでPyTorchをインストールする際に遭遇した問題と、その対処法をまとめました。

uvの仕様は頻繁に更新されるため、まずはuvを最新化するのも有効だと思います。効率的で堅牢なパッケージ管理の役に立てば幸いです。
