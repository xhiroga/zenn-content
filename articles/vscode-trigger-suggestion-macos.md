---
title: "VSCodeのTrigger Suggestionの紹介と、macOSで有効化するための工夫"
emoji: "🐘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vscode", "typescript"]
published: true
---

VSCodeでTypescriptを書くときに、「一文字も入力していないんだけど、予測変換が出てほしいな...」と思うことはありませんか？  
それ、`Trigger Suggestion` で可能です。macOSユーザーで「何それ？」って人は、私のようにOSのショートカットと衝突しているせいで見逃しているかもしれないです。

## デモ

`Ctrl + Space`でショートカットを表示しています。

![image.gif](https://i.gyazo.com/35737b7708cdf79a1750bb6e720e1c72.gif)

## Trigger Suggestionについて

VSCodeには `Trigger Suggestion`というショートカットがあります。  
いまVSCodeを開いている人は、`Cmd + Shift + P` でコマンドパレットを開いて、`Trigger Suggestion`と打ち込んでみてください。

![image.gif](https://i.gyazo.com/417a9be3034d3bf2cab2b9eb07610eb5.gif)

実行すると、プロパティのはじめの一文字を入力しなくても予測変換が表示されると思います。

## ショートカット

`Trigger Suggestion`にはデフォルトで3つのショートカットが設定されています。

 - `Ctrl + Space`
 - `Option + Ecs`
 - `Command + |`

このうち一番押しやすいであろう `Ctrl + Space` は、macOSだと入力ソースの切り替えに割り当てられています。そのせいで気づかなかったのかもしれませんね。  

以下のGitHubのIssueにも同じように引っかかった人がおり、システム環境設定で入力ソースの切り替えのショートカットをOFFにする方法も載っています。ご参考まで。  
[Trigger suggestion shortcut on Mac not working · Issue \#85643 · microsoft/vscode](https://github.com/microsoft/vscode/issues/85643)

本当に豆知識ですが、入力ソースの切り替えのショートカットをコマンドラインから無効化することも可能です（自分以外のマシンで試したことがないので、自己責任でお願いします）  
...もうちょっと良い書き方があるかもしれませんが、それはむしろ教えて下さい。

```shell
defaults write com.apple.symbolichotkeys AppleSymbolicHotKeys -dict-add 60 "<dict><key>enabled</key><false/><key>value</key><dict><key>parameters</key><array><integer>32</integer><integer>49</integer><integer>262144</integer></array><key>type</key><string>standard</string></dict></dict>"
defaults write com.apple.symbolichotkeys AppleSymbolicHotKeys -dict-add 61 "<dict><key>enabled</key><false/><key>value</key><dict><key>parameters</key><array><integer>32</integer><integer>49</integer><integer>786432</integer></array><key>type</key><string>standard</string></dict></dict>"
```

なお個人的には、`Option + Ecs`で十分かな、と思っています。

## まとめ

ショートカットからの呼び出しで、プロパティの一文字目を入力することなく予測変換を呼び出せるようになりました。

