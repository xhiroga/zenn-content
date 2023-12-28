---
title: "Stable Diffusionで獣人化するための試行錯誤"
emoji: "🐕"
type: "tech" # 迷う...
topics: ["stablediffusion", "automatic1111"]
published: true
---

Stable Diffusionで、自分の顔写真を元に獣人になるための試行錯誤です。
![生成結果（厳選したもの）](/images/2023-12-28-selected.png)

## TL;DR

- 服装や背景まで含めた印象にこだわるなら、全体を生成する `img2img`
  - `Denoising strength`でAIに自由に描かせ、プロンプトで服装や構図崩れを予防する
  - `Denoising strength`でAIに自由に描かせ、Negative Promptで人間から離れさせる
  - 低解像度画像で予想外のオブジェクトを減らす
- 動物顔の打率を上げたいなら、顔をゼロから生成する`Inpaint`
  - `Mask blur`＋低解像度画像の組み合わせで、Inpaint以外の箇所も一応描きなおせる
- 部分ごとに`Denoising strength`を変える方法は発見できなかった


## 動機

AI関連のDiscordコミュニティに参加するうちに、顔が写真だと浮くな...と思い始めました。とはいえ、アイコンは一つの方が知り合いに分かりやすいので、間をとってプロフィール写真をベースにした獣人になることにしました。  
服装や画像全体の印象は元のまま獣人になりたかったので、Stable Diffusionの`img2img`を使うことにしました。

なお、最終的には手書きするので、あくまでアイデア出しが目的です。そのため仕上げのテクニックには言及していません。Stable Diffusionだけで仕上げるなら、神Seedを引いてからさらに調整するのがいいと思います。

ちなみに元のプロフィール写真はこちらです。

![@xhiroga](/images/hiroga.jpg)

## 服装や背景まで含めた印象にこだわるなら、全体を生成する `img2img`

`img2img`では、服装や背景までプロンプトで指定することができるので、元画像の印象を残したまま獣人化できます。

（再掲）`img2img`で生成した結果
![生成結果（厳選したもの）](/images/2023-12-28-selected.png)

逆に、顔の部分が獣人化されないことがあります。上段中央に人間がいますね。
![Alt text](/images/2023-12-28-grid-0148.png)

後述するように`Denoising strength`を引き上げることで対策が可能ですが、それでも人間を引く確率が50%→20%程度に下げる程度の効果なので、最後には人間が選別する必要がありますね。

### `Denoising strength`でAIに自由に描かせ、プロンプトで服装や構図崩れを予防する

`Denoising strength`とは、元画像にどれだけ忠実に描くかです。  
獣人化のハードルは元画像に引っ張られて人間になってしまうことなので、`Denoising strength`を高くすればそれを防げます...が、代わりに独自の服装や構図が現れます。

`Denoising strength`を1.0にすると、元画像はすべて無視されます。
![Alt text](/images/2023-12-28-xyz_grid-0003-1000192234.png)

したがって、`Denoising strength`を高めにして、服装や構図は以下のようにプロンプトで指定しました。

```stable diffusion prompt
a character illustration of anthropomorphic goat wearing blue shirt with smiling, looking at the camera, 
```

### `Denoising strength`でAIに自由に描かせ、Negative Promptで人間から離れさせる

ここは賛否別れそうですが、<del>たくさんあるとお得な気持ちになるので</del> 生成速度に影響しないので実績あるNegative Promptは全部入れました。  
獣人限定のテクニックですが、特に人体のパーツをNegative Promptで狙い撃ちするのは効果があったと思います。

```stable diffusion prompt
(human:1.5), (person:1.5), (people:1.5), man, woman, boy, girl,
human face,
hair, long hair, medium hair, short hair,
beard, moustache, stubble, sideburns,
eyebrow, (human ear:1.5), (earlobe:1.5), (ear hole:1.5), human nose, nostril, human mouth, human tooth, lips, skin, hairless, smooth skin,
```

感想ですが、獣人を獣人たらしめているのは何か、という本質的な問いができるのでいいですね。耳たぶ（earlobe）や唇（lips）のウェイトは大きいと思います。

### 低解像度画像で予想外のオブジェクトを減らす

意外に効果があった手法です。AIが余計な細部に囚われなくなります。  
元画像を無視するパラメータには`Denoising strength`もあります。人間のお絵描きに例えるなら、`Denoising strength`は「元画像と何度見比べるか」、解像度は「元画像を薄目で見るか」といった違いがあると解釈しています。

低解像度の画像（16x16px）から生成
![低解像度の画像（16x16px）からヤギ獣人を生成](/images/2023-12-28-xyz_grid-0003-1000192234.png)
オリジナルの解像度（332x332px）から生成
![オリジナルの解像度（332x332px）からヤギ獣人を生成](/images/2023-12-28-xyz_grid-0002-1000192234.png)

オリジナルの解像度では、顔部分を人間以外に見れなくなってしまったのか、ヤギは肩の後ろから生成されています。  
なお、解像度が低すぎると色味が意図しない感じになるので、実際の解像度は何パターンか試して決めるのがよさそうです。

## 動物顔の打率を上げたいなら、顔をゼロから生成する`Inpaint`

`Inpaint`を使うことで、プロンプト通りに動物の顔を生成できます。
![Inpaint](/images/2023-12-28-inpaint.png)

ただし、単に`Inpaint`を使うだけでは、`Mask blur`を設定しても服装や背景と顔がなじみません。そこで、再び低解像度画像を用います。

### `Mask blur`＋低解像度画像の組み合わせで、Inpaint以外の箇所も一応描きなおせる

オリジナル画像の顔部分を`Inpaint`の対象に取ってみました。`Mask blur`は`64`(AUTOMATIC1111で設定できる最大値)に設定していますが、コラ画像感が半端ないですね。
![オリジナル画像でInpaint](/images/2023-12-28-inpaint-from-original.png)

そこで、元画像を低解像度にしてみます。すると、いい感じになりました。
![低解像度画像でInpaint](/images/2023-12-28-inpaint-from-lowres.png)

## 部分ごとに`Denoising strength`を変える方法は発見できなかった

この記事でやりたいことは、要するに「顔は`Denoising strength`高め、服装と構図は`Denoising strength`低め」です。

そこで、顔の部分を透過PNGにして、服装と構図をそのままにすることで、顔の部分だけ獣人にならないかと思いました。結論としては、透過PNGの透明部分が白として認識されるので、サモエドとか羊を生成したいならアリかもしれません。

顔部分を透過させます。
![hirogaの顔部分を透過させる](/images/2023-12-28-transparent.png)

透過部分が白として認識されちゃいました。
![透過部分が白として認識された生成結果](/images/2023-12-28-transparent-white.png)

もしかしたら生のStable Diffusionを使ったり、他のクライアントを使ったり、拡張機能を開発すればクリアできるかもしれませんが、今後の課題ですね。

## まとめ

Stable Diffusionを使って、自分の顔写真を元に獣人に慣れることを示しました。  
また、求めている画像の方向性によって`img2img`と`Inpaint`を使い分けることの有用性や、あえて低解像度を用いることでAIの描き方をコントロールできることを示しました。

誰かのお役に立てば幸いです。バッジを贈るもお待ちしています☺
