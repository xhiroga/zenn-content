

- Ansibleのインベントリには、JSONを返すスクリプトを指定できる
- Pythonでもよい
- シェバンでenv で検索した？Pythonを指定する場合、AnsibleをUVで実行していたら、その環境になる
- 動的にインベントリを出した後、ホスト一覧をlimitで絞り込む


## デモ

[![asciicast](https://asciinema.org/a/709732.svg)](https://asciinema.org/a/709732)

## 動機

機械学習をやっているとGPUクラウドを立てたり落としたりする
接続先を複数個所に書くのは結構ストレスだった

## 大まかな構成

- ondemand.py には二つの機能がある
- -i オプションに渡す機能
- --limit オプションに渡す出力のための機能 ホスト名を選ぶ


## コード

https://github.com/xhiroga/homelab/blob/93ea84375b9d6aa7e134ebbfec8806fe837be24e/playbooks/ondemand.py
```py
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

- Paramikoが有名だがincludeディレクティブに対応していない
- fzfの採用について...
  - Pythonで完結させたかったが、Pythonスクリプトからの標準入出力がAnsibleに吸われる性質上難しかった

## まとめ

DRYができてすっきり

