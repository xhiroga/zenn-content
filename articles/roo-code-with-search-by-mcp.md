（後で記事にする）

TLDR
- Roo Code上でDeep Reseachのような検索・レポート作成ができる
- MCPサーバーで検索・Web閲覧を行う
- Brave Searchとfetchを使う

デモ

https://youtu.be/wvF1nui1bAc


そもそもMCPって？

ここにあるよ
https://mcp.so/

別に元々ブラウザあるくない？
→ 速度が全然違う
→ レンダリングが必要なものはブラウザ、そうでないものはfetchで分けると良い

2025-03-15時点のトラブルシュート

- 何故かClaude desktopで動くnpxの形式だとRoo Codeでは動かない
  - clineやcursorでは試していない

2025-03-15 時点のトラブルシュート
- Allways allowが有効になるのはチェックを入れたタスクの次からっぽい

注意

MCPサーバーはコミュニティから公開されているものです
自己責任でお願いします。


設定

YouTube

https://youtu.be/La_UCAMLvgk

```
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
APIキーはここから登録
https://api-dashboard.search.brave.com/app/keys



Prompt

指定しないとブラウザで検索をする

Custom Instructions for All Modelsで指定しましょう

```
Use "brave-search" MCP as default search. Use "fetch" MCP as default web viewer
```

まとめ
そうは言ってもOpenAIのAgent SDKで検索できるらしいし
でもOperatorよりは楽かも
