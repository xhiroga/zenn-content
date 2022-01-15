---
title: "poetry init では 3.10 が使えるのに、 poetry env use python3.10 ではエラーになる"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Python","poetry"]
published: true
---

## TL;DR

PoetryをHomebrewでインストールしている場合、Python3.10が同梱される（2022年1月現在）。  
Homebrewで依存関係のためにインストールされるモジュールは通常PATHに存在しないので、pyenvなどでインストールしていない限りは`poetry env use python3.10`がエラーになる。

## Poetryの仕様

### poetry init

poetry init で --python オプションを指定しない場合、Poetryが利用しているPythonを利用する。

参考: [`current_env = SystemEnv(Path(sys.executable))`](https://github.com/python-poetry/poetry/blob/ecb030e1f0b7c13cc11971f00ee5012e82a892bc/src/poetry/console/commands/init.py#L151-L152)

ちなみに、PATHにpython3.10が存在しない状態で`--python 3.10`を指定すると `poetry install`時にエラーになる。

```bash
% poetry init --python 3.10
# 省略
Do you confirm generation? (yes/no) [yes]

% poetry install
The currently activated Python version 3.10.1 is not supported by the project (3.10).
Trying to find and use a compatible version. 

  NoCompatiblePythonVersionFound

  Poetry was unable to find a compatible version. If you have one, you can explicitly use it via the "env use" command.
```

### poetry env use python3.10

PATHにpython3.10が存在しない場合、エラーになる。

```bash
% poetry env use python3.10
/bin/sh: python3.10: command not found

  EnvCommandError

  Command python3.10 -c "import sys; print('.'.join([str(s) for s in sys.version_info[:3]]))" errored with the following return code 127, and output:
```

## 解決方法

ドキュメントに記載がある通り、pyenvでpython3.10を利用する。

```bash
pyenv local 3.10.1

poetry config virtualenvs.in-project true --local
# OR
export POETRY_VIRTUALENVS_IN_PROJECT=true

poetry init --python ^3.10
poetry install
```

## まとめ

pyenvで明示的にバージョンを指定することで、Poetryがデフォルトで利用しているPythonを避けることができる。
