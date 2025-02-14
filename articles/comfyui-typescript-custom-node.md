ComfyUIのCustom NodeをTypeScriptで作る


ComfyUIは拡散モデルやLLMなどを実行するためのワークフローをGUI上で実装・管理できるアプリケーションです。
主に画像・動画生成で用いられていますが、データの可視化においてそれに留まらないポテンシャルを秘めているソフトウェアだと思います。

今回、LLMの動作を可視化するうえでカスタムノードを開発していたのですが、
TypeScript化にめっちゃ苦労したのでテンプレートとして公開しました。

https://github.com/xhiroga/ComfyUI-TypeScript-CustomNode

ComfyUIでは、PythonとJavaScriptでカスタムノードを開発することができます。

ComfyUIカスタムノードとは？

公式を参照
- [Custom Node Overview](https://docs.comfy.org/custom-nodes/custom_node_overview)

どうして型情報を追加したの？

- ComfyUIのカスタムノード開発では、本家のlitegraphには存在しないイベントに対してフックを登録する。まぎらわしい
- 
例

```
// 本家には onExecute は存在するが、 onExecuted は無い!!!
nodeType.prototype.onExecuted = function (message) {
      originalOnExecuted?.apply(this, arguments);
      debugString(this, message.content);
    };
```



ComfyUIカスタムノードのフロントエンドの仕様

重要なポイントのみ

ComfyUIのカスタムノードがフロントエンドで行うこととしては、イベントリスナーの登録です。
appのイベントリスナーは、カスタムノードのスクリプトから見て相対パスで2階層上から取得します。

このようにデプロイされる

```
```

TypeScriptで開発するうえでのポイント

1. ComfyUI_frontendをGitHubから依存関係に追加し、型情報を取得する
2. appをComfyUI_frontendからimportします
3. JavaScriptへのトランスパイル時に、importのパスを書き換えます

詳細はソースコードをご覧ください。

https://github.com/xhiroga/ComfyUI-TypeScript-CustomNode/blob/main/web/src/debugString.ts

https://github.com/xhiroga/ComfyUI-TypeScript-CustomNode/blob/main/web/vite.config.mts
