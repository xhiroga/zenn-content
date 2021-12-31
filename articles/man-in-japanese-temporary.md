---
title: "man pagesを一時的に日本語で読む"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["man","debian"]
published: true
---

基本的には `manpage` を英語で読みたいが、どうしても意味が取れないときには日本語で読みたい、でも環境変数は汚したくない、ということがあったので、対策を考えました。
なお、debianの場合です。

## TL;DR

`-L` オプションを使わず、コマンドの実行時限定で環境変数を渡す（デフォルトの文字コードがUTF-8ではないディストリビューション・バージョンの場合）

```shell
LANG=ja_JP.utf8 man ls
```

文字コードに UTF-8 が指定されている場合、`-L` オプションで問題ない。

```shell
# export LANG=C.UTF-8
man -L ja_JP.utf8 ls
# または、単に
man -L ja ls
```

## 前提

`locale -a`を実行して`ja_JP.utf8`が表示される環境で実行しています。
また、`manpages-ja`をインストール済みです。

Dockerイメージはこちら。
[docker\-images/Dockerfile at main · xhiroga/docker\-images](https://github.com/xhiroga/docker-images/blob/main/manpages-ja/Dockerfile)


## 実験

| コマンドライン/環境変数 | unset LANG | export LANG=C.UTF-8 | export LANG=ja_JP.utf8 |
| --- | --- | --- | --- |
| man ls | English | English | 日本語 |
| man -L C.UTF-8 ls | English | English | English |
| man -L ja_JP.utf8 ls | 日本語だがUTF-8が表示されない | 日本語 | 日本語 |

### 補足

`man -E UTF-8 -L ja_JP.UTF-8 ls`だと、惜しいところまで行くのですが表示崩れがあります（条件などは要調査）

## まとめ

さすがに便利なオプション（`-L`）があることを知れてよかったです。
UTF-8がデフォルトかどうかはディストリビューションやバージョンにも寄りそうなので、あくまで私の環境（ubuntu:20.04）での結果、ということで。
