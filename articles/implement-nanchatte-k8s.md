---
title: "Kubernetesを学ぶために、なんちゃってKubernetesを実装する。"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["kubernetes", "typescript"]
published: false
---

## Steps

1. ローカルの k3s で Kubernetes クラスターを実行する。
2. kubectl と k3s を使って一般的な Kubernetes のチュートリアルをする。その際、ちなみに、Verbosity level 6(-v=6)以上で実行して API エンドポイントを露出させる。
3. kubectl ではなく開発用 API クライアント(Postman, Insomnia など)と k3ss を使ってチュートリアルをする。
4. API クライアントで記録した Request, Response をそっくり返すアプリケーションを実装する。

## 2.

ちなみに、Verbosity level 8(-v=8)で実行すると Request, Response の Content も丸見えになります。

参考: [Exploring kubectl HTTP request verbose option and custom kube-config based on kubernetes RBAC](https://goglides.io/kubectl-http-request-to-formulate-rbac/123/)

## 3. kubectl ではなく開発用 API クライアント(Postman, Insomnia など)と k3ss を使ってチュートリアルをする。

k3s へのアクセスにはクライアント側証明書が必要なので、作成します。
ここでは macOS に合わせて PFX ファイルを作成しますが、各位の環境に合わせて適当にアレンジしてください。

```shell
mkdir .local
k3d kubeconfig get --all | grep client-key-data | awk '{print $2}' | base64 -d > .local/client-key-data
k3d kubeconfig get --all | grep client-certificate-data | awk '{print $2}' | base64 -d > .local/client-certificate-data
openssl pkcs12 -export -out .local/k3d.pfx -inkey .local/client-key-data -in .local/client-certificate-data
```
