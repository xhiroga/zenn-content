---
title: "個人的に感動したシェルの小技集"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## マニュアルを VSCode で開く

```shell
man curl | col -b | code -
```

## .env の中身を環境変数に展開する

```shell
export $(cat .env)
```

ポイント: export コマンドは複数の値を取ることができます。
