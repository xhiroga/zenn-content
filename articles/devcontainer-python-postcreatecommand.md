---
title: "なぜpip install -r requirements.txtをpostCreateCommandに書くべきか（オレオレDev Containers最強開発環境）"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vscode","python"]
published: false
---

## TL;DR



## 動機

Windows環境でPlaywrightを使ってスクレイピングをするにあたって、ローカルを汚さないようにDev Containersを使って開発環境を構築しました。その際、`pip install -r requirements.txt`をDockerfileに書いてしまい、結果`ModuleNotFoundError`が発生しました。

## なにをしたか


## なぜエラーが発生したか


## そもそもDev Containersとは

コンテナが普及するに伴って、CIやローカルのコーディングでもコンテナを利用したいという要望が増えてきたが、それぞれのシナリオごとに必要なツールが違うことに応えるもの。
`devcontainer.json`に構成を記述することで、VSCodeやIntelliJなど対応するエディタでコンテナを利用できるようになります。

Dev Containerには`Features reference`という機能があります。VSCodeのコマンドパレットからインタラクティブにDev Containerの設定を行う際、Select Featuresとして表示される機能の一覧のことです。

![Select Features](devcontainer-select-features.png)

これらの機能が何かというと、`devcontainer-feature.json`と`install.sh`を含むフォルダです。

### 余談1: Remote ContainerとDev Container

Dev Containerは元々Remote Containerという名前でしたが、VSCode以外でも利用できるようにDev Containerという名前に変更された経緯があります。

[VSCode Remote Container 改め Dev Containersがかなり変わっていて困惑した](https://zenn.dev/newgyu/scraps/4c24bf3df804bd) が詳しいです。

### 余談2: Dev Container Featureの中身を覗きたかった

Dev ContainerのFeatureはOCIレジストリへの参照として公開されており、つまりコンテナです。実際に`devcontainer-feature.json`と`install.sh`を含んでいるか、確認してみましょう。

```powershell
PS C:\Users\hiroga\Documents\GitHub\zenn-content> docker run --entrypoint /bin/bash --rm -it ghcr.io/devcontainers/features/python
Unable to find image 'ghcr.io/devcontainers/features/python:latest' locally
latest: Pulling from devcontainers/features/python
docker: unsupported media type application/vnd.devcontainers.
See 'docker run --help'.
```

あれ？Pullできませんでした。`unsupported media type application/vnd.devcontainers`とはなんでしょうか？

実は、OCI(Open Container Initiative)の定めるイメージフォーマットには、MediaTypeというプロパティがあります。コンテナイメージはManifestやIndexなど様々な要素で構成されており、どの要素かを示す値のようです。[^OCI Image Media Types]
[^OCI Image Media Types]: [image-spec/media-types.md at main · opencontainers/image-spec](https://github.com/opencontainers/image-spec/blob/main/media-types.md)

Dev ContainerのFeaturesは独自の形式を持っているため、Dockerイメージとして実行することはできなかったんですね。Dev ContainerのFeatureの定義は、GitHubリポジトリを参照するのが早いようです。

https://github.com/devcontainers/features/tree/main/src/python

### なぜDev ContainersはPythonのパスを書き換えるのか

[features/src/python at main · devcontainers/features](https://github.com/devcontainers/features/tree/main/src/python)

## 







## 参考資料

- [Development containers](https://containers.dev/)
- [Developing inside a Container using Visual Studio Code Remote Development](https://code.visualstudio.com/docs/devcontainers/containers)
- [Get started with development Containers in Visual Studio Code](https://code.visualstudio.com/docs/devcontainers/tutorial)
- [Build and run a Python app in a container](https://code.visualstudio.com/docs/containers/quickstart-python)
- [Develop in containers with Visual Studio Code](https://code.visualstudio.com/learn/develop-cloud/containers)
- [Dev container featuresについて調べてみる](https://zenn.dev/nmemoto/scraps/9eee0f54dc30ed)
