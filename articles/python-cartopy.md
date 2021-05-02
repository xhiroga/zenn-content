---
title: "Pythonで地理情報のマッピング！cartopy使ってみた"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Python","cartopy"]
published: true
---
Pythonで地図にデータをマッピングできるライブラリの[cartopy](http://scitools.org.uk/cartopy/docs/v0.15/index.html)を使おうとしたらハマったのでやり方を残しておきます。
Anacondaを使うと簡単にインストールできるらしいのですが、この記事は素のPythonでやる人向けです。

# セットアップ
cartopy自体はpipでインストールできるのですが、別言語で書かれたライブラリを使ったりして依存関係が複雑なのでメモ🖋  
大まかな流れは...
0. Xcode’s command-line の確認
1. PROJ.4のインストール
2. GEOSのインストール
3. shapelyおよびcartopyのインストール（但しバイナリファイルを除く）

まずは[公式インストールガイド](http://scitools.org.uk/cartopy/docs/latest/installing.html#installing)をみてね。

## 0. Xcode’s command-line の確認
ガイドには記載がないけどハマったのでメモ。
macOSをアップデートした際にコマンドラインツールが正常にインストールされていないことがあるらしい。
[stdio.h file not found error on macOS Sierra - GitHub](https://github.com/frida/frida/issues/338)

解決するには、
```
$ xcode-select --install
```
でOK。

## 1.PROJ.4のインストール
[PRO4.J](http://proj4.org/)は緯度経度を扱うためのライブラリ。
UNIXのプログラムのビルドに詳しい人ならなんてことないのかもしれないけど、自分はAutoToolの使い方をしらなかったので困りました。

まずは[PRO4.Jのガイド](https://github.com/OSGeo/proj.4)を読んで、GitHubからソースを取得。
その場合自分でコンパイル→インストールする必要があるので...

1. pro4.j のディレクトリ内で autogen.sh を実行。
2. ./configure コマンドでmakeコマンドのための設定ファイルを作成
3. make → make install でコンパイルしてインストール。

## 2. GEOSのインストール
[GEOS](https://trac.osgeo.org/geos/)は（多分）3Dの描画をするライブラリ。

こちらは初めからconfigureが用意されているので、
./configure → make → make install でOK。

## 3.　shapelyおよびcartopyのインストール（但しバイナリファイルを除く）
cartopyの依存ライブラリがDLL地獄（依存ライブラリのバージョンが違うことによる問題）に陥っているらしく、pipでインストールする際にはバイナリファイルを除く必要がある。以下も参照。
[cartopy possibly kills the kernel when using ipython/jupyter notebook - GitHub](https://github.com/SciTools/cartopy/issues/738)

1. すでにshapely, cartopyがインストールされている場合は削除。  
```
pip uninstall shapely
pip uninstall cartopy
```
2. バイナリファイルを除くオプションを付けてインストール  
```
pip install shapely cartopy --no-binary shapely --no-binary cartopy  
```
セットアップ手順は以上です。

# 使ってみる
[公式サイト](http://scitools.org.uk/cartopy/docs/v0.15/matplotlib/intro.html)のソースコードを引用。

```
import cartopy.crs as ccrs
import matplotlib.pyplot as plt

ax = plt.axes(projection=ccrs.PlateCarree())
ax.coastlines()

plt.show()
```
結果  
![Screen Shot 2017-12-10 at 16.15.32.png](https://qiita-image-store.s3.amazonaws.com/0/96286/76fd287c-5f7d-1839-f38c-457ab922dcdd.png)

これは一番シンプルなやつだけど、色も形も幅広く変えられそうなので楽しみ！

