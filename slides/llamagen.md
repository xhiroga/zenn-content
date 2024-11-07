---
title: "Diffusionなし！LlamaのNext-Token Predictionを使った画像生成"
marp: true
theme: uncover
---

<!-- 方針: LLMコミュニティでの発表であり、自分の学習であることも踏まえ、わからない概念を細かくケアする -->

LlamaGen[^Sun_et_al_2024]
[^Sun_et_al_2024]: P. Sun et al., “Autoregressive Model Beats Diffusion: Llama for Scalable Image Generation,” Jun. 10, 2024, arXiv: arXiv:2406.06525. doi: 10.48550/arXiv.2406.06525.

---

## この論文を選んだのはなぜ？

- 人間と同じように絵を描くAIに興味がある
- 自己回帰モデルによる画像生成に可能性を感じていた
- Llamaを使った画像生成は条件に当てはまっていた

---

## TL;DR

<!-- TODO: もっと研究者に伝わる具体的な表現にする -->

1. 性能の良いImage Tokenizer
2. Next-Token Predictionによる画像生成
3. text-to-imageのような下流タスクでの性能
4. LLMコミュニティの資産の流用

---

## アーキテクチャ

（本文中に適当な図がないので作るかも）

訓練

画像 -> 畳み込み -> ベクトル量子化 -> Transformerで自己教師あり学習（のはず）

推論

初期条件 -> Transformer -> トークン列 -> ベクトル量子化の逆 -> 畳み込みの逆 -> 画像

---

## Image Tokenizer

---

ImageGPTの画像

---

TODO: 実演するかも

---

- Transformerで画像を予測するには、極端な話1ピクセルごとにトークンとして扱えば良い
- それだと効率が悪いので、「複数ピクセルをまとめる（畳み込み）」「にたピクセルを1括りにする（ベクトル量子化）」を行う
- 本論文ではVQGANで提案された手法を用いた

---

先行研究: VQGAN

![Taming Transformers for High-Resolution Image Synthesis](https://github.com/CompVis/taming-transformers/blob/master/assets/teaser.png?raw=true)

- 実はTransformer部分がLlamaに代わった以外はVQGAN（要確認）

---

ポイント

- 画像トランスフォーマーのパラメータは「ダウンサンプル比」と「コードブックの語彙数」
- ...

---

ベクトル量子化とは？

> ベクトル量子化とは
> ベクトル量子化は、データを圧縮する際に、連続的な値を一定の数の離散的な値に変換するプロセスです。これにより、データはより効率的に圧縮され、特に画像のような複雑なデータを扱う際に有効です。

https://www.softech.co.jp/mm_120704_pc.htm

---

なぜベクトル量子化を採用したのか？

TODO: VQVAEの論文を読んで考える

ベクトル量子化の方が、JPEGのような圧縮方式よりも、形状を保つことができて有利ってこと？

---

性能

97%のコードブック使用率 → これどうやって測ったの？
VAEにはコードブック使用率の概念はないと思うが、（下流タスク以外に）どうやって比較するの？

コードブックはどれだけある？単純に考えれば256^3 = 16,777,216
ダウンサンプル比8と16らしいが...それって単純に考えれば2Mってこと（多いがLLMと比較して大外れではない）

Llamaで使われているトークナイザの場合はどう？
> Llama 3では128Kトークンの語彙を持つトークナイザーを使用
https://note.com/yutohub/n/n5fd752f212d6

---

## Next-Token Predictionによる画像生成

---

![Image generation by next-image-token prediction | VAR](https://github.com/FoundationVision/VAR/assets/39692511/3e12655c-37dc-4528-b923-ec6c4cfef178)

---

## 自己回帰モデルとは？

出力が入力に入るモデル。Llamaとか...

←→ 一度にすべての出力を出すタイプ

---

- Llamaをそのまま使っている
- クラス分類をするために、クラストークンを用いている。ViTに出てくるやつ
- Transformerで画像トークンというのはDiTなどに倣っている

---

## 先行研究

- DiT: 拡散モデル、ただしノイズの除去にCNNの代わりにTransformerを用いる
- PixArt-α: SD1.5の1割の学習時間で住むやつ
- VQGAN: 画像トークナイザーで紹介した
- 更に遡ればImageGPTがある？

---

## クラスからの画像生成・テキスト条件付き画像生成

クラストークンを入力に用いるらしい

---

## 学習

### Scale Up

DDPって？FSDPって？
https://zenn.dev/syoyo/scraps/5fc9c6edb48511

### IS

[^Salimans_et_al_2016]

確信度合いと多様性によって判断するやつ

https://data-analytics.fun/2021/12/12/understanding-inception-score/

50000枚生成する

---

### FID (Fréchet inception distance)

https://data-analytics.fun/2021/12/31/understanding-fid/

<!-- 評価指標。主な指標としてFréchet inception distance (FID) [Heusel et al. 2017]を用いる。また、二次指標とし て、Inception Score (IS) [Salimans et al. 2016]、sFID [Nash et al. 2021]、Precision/Recall [Kynkänniemi et al. 2019]を報告する。すべての評価は、公正な比較のためにADMのTensorFlowスクリプト[Dhariwal & Nichol 2021]を 使用して実装されている。 -->

<!-- https://claude.ai/chat/fd1ccb06-f197-469b-a2af-36ace1d784b9 -->

FID (Fréchet inception distance)[^Heusel_et_al_2017]

本物の画像と、埋め込み表現を計算してその距離を測る

1万サンプル以上

---

### rFID

どのくらいだと高いの？低いの？

---

## LLM サービングフレームワークの貢献

（こういうの他にもある？）

---

## LlamaGenの貢献は？

- 総合的に〇〇でも拡散モデル並みのパフォーマンスを出せる → この〇〇
- 自己回帰モデルのほうが〇〇の面で優れている
- 拡散モデルは逆に〇〇で優れている

---

## まとめ

---

[^Salimans_et_al_2016]: T. Salimans, I. Goodfellow, W. Zaremba, V. Cheung, A. Radford, and X. Chen, “Improved Techniques for Training GANs,” Jun. 10, 2016, arXiv: arXiv:1606.03498. doi: 10.48550/arXiv.1606.03498.
[^Heusel_et_al_2017]: M. Heusel, H. Ramsauer, T. Unterthiner, B. Nessler, and S. Hochreiter, “GANs Trained by a Two Time-Scale Update Rule Converge to a Local Nash Equilibrium,” Jan. 12, 2018, arXiv: arXiv:1706.08500. doi: 10.48550/arXiv.1706.08500.
[^fmuuly] “Llamaで画像生成：LlamaGen【論文】,” Zenn. Accessed: Nov. 05, 2024. [Online]. Available: <https://zenn.dev/fmuuly/articles/40f4863385b7d8>

---

## TODO

- inductive biasesって何？
