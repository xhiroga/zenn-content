---
title: ""
emoji: "💙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["blender","bpy", "python"]
published: false
---


## そもそもAdd-onじゃなくてExtensionになった背景は？

Add-onって言ったときに、ExtensionのTypeとしてのAdd-onと古い拡張機能のがあるから注意

## インストールして使うには？


https://extensions.blender.org/

## どういう管理方法になっているの？

4.3\extensions\user_default\blendier  に解凍されているっぽい
じゃあZipにする意味は...? → 不要ファイルの除去？

## ビルド

__init__.py にはbl_infoは相変わらず必要なのか？
https://docs.blender.org/manual/en/latest/advanced/scripting/addon_tutorial.html

いるっぽい

```
validate:
	/mnt/c/Program\ Files/Blender\ Foundation/Blender\ 4.3/blender.exe --debug-all --log-level -1 --command extension validate
```

なんか --verbose はエラー出る、正しい使い方は？

デプロイはDrag^Dropで良いのスゲー良い Install From Disk
https://docs.blender.org/manual/en/latest/editors/preferences/extensions.html#prefs-extensions-install

公開するにはどのくらいの審査期間がある？

module 'bl_ext.user_default.blendier' has no attribute 'register'

#### CI



#### WSL

WSLで開発することは推奨される？その場合はどのように各種指定したらよい？

→ 特にトラブルなし


## デバッグ

register() はどこで見えるの？

> Windows: Blenderのメニューバーから Window -> Toggle System Console を選択します。
https://aistudio.google.com/prompts/1bQbj-TdZA-ffMD9XZgHTV7jhHd6uYF-t

printデバッグの結果は...

Add-on not loaded: "addon", cause: No module named 'addon'
FATAL_ERROR: Error, file ".\blender_manifest.toml" not found!

基本的にはExtension内で開発することを推奨して要るっぽい

フォルダごとに設定を持たせたい場合は？

ワークスペースの概念は存在する？

動的にテーマを変えることはできる？

## 開発

型が欲しい！！！

## 運用

動的な更新は？

## References

### 日本語記事


### それ以外


## Preferenceについて...

Config以下に雑に保存することもできる


一方で、https://docs.blender.org/api/current/bpy.types.AddonPreferences.html のとおりに保存するとこんな感じ

![alt text](image.png)

内容は保存される...これは何ごとに保存されるの？
.blendファイル単位では、少なくともないらしい。

config/userpref.blend をこじ開けてみたものの、それらしい設定はなし。ほんとどこに保存してるんだ？

https://colorful-pico.net/introduction-to-addon-development-in-blender/2.8/html/chapter_03/07_Use_Preference.html
> プリファレンスに配置する情報として、適切だと考えられるものを次に示します。
アドオンの機能に割り当てる、キーボードのキーやマウスのボタン
3-4節 や 3-5節 で紹介した、独自UIにおけるフォントなどのアドオンのUI設定
文字数の関係上、bl_info に記載できなかったアドオンの詳細情報（特に、location や description は長文になることが多い）
アドオン内の機能の有効化/無効化（アドオンが複数の機能で構成される場合）
アドオン全体で共有したい設定（アドオンが複数の機能で構成される場有）

おそらくはconfig以下にフォルダを作って保存するのが正しいと思うのだがもう一押し
https://chatgpt.com/c/6812aab9-7918-8010-8c45-4dd5de619167
https://docs.blender.org/api/current/bpy.utils.html
