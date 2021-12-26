---
title: "defaults, PlistBuddy, plutilをdockerで利用する"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["macos", "defaults", "PlistBuddy", "plutil", "dotfiles"]
published: true
---

## TL;DR

dotfilesの設定に対してテストを実行したかったので、`defaults`, `PlistBuddy`, `plutil`をdockerで動かせるようにしました。

```shell
docker run --rm -it --entrypoint bash ghcr.io/xhiroga/defaults-plutil:latest
```

- イメージ: [Package defaults\-plutil](https://github.com/xhiroga/docker-images/pkgs/container/defaults-plutil)
- ソースコード: [docker\-images/Dockerfile at main · xhiroga/docker\-images](https://github.com/xhiroga/docker-images/blob/main/defaults-plutil/Dockerfile)

## `defaults`

GNUstepのおかげで、パッケージをインストールするだけで動かせました。

```shell
apt update && apt install gnustep-base-runtime -y
```

GNUstepやdefaultsの歴史については、拙稿 [defaultsの歴史](https://zenn.dev/hiroga/articles/defaults-command-history) をご覧ください。

ちなみに、macOS実装とGNUstep実装では細かい挙動の差異があります。

### オプションの違い

GNUstepは値の引数に型を取ることができません。macOSの方が充実している印象です。

```shell
# macOS
% defaults write com.apple.dock autohide -bool false
% defaults read com.apple.dock autohide
0

# GNUstep
% defaults write com.apple.dock autohide -bool false
% defaults read com.apple.dock autohide
com.apple.dock autohide -bool
```

違いはそれぞれのマニュアルからも分かります。

**GNUstep**

> defaults write domain key value
> 
> write 'value' as default 'key' in the specified domain. 'value' must be a property list in single quotes.
> 
> -- <cite>[defaults\(1\) — gnustep\-base\-runtime — Debian stretch — Debian Manpages](https://manpages.debian.org/stretch/gnustep-base-runtime/defaults.1.en.html)</cite>

**macOS**

> Command line interface to a user's defaults.  
> Syntax:
> 
> 
> 'defaults' [-currentHost | -host <hostname>] followed by one of the following:
> 
> （省略）
> 
>   write \<domain> \<domain_rep>          writes domain (overwrites existing)  
>   write \<domain> \<key> \<value>         writes key for domain
> 
> （省略）
> 
> \<value> is one of:  
> \<value_rep>  
> -string \<string_value>  
> -data \<hex_digits>  
> -int[eger] \<integer_value>  
> -float  \<floating-point_value>  
> -bool[ean] (true | false | yes | no)  
> -date \<date_rep>  
> -array \<value1> \<value2> ...  
> -array-add \<value1> \<value2> ...  
> -dict \<key1> \<value1> \<key2> \<value2> ...  
> -dict-add \<key1> \<value1> ...


### Preference Domainの検索パスの違い

macOS実装では `~/Library/Preferences/identifier.plist`が検索対象になるのに対して、GNUstep実装をUbuntuで動かした場合は `~/GNUstep/Defaults/identifier.plist` が検索対象になります。  
（もしかしたらGNUstepをビルドした環境や実行環境によって変わるかもしれません）

## `PlistBuddy` & `plutil`

クロスプラットフォームな実装が配布されているため、cloneしてbuildすることで動かすことができました。
[screenplaydev/plutil: A cross\-platform, drop\-in replacement for Apple's plutil\.](https://github.com/screenplaydev/plutil)

2016年頃にFacebookが発表した[xcbuild](https://github.com/facebookarchive/xcbuild)というXCodeの互換ツールがあり、そのコードを移植したプロジェクトとのことです。

### 苦労した点

ライブラリが正しくインストールされないバグのため、当初`PlistBuddy`が動きませんでした。  
Issueを起票しつつ、手動でinstallすることで一応解決。

[liblinenoise\.so was built but not installed\. · Issue \#6 · screenplaydev/plutil](https://github.com/screenplaydev/plutil/issues/6)

## まとめ

`defaults` や `PlistBuddy`, `plutil` がテストできるようになり、とても嬉しいです。  
皆様も充実した dotfiles ライフをお過ごし下さい。
