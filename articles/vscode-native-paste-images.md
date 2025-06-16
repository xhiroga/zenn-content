---
title: "VSCodeのmarkdown.copyFiles設定の詳細な挙動"
emoji: "📸"
type: "tech"
topics: ["vscode", "markdown"]
published: true
---

## TL;DR

- VSCode v1.78（2023年4月）時点で　`Paste Image` 拡張機能は不要になっていました
- VSCode v1.79（2023年5月）からは `markdown.copyFiles.destination` の設定が `experimental`ではなくなりました
- Markdownのファイル名を画像のフォルダ・ファイル名に用いるなど、柔軟に設定できます

## 動機

VSCodeを長く使っている方は、画像の貼り付けに Paste Image 拡張機能を使っているかもしれません。しかし、VSCode v1.79（2023年5月）以降では標準機能で同じことができるようになりました。

## すること

VSCodeの標準機能を使って、Markdownファイルに画像を貼り付ける設定を行います。Paste Image拡張機能と同等以上の柔軟な設定が可能です。

## やり方

### 基本的な使い方

VSCode v1.79（2023年5月）以降では、Markdownファイルを編集中に画像をクリップボードからペーストすると、自動的にワークスペース内にコピーされます。[^vscode_v179_update]

[^vscode_v179_update]: <https://code.visualstudio.com/updates/v1_79#_copy-external-media-files-into-workspace-on-drop-or-paste-for-markdown>

### 画像の保存先を設定する

`settings.json` に次のような設定を追加することで、画像の保存先をカスタマイズできます。

```json
{
  "markdown.copyFiles.destination": {
    "**/*": "${documentDirName}/attachments/${documentBaseName}.${fileExtName}"
  }
}
```

この場合、`docs/manual.md`に対して貼り付けた画像は`docs/manual.png`となります。

### 特定のプロジェクトで異なる設定を使う

複数のパターンを指定できます。より詳細なパターンが優先されます。

```json
{
  "markdown.copyFiles.destination": {
    "zenn-content/**/*": "${documentWorkspaceFolder}/images/${documentBaseName}.${fileExtName}",
    "**/*": "${documentDirName}/attachments/${documentBaseName}.${fileExtName}"
    # 順番を入れ替えても詳細な方が優先される（っぽい）
  }
}
```

この設定では、`zenn-content` フォルダ内のMarkdownファイルは `images` フォルダに、それ以外は各ドキュメントの `attachments` フォルダに画像が保存されます。

（もっとも、同様のユースケースではワークスペースごとにVSCodeの設定を置いた方が良さそうです）

### 画像の重複を自動的に処理する

同じ名前の画像を貼り付けた場合、VSCodeは自動的に `image-1.png` のようにリネームします。[^vscode_overwrite]

[^vscode_overwrite]: <https://code.visualstudio.com/updates/v1_79#_markdowncopyfilesoverwritebehavior>

## まとめ

VSCode v1.79（2023年5月）以降では、Paste Image拡張機能を使わなくても標準機能で画像の貼り付けができるようになりました。設定も柔軟で、プロジェクトごとに保存先を変えることも可能です。
