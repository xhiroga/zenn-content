---
title: "Ansible CLIをVSCodeのRun and Debugから実行する"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ansible","vscode"]
published: true
---

## TL;DR

VSCode の Run & Debug から `ansible` を起動する設定です。

![](/images/2021-12-31-ansible-debug-cli-from-vscode.png)

### 設定

[ansible/launch\.json at hiroga/vscode\-debug · xhiroga/ansible](https://github.com/xhiroga/ansible/blob/hiroga/vscode-debug/.vscode/launch.json)

## 解説

Ansible公式の[ソース \(devel\) からの Ansible の実行](https://docs.ansible.com/ansible/2.9_ja/installation_guide/intro_installation.html#devel-ansible)を参考に、VSCodeからデバッグできるように設定しました。

```jsonc
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "ansible-playbook",
            "type": "python",
            "request": "launch",
            "program": "${workspaceFolder}/bin/ansible-playbook",
            "console": "integratedTerminal",
            "env": {
                "PYTHONPATH": "${workspaceFolder}/test/lib:${workspaceFolder}/lib"
            },
            "args": [
                "${file}"
            ],
        },
    ]
}
```

### ポイント

- 本来は `./hacking/env-setup` が設定する `PYTHONPATH` を、手動で設定する必要があった。
- 現在開いているPlaybookをデバッグできるような設定にした。


## 参考

- [Ansible のインストール — Ansible Documentation](https://docs.ansible.com/ansible/2.9_ja/installation_guide/intro_installation.html#devel-ansible)
- [Using Python Environments in Visual Studio Code](https://code.visualstudio.com/docs/python/environments#_global-and-virtual-environments)
