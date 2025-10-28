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

## VSRとは

VSR(Visual Speech Recognition)とは、視覚データから発話内容を認識する、いわば読唇術のタスクです。それ以外にA-VSR(Auto VSR)やLip ReadingやSilent Speech Recognitionと呼ばれることもあります。

関連するタスクとしては、音声からの文字起こしを行うASR(Audio Visual Recognition)や、視覚を用いてノイズなどの影響を取り除くAVSR(Audio Visual Speech Recognition)などがあります。

## ASR/VSRの発展



ASRリーダーボード
https://huggingface.co/spaces/hf-audio/open_asr_leaderboard

<!-- これ多言語のほうが低いのはなんで？ -->

https://superbbenchmark.github.io?subset=Paper#/leaderboard

VLM
https://huggingface.co/spaces/opencompass/open_vlm_leaderboard

### モデル一覧

- タスクはASVかVSRのいずれかのみ記載し、AVT, VSR, AVSRなどは省略した。
- 筆者の主観で代表的なモデルを選んだ。
- ASRモデルには複数の事前学習済み重みが存在するため、[Open ASR Leaderboard](https://huggingface.co/spaces/hf-audio/open_asr_leaderboard)の記載を参照した。
- VSRモデルでは主にLRS3のWERを記載した。ASRのベンチマークよりデータの多様性が乏しいため数値の単純比較をすべきでない点に注意。
- データセットの名前については、次のとおり省略した。
  - LL: Libri-Light
  - LS: LibriSpeech
  - VC2: VoxCeleb2
  - YT: YouTube

|年月|タスク|モデル|アーキテクチャ|学習データ|WER|コメント|
|-|-|-|-|-|-|-|
|2020|ASR|wav2vec 2.0|CNN+TF(Enc)|LL(60,000h)+LS(FT,960h)|22.55%||
|2020-12|VSR|VTP|3dCNN+VTP+TF(Enc/Dec)|LRS2+LRS3+TEDxext(2,676h)|30.7%||
|2021|ASR|HuBERT|CNN+TF(Enc)|LL(60,000h)+LS(FT,960h)|22.55%||
|2021|-|WavLM|
|2022|**VSR**|AV-HuBERT|CNN+TF(Enc)|VC2(1,326h)+LRS3(433h)+LRS3(FT,433h)|26.9%||
|2022|ASR|Whisper v1||680,000h|10.32%||
|2022-10|**VSR**|Ma et al.|3dCNN+2dCNN+CF+TF(Dec)|1,459h|31.5%||
|2023-04|**VSR**|RAVEn|||||
|2023-06|**VSR**|Auto-AVSR|3dCNN+RN18+CF+TF(Dec)||||
|2023|**VSR**|LP-Conformer|Frontend+Conformer|YT(100,000h)+LRS3(FT,400h)|12.8%|VSRのSOTA|
|2023|ASR|Whisper large-v3||5,000,000h|7.44%||
|2024|ASR|Granite-Speech|||5.74%||
|2024|**VSR**|BRAVEn|||||
|2024-05|**VSR**|VSP-LLM|AV-HuBERT+Llama|VC2(1,326h)+LRS3(433h)+LRS3(FT,433h)|25.4%||
|2024-09|**VSR**|Whisperer|AV-HuBERT+Adpt+Whisper|VC2(1,326h)+LRS3(433h)+LRS3(FT,433h)|24.3%||
|2025|**VSR**|VALLR|ViT-Base+Adpt+CTC+Llama|(ViT-Base+Llama3.2-3B)+30h|18.7%||

<!-- ちなみにプロのLip Readerの正解率は？ -->

### 教師あり学習

超・古典的な技術


DNN


RNNベースのMakino 2019



### 自己教師あり学習・反教師あり学習





#### wav2vec[^wav2vec]

[^wav2vec]: [S. Schneider, A. Baevski, R. Collobert, and M. Auli, “wav2vec: Unsupervised Pre-training for Speech Recognition,” Sept. 11, 2019, arXiv: arXiv:1904.05862. doi: 10.48550/arXiv.1904.05862.](https://arxiv.org/abs/1904.05862)


CNNベースの手法

#### VTP[^VTP]

[^VTP]: K. R. Prajwal, T. Afouras, and A. Zisserman, “Sub-word Level Lip Reading With Visual Attention,” Dec. 03, 2021, arXiv: arXiv:2110.07603. doi: 10.48550/arXiv.2110.07603.

3D CNN + Visual Transformer Pooling + Transformer Encoder-Decoderによる、文字単位ではなくサブワード単位の認識が特徴のモデル。VTPという略称は公式ではないが、少なくともVALLR[^VALLR]でそう言及されている。

#### wav2vec 2.0[^wav2vec2.0]

[^wav2vec2.0]: [A. Baevski, H. Zhou, A. Mohamed, and M. Auli, “wav2vec 2.0: A Framework for Self-Supervised Learning of Speech Representations,” Oct. 22, 2020, arXiv: arXiv:2006.11477. doi: 10.48550/arXiv.2006.11477.](https://arxiv.org/abs/2006.11477)


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

https://www.youtube.com/watch?v=FIau-6JA9Po

#### Auto-AVSR[^Auto-AVSR]

[^Auto-AVSR]: P. Ma, A. Haliassos, A. Fernandez-Lopez, H. Chen, S. Petridis, and M. Pantic, “Auto-AVSR: Audio-Visual Speech Recognition with Automatic Labels,” in ICASSP 2023 - 2023 IEEE International Conference on Acoustics, Speech and Signal Processing (ICASSP), June 2023, pp. 1–5. doi: 10.1109/ICASSP49357.2023.10096889.

[事前学習済み重み](https://github.com/mpc001/auto_avsr)が公開されている。学習データ3,291時間で学習した重みがあり、これはVSRの公開されている重みの中でも最大。

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

<!--
結構理に適っている気がするのだけど、ViT採用のアイデアはどこから？
（他のモデルがAV-HuBERTにこだわっているのはなぜ？）
（9月にVideoMAEで自作した際はあまりうまくいかなかったが、この論文との差分は？）

他の論文への言及
- VSP-LLM, Personal Lip Reading: LLM活用の先駆者として言及。ただし制御性の低さを指摘。Zero-AVSRはほぼ登場が同時なので引用されていない。
- VTP, LP: スコアのみ
- Whisperer: ASRモデルの転移学習の最新手法として紹介し、大量のデータが必要な点を指摘。ほか、CTCヘッドの学習戦略のきっかけとして言及
- Zero-shot keyword spotting: 先駆けとして（のはず）
-->

## VSRのデータセット

|年月|名前|言語|時間|ソース|特徴|
|-|-|-|-|-|-|
||GRID|英語|||
||LRW|英語|173h|||
|2016-11|LRS|英語|?h|BBC||
||AVSpeech|英語||||
|2018|VoxCeleb2|英語||||
|2018-09|LRS2|英語|224h|BBC||
|2018-09|LRS3|英語|437h|TED&TEDx||
||WildVSR||||
|2023-01|OLKAVS|韓国語|1,150h||


なお、各データセットの図表は公式およびサーベイ[^Changchong]からの引用である。

[^Changchong]: [1] S. Changchong et al., “Deep Learning for Visual Speech Analysis: A Survey.” Accessed: Oct. 28, 2025. [Online]. Available: https://www.computer.org/csdl/journal/tp/2024/09/10472054/1VhFlotHb2w

### LRW[^LRW]

[^LRW]: [J. S. Chung and A. Zisserman, “Lip Reading in the Wild,” S.-H. Lai, V. Lepetit, K. Nishino, and Y. Sato, Eds., in Lecture Notes in Computer Science, vol. 10112. Cham: Springer International Publishing, 2017, pp. 87–103. doi: 10.1007/978-3-319-54184-6_6.](https://link.springer.com/chapter/10.1007/978-3-319-54184-6_6)

### LRS[^LRS]

[^LRS]: [J. S. Chung, A. Senior, O. Vinyals, and A. Zisserman, “Lip Reading Sentences in the Wild,” in 2017 IEEE Conference on Computer Vision and Pattern Recognition (CVPR), July 2017, pp. 3444–3453. doi: 10.1109/CVPR.2017.367.](https://arxiv.org/abs/1611.05358)

![LRS](/images/vsr-visual-speech-recognition-survey/lrs-fig3.png)

LRWが1クリップ1単語だったのに対して、1クリップ1センテンスとなっている。

### LRS2[^LRS2]

[^LRS2]: [T. Afouras, J. S. Chung, A. Senior, O. Vinyals, and A. Zisserman, “Deep Audio-Visual Speech Recognition,” IEEE Trans. Pattern Anal. Mach. Intell., vol. 44, no. 12, pp. 8717–8727, Dec. 2022, doi: 10.1109/TPAMI.2018.2889052.](https://arxiv.org/abs/1809.02108)

![LRS2](/images/vsr-visual-speech-recognition-survey/lrs2-fig3.png)

LRS1の上位互換と見做されているのか、LRS1を飛ばしてLRS2が言及されることが多い。現在は公式サイトの配布が停止している。

### LRS3[^LRS3]

[^LRS3]: [T. Afouras, J. S. Chung, and A. Zisserman, “LRS3-TED: a large-scale dataset for visual speech recognition,” Oct. 28, 2018, arXiv: arXiv:1809.00496. doi: 10.48550/arXiv.1809.00496.](https://arxiv.org/abs/1809.00496)

![LRS3](/images/vsr-visual-speech-recognition-survey/dl-for-vsa-lrs3.png)

2025年時点におけるVSRの事実上の標準データセット。現在は[公式サイト](https://www.robots.ox.ac.uk/~vgg/data/lip_reading/)の配布が停止している。Train/Val/Testで話者分離をしていないため、LRS3のみで学習すると話者ごとの充分な汎化性能が得られない可能性がある。

TEDの映像から自動で顔認識してクリップを作成している。登壇者ではなく背後のスライドが切り抜かれてしまっているクリップも意外とあるらしい？

### WildVSR[^WildVSR]

[^WildVSR]: Y. A. D. Djilali, S. Narayan, E. L. Bihan, H. Boussaid, E. Almazrouei, and M. Debbah, “Do VSR Models Generalize Beyond LRS3?,” Nov. 23, 2023, arXiv: arXiv:2311.14063. doi: 10.48550/arXiv.2311.14063.

<!-- 角度とかはこだわりある？ -->

### OLKAVS[^OLKAVS]

[^OLKAVS]: [J. Park et al., “OLKAVS: An Open Large-Scale Korean Audio-Visual Speech Dataset,” Aug. 28, 2025, arXiv: arXiv:2301.06375. doi: 10.48550/arXiv.2301.06375.]([1] J. Park et al., “OLKAVS: An Open Large-Scale Korean Audio-Visual Speech Dataset,” Aug. 28, 2025, arXiv: arXiv:2301.06375. doi: 10.48550/arXiv.2301.06375.)

非英語、総映像時間1,150時間というだけでもすごいのに、5点から収録しているので実質データ量が5倍。

## VSRの学習

時間マスキング
(ma et all 2022など)
文脈情報に応じて口のかたちを推定する力を養う

モダリティdropout

擬似ラベル

転移学習メソッド

多言語対応メソッド

### QLoRA, LoRA, フルトレーニング

VSP-LLMを用いて、LLMの追加学習方法について比較がなされています。LRS3による学習においては、学習ステップ数が30kステップ数程度である場合、QLoRAやLoRAでは損失が収束するが、フルトレーニングでは収束に至らないことが示されています。

## 今後の課題

- モデルは性能の良さというより学習効率で選んだ方が良いかもしれない
(AV-HuBERT? ViT? VideoMAE?)
(Phonemeベース? トークンベース?)
(直接統合? プロンプト経由の統合?)
(セミオープンな語彙の場合は？)

## 参考文献（サーベイなど）

- ASR
  - [C. Wu, Y. Pan, H. Wu, and L. Ning, “Integrating Speech Recognition into Intelligent Information Systems: From Statistical Models to Deep Learning,” Informatics, vol. 12, no. 4, p. 107, Oct. 2025, doi: 10.3390/informatics12040107.](https://www.mdpi.com/2227-9709/12/4/107)
  - [Y. Yang et al., “Towards Universal Speech Discrete Tokens: A Case Study for ASR and TTS,” Dec. 14, 2023, arXiv: arXiv:2309.07377. doi: 10.48550/arXiv.2309.07377.](https://arxiv.org/abs/2309.07377)
- VSR
  - [J. Rishabh and H. Naomi, “From Hype to Insight: Rethinking Large Language Model Integration in Visual Speech Recognition.” Accessed: Oct. 27, 2025. [Online]. Available: https://arxiv.org/abs/2509.14880v1](https://arxiv.org/abs/2509.14880v1): サーベイではなく論文ですが、最近のVSR+LLMの流れをまとめている（オープンアクセスでは）現状唯一の論文かも？
  - [P. Nemani, G. S. Krishna, and S. Kundrapu, “Automated Speaker Independent Visual Speech Recognition: A Comprehensive Survey,” Image and Vision Computing, vol. 138, p. 104787, Oct. 2023, doi: 10.1016/j.imavis.2023.104787.](https://arxiv.org/abs/2306.08314): 包括的。さらにOLKAVSに言及している数少ない論文。
  - [K. Rezaee and M. Yeganeh, “Automatic Visual Lip Reading: A Comparative Review of Machine-Learning Approaches,” Results in Engineering, p. 107171, Sept. 2025, doi: 10.1016/j.rineng.2025.107171.](https://www.sciencedirect.com/science/article/pii/S2590123025032268)
- Computer Vision
  - [N. Madan, A. Moegelmose, R. Modi, Y. S. Rawat, and T. B. Moeslund, “Foundation Models for Video Understanding: A Survey,” May 06, 2024, arXiv: arXiv:2405.03770. doi: 10.48550/arXiv.2405.03770.](http://arxiv.org/abs/2405.03770)
