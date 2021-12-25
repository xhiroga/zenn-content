---
title: "defaultsの歴史"
emoji: "🔖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["macos", "defaults", "PlistBuddy", "plutil", "dotfiles"]
published: true
---

macOSの設定を編集できるコマンド `defaults` の歴史について、調べたことをまとめました。
私も勉強中ですので、間違っている点や疑問があればぜひ教えて下さい。

## 年表

| 年 | イベント |
| --- | --- |
| 1985 | Steve JobsがAppleを退職し、NeXT Computerを創業 |
| 1989 | NeXTSTEP 1.0のリリース |
| 1990-12-25 | 世界初のブラウザ world-wide-web が NeXT Computerで開発される |
| 1993-05-21 | GNUstepの前身の一つである、Collection Library for GNU Objective-Cのα版が公開される |
| 1994 | NeXT Computer と Sun Systemsが共同でOPENSTEPの仕様を公開する |
| 1995 | OPENSTEP仕様の公開を受け、同様の取り組みをしていた開発者が合流してGNUstepプロジェクトが発足する |
| 1996 | AppleがNeXT Computerを買収。Steve JobsもAppleに戻る |
| 1997-08-31 | 後のOS X（macOS）である RhapsodyのDeveloper Releaseがリリースされる |
| 2001-03-22 | OS Xが販売開始される |
| 2002 | GNUstepがリリースされる |

## defaultsの歴史

`defaults` コマンドって何？という方は、以下のコマンドを実行してみて下さい。

```shell
defaults read com.apple.dock orientation
> bottom # bottom, right or left
```

Dock（アプリのアイコンが並んているバー）の位置が表示されないでしょうか？
このように、`defaults`コマンドではmacOS上のアプリケーションのユーザー設定を表示・編集することができます（ユーザーだけでなく、マシン固有の設定も編集できます）

これらの設定の本体は通常 `~/Library/Preferences/com.apple.dock.plist` にバイナリ形式で格納されており、`defaults`はそれを編集するためのユーティリティというわけです。

### NeXTSTEP時代

プログラミング以外の方法でUserDefaultsを編集するために、当時3種類のコマンドラインツールが提供されていました。NeXTのコンピュータ・OS・ソフトウェアを扱っていた雑誌 "NeXT WORLD"の寄稿から引用します。

> In addition to the programmer's interface, NeXTstep provides three command line programs **dread, dwrite, and dremove** that give you access to the defaults system. To see all of your defaults, type:
> 
> -- <cite>[Using NeXTstep's Defaults System by Ray Ryan](http://www.nextcomputers.org/NeXTfiles/Articles/NeXTWORLD/92.2/92.2.Summer.HowToU4.html)</cite>

ちなみに、Internet Archiveから当時の誌面を読むことができます。

![](/images/2012-12-26-internet-archive-next-world.png)

[NeXTWORLD Vol\. 2 No\. 2 Summer 1992 : Free Download, Borrow, and Streaming : Internet Archive](https://archive.org/details/NeXTWORLDVol.2No.2Summer1992/page/n95/mode/2up)


### OPENSTEP時代

Rhapsodyの開発者向けドキュメントによれば、当時から`defaults`コマンドは存在したようです。

> Name: defaults  
> Description: Reads, writes, searches, and deletes user defaults. The defaults system records user preferences that persist when the application isn’t running. When users specify defaults in an application’s Preferences panel, NSUserDefaults methods are used to write the defaults.  
> Location: /usr/bin
> 
> -- <cite>[DISCOVERING OPENSTEP: A Developer Tutorial](http://www.nextcomputers.org/NeXTfiles/Software/OPENSTEP/Info/discovering.pdf)</cite>

ちなみに Big Sur現在でもパスは変わっていません。

```shell
% which defaults
/usr/bin/defaults
```

GNUstepのドキュメントによれば、やはりNeXTSTEP時代の`dread`, `dwrite`, `dremove` の3つのコマンドの機能を併せ持つとのことです。

> The 'defaults' command appeared in OpenStep and combined the capabilities of the earlier NeXTstep commands 'dread', 'dwrite', and 'dremove'.
> 
> -- <cite>[defaults\(1\) — gnustep\-base\-runtime — Debian stretch — Debian Manpages](https://manpages.debian.org/stretch/gnustep-base-runtime/defaults.1.en.html)</cite>

### GNUstep

[GNUstep](https://github.com/gnustep/)はオープンソースなので、defaultsについてもソースを読むのが早いと思います。
GNUstep自体の歴史は[GNUstep History](http://gnustep.made-it.com/Guides/History.html)がとても詳しかったです。

### 現在

defaultsは隠しコマンドと呼ばれることもありますが、現在でも[Appleの開発者向けドキュメント](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/UserDefaults/AboutPreferenceDomains/AboutPreferenceDomains.html)に記載があります。隠し設定の編集もできるコマンド、の方が正確ですね。

ちなみに設定可能な項目は、[Dock](https://developer.apple.com/documentation/devicemanagement/dock)のように一部のプロパティを公開しているアプリを除けば見つけられないものがほとんどでした。
人によって様々な探し方がありますが、以下が参考になると思います。

- [OS X ハッキング\!\(315\) stringsコマンドで裏オプション探し \| マイナビニュース](https://news.mynavi.jp/article/osx-315/)
- [Macの設定を自動化するdefaultsコマンドと、それを助けるpdef \- memo\.yammer\.jp](https://memo.yammer.jp/posts/pdef)
- [Mac を買ったら必ずやっておきたい初期設定を、全て自動化してみた](https://zenn.dev/ulwlu/articles/1c3a1da12887ed)


## 参考文献

- [NeXTSTEP \- Wikipedia](https://en.wikipedia.org/wiki/NeXTSTEP)
- [Welcome To The NeXT World](http://www.nextcomputers.org/)
- [GNUstep \- Wikipedia](https://en.wikipedia.org/wiki/GNUstep)
- [GNUstep History](http://gnustep.made-it.com/Guides/History.html)
- [GNUstepはOpenStepの代替フリーソフトウェア](https://www.codereading.com/nb/gnustep.html)
- [Mac OS X、今週末から販売開始 \- Apple \(日本\)](https://www.apple.com/jp/newsroom/2001/03/21Mac-OS-X-Hits-Stores-This-Weekend/)
- [第3回　plist（プロパティリスト）とFoundation【前編】：Undocumented Mac OS X（1/4 ページ） \- ITmedia エンタープライズ](https://www.itmedia.co.jp/enterprise/articles/0705/14/news013.html)
- [Property Lists, Preferences and Profiles for Apple Administrators](https://books.apple.com/us/book/property-lists-preferences-and-profiles-for-apple/id1213903756)
- [Defaults](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/UserDefaults/AboutPreferenceDomains/AboutPreferenceDomains.html)