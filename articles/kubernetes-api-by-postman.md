---
title: "k3sにPostmanからアクセスする"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["kubernetes", "k3s", "Postman"]
published: true
---

## TL;DR

- `k3d kubeconfig` で接続用の設定を出力する
- クライアント証明書を base64 デコードする
- サーバー証明書の検証を無効化する

## How to

### `k3d kubeconfig` で接続用の設定を出力する

`k3d kubeconfig get` で接続先を確認。

```shell
k3d kubeconfig get --all

---
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data:
    # 省略
    server: https://0.0.0.0:52224
  name: k3d-k3s-default
```

すでに接続先を設定済みの場合、 `cat ~/.kube/config` でも OK。

### クライアント証明書を base64 デコードする

```shell
k3d kubeconfig get --all | grep client-key-data | awk '{print $2}' | base64 -d > client-key-data
k3d kubeconfig get --all | grep client-certificate-data | awk '{print $2}' | base64 -d > client-certificate-data
```

デコードしたクライアント証明書を Postman に設定（ちなみに Insomnia でも可能でした。）

![image.png](https://i.gyazo.com/99a558c2fdf5ae2f8bd7dabd5b4f2fc4.png)

### サーバー証明書の検証を無効化する

k3s の SSL 証明書は k3s 自身によるオレオレ証明書のため、検証ができません。

![image.png](https://i.gyazo.com/b67f5ba416e072cbe45aeaee999a2118.png)

（逆になんで kubectl はリクエストに成功しているんだろう...?）

そのままリクエストすると失敗するので、SSL 証明書の検証を無効化してください。

## 補足

クラスターを削除して再作成すると、接続先ポートとクライアント証明書が新たに設定されます。ご注意ください。
