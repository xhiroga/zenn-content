---
title: '読唇術の研究まとめ【サーベイ】'
emoji: "💄"
type: "tech"
topics: ["vsr", "machinelearning", "deeplearning", "LLM"]
published: false
references:
- OpenASR Leaderboard: https://chatgpt.com/c/68faf914-d57c-8322-ac80-2f178b8eae6e?ref=mini
- https://chatgpt.com/c/68fb1d9c-1540-8322-9307-4bef509a67f8?ref=mini
- RAVEn/BRAVEn: https://chatgpt.com/c/68fb298e-7ccc-8320-a573-c31bf48e2bad?ref=mini
- 弱教師あり学習など: https://chatgpt.com/c/68fd58f1-24b8-8323-bdd8-5832a50ae0ce?ref=mini
- LP-Conformer: https://chatgpt.com/c/68fd5331-2e30-8320-bd0d-817e1a448489?ref=mini
---

## TL;DR

## 動機

読唇術のモデルを開発するためには、視覚音声認識(VSR)だけでなく音声認識(ASR)のトレンドを抑える必要があります。そこで、ASR/VSRにおける代表的なモデル・データセット・学習手法などについてまとめました。

時間をかけて注意深く調べていますが、筆者は専門的な訓練を受けていないため、本記事には誤りがある可能性があります。

## ASR/VSRの発展の概要

ASRリーダーボード
https://huggingface.co/spaces/hf-audio/open_asr_leaderboard

<!-- これ多言語のほうが低いのはなんで？ -->

VLM
https://huggingface.co/spaces/opencompass/open_vlm_leaderboard


## 歴史

超・古典的な技術


DNN


RNNベースのMakino 2019



### 自己教師あり時代

- タスクはASVかVSRのいずれかのみ記載し、AVT, VSR, AVSRなどは省略した。
- 筆者の主観で代表的なモデルを選んだ。
- ASRモデルの学習データおよびWERについては、[Open ASR Leaderboard](https://huggingface.co/spaces/hf-audio/open_asr_leaderboard)に記載されているモデルを優先した。
- VSRモデルのWERについては、論文の記載を優先した。評価データセットの多様性に乏しい点に注意。
- データセットの名前については、次のとおり省略した。
  - LL: Libri-Light
  - LS: LibriSpeech
  - VC2: VoxCeleb2
  - YT: YouTube

|年月|タスク|モデル|アーキテクチャ|学習データ|WER|コメント|
|-|-|-|-|-|-|-|
|2020|ASR|wav2vec 2.0|CNN+Transformer(Enc)|LL(60,000h)+LS(FT,960h)|22.55%||
|2021|ASR|HuBERT|CNN+Transformer(Enc)|LL(60,000h)+LS(FT,960h)|22.55%||
|2022|**VSR**|AV-HuBERT|CNN+Transformer(Enc)|VC2(1,326h)+LRS3(433h)+LRS3(FT,433h)|26.9%||
|2022|ASR|Whisper v1||680,000h|10.32%||
|2022-10|**VSR**|Ma et al. (2022)|3dCNN+2dCNN+CF+TF(Dec)|1,459h|31.5%||
|2023|**VSR**|LP-Conformer|Frontend+Conformer|YT(100,000h)+LRS3(FT,400h)|12.8%|VSRのSOTA|
|2023|ASR|Whisper large-v3||5,000,000h|7.44%||
|2024|ASR|Granite-Speech|||5.74%||
|2024-05|**VSR**|VSP-LLM|AV-HuBERT+Llama|VC2(1,326h)+LRS3(433h)+LRS3(FT,433h)|25.4%||
|2024-09|**VSR**|Whisperer|AV-HuBERT+Adpt+Whisper|VC2(1,326h)+LRS3(433h)+LRS3(FT,433h)|24.3%||
|2025|**VSR**|VALLR|ViT-Base+Adpt+CTC+Llama|(ViT-Base+Llama3.2-3B)+30h|18.7%||


#### wav2vec

CNNベースの手法

#### wav2vec 2.0

https://huggingface.co/speechbrain/asr-wav2vec2-librispeech
https://huggingface.co/facebook/wav2vec2-large-960h-lv60-self

transformer

#### HuBERT

https://huggingface.co/facebook/hubert-xlarge-ls960-ft


LRS3のWER 26.9%
埋め込みモデルとしても使われるが、文字起こしも普通にできる
https://chatgpt.com/g/g-p-68e8381a0b7081918da1bbc98d7d060b-vsr/project

? BERTの型としては？ちょっと古いのか？
RoPE+Pre-LN+GeGLU、
交互Attention（ローカル窓＋散発グローバル）、
Unpadding＋FlashAttention 、といった工夫はなかった時代

実装

このエンコーダーを使っている世代は...
VSP-LLM

RAVEn / BRAVEn
HuBERT + BYOL...らしい

WavLM
離散音声トークンのデファクトの一つらしいが、不思議と見ない...?

#### AV-HuBERT[^AV-HuBERT]

[^AV-HuBERT]: [B. Shi, W.-N. Hsu, K. Lakhotia, and A. Mohamed, “Learning Audio-Visual Speech Representation by Masked Multimodal Cluster Prediction,” Mar. 13, 2022, arXiv: arXiv:2201.02184. doi: 10.48550/arXiv.2201.02184.](https://arxiv.org/abs/2201.02184)

HuBERTをVSR/AVSRに拡張したモデル。音声フレーム4個分(=40ms)からなる1音声ベクトルに対して、映像を25fpsで扱って1映像フレーム(=40ms)をフレームごとに結合し、Transformerに入力している。

公開されている事前学習済みモデルは次の通り。

- [Meta公式](https://facebookresearch.github.io/av_hubert/): VSR/AVSR, LRS3 + VoxCeleb2など
- [MuAViC](https://github.com/facebookresearch/muavic): AVSR, 多言語

感想になるが、特にLRS3 + VoxCeleb2の事前学習済みモデルにはWERほどの性能はないかもしれない。AV-HuBERTの埋め込みを入力に用いた別のモデルで試した経験から言うと、学習データセットにない話者（私自身やランダムな英語の映像）の視覚音声認識はまず成功しない。

#### Whisper[^Whisper]

[^Whisper]: A. Radford, J. W. Kim, T. Xu, G. Brockman, C. McLeavey, and I. Sutskever, “Robust Speech Recognition via Large-Scale Weak Supervision”.

2022年にOpenAIが公開したASRモデル。

PyTorch実装で扱いやすく、エコシステムも充実している。最近だと[FFmpegにWhisperを用いた文字起こし機能が搭載されるとして話題になった。](https://gigazine.net/news/20250814-ffmpeg-whisper-transcription)

#### Ma et al. (2022) VSR model[^Ma-VSR]

[^Ma-VSR]: P. Ma, S. Petridis, and M. Pantic, “Visual Speech Recognition for Multiple Languages in the Wild,” Nat Mach Intell, vol. 4, no. 11, pp. 930–939, Oct. 2022, doi: 10.1038/s42256-022-00550-z.


#### LP-Conformer[^LP-Conformer]

[^LP-Conformer]: [O. Chang, H. Liao, D. Serdyuk, A. Shah, and O. Siohan, “Conformers are All You Need for Visual Speech Recognition,” Dec. 13, 2023, arXiv: arXiv:2302.10915. doi: 10.48550/arXiv.2302.10915.](https://arxiv.org/pdf/2302.10915)

LP-Conformerという名前は論文内にはなく、ブログなどで呼ばれているもの。[^anwarvic-LP_Conformer]
[^anwarvic-LP_Conformer]: https://anwarvic.github.io/speech-recognition/LP_Conformer

#### NVIDIA Canary (2024) - Comformer+LLM

言ってみればwav2vec2+llmってこと？ → transformer-decoderらしい
NeMOシリーズのCannary
Comformer部分だけ使ったらいい感じのローマ字が取得できたりするのか...?
Comformer+Llamaはないのか？

学習データは86k時間

#### Microsoft Phi-4 Multimodal Instruct (2024–25) - 直接音声を理解
すでにマルチモーダル...

#### IBM Granite Speech

#### Whisperer[^Whisperer]

[^Whisperer]: K. R. Prajwal, T. Afouras, and A. Zisserman, “Speech Recognition Models are Strong Lip-readers,” in Interspeech 2024, ISCA, Sept. 2024, pp. 2425–2429. doi: 10.21437/Interspeech.2024-2290.



#### VALLR[^VALLR]

[^VALLR]: M. Thomas, E. Fish, and R. Bowden, “VALLR: Visual ASR Language Model for Lip Reading,” Mar. 27, 2025, arXiv: arXiv:2503.21408. doi: 10.48550/arXiv.2503.21408.

ViT-Baseを組み込んだ音素エンコーダーを開発し、Llama3等と接続。具体的には、映像の各フレームを静止画としてベクトル化し、時系列方向に1次元畳み込み、その後CTC処理を加えている。

ViT-Baseのアーキテクチャだけでなく、事前学習済み重みを使ってこその性能と思われる。LRS3のわずか30時間のデータを用いるという前提のためViT-Baseを採用しているのかもしれない。その場合、学習・推論速度を無視すれば、より大きなデータセット + ViT-Largeの採用で簡単に性能の向上が期待できる。

## データセット

|年月|名前|時間|ソース|特徴|
|-|-|-|-|-|
||LRW||||
||LRS2||||
||AVSpeech||||
|2018|VoxCeleb2||||
||LRS3||||
||WildVSR|||



### LRS3

VSRの事実上の標準データセット。現在は公式サイトから配布されていない。Train/Val/Testで話者分離をしていないため、LRS3のみで学習すると話者ごとの充分な汎化性能が得られない可能性がある。

<!-- LRS2よりも難しかったはず -->

### WildVSR[^WildVSR]

[^WildVSR]: Y. A. D. Djilali, S. Narayan, E. L. Bihan, H. Boussaid, E. Almazrouei, and M. Debbah, “Do VSR Models Generalize Beyond LRS3?,” Nov. 23, 2023, arXiv: arXiv:2311.14063. doi: 10.48550/arXiv.2311.14063.

<!-- 角度とかはこだわりある？ -->

## ASR to VSRの技術

時間マスキング
(ma et all 2022など)
文脈情報に応じて口のかたちを推定する力を養う

モダリティdropout

擬似ラベル

転移学習メソッド

多言語対応メソッド

## 今後の課題

- モデルは性能の良さというより学習効率で選んだ方が良いかもしれない

## サーベイ

VSR
K. Rezaee and M. Yeganeh, “Automatic Visual Lip Reading: A Comparative Review of Machine-Learning Approaches,” Results in Engineering, p. 107171, Sept. 2025, doi: 10.1016/j.rineng.2025.107171.
（でもAV-HuBERTに触れてないのが気になる）

Integrating Speech Recognition into Intelligent Information
Systems: From Statistical Models to Deep Learning


動画認識側
[1] N. Madan, A. Moegelmose, R. Modi, Y. S. Rawat, and T. B. Moeslund, “Foundation Models for Video Understanding: A Survey,” May 06, 2024, arXiv: arXiv:2405.03770. doi: 10.48550/arXiv.2405.03770.
