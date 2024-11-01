---
title: "uvのデバッグ環境をVS Codeで構築する"
emoji: "🔍"
type: "tech"
topics: ["rust", "uv", "vscode", "debug"]
published: true
---

## TL;DR

Rustで書かれたPythonパッケージマネージャー[uv](https://github.com/astral-sh/uv)のデバッグ環境をVisual Studio Codeで構築する方法を紹介します。

## 動機

uvの挙動を詳しく調査したいケースがあります。特に、ブレークポイントを打って動作を確認したい場合があります。

## すること

1. uvのリポジトリをフォークします。
2. VS Codeに必要な拡張機能をインストールします。
3. デバッグ設定ファイル（launch.json）を作成します。
4. デバッグを実行します。

## やり方

### 1. リポジトリのフォーク

GitHubでuvのリポジトリをフォークし、ローカルにクローンします。

### 2. VS Code拡張機能のインストール

以下のコマンドを実行して、必要な拡張機能をインストールします。

```shell
$ code --install-extension rust-lang.rust-analyzer
$ code --install-extension vadimcn.vscode-lldb
```

### 3. launch.jsonの作成

1. VS Codeの "Run and Debug" メニューを開きます。
2. "create a launch.json file." を選択します。
3. デバッガーとして "LLDB" を選択します。

### 4. launch.jsonの編集

作成されたlaunch.jsonを以下のように編集します。`args`にデバッグしたいuvコマンドの引数を指定します。

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "lldb",
            "request": "launch",
            "name": "Debug executable 'uv'",
            "cargo": {
                "args": [
                    "build",
                    "--bin=uv",
                    "--package=uv"
                ],
                "filter": {
                    "name": "uv",
                    "kind": "bin"
                }
            },
            "args": ["add", "markupsafe", "--directory", "test", "--index", "https://download.pytorch.org/whl/cu124", "--verbose"],
            "cwd": "${workspaceFolder}"
        }
    ]
}
```

この設定例では、`uv add markupsafe --directory test --index https://download.pytorch.org/whl/cu124 --verbose` コマンドをデバッグ実行します。

### 5. デバッグの実行

1. ソースコードにブレークポイントを設定します。
2. 「実行とデバッグ」メニューから「Debug executable 'uv'」を選択します。
3. デバッグが開始され、ブレークポイントで実行が停止します。

以下は、実際にデバッグを実行している様子です。

![VS CodeでのUVデバッグ画面](/images/vscode-debugging-for-uv-rust.png)

## まとめ

VS Codeを使ってuvのデバッグ環境を構築する方法を紹介しました。この環境を使うことで、uvの内部動作を詳細に調査できます。

この方法を使えば、uvの動作をステップ実行したり、変数の値を確認したりすることができます。Rustプロジェクトのデバッグに慣れていない方でも、視覚的に分かりやすく追跡できるでしょう。
