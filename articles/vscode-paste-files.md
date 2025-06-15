# VSCodeで
（なんか細かすぎるTIPS的な感じの）

NOTE: この記事はChatGPT o3で清書しています。清書前の記事は[ここにコミットID]()から確認できます。

TLDR

- Paste Image拡張機能は不要になったよ
- `markdown.copyFiles.destination` はexperimentalではなくなったよ

VSCodeを長く使っている方は、Ctrl+ Option + Vで画像の貼り付けができる Paste Imageを使っている方がいるかもしれません。

https://github.com/mushanshitiancai/vscode-paste-image


https://code.visualstudio.com/updates/v1_79#_copy-external-media-files-into-workspace-on-drop-or-paste-for-markdown

TIPS

https://code.visualstudio.com/updates/v1_79#_markdowncopyfilesoverwritebehavior
公式ドキュメントにあるように、画像を複数枚貼り付けると自動で image-1のようにリネームする


次のどっちでも zenn-content の場合は images 以下に画像が配置される。詳細な記述が勝つルールと思われる

なお、Workspace機能で複数リポジトリ等を単一Workspaceに束ねて表示している場合でも、documentWorkspaceFolderは問題なくリポジトリルート等を指す


```
"**/*": "${documentDirName}/attachments/${documentBaseName}.${fileExtName}",
"zenn-content/**/*": "${documentWorkspaceFolder}/images/${documentBaseName}.${fileExtName}",
```

または

```
"zenn-content/**/*": "${documentWorkspaceFolder}/images/${documentBaseName}.${fileExtName}",
"**/*": "${documentDirName}/attachments/${documentBaseName}.${fileExtName}",
```


個人的なTIPS

- Obisdianと同期しています
- 年に1度程度フォルダの移動をするので、In subfolder under current folderの設定にしています
- ObsidianのデフォルトのSubfolder nameがattatchments なので、それに合わせています。
