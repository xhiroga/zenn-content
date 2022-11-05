---
title: "git-secretsのアンインストールに伴い、templateで作ったhooksの一括検索ツール作った"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["git", "git-secrets"]
published: false
---

[git-secrets](https://github.com/awslabs/git-secrets)、利用していらっしゃるでしょうか？  
gitleaksに乗り換えるにあたり、`brew uninstall git-secrets`だけでは不十分だと分かりました。大量の手作業が必要だったので、Goでツールを作成しました。

## アンインストール手順

1. `brew uninstall git-secrets` （brewでインストールしている場合）
2. `~/.gitconfig` から `templateDir = /Users/hiroga/.git-templates/git-secrets` の行を削除
3. `~/.git-templates/git-secrets` を削除
4. テンプレートを用いて作成されたリポジトリからhooksを削除 ← これが面倒

## テンプレートを用いたhooksを一括検索するツールを作った

https://github.com/xhiroga/git-secrets-hooks-cleaner

任意のフォルダから、`git-secrets`のテンプレートで生成されたhooksを再帰的に検索します。

構想段階では削除まで自動化するつもりでしたが、いざ検索したら手で消せる量だったので途中で実装を止めました。

### 使い方

```shell
go run main.go list --base $YOUR_GIT_REPO_ROOT_DIR
```

![](/images/2022-11-05-14-13-17.png)

`git-secrets`のテンプレートで生成されたhooksが `Template Compatible` の文字と共に表示されます。`git secrets`を含むが、テンプレートと同一ではない場合は赤字で`Modified`と表示されます。

## 開発にあたって

Go言語を使えて満足です。Goに詳しい方のツッコミ、大歓迎です。

CobraでCLIのテンプレートをサクッと作れたのは、今後のCLI制作が捗りそうな予感がしました。

## まとめ

よいGit&CLIライフをお過ごしください。
