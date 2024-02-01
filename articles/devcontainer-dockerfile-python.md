---
title: "devcontainerのためにDockerfileを書くべきか考察"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vscode","python"]
published: true
---

## TL;DR

- Dockerfileで構築したPython環境をdevcontainerのFeatureで上書きしてしまい、`ModuleNotFoundError`が発生した
- Dev ContainerのFeatureは、VSCodeの拡張機能だけでなく、ランタイムのインストールも行う
- Dev Containerを使うときは、ランタイムの設定はDockerfile側かdevcontainer側のどちらかに統一するべき

## 動機

Windows環境でPlaywrightを使ってスクレイピングをするにあたって、ローカルを汚さないようにDev Containersを使って開発環境を構築しました。しかし、`docker run`では動くのに、VSCodeのデバッグで実行すると`ModuleNotFoundError`が発生してハマりました。

## そもそもDev Containersとは

コンテナが普及するに伴って、CIやローカルのコーディングでもコンテナを利用したいという要望が増えてきたが、それぞれのシナリオごとに必要なツールが違うことに応えるもの。
`devcontainer.json`に構成を記述することで、VSCodeやIntelliJなど対応するエディタでコンテナを利用できるようになります。

Dev Containerには`Features reference`という機能があります。VSCodeのコマンドパレットからインタラクティブにDev Containerの設定を行う際、Select Featuresとして表示される機能の一覧のことです。

![Select Features](/images/devcontainer-select-features.png)

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

実は、OCI(Open Container Initiative)の定めるイメージフォーマットには、MediaTypeというプロパティがあります。コンテナイメージはManifestやIndexなど様々な要素で構成されており、どの要素かを示す値のようです。[^oci-image-media-types]
[^oci-image-media-types]: [image-spec/media-types.md at main · opencontainers/image-spec](https://github.com/opencontainers/image-spec/blob/main/media-types.md)

Dev ContainerのFeaturesは独自の形式を持っているため、Dockerイメージとして実行することはできなかったんですね。Dev ContainerのFeatureの定義を見たい場合、GitHubリポジトリを参照するのが早いようです。

https://github.com/devcontainers/features/tree/main/src/python

## なにをしたか

開発用のDockerイメージをDockerfileで定義した後、VSCodeでデバッグできるようにdevcontainerで指定しました。
その際、Pythonの拡張機能を利用したかったので、Dev Container構築時のFeatureとしてPythonを選択しました。

## なぜエラーが発生したか

Dev ContainerのFeatureのPythonは、VSCodeのPythonの拡張機能だけでなく、Pythonのインストールも行います。その際、`/usr/local/python/current/bin`にPythonのパスを設定します。

```terminal
root@4d9eb819d46e:/workspaces/til/software-engineering/playwright/_src/docker-play
wright# which python
/usr/local/python/current/bin/python
root@4d9eb819d46e:/workspaces/til/software-engineering/playwright/_src/
docker-playwright# echo $PATH
/vscode/vscode-server/bin/linux-x64/8b3775030ed1a69b13e4f4c628c612102e30a681/bin/remote-cli:/usr/local/python/current/bin:/usr/local/py-utils/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
root@4d9eb819d46e:/workspaces/til/software-engineering/playwright/_src/
```

https://github.com/devcontainers/features/blob/b08484e74f82ceaa35a48bef4b126b2c25343790/src/python/install.sh#L403

## どうすればよいか

Dev Containerによる環境構築は、次の2つに分けられると考えています。

1. デプロイ用などのDockerイメージをDockerfileで定義し、開発用ツールをdevcontainerでインストールする
2. ベースイメージやランタイムの選定をdevcontainerに任せる

今回の場合、PlaywrightのFeatureがなかったので、1を選択します。

## まとめ

Dev Containerを利用する際は、ランタイムの設定はDockerfile側かdevcontainer側のどちらかに統一すると、トラブルを避けられます。

## 参考資料

- [Development containers](https://containers.dev/)
- [Developing inside a Container using Visual Studio Code Remote Development](https://code.visualstudio.com/docs/devcontainers/containers)
- [Get started with development Containers in Visual Studio Code](https://code.visualstudio.com/docs/devcontainers/tutorial)
- [Build and run a Python app in a container](https://code.visualstudio.com/docs/containers/quickstart-python)
- [Develop in containers with Visual Studio Code](https://code.visualstudio.com/learn/develop-cloud/containers)
- [Dev container featuresについて調べてみる](https://zenn.dev/nmemoto/scraps/9eee0f54dc30ed)
