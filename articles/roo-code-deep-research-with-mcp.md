---
title: "Roo Code(Cline)でDeep Researchする"
emoji: "🔍"
type: "tech"
topics: ["roocode", "cline", "mcp", "claude", "ai", "llm"]
published: true
---

## TLDR

- Roo Code上でDeep Researchのような検索ができて非常に便利
- MCPサーバーで検索(Brave Search)・Web閲覧(Fetch)を行う

## デモ

まずはデモをご覧ください。

https://youtu.be/wvF1nui1bAc

## そもそもMCPって？

MCP（Model Context Protocol）は、AIアシスタントとデータソースを接続するためのオープンスタンダードプロトコルです。2024年11月にAnthropicによってオープンソース化され、AIモデルが外部システムと安全に通信できるようにします。

MCPの公式サイトから、公開されているMCPサーバーを参照できます。
https://mcp.so/

## Brave Search & Fetch とは？

### Brave Search

Brave SearchはBrave Browserを開発している会社が提供する検索エンジンAPIです。2024年11月にコミュニティによってMCPサーバーの実装が公開されています（[Pull Request](https://github.com/modelcontextprotocol/servers/pull/15)）

### Fetch

FetchはウェブページのコンテンツをMarkdown形式で取得するシンプルなMCPサーバーです。

## Roo Codeってブラウザを使えるのでは？MCPは必要？

必要です。速度が全然違います。といっても、Fetchではレンダリングができないサイトなどでブラウザを使うなど、棲み分けがありそうな気がします。

## 設定方法

### 1. MCPサーバーの設定

最低限、MCPサーバーの設定をすれば使えます。動画もあるのでご参考まで。

https://youtu.be/La_UCAMLvgk

Roo CodeのタブからMCP Serversを開き、`Edit MCP Settings`をクリックして`cline_mcp_settings.json`を開いて、次のとおり編集します。

```json
{
  "mcpServers": {
    "brave-search": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "BRAVE_API_KEY",
        "mcp/brave-search"
      ],
      "env": {
        "BRAVE_API_KEY": "***"
      },
      "alwaysAllow": [
        "brave_web_search"
      ]
    },
    "fetch": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "mcp/fetch"
      ],
      "alwaysAllow": [
        "fetch"
      ]
    }
  }
}
```

注意点として、2025-03-15現在、私の環境ではランタイムが`npx`形式だと動きません。なので私は`docker`で動かしています。

### 2. Brave Search APIキーの取得

Brave SearchのAPIキーは以下のサイトから登録・取得できます：
https://api-dashboard.search.brave.com/app/keys

### 3. カスタム指示の設定

Roo CodeはMCPに登録しただけだとBrave SearchやFetchを使わないことがあるので、Custom Instructions for All Modelsに以下を追加しました。

```txt
Use "brave-search" MCP as default search. Use "fetch" MCP as default web viewer
```

## まとめ

これまではコーディング中のトラブルシュートのためにソースコードをo1 Proに貼り付けていましたが、その手間がなくせる可能性を感じており、非常に興奮しています！  
プロンプトの工夫次第では実際にDeep Research並の性能も狙えると思うので、ぜひ皆さんも使ってみて感想を教えてください！
