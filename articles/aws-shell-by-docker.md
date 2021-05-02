---
title: "aws-shell を dockerでクリーンに動かす"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","Docker","aws-cli"]
published: true
---
# TL;DR
https://github.com/hiroga-cc/docker-images/blob/master/aws-shell/Dockerfile

# 詳細

`aws-shell` の調子が良くないので、dockerでクリーンに実行します。  

aliasでdockerを動かし、かつ `~/.aws` をボリュームとしてマウントする、という構成です。

```Dockerfile
# alias aws-shell='docker run --rm -it -v "$HOME/.aws:/root/.aws" hiroga/aws-shell:latest'
FROM python:3.7.7-alpine3.11

RUN pip install --upgrade pip;\
    pip install --upgrade aws-shell
ENTRYPOINT [ "aws-shell" ]
```

# 今後の改善点

* できれば毎度イメージを作るのを避けたいので `start` を利用したいのだが、 startにprofileオプションを渡す方法が思いつかないです...
* Creating doc index が毎回走るのもなんとかしたい。コンテナビルド時にindexを作るという手もあるが...うーん。


