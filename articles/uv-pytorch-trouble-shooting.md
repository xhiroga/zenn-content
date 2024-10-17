---
title: "uv add torchで2度とNo solution foundと言わせない"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "uv", "rye", "pytorch"]
published: false
---

## Pythonのパッケージのバージョンについて

Pythonのパッケージは特別な形式のZipファイルとして共有されており、その形式やファイルをwheel（ホイール）という。
モンティ・パイソンのコメディより、お店にストックしてあった円盤型のチーズが由来。https://qiita.com/misohagi/items/21d5f383f3087118eda0
（勝手に車輪の再発明の「車輪」から来ているのかと思ってた）

以下を具体例としてフォーマットについて解説

torch==2.1.2+cu118-cp310-cp310-win_amd64
[numpy-2.1.2-pp310-pypy310_pp73-win_amd64.whl](https://files.pythonhosted.org/packages/c0/ec/0c04903b48dfea6be1d7b47ba70f98709fb7198fd970784a1400c391d522/numpy-2.1.2-pp310-pypy310_pp73-win_amd64.whl)

## ホイールの名前のスペック

https://packaging.python.org/en/latest/specifications/binary-distribution-format/#file-format

### バージョン番号

https://packaging.python.org/en/latest/specifications/version-specifiers/

2.1.2 public version identifier
+cu118 local version label

+cu118まで含めてバージョン番号というのが誤解しやすそうなので注意。私は誤解していた。

### ビルドタグ

-cp310-cp310-win_amd64

## (インストール関連: 後でタイトル考える）

### pip install

そもそもpip installで指定する対象
https://pip.pypa.io/en/stable/cli/pip_install/

requirement specifierに従う
https://pip.pypa.io/en/stable/reference/requirement-specifiers/

requirement specifierがversion specifierに従う

## uv add

hiroga@HIROGA-RTX4090:/mnt/c/Users/hiroga/Documents/GitHub/zenn-content/.local/test$ uv add torch
"torch>=2.4.1",
何もしなくても最低限のバージョンを記録するようになっている

実はPyPI（パイピーアイ）にもある
[[package]]
name = "torch"
version = "2.4.1"
source = { registry = "https://pypi.org/simple" }
dependencies = [

```shell
> uv add torch --index-url https://download.pytorch.org/whl/cu118
Resolved 23 packages in 2.15s
error: Failed to prepare distributions
  Caused by: Failed to fetch wheel: test @ file:///C:/Users/hiroga/Documents/GitHub/zenn-content/.local/test
  Caused by: Failed to install requirements from build-system.requires (resolve)
  Caused by: No solution found when resolving: setuptools>=40.8.0, wheel
  Caused by: Because wheel was not found in the package registry and you require wheel, we can conclude that your requirements are unsatisfiable.
```

```shell
uv add torch==2.1.2 --index-url https://download.pytorch.org/whl/cu118 --verbose
DEBUG No cache entry for: https://download.pytorch.org/whl/cu118/torch/
DEBUG Searching for a compatible version of torch (==2.1.2)
DEBUG No compatible version found for: torch
DEBUG Searching for a compatible version of test @ file:///C:/Users/hiroga/Documents/GitHub/zenn-content/.local/test (<0.1.0 | >0.1.0)
DEBUG No compatible version found for: test
  × No solution found when resolving dependencies:
  ╰─▶ Because there is no version of torch==2.1.2 and your project depends on torch==2.1.2, we can conclude that your project's requirements are unsatisfiable.
  help: If this is intentional, run `uv add --frozen` to skip the lock and sync steps.
```

```shell
uv add torch==2.1.2 --index-url https://download.pytorch.org/whl/cu118 --verbose --index-strategy unsafe-best-match
DEBUG No cache entry for: https://download.pytorch.org/whl/cu118/torch/
DEBUG Searching for a compatible version of torch (==2.1.2)
DEBUG No compatible version found for: torch
DEBUG Searching for a compatible version of test @ file:///C:/Users/hiroga/Documents/GitHub/zenn-content/.local/test (<0.1.0 | >0.1.0)
DEBUG No compatible version found for: test
  × No solution found when resolving dependencies:
  ╰─▶ Because there is no version of torch==2.1.2 and your project depends on torch==2.1.2, we can conclude that your project's requirements are unsatisfiable.
  help: If this is intentional, run `uv add --frozen` to skip the lock and sync steps.

```

```shell
uv add torch==2.1.2+cu118 --index-url https://download.pytorch.org/whl/cu118 --verbose

DEBUG No cache entry for: https://download.pytorch.org/whl/cu118/torch/
DEBUG Searching for a compatible version of torch (==2.1.2+cu118)
DEBUG Selecting: torch==2.1.2+cu118 [compatible] (torch-2.1.2+cu118-cp310-cp310-linux_x86_64.whl)
DEBUG Found stale response for: https://download.pytorch.org/whl/cu118/torch-2.1.2%2Bcu118-cp310-cp310-linux_x86_64.whl#sha256=60396358193f238888540f4a38d78485f161e28ec17fa445f0373b5350ef21f0
DEBUG Sending revalidation request for: https://download.pytorch.org/whl/cu118/torch-2.1.2%2Bcu118-cp310-cp310-linux_x86_64.whl#sha256=60396358193f238888540f4a38d78485f161e28ec17fa445f0373b5350ef21f0
DEBUG Found not-modified response for: https://download.pytorch.org/whl/cu118/torch-2.1.2%2Bcu118-cp310-cp310-linux_x86_64.whl#sha256=60396358193f238888540f4a38d78485f161e28ec17fa445f0373b5350ef21f0
```

```
> uv add "torch==2.1.2+cu118" "torchvision==0.16.2+cu118" --index-strategy unsafe-best-match
Resolved 19 packages in 1ms
error: distribution torch==2.1.2+cu118 @ registry+https://download.pytorch.org/whl/cu118 can't be installed because it doesn't have a source distribution or wheel for the current platform
```
これは様子が異なる。

なんでさっきできていまダメなんだ？
```shell
uv add "torch==2.1.2+cu118" "torchvision==0.16.2+cu118" --index-url https://download.pytorch.org/whl/cu118 --verbose
DEBUG Searching for a compatible version of urllib3 (>=1.21.1, <1.27)
DEBUG Selecting: urllib3==1.26.13 [compatible] (urllib3-1.26.13-py2.py3-none-any.whl)
DEBUG Tried 19 versions: certifi 1, charset-normalizer 1, filelock 1, fsspec 1, idna 1, jinja2 1, markupsafe 1, mpmath 1, networkx 1, numpy 1, pillow 1, requests 1, sympy 1, torch 1, torchvision 1, train 1, triton 1, typing-extensions 1, urllib3 1
DEBUG Split universal resolution took 3.455s
Resolved 19 packages in 3.45s
error: distribution torch==2.1.2+cu118 @ registry+https://download.pytorch.org/whl/cu118 can't be installed because it doesn't have a source distribution or wheel for the current platform
```

```
> uv run python --version
   Built train @ file:///.../train
Uninstalled 1 package in 1ms
Installed 1 package in 7ms
Python 3.12.0
```

解答: uv initは元々バージョンを固定する仕様ではなかった。結果としてcpythonのバージョンが異なり、厳密にマッチするwheelがないのでエラーになっていた。
https://github.com/astral-sh/uv/pull/6869

uvのバージョンが壊れないようにするための挙動

バージョン解決の経過を見ることは出来ないのか？
--verboseでできる

https://github.com/astral-sh/uv/issues/6821

uv add と uv pip install
https://docs.astral.sh/uv/pip/compatibility/

関連Issue
https://github.com/astral-sh/uv/issues/1497
https://github.com/astral-sh/uv/issues/2037

## 参考

https://qiita.com/fujine/items/11acac9aaa56791378b4

https://www.pythonic-exam.com/archives/8336

https://claude.ai/chat/3c53137a-6db0-4df8-88e4-cf1a0ec86df7
