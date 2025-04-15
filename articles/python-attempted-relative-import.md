---
title: ""
emoji: "🐍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python"]
published: false
notebook_url:
- https://aistudio.google.com/prompts/1rOi3MDeQO4139vFrn2jpbT2rLwcOO--f
- https://gemini.google.com/app/469caba8d706bb2e?
---


エラーメッセージの意味を理解する

ImportError: attempted relative import with no known parent package


このエラーはスクリプト実行しているときに発生するが、parent packageが存在しないというエラーメッセージは直接的すぎてわかりづらい。

より丁寧に言えば、スクリプトとして実行している場合は、Pythonはそのファイルからパスをたどって所属パッケージを探したりしないので、パッケージを知らない、ということになる。

解決方法としては「モジュールで実行」すなわち -m になるのだが、．．．

JavaScriptでは次のようなコードが許される。

```
- dir/
  - index.js
  - utils.js
```

```index.js
import greet from utils

greet()
```

```
node index.js
```

そもそも、Pythonの相対インポートはファイルパスによる相対インポートではない点で、JSとは仕様が異なる。
名前空間パッケージという仕様がある
（ここで名前空間パッケージを使った便利な例をAIに調べてもらう）

ここでPythonではインポート時に.py拡張子があるとエラーになるとか、インストールするPyPIの名前とパッケージの名前が違うことを思い出してほしい。
あれはあくまでパッケージを扱っているため

Pythonの相対インポートはファイルパスではなくパッケージ空間のための文法なのだ。
ちなみにファイルパスに従った相対インポートに関連するIssueも当然あり、次のようになっている

（ここAIに調べてもらう）


また、解決済みのパッケージ構造を表示してくれる開発ツールもある。
（ここAIに調べてもらう）



モジュールに親しむ

アドオン開発などで相対インポートが必須で、かつ個別のファイルを実行してデバッグなどをしたい場合は、もうモジュールに親しむしかない。

モジュール実行は意外と使われている。例えば python -m venv .venv とか。これらは文法的には同じである。


```
python3 -c "import pkgutil; print([name for _, name, _ in pkgutil.iter_modules()])"

# uvの場合
uv run python  -c "import pkgutil; print([name for _, name, _ in pkgutil.iter_modules()])"
```

おいおい、そんなことしｔら、インストールしたパッケージのすべてのファイルが表示されないか？と思うかもしれないが（私は思った）、ならない。

pkgutil.iter_module() が賢いので。

ちなみに最近mcpでもよく見かけるuvxで実行できるのは...
（おそらくモジュールを指定するのだと思うがAIに調べてもらう）



