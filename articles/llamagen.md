---
title: "LlamaGen(仮)"
emoji: "🐘"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["AI", "機械学習", "ディープラーニング", "survey"]
published: false
---

<!-- 方針: LLMコミュニティでの発表であり、自分の学習であることも踏まえ、わからない概念を細かくケアする -->

## 動機

## LlamaGenとは？

## 自己回帰モデルとは？

## Image Tokenizer

実演したい
（割とズレや回転にセンシティブなのか？）

## 先行研究

VQVAE, VQGAN

### FID

https://data-analytics.fun/2021/12/31/understanding-fid/

<!-- 評価指標。主な指標としてFréchet inception distance (FID) [Heusel et al. 2017]を用いる。また、二次指標とし て、Inception Score (IS) [Salimans et al. 2016]、sFID [Nash et al. 2021]、Precision/Recall [Kynkänniemi et al. 2019]を報告する。すべての評価は、公正な比較のためにADMのTensorFlowスクリプト[Dhariwal & Nichol 2021]を 使用して実装されている。 -->

<!-- https://claude.ai/chat/fd1ccb06-f197-469b-a2af-36ace1d784b9 -->

### IS

[^Salimans_et_al_2016]

確信度合いと多様性によって判断するやつ

https://data-analytics.fun/2021/12/12/understanding-inception-score/

50000枚生成する

### FID (Fréchet inception distance)

FID (Fréchet inception distance)[^Heusel_et_al_2017]

本物の画像と、埋め込み表現を計算してその距離を測る

1万サンプル以上

### rFID

どのくらいだと高いの？低いの？

## 下流タスクでの評価

## LlamaGenの貢献は？

- 総合的に〇〇でも拡散モデル並みのパフォーマンスを出せる → この〇〇
- 自己回帰モデルのほうが〇〇の面で優れている
- 拡散モデルは逆に〇〇で優れている

## まとめ

---

[^Salimans_et_al_2016]: T. Salimans, I. Goodfellow, W. Zaremba, V. Cheung, A. Radford, and X. Chen, “Improved Techniques for Training GANs,” Jun. 10, 2016, arXiv: arXiv:1606.03498. doi: 10.48550/arXiv.1606.03498.
[^Heusel_et_al_2017]: M. Heusel, H. Ramsauer, T. Unterthiner, B. Nessler, and S. Hochreiter, “GANs Trained by a Two Time-Scale Update Rule Converge to a Local Nash Equilibrium,” Jan. 12, 2018, arXiv: arXiv:1706.08500. doi: 10.48550/arXiv.1706.08500.
[^fmuuly] “Llamaで画像生成：LlamaGen【論文】,” Zenn. Accessed: Nov. 05, 2024. [Online]. Available: <https://zenn.dev/fmuuly/articles/40f4863385b7d8>
