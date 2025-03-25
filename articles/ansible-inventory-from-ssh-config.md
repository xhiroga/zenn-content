---
title: "Ansibleのインベントリを~/.ssh/configから動的に生成"
emoji: "🔧"
type: "tech"
topics: ["ansible", "python", "ssh", "sshconf"]
published: true
---

## TL;DR

- SSH設定ファイル（~/.ssh/config）からAnsibleインベントリを動的に生成するPythonスクリプトを書きました
- SSH Configの読み取りには `sshconf` を用いました
- AnsibleのランタイムをPythonの仮想環境にすると、動的インベントリも同じ環境で実行できます

## 動機

機械学習の実験をしていると、GPUクラウドを頻繁に立てたり落としたりします。その度に接続先情報を複数の場所に書き換えるのがストレスでした。

SSH設定は `~/.ssh/config` に書いていますが、Ansibleのインベントリファイルにもホスト名を記述する必要があります。この二重管理をなくし、DRY（Don't Repeat Yourself）の原則に従いたいと考えました。

## デモ

実際の動作の様子です。SSH設定からホスト一覧を取得し、インタラクティブに実行対象を選択できます。

[![asciicast](https://asciinema.org/a/709734.svg)](https://asciinema.org/a/709734)

## 実装方法

### 大まかな構成

作成したスクリプト `ondemand.py` には主に二つの機能があります：

1. Ansibleの `-i` オプションに渡すためのJSON形式インベントリを出力する機能
2. `--limit` オプションに渡すためのホスト名リストを出力し、fzfなどで選択できるようにする機能

### コード

次のように実行します。

```console
$ uv run ansible-playbook -i ondemand.py playbook.yml --limit $(ondemand.py | fzf)
```

https://github.com/xhiroga/homelab/blob/93ea84375b9d6aa7e134ebbfec8806fe837be24e/playbooks/ondemand.py

```python
#!/usr/bin/env python
import argparse
import json
import os

from sshconf import read_ssh_config


def parse(ssh_config_path: str) -> list[str]:
    """
    ssh接続情報は ssh_config に記載されている前提だから、ansible_user や ansible_ssh_private_key_file は返さない。
    簡単のため _meta は返さない。
    """
    config = read_ssh_config(ssh_config_path)
    hostnames = config.hosts()

    if "*" in hostnames:
        hostnames.remove("*")

    return list(hostnames)


def hostname_to_inventory(hostnames: list[str]):
    return {"all": {"hosts": hostnames}}


def main():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "ssh_config",
        nargs="?",
        default="~/.ssh/config",
        help="Path to SSH config file (default: ~/.ssh/config)",
    )
    parser.add_argument("--list", action="store_true", help="for ansible")
    parser.add_argument("--host", help="for ansible")
    args = parser.parse_args()

    expanded = os.path.expanduser(args.ssh_config)
    hostnames = parse(expanded)

    if args.list:
        print(json.dumps(hostname_to_inventory(hostnames)))
        return

    if args.host:
        print(json.dumps({}))
        return

    # --limit オプションの引数を fzf などで選択するために用いる
    # questionary や pick の採用を検討したが、いずれもコマンド置換・パイプとの相性が悪かった。
    print("\n".join(hostnames))


if __name__ == "__main__":
    main()
```

#### Ansibleの動的インベントリ

Ansibleのインベントリには、JSONを返すスクリプトを指定できます。Pythonで書いたスクリプトも使用可能です。

スクリプトは `--list` オプションが指定された場合、全ホストの情報をJSON形式で返す必要があります。また、`--host <hostname>` オプションが指定された場合は、特定ホストの変数情報を返します。

#### 環境変数の扱い

シェバンで `#!/usr/bin/env python` と指定しているため、Ansibleを[UV](https://github.com/astral-sh/uv)などの環境で実行している場合、そのPython環境が使用されます。

#### ホスト選択機能

動的にインベントリを生成した後、実行対象のホストを `--limit` オプションで絞り込めるようにしています。オプションなしで実行した場合はホスト名のリストを出力し、これを `fzf` などのツールと組み合わせることでインタラクティブな選択が可能になります。

#### 実装上の選択

- [Paramiko](https://www.paramiko.org/)は有名なSSHライブラリですが、SSH設定の `Include` ディレクティブに対応していないため、[sshconf](https://pypi.org/project/sshconf/)を使用しました
- ホスト選択をPythonで完結させることも検討しましたが、Pythonスクリプトからの標準入出力がAnsibleに吸収される性質上難しかったため、外部ツール（fzf）との連携を選択しました

## まとめ

インタラクティブに環境構築ができ、開発者体験が良いのでオススメです。
