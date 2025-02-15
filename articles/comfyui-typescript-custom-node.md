---
title: "ComfyUI Custom Nodes用のTypeScriptテンプレートを公開しました。"
emoji: "🛠️"
type: "tech"
topics: ["ComfyUI", "TypeScript", "JavaScript"]
published: false
---

## TL;DR

ComfyUIのCustom NodesをTypeScriptで開発するためのテンプレートを公開しました。

https://github.com/xhiroga/ComfyUI-TypeScript-CustomNode

Star☆お待ちしております！

## ComfyUI Custom Nodeとは？

ComfyUI は、拡散モデルや LLM などを実行するためのワークフローを GUI 上で実装・管理できるアプリケーションです。ノードベースのインターフェースを採用しており、各ノードが特定の処理を実行し、それらを繋ぎ合わせることで複雑なワークフローを構築できます。

Custom Nodesを用いることで、公式には用意されていない処理（例えばCV系の処理）を組み込むことができます。また、ノードのUIをカスタマイズすることができます。

詳細は[公式ドキュメント](https://docs.comfy.org/custom-nodes/custom_node_overview)をご覧ください。

:::message
ComfyUIのCustom Nodes初心者の方へ: Custom Nodes開発はPythonだけで事足りることも多いです。この記事はニッチな内容を扱っているので、まずは公式ドキュメントを読むことをオススメします。
:::

## Custom NodesのUI開発の課題

ComfyUIのCustom NodesのUIの開発とは、ざっくり言えば「本体からimportしたappのインスタンスのイベントリスナーにフックを登録する」プログラムを書くことです。

具体的には、次のようなコードになります。

```js
import { app } from "../../../scripts/app.js";

const custom_function = () => {
  // ここにカスタムコード
}

app.registerExtension({
  name: "SomeCustomNode",
  async beforeRegisterNodeDef(nodeType, nodeData, app) {
    if (nodeData.name === "SomeCustomNode") {
      const onExecuted = nodeType.prototype.onExecuted;
      nodeType.prototype.onExecuted = function(message) {
        onExecuted?.apply(this, arguments);
        custom_function.call(this. message)
      }
    }
  }
})
```

### ツラいポイント

ComfyUIのCustom NodesのUI開発を行う場合の辛い点は次のとおりです。

1. 公式ドキュメントが不足しています。利用可能なイベントリスナーや引数の型についての情報が少なく、実際に動かしながらの開発が必須です。
2. TypeScriptで開発されたCustom Nodesのサンプルがありません。ComfyUIのFrontendは型情報を`@comfyorg/comfyui-frontend-types`としてnpmに公開しているのですが、2025年2月24日時点でGitHub上にそれをimportしているソースコードは存在しませんでした。

## ComfyUI-TypeScript-CustomNodeの紹介

前述の理由からComfyUIのCustom NodesのUIをTypeScriptで開発したいところですが、サンプルが少ないため雛形を作るまでに数時間程度を要しました。

非常に徒労を感じたので、テンプレートを作って公開した、という流れです。

### ComfyUI-TypeScript-CustomNodeを用いた開発手順

テンプレートを利用した ComfyUI カスタムノード開発の基本的な手順は以下の通りです。

1. [テンプレートリポジトリのページ](https://github.com/xhiroga/ComfyUI-TypeScript-CustomNode/)を開く
2. `use this templateを押下`
3. `make install` コマンドを実行し、必要な依存ライブラリをインストール。
4. カスタムノードを開発: `web/src` ディレクトリに TypeScript コードを作成します。
5. 開発サーバーを起動: `make dev` コマンドを実行し、ComfyUI を起動します。

### ComfyUI-TypeScript-CustomNodeの実装上のポイント

本テンプレートを実装する上で工夫した点です。テンプレートを用いずにTypeScriptで開発される方にも参考になると思います。

#### 1. `@comfyorg/comfyui-frontend-types`のバグ解消

ComfyUIのフロントエンドは`litegraph`を用いて開発されていますが、独自の拡張が入っています。

それらはアンビエント宣言(`declare`)で`module`を拡張することで定義されていたのですが、`@comfyorg/comfyui-frontend-types`の`v1.10.0`まではその情報が含まれていませんでした。

[PRを送ったところすぐにmergeしていただいた](https://github.com/Comfy-Org/ComfyUI_frontend/pull/2560)ので、現在は修正済みです。

#### 2. 本体からの相対importをコンパイル時に差し込む

ComfyUIのCustom Nodesのソースコードは、デプロイ後に次の位置にコピーされます。

```

```

したがって、イベントリスナーなどを備えた`app`インスタンスはインスタンスは本体から相対インポートすることになります。

Custom Nodes開発時点では`../../scripts/app.js`は存在しないので、コンパイル時に挿入することになります。Custom Nodesのコードとして扱われないようにextermalとしての指定も必要です。

```ts
// vite.config.mts
import { defineConfig } from 'vite'

export default defineConfig({
  build: {
    ...,
    rollupOptions: {
      plugins: [
        inject({
          app: ['../../scripts/app.js', 'app']
        })
      ],
      external: (id) => id.startsWith('../../scripts/'),
    }
  },
  ...,
})
```

## まとめ

この記事では、ComfyUI のカスタムノードを TypeScript で開発する方法とそのメリット、具体的な開発手順とポイントについて解説しました。

TypeScript を活用することで、ComfyUI カスタムノード開発における型安全性を高め、開発効率を向上させることができます。

ぜひ [ComfyUI-TypeScript-CustomNode](https://github.com/xhiroga/ComfyUI-TypeScript-CustomNode) テンプレートを利用して、TypeScript での ComfyUI カスタムノード開発を試してみてください。
