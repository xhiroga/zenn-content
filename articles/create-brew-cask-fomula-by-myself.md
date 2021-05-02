---
title: "brew caskで欲しいバージョンのfomulaがないのでtap自作する"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["homebrew"]
published: true
---

Virtualboxの5.2をインストールしたいのですが、2019-12-27現在 `brew install virtualbox` すると 6.1がインストールされてしまいます。  
tapを自作して解決しましょう。

# 1. 公式のCask Codeを参照する
https://formulae.brew.sh/cask/virtualbox  
https://github.com/Homebrew/homebrew-cask/blob/master/Casks/virtualbox.rb  

# 2. Tapリポジトリを公開する

やり方はこちらを参考にしました。  
https://qiita.com/tarappo/items/8d6cb62be3b91f0be583

dmgファイルのダウンロードURLを公式サイトから控え、バイナリの SHA256を算出した後、公式のCask Codeをコピペして置換した結果がこちらです。  
https://github.com/hiroga-cc/homebrew-virtualbox-5.2

## ハマりどころ
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/44f4285e-14c7-d79d-8fbc-3251082ddecf.png)

ルートディレクトリに .rbファイルを作成するのではなく、 `プロジェクトルート/Casks/***.rb` の構成で作る必要があります。  
誤ってルートディレクトリに作成した場合は以下のエラーが発生し、私は原因特定まで1時間を費やしました...。

```
$ brew tap hiroga-cc/virtualbox-5.2
==> Tapping hiroga-cc/virtualbox-5.2
Cloning into '/usr/local/Homebrew/Library/Taps/hiroga-cc/homebrew-virtualbox-5.2'...
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 4 (delta 0), reused 4 (delta 0), pack-reused 0
Receiving objects: 100% (4/4), done.
Error: Invalid formula: /usr/local/Homebrew/Library/Taps/hiroga-cc/homebrew-virtualbox/virtualbox-5.2.rb
virtualbox: undefined method `cask' for Formulary::FormulaNamespacead380ff8d454670d2504ca62549826bd:Module
Error: Cannot tap hiroga-cc/virtualbox: invalid syntax in tap!
```

（Ruby分かる人だと即解決できたりするんですかね？）

# 3. インストールする

```
brew tap hiroga-cc/virtualbox-5.2
brew cask install virtualbox-5.2
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/1d4647ea-3134-aef8-8825-d7995c7064ed.png)
やったぜ！完了。

