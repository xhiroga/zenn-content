---
title: "VirtualBox on macでクリップボードの共有が有効にできなかった時の対応"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Ubuntu","VirtualBox"]
published: true
---

# TL;DR
Guest AdditionsはVirtual Boxのバージョンに合ったものを導入しましょう。


# 概要
VirtualBoxで起動したゲストOSでは、Guest Additionsというプログラムを読み込むことでクリップボードの共有ができるようになります。  
しかし、 `sudo apt-get install virtualbox-guest-dkms` で導入されるGuest Additionsは最新とは限らず、前準備も必要です。包括的な手順を見つけられなかったので、メモしておきます。


# 手順
## 1. Extension Packの導入
参考: https://www.nakivo.com/blog/how-to-install-virtualbox-extension-pack/

## 2. Guest Additionsの導入
参考: https://pc-karuma.net/virtualbox-install-guest-additions/

## 3. ゲストOSの再起動
`reboot`


