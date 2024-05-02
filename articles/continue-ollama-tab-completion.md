---
title: "Continue + Ollama でタブ補完(β)を機能させるまで"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ollama", "continue", "LLM"]
published: true
---

ローカルLLMでGitHub Copilotのような開発ができるようにしました。

[Continue](https://www.continue.dev/) と [Ollama](https://ollama.com/) を用いましたが、[タブ補完](https://docs.continue.dev/walkthroughs/tab-autocomplete)がβ版ということもあってか設定で躓いたので、記事にしました。

## TL;DR

- Ollamaを起動し、API経由のアクセスが有効か確かめます
- config.jsonの設定後、VSCodeを再起動します（2024-05-02時点では必要）
- Continueのタブ補完実行時、VSCodeのOutputタブとOllamaのserver.logでデバッグを行います。

https://www.youtube.com/watch?v=s_gf-4t2vZU

## 技術選定

GitHub Copilotに代わるアシスタントとしては、CursorとContinueを比較しました。  
今回は完全ローカルで動作するContinueを選択しました。Cursorは2024‐05‐01時点ではCursorのサーバーを経由してLLMにアクセスしているため、ngrokなどでプロキシしないとローカルLLMが利用できないためです。

https://zenn.dev/ryoppippi/articles/02c618452a1c9f

また、LLMのProviderとしてはOllamaを選択しました。理由はContinue公式のガイドが豊富だからです。  
ちなみに2024‐05‐02時点で、ローカルだけでもLM StudioやLllama.cppを含む10種類のProviderがサポートされています。

## 手順

### 1. Ollamaのインストール・起動確認

[Ollamaをインストールし、モデルを起動します。](https://ollama.com/download/windows)  
タブ補完用とチャット用でそれぞれ別モデルを利用するのが良さげです。

```powershell
# チャット用
ollama run llama3
# タブ補完用
ollama run starcoder2:3b
```

次の通り、インタラクティブシェルが元気よく起動しました。

https://www.youtube.com/watch?v=21-Z6Gh5G_A

インタラクティブシェルを起動するのは、初回起動時にモデルをダウンロードするのが目的です。　　
実際には、次のようにOllamaがバックグラウンドで起動しており、かつモデルがダウンロードできていれば、インタラクティブシェルを起動させておく必要はありません。

![ollama on background](/images/ollama.png)

モデルのダウンロードが完了したら、OllamaのAPIでモデルを確認します。

```powershell
curl http://localhost:11434/api/tags
```

```json
{
  "models": [
    {
      "name": "llama3:latest",
      "model": "llama3:latest",
      "modified_at": "2024-05-01T21:16:11.7124241+09:00",
      "size": 4661224578,
      "digest": "a6990ed6be412c6a217614b0ec8e9cd68 00a743d5dd7e1d7fbe9df09e61d56 15",
      "details": {
        "parent_model": "",
        "format": "gguf",
        "family": "llama",
        "families": ["llama"],
        "parameter_size": "8B",
        "quantization_level": "Q4_0"
      }
    },
    {
      "name": "mistral:latest",
      "model": "mistral:latest",
      "modified_at": "2024-05-01T21:12:46.3880083+09:00",
      "size": 4109865159,
      "digest": "61e88e884507ba5e06c49b40e6 226884b2a16e872382c2b44a42f2d119 d804a5",
      "details": {
        "parent_model": "",
        "format": "gguf",
        "family": "llama",
        "families": ["llama"],
        "parameter_size": "7B",
        "quantization_level": "Q4_0"
      }
    },
    {
      "name": "starcoder2:3b",
      "model": "starcoder2:3b",
      "modified_at": "2024-05-02T07:09:30.9950013+09:00",
      "size": 1709901545,
      "digest": "f67ae0f646584a4d1d7c609bf47 78dd0d07054582362d21ca4f0edea 22aafd0c",
      "details": {
        "parent_model": "",
        "format": "gguf",
        "family": "starcoder2",
        "families": ["starcoder2"],
        "parameter_size": "3B",
        "quantization_level": "Q4_0"
      }
    }
  ]
}
```

### 2. config.json の設定・VSCodeの再起動

Continueのタブ補完は2024-05-02時点でβ版なので、自らconfig.jsonを編集する必要があります。

私は[ドキュメント](https://docs.continue.dev/walkthroughs/tab-autocomplete)に従って次のように設定しました。

```json
  "tabAutocompleteModel": {
    "title": "Starcoder 3b",
    "provider": "ollama",
    "model": "starcoder2:3b",
    "apiBase": "http://localhost:11434"
  },
  "allowAnonymousTelemetry": false, # インターネット切断時の利用を想定しているため。もっとも、通信切断時にtrueで使っても特におかしくはならない。
```

設定後、VSCodeを再起動します。

### 3. タブ補完の実施とデバッグ

任意のソースコードでタブ補完が有効になっていることを確認します。  
なお、VSCodeの右下にもContinueのタブ補完の有効・無効を切り替えるスイッチがあるので、注意して下さい。

補完候補が示されない場合、タブ補完が試みられているかは次の2箇所から確認できます。

#### VSCodeのOutputタブ

VSCodeのOutputタブ（Ctrl+Shift+U）で「Continue - LLM Prompt/Completion」を選択します。  
補完が行われていれば、その内容が表示されます。

#### Ollamaのserver.log

Ollamaのアイコンをクリック > View logs で、Ollamaのserver.logのフォルダを開くことが出来ます。  

エクスプローラから直接開く場合は `%USERPROFILE%\AppData\Local\Ollama` (私の場合は `C:\Users\hiroga\AppData\Local\Ollama`) です。

補完が行われている場合、次のようなログがあるはずです。余談ですがOllamaって内部で[GIN](https://github.com/gin-gonic/gin)(Go言語で書かれたWebサーバー)を使ってるんですね。

```log
[GIN] 2024/05/02 - 10:44:09 | 200 |    1.2802042s |       127.0.0.1 | POST     "/api/generate"
```

## まとめ

APIやログを見ながら、ContinueとOllamaによるプログラミング環境を整えることができました。
