---
title: "LlamaGen: LlamaのNext-Token予測を使った画像生成【論文】"
emoji: "🦙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["論文", "machinelearning", "computervision", "deeplearning", "llm", "llama"]
published: true
notebook_urls:
  - LlamaGen: https://notebooklm.google.com/notebook/3b48c448-56cd-44d4-a259-7246d00f5108
  - LlamaGen: https://aistudio.google.com/prompts/1hRYVjlnhrsyim6hOgD8F23-l-O4Oe8Hj
  - 関連研究: https://aistudio.google.com/prompts/1bmhZcwjdpElxJcKqyehL7WoZEP3Uf8I9
  - ViT: https://aistudio.google.com/prompts/1Z0JvJe5WNEhPa2rKy8YcV69jVADoDt4G
  - VQGAN: https://aistudio.google.com/prompts/1Re2PEOv2zcIzic2fOejI0R5AqoH6HRk0
  - ベクトル量子化: https://chatgpt.com/c/67457c7f-84cc-8010-b422-d0a0068dd127
  - CFG: https://aistudio.google.com/prompts/1o53DQY58Yjsr4suqx63vbJnhMXxOhz4o?pli=1
  - Precision/Recall: https://aistudio.google.com/prompts/1JhKUhFlhdecNVxaz33-m31pNyjmCTd5p
---

## Autoregressive Model Beats Diffusion: Llama for Scalable Image Generation[^Sun_et_al_2024]

[^Sun_et_al_2024]: P. Sun et al., “Autoregressive Model Beats Diffusion: Llama for Scalable Image Generation,” Jun. 10, 2024, arXiv: arXiv:2406.06525. doi: 10.48550/arXiv.2406.06525.

https://arxiv.org/abs/2406.06525

なお、[fmuuly氏による](https://zenn.dev/fmuuly)「[Llamaで画像生成：LlamaGen【論文】](https://zenn.dev/fmuuly/articles/40f4863385b7d8)」も分かりやすくてオススメです。

## Note

この記事の内容を、2024-12-03に行われる [松尾研LLMコミュニティ【Paper & Hacks】#28](https://matsuolab-community.connpass.com/event/338122/) にて発表します。

https://matsuolab-community.connpass.com/event/338122/

## はじめに

近年の画像生成AIは、高品質な画像を生成できる一方で、大規模言語モデルと統合されているとは言えません。例えばChatGPTには画像生成機能がありますが、内部ではDALL-Eという別のモデルを呼び出しています。[^DALL-E3]
[^DALL-E3]: https://openai.com/index/dall-e-3/

これは、画像生成AIと大規模言語モデルのアーキテクチャが根本的に異なるためです。Stable DiffusionやDALL-E2以降で用いられる拡散モデルは、画像にノイズを徐々に加えていき、最終的に完全なノイズになった状態から、逆向きにノイズを除去していくことで画像を生成します。拡散モデルは高品質な画像を生成できますが、ノイズ除去を何度も繰り返す必要があるため計算コストが高いという課題があります。一方で、GPTに代表される大規模言語モデルは、前のトークンから次のトークンを予測することで文章を生成します。自己回帰モデルは並列処理が容易なため高速に推論できますが、高解像度の画像をピクセル列として扱うと系列長が長くなりすぎてしまい、計算量が爆発するという問題がありました。

本稿で紹介する **LlamaGen** は、LLMであるLlamaを自己回帰型画像生成モデルに応用した研究です。LlamaGenは、画像をトークン化し、そのトークン列をLlamaを用いて自己回帰的に生成することで、高品質な画像生成を実現します。将来的には、例えば図の入った画像を前処理なしで訓練に用いるようなことができるかもしれませんね。

### ソースコード

LlamaGenはソースコードと重みが公開されています。[^GitHub_LlamaGen]動かすのに非常に苦労したので、フォークして`uv`で動くように整えたリポジトリを公開します。
[^GitHub_LlamaGen]: https://github.com/FoundationVision/LlamaGen

https://github.com/xhiroga/LlamaGen/tree/chore/uv

## 関連研究

自己回帰モデルで画像を扱おうという研究は、実はこの研究が初めてではありません。むしろ、拡散モデルよりも長い歴史があります。

* **PixelCNN (2016)**[^Oord_et_al_2016a][^Oord_et_al_2016b]: 自己回帰モデルによる画像生成の先駆け的な研究。ピクセルを直接予測するモデルで、マスク畳み込みを用いて計算量を抑えながら自己回帰を実現しました。
* **ImageGPT (2020)**[^Chen_et_al_2020]: Transformer を用いた自己回帰型画像生成モデル。画像を低解像度化し、ピクセルをトークンとして扱います。大規模言語モデルの事前学習と同様に、大量の画像データで事前学習を行うことで、様々な画像生成タスクに適用できる汎用的な表現を獲得することを目指しました。
* **ViT (2020)**[^Dosovitskiy_et_al_2020]: 画像認識のための Transformer ベースのモデル。画像をパッチに分割し、各パッチをトークンとして Transformer エンコーダーに入力します。画像分類タスクで高い性能を達成し、Transformer が画像認識にも有効であることを示しました。
* **DALL-E (2021)**[^Ramesh_et_al_2021]: Transformer を用いた画像生成モデル。画像を VAE で離散トークン化し、テキストと画像のペアデータで学習します。テキストから高品質な画像を生成できることを示しました。
* **VQGAN (2021)**[^Esser_et_al_2021]: ベクトル量子化を用いた Image Tokenizer と Transformer を用いた自己回帰型画像生成モデル。高解像度画像生成において、Transformer が CNN よりも優れた性能を持つことを示しました。
* **DiT (2023)**[^Peebles_and_Xie_2023]: 拡散モデルの一種で、ノイズ除去に Transformer を用います。拡散モデルと Transformer の利点を組み合わせることで、高品質な画像生成を実現しました。

[^Oord_et_al_2016a]: A. van den Oord, N. Kalchbrenner, and K. Kavukcuoglu, “Pixel Recurrent Neural Networks,” Aug. 19, 2016, arXiv: arXiv:1601.06759. Accessed: Nov. 28, 2024. [Online]. Available: http://arxiv.org/abs/1601.06759
[^Oord_et_al_2016b]: A. van den Oord, N. Kalchbrenner, O. Vinyals, L. Espeholt, A. Graves, and K. Kavukcuoglu, “Conditional Image Generation with PixelCNN Decoders,” Jun. 18, 2016, arXiv: arXiv:1606.05328. doi: 10.48550/arXiv.1606.05328.
[^Chen_et_al_2020]: M. Chen et al., “Generative Pretraining from Pixels,” 2020.
[^Dosovitskiy_et_al_2020]: A. Dosovitskiy et al., “An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale,” Oct. 22, 2020, arXiv: arXiv:2010.11929. doi: 10.48550/arXiv.2010.11929.
[^Ramesh_et_al_2021]: A. Ramesh et al., “Zero-Shot Text-to-Image Generation,” Feb. 26, 2021, arXiv: arXiv:2102.12092. doi: 10.48550/arXiv.2102.12092.
[^Esser_et_al_2021]: P. Esser, R. Rombach, and B. Ommer, “Taming Transformers for High-Resolution Image Synthesis,” Jun. 23, 2021, arXiv: arXiv:2012.09841. doi: 10.48550/arXiv.2012.09841.
[^Peebles_and_Xie_2023]: W. Peebles and S. Xie, “Scalable Diffusion Models with Transformers,” Mar. 02, 2023, arXiv: arXiv:2212.09748. Accessed: Nov. 07, 2024. [Online]. Available: http://arxiv.org/abs/2212.09748

LlamaGen の貢献は、大規模言語モデルのノウハウ（アーキテクチャや訓練手法など）を画像生成に適用することで、高品質な画像生成が可能になることを示した点にあります。

## LlamaGenのアーキテクチャ

LlamaGenは、主に2つのモジュールから構成されています。

1. **Image Tokenizer:** 画像を離散的なトークン列に変換するモジュール。VQGAN[^Esser_et_al_2021] で提案されたアーキテクチャをベースにしています。
2. **Llama:** トークン列を入力として受け取り、自己回帰的に次のトークンを予測することで画像を生成するモジュール。LLMであるLlamaをベースにしています。

```mermaid
graph LR
    i[画像 🏞️]
    t[テキスト・クラス 💬]
    
    i --> enc[エンコーダー<br/>量子化器]
    subgraph Image Tokenizer
    enc --コードブックを参照--> token([グリッドトークン<br/>🟦 🟩 ⬜️ 🟩 ...])
    token --> dec[デコーダー]
    end

    t --> llm[Llama 🦙]
    token --> llm
    llm --> token

    dec --> o[出力画像 🏞️]
```

## LlamaGenとViTの比較

Transformerを画像言語モデルで用いた例としては、ViT[^Dosovitskiy_et_al_2020]が有名です。

LlamaGenとViTの違いをまとめました。

| 特徴             | LlamaGen                             | ViT                                          |
| ---------------- | ------------------------------------ | -------------------------------------------- |
| 主なタスク       | 画像生成                             | 画像のクラス分類                             |
| アーキテクチャ   | GPT (=Transformer Decoder)           | Transformer Encoder                          |
| トークン化の対象 | 画像のパッチ                         | 画像のパッチ                                 |
| トークン化の方法 | ベクトルを量子化しコードブックに対応 | ベクトルを全結合層で変換しパッチ埋込みを得る |

Transformerを用いる点とトークン化の対象は同じですが、利用する型（エンコーダ・デコーダ）、タスク、トークン化の方法が異なります。

## Image Tokenizer

Image Tokenizerは、高解像度画像を効率的に処理するために、画像をトークンの系列に変換する重要なモジュールです。Transformer ベースのモデルは系列長に対して計算コストが二次関数的に増加するため、高解像度画像をピクセルごとにトークン化すると、計算量が爆発的に増加してしまいます。Image Tokenizer は、画像をより短いトークン列に変換することで、この問題を解決します。

### Image Tokenizerの関連研究

LlamaGen で用いられた Image Tokenizer は、VQGAN[^Esser_et_al_2021] で提案されたものとほぼ同じアーキテクチャです。VQGAN は、Image Tokenizer と自己回帰モデルを組み合わせることで、高解像度画像の生成を可能にしました。

:::details VQGANとLlamaGenの違い
VQGANのアーキテクチャを次の通り示します。LlamaGenのアーキテクチャと比較すると、自己回帰モデルが異なる（Transformer or Llama）ことが分かります。

![Taming Transformers for High-Resolution Image Synthesis](https://github.com/CompVis/taming-transformers/blob/master/assets/teaser.png?raw=true)
:::

### Image Tokenizerの仕組み

VQGAN（および LlamaGen）の Image Tokenizer は、エンコーダー、量子化器、デコーダーの3つの部分から構成されています。

1. **エンコーダー:** 入力画像を低次元の特徴マップに変換します。
2. **量子化器:** 特徴マップの各ベクトルを、学習済みコードブック中の最も近いコードベクトルに置き換えます。このコードベクトルがトークンとなります。
3. **デコーダー:** トークン列から画像を再構成します。

このプロセスにより、画像は少数のトークンで表現され、Transformer で効率的に処理できるようになります。

### ベクトル量子化

ベクトル量子化は、連続的なベクトル空間を離散的なコードブックで表現する手法です。VQ-VAE (Vector Quantized Variational AutoEncoder) などで利用されており、画像や音声などの高次元データを効率的に圧縮・表現することができます。VQ-VAE は、画像を潜在表現にエンコードし、その潜在表現を量子化することでトークン化します。これにより、連続的な画像データを離散的なトークン列に変換することができます。

なお、ベクトル量子化については「ソフテックだより」の記事[^Softech_2012]が分かりやすかったです。
[^Softech_2012]: https://www.softech.co.jp/mm_120704_pc.htm

### Image Tokenizerのパラメータ

画像の情報の圧縮率を測るための値として、「ダウンサンプル比」と「コードブックの語彙数」があります。カメラで例えれば、「解像度」と「色の階調」ということになるでしょうか。エンコーダーが画像を畳み込む前後の比率がダウンサンプル比であり、量子化器がマッピングする先のベクトルの種類がコードブックの語彙数です。

したがって、例えば256x256の画像をダウンサンプル比8のImage Tokenizerでトークン化すると、必要なトークン数は1024となります（256/8 * 256/8 = 1024）。

LlamaGenのImage Tokenizerは、ダウンサンプル比が8と16の場合で、コードブックの語彙数が4096から32768の場合でそれぞれ学習されています。ちなみにLlama3のボキャブラリーの数は128Kトークン[^Llama3]なので、桁が違いますね。
[^Llama3]: https://ai.meta.com/blog/meta-llama-3/

### Image Tokenizerの訓練

Image Tokenizer の訓練では、生成された画像が入力画像に近づくように、以下の損失関数を最小化します。

$$
\begin{align}
L_{AE} = L_2 (x, \widehat{x}) + L_P (x, \widehat{x}) + \lambda_G L_G (\widehat{x})
\end{align}
$$

ここで、

* $L_2(x, \hat{x})$ は、入力画像 $x$ と生成画像 $\hat{x}$ の間のピクセル単位の平均二乗誤差です。
* $L_P(x, \hat{x})$ は、入力画像 $x$ と生成画像 $\hat{x}$ の間の知覚的損失です。LPIPS[^Zhang_et_al_2018] を使用します。LPIPSは、2つの画像の知覚的類似性を測定するために設計された指標です。AlexNetのような事前学習済みネットワークの内部活性化を用いて計算され、ピクセル単位の差よりも人間の知覚との相関性が高いことが示されています。
* $L_G(\hat{x})$ は、生成画像 $\hat{x}$ に対する敵対的損失です。PatchGAN 識別器を用います。PatchGANは、画像をパッチに分割し、各パッチが本物か偽物かを識別することで、画像全体のリアルさを評価します。

[^Zhang_et_al_2018]: R. Zhang, P. Isola, A. A. Efros, E. Shechtman, and O. Wang, “The Unreasonable Effectiveness of Deep Features as a Perceptual Metric,” Apr. 10, 2018, arXiv: arXiv:1801.03924. doi: 10.48550/arXiv.1801.03924.

## Next-Token予測による画像生成

LlamaGen は、Image Tokenizer によって生成されたトークン列を LLM (Llama) に入力し、自己回帰的に次のトークンを予測することで画像を生成します。これは、自然言語処理における文章生成と同様のアプローチです。

### Next-Token予測による画像生成の関連研究

自己回帰モデルによる画像生成は、PixelCNN や ImageGPT など、以前から研究されてきました。これらの研究では、画像をピクセルや離散トークンの列として扱い、自己回帰的に生成することで高品質な画像生成を実現しています。LlamaGen は、これらの先行研究と同様に、自己回帰モデルを利用していますが、LLM である Llama を採用することで、スケーラビリティと生成品質の向上を実現しています。

### CFG (Classifier-Free Guidance)

LlamaGen は、Stable Diffusion と同様に、Classifier-Free Guidance (CFG) を用いて条件付き画像生成を行います。CFG は、分類器を用いることなく、テキストなどの条件情報を画像生成プロセスに組み込むことができる手法です。

LlamaGenは条件付きの画像生成に対応しています。それを行う場合、クラス条件付けやテキスト条件付けを画像トークンをコンテキストとして用います。

具体的には、テキスト条件付けの場合、T5によってテキストを埋め込みに変換した後、画像のグリッドトークンを変換した埋め込みと連結して入力に用いています。

条件付き画像生成では、Stable Diffusionと同様にCFG (分類機なしガイダンス)を用いています。

拡散モデルで条件付きでノイズを除去した画像を推論するために、生成された画像とキャプションの適合度からノイズ除去の方向性として用いる手法があります。なお、この適合度を計算する部品を分類器 (classifier)といいます。

数式で言えば、キャプションを前提として画像が生成される条件付き確率$p(x|y)$を最大化するには、$p(y|x)p(x)$を最大化する必要がある、ということになります。

ところが、拡散モデルではノイズを徐々に取り除く形で画像を生成します。だから、ノイズが含まれる各段階ごとにキャプションとの適合度を計算する必要があり、手間が大きかったのでした。

また、生成された画像を評価する際の指標であるFIDなどが分類を内部で使っているので、学習時に分類器からフィードバックを受けると数値指標をハックする形になることも懸念でした。

そこで、分類器を用いることなく$p(y|x)$を推定するClassifier-Free Guidance (CFG)が提案されました。 CFGでは、条件付きの拡散モデル$p(x|y)$と条件なしの拡散モデル$p(x)$を同時に学習します。推論時には、両方のモデルを用いてノイズ除去の方向を計算し、その差分を利用することで、あたかも分類器を用いたかのようなガイダンスを実現します。具体的には、条件付きのノイズ除去方向と条件なしのノイズ除去方向を計算し、その差分に重みをかけて条件なしのノイズ除去方向に加算することで、条件の影響の強さを調整します。これにより、分類器を用いることなく、効率的に条件付き画像生成を行うことができます。

なお、CFGを理解するにあたってかくびー氏のブログ[^cakkby6_2023]が分かりやすかったです。
[^cakkby6_2023]: https://cake-by-the-river.hatenablog.jp/entry/stable_diffusion_8

### Next-Token予測による画像生成の訓練

画像生成の訓練は、大規模言語モデルの事前学習と変わりません。予測したトークンに対する交差エントロピー誤差を計算し、誤差逆伝播を行います。LlamaGenでは、この損失関数に、条件付き損失と条件なし損失の2種類があります。条件付き損失は、テキストなどの条件情報が与えられた場合の損失で、条件なし損失は、条件情報が与えられない場合の損失です。CFGでは、これらの2つの損失を組み合わせて、最終的な損失を計算します。具体的には、条件付き損失と条件なし損失の差を計算し、その差に重みをかけて条件なし損失に加算します。この重みは、CFGスケールと呼ばれ、条件情報の影響の強さを制御します。

## 評価

LlamaGen の性能は、FID (Fréchet Inception Distance)、IS (Inception Score)、rFID (Reconstruction FID)、sFID (sliced FID)、Precision/Recall、PSNR、SSIM などの指標を用いて評価されます。

### 評価指標

* **IS (Inception Score):** 生成画像の品質と多様性を測定する指標。高いほど良い。
* **FID (Fréchet Inception Distance):** 生成画像と実画像の分布間の距離を測定する指標。小さいほど良い。
* **rFID (Reconstruction FID):** Image Tokenizer によって再構成された画像と元画像の FID。Image Tokenizer の性能を測るために使われます。低いほど、再構成された画像が元画像に近いことを示します。
* **sFID:** FID の改良版。ノイズに強く、より安定した評価ができます。
* **Precision/Recall:** 生成画像の多様性とプロンプトへの適合性を評価します。Precision は生成された画像がどれだけ真の分布に近いかを、Recall は真の分布がどれだけ生成された画像に含まれているかを測定します。
* **PSNR (Peak Signal-to-Noise Ratio):**  画質の客観的評価指標。高いほど良い。ただし、人間の知覚とは必ずしも一致しません。
* **SSIM (Structural Similarity Index):** 画質の客観的評価指標。1に近いほど良い。人間の知覚との相関性が PSNR よりも高いとされています。

### IS (Inception Score)[^Salimans_et_al_2016]

[^Salimans_et_al_2016]: T. Salimans, I. Goodfellow, W. Zaremba, V. Cheung, A. Radford, and X. Chen, “Improved Techniques for Training GANs,” Jun. 10, 2016, arXiv: arXiv:1606.03498. doi: 10.48550/arXiv.1606.03498.

画像のクラス分類モデルであるInceptionNetを用いて、画像生成モデルの品質を測る手法です。

評価対象の画像生成モデルでまとまった量の画像を生成し、それらの画像について次の確率分布を測ります。

1. ある画像の分類結果の分布。特定のラベルに集中しているほど良い
2. 画像全体の分類結果の分布。全てのラベルに広がっているほど良い

2つの確率分布を算出したら、それらの分布の違いを測ります。具体的には、KDL（カルバック・ライブラー・ダイバージェンス）を計算します。そして、すべての画像におけるKLDの平均を取り、そのexpを取った値がインセプションスコアとなります。

値は高いほど良く、GANで約220、潜在拡散モデルで約250、LlamaGenでは約310となっています。

なお、mm_0824氏のブログ[^mm_0824_2021]が分かりやすかったです。
[^mm_0824_2021]: <https://data-analytics.fun/2021/12/12/understanding-inception-score/>

### FID (Fréchet inception distance)[^Heusel_et_al_2017]

[^Heusel_et_al_2017]: M. Heusel, H. Ramsauer, T. Unterthiner, B. Nessler, and S. Hochreiter, “GANs Trained by a Two Time-Scale Update Rule Converge to a Local Nash Equilibrium,” Jan. 12, 2018, arXiv: arXiv:1706.08500. doi: 10.48550/arXiv.1706.08500.

日本語にするとフレチェインセプション距離でしょうか。2つの分布の違いを、中心の違いと散らばり方の違いの2つの観点から測り、足し合わせた値です。

具体的には、次の2つの項を計算します。

1. 2つの分布の平均べクトルの間の距離
2. 2つの分布の分散共分散行列の差異のトレース（詳細は計算式を参照してください）

値は小さいほど良く、BigGANで約7, 最新の評価にはDiffusionで約2となっています。

計算にあたってはベンチマーク対象が必要で、LlamaGenではImageNetでベンチマークを行っています。

### Precision/Recall[^Kynkaanniemi_et_al_2019]

[^Kynkaanniemi_et_al_2019]: T. Kynkäänniemi, T. Karras, S. Laine, J. Lehtinen, and T. Aila, “Improved Precision and Recall Metric for Assessing Generative Models,” Oct. 30, 2019, arXiv: arXiv:1904.06991. doi: 10.48550/arXiv.1904.06991.

適合度と再現率です。実際の画像の分布と生成した画像の分布の重なり合う範囲を比較します。

といっても、画像生成モデルに、学習データと全く同じ画像を再現させることは望めませんし、求められていません。そこで、実際の画像と生成した画像のそれぞれに似た画像がお互いに含まれているかを判断します。具体的には、画像をVGGなどでベクトル化し、特徴空間においてn番目に近い画像までの距離を半径とした超球を作り、比較対象の画像がその超球に含まれるかによって判断します。

値は高いほどよく、LlamaGenのPrecisionは約0.8、Recallは約0.5となっています。

## LlamaGenのスケール則

LlamaGen は、モデルのパラメータ数を増やすことで、より高品質な画像を生成できることが示されています。これは、大規模言語モデルで観測されるスケール則と同様の傾向です。具体的には、パラメータ数を増やすことで、FID などの評価指標が改善されることが確認されています。

## LLMエコシステム

### スケールアップ

LlamaGenのアーキテクチャはLlamaとほぼ同じであるため、LLMコミュニティの最適化技術（大規模なモデルを効率的に学習するための技術）がそのまま利用できます。具体的には、AdamW オプティマイザ、勾配クリッピング、学習率のウォームアップなどが利用されています。これらの技術により、LlamaGen は大規模なデータセットで効率的に学習することが可能になっています。

### vLLM

LlamaGen は、vLLM を用いることで推論を高速化しています。vLLM は、大規模言語モデルの推論に特化したライブラリで、効率的なメモリ管理や並列処理など、様々な最適化技術を提供しています。vLLM を使用することで、LlamaGen の推論速度が最大 4 倍程度向上することが報告されています。これにより、より高速な画像生成が可能になります。

## まとめ

LlamaGen は、LLM である Llama を自己回帰型画像生成に応用した、新しい画像生成モデルです。高品質な Image Tokenizer と効率的な訓練手法により、拡散モデルに匹敵する生成品質を達成しながら、LLM のスケール則に従って性能向上していくことが期待されます。また、vLLMなどのLLMエコシステムを活用することで高速な推論も可能になっています。

## 感想

LlamaGen は、LLM を画像生成に応用するという点で非常に興味深い研究です。大規模言語モデルのスケール則と同様の傾向が確認されているため、今後更にモデルサイズを大きくすることで、更なる高品質な画像生成が期待されます。

一方で、トークン数の制御やマルチモーダルへの拡張など、いくつかの課題も残されています。トークン数の制御は、生成される画像の解像度や細部表現を調整するために重要です。また、マルチモーダルへの拡張は、テキスト、画像、音声など、複数のモダリティを統合的に扱うことで、より高度な表現を獲得するために重要です。これらの課題を解決することで、LlamaGen はより強力な画像生成モデルになると期待されます。
