---
title: "OpenStreetMapの地形データをUnityで使える3Dデータとして書き出す方法"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Unity","OpenStreetMap"]
published: true
---
ハッカソン中に試行錯誤してたどり着いたやり方をまとめておきます。

# 概要
1. 地図の任意の範囲の.osmファイルを出力する
2. OSM2Worldを使って.objファイル形式に変換する


# 1. 地図の任意の範囲の.osmファイルを出力する
http://www.openstreetmap.org/ にアクセスし、任意の範囲を選択してから「エクスポート」します。

![Screenshot 2017-08-21 06.43.30.png](https://qiita-image-store.s3.amazonaws.com/0/96286/1c9d06d7-8ec4-918d-2a80-351aadf6be64.png)


# 2. OSM2Worldを使って.objファイル形式に変換する
OSM2Worldをインストール→起動し、osmファイルを取り込みます。
http://wiki.openstreetmap.org/wiki/JA:OSM2World の記述を参考にすると良いです。

起動画面
![Screenshot 2017-08-21 06.47.53.png](https://qiita-image-store.s3.amazonaws.com/0/96286/01e714c8-f97b-e471-64dc-2a164497d8d6.png)

OSMファイルを開いたところ(この後、File > Export OBJ file を選択)
![Screenshot 2017-08-21 06.58.57.png](https://qiita-image-store.s3.amazonaws.com/0/96286/be679c44-7c30-8fc9-ffb8-eca397fa186c.png)

Unityに取り込むとこんな感じです。
![Screenshot 2017-08-21 06.53.00.png](https://qiita-image-store.s3.amazonaws.com/0/96286/4450ba3d-e328-1016-c247-7ac26f4cae05.png)


# 今後の課題
建物同士がオブジェクトとして分かれていない。
UV値が全てゼロに設定されているのでうまく画像を貼り付けられない。
解消する方法がありそうな気がするんですが、知ってる人がいたらぜひ教えて下さい。


# 参考にしたサイト
http://wiki.openstreetmap.org/wiki/JA:3D#.E3.82.A8.E3.82.AF.E3.82.B9.E3.83.9D.E3.83.BC.E3.83.88

その他にこんなやり方があるようです。
[ActionStreetMap](http://actionstreetmap.github.io/demo/)
こちらは地図を動的に生成可能。
[GoMap](https://www.assetstore.unity3d.com/jp/#!/content/68889)
[MapBox](https://www.mapbox.com/unity/)

