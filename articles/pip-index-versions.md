
- pip indexとは？
  - pipのドキュメントには存在しない
    - https://pip.pypa.io/en/stable/cli/

もともとあったsearchコマンドは？
PyPIが対応しなくなった。なぜ？
PyPIがXML-RPCのAPIに対応しなくなったらしい。
https://warehouse.pypa.io/api-reference/xml-rpc.html#deprecated-methods
このWarehouseというのは？PyPIで使われているWebアプリケーションフレームワーク

PyPI以外ではsearchはサポートされているらしい。ほんとに？
いずれにせよあまりサポートはされてなさそうだ。
（ほかのインデックスサーバーのURLください）

```shell
pip search torch
ERROR: XMLRPC request failed [code: -32500]
RuntimeError: PyPI no longer supports 'pip search' (or XML-RPC search). Please use https://pypi.org/search (via a browser) instead. See https://warehouse.pypa.io/api-reference/xml-rpc.html#deprecated-methods for more information.

pip search torch --index "https://download.pytorch.org/whl/cu124"
ERROR: Exception:
Traceback (most recent call last):
  File "C:\Users\hiroga\AppData\Local\Packages\PythonSoftwareFoundation.Python.3.11_qbz5n2kfra8p0\LocalCache\local-packages\Python311\site-packages\pip\_internal\network\xmlrpc.py", line 52, in request
    raise_for_status(response)
  File "C:\Users\hiroga\AppData\Local\Packages\PythonSoftwareFoundation.Python.3.11_qbz5n2kfra8p0\LocalCache\local-packages\Python311\site-packages\pip\_internal\network\utils.py", line 56, in raise_for_status
    raise NetworkConnectionError(http_error_msg, response=resp)
pip._internal.exceptions.NetworkConnectionError: 403 Client Error: Forbidden for url: https://download.pytorch.org/whl/cu124

```

ドキュメントにないのでソースを探す
https://github.com/pypa/pip


2021年に実装されている。割と新しい？
https://github.com/pypa/pip/commit/3751878b42c9914e1c71aeec62b97ec47fb9accf

Release noteより抜粋
https://pip.pypa.io/en/stable/news/#id419

> Add new subcommand pip index used to interact with indexes, and implement pip index version to list available versions of a package. (#7975)

そのモチベーションのIssue
https://github.com/pypa/pip/issues/7975


pip install には dry runがある。
>   --dry-run                   Don't actually install anything, just print what would be. Can be used in combination with --ignore-installed to 'resolve' the
                              requirements.

uv pip installにもdry runがある
https://docs.astral.sh/uv/reference/cli/#uv-pip-install
> --dry-run
Perform a dry run, i.e., don’t actually install anything but resolve the dependencies and print the resulting plan

おそらく--dry-runだとBest Matchを探してしまうのでは？

Searchとは異なる方法なんだろうか？
PackageFinder は（index version 内部で使われているコマンド）

名前はそこそこ議論があった
https://github.com/pypa/pip/issues/8516

pip searchだと面倒そう
I feel we are free to do anything with pip search now. I feel we can just accept this to replace pip search <term> and slap on a big red banner whenever it’s called telling the user this is experimental and will change at any time without notice whatsoever, so users should use it at their own risk.

最終的には名前は実装のPRで議論されている
https://github.com/pypa/pip/pull/8978


また立ち位置は？

uvでの対応予定は？
Feature request: show available package versions (equivalent of pip index versions) #2111
https://github.com/astral-sh/uv/issues/2111

わかる。ほしい。

> # or, the old hack:
pip install <some-package>==
こんなテクが...

ここでも紹介されてる。
https://stackoverflow.com/questions/4888027/how-to-list-all-available-package-versions-with-pip

> For pip >= 21.2 use:

pip index versions pylibmc
Note that this command is experimental, and might change in the future!

Below approach breaks with pip 24.1 released on 2024-06-21.

For pip >= 21.1 use:

pip install pylibmc==
For pip >= 20.3 use:

pip install --use-deprecated=legacy-resolver pylibmc==
For pip >= 9.0 use:

$ pip install pylibmc==
Collecting pylibmc==
  Could not find a version that satisfies the requirement pylibmc== (from 
  versions: 0.2, 0.3, 0.4, 0.5.1, 0.5.2, 0.5.3, 0.5.4, 0.5.5, 0.5, 0.6.1, 0.6, 
  0.7.1, 0.7.2, 0.7.3, 0.7.4, 0.7, 0.8.1, 0.8.2, 0.8, 0.9.1, 0.9.2, 0.9, 
  1.0-alpha, 1.0-beta, 1.0, 1.1.1, 1.1, 1.2.0, 1.2.1, 1.2.2, 1.2.3, 1.3.0)
No matching distribution found for pylibmc==
The available versions will be printed without actually downloading or installing any packages.

For pip < 9.0 use:

pip install pylibmc==blork

<<<

pip search と pip index versionsの違い

> First find the latest version available with pip search <package name>. 
latestしかないっぽい。検証する...と思ったがsearchが対応しているインデックスサーバーを見つけられなかった

