---
title: '多言語対応の読唇モデル "Zero-AVSR" を動かす'
emoji: "💄"
type: "tech"
topics: ["zero-avsr", "marc", "av-hubert", "hubert"]
published: false
---

## TL;DR


## 動機

とある理由から読唇の研究に興味があり、中でも多言語対応かつソースが公開されている[Zero-AVSR](https://arxiv.org/abs/2503.06273)に興味を持ちました。

しかし、特に任意の音声・映像に対して推論を動かすのが結構しんどかったので、記事にしました。併せて論文の解説もしています。

## AVSRとは？

音声認識に関する研究をASR, 読唇に関する研究をVSRといいます。また、音声と読唇の両方の情報を使って発話内容を認識する研究を AVSR (Audio-Visual Speech Recognition) といいます。

ASR, VSR, AVSRのいずれも、性能の評価にはCERやWERといった指標が用いられます。生成した文字起こしと教師データを比べて、文字や単語レベルでどれだけ誤りが少ないかを測る指標です。

## Zero-AVSRについて

Zero-AVSRは、2025年3月にarXivにて初版が公開された、多言語対応のAVSRフレームワークです。

構成を大きく分けると、音声・読唇情報をローマ字に変換する "AV-Romanizer" と、ローマ字を文章に変換するLLMの2段から成り立っています。特に "AV-Romanizer" は、音声と読唇のデファクトの特徴抽出器の1つである "AV-HuBERT" のヘッドをローマ字分類タスク用にファインチューニングしたモデルとなっており、全体像を読み解くのが少々大変でした。詳しく解説します。

### 先行研究: HuBERT および AV-HuBERT

音声データを連続するベクトルとして表現するモデルのうち、Metaが自己教師あり学習を用いて開発したのが[HuBERT](https://arxiv.org/abs/2106.07447)です。[AV-HuBERT](https://arxiv.org/abs/2201.02184)はHuBERTを音声・視覚に拡張したモデルです。

ごく簡単に言えば、HuBERTは音声ファイルを20msごとのフレームに分割し、それぞれのフレームを256~1024次元（モデルサイズによって異なります）のベクトルに変換するモデルです。次のデモでは、HuBERTによって4秒の音声を200フレームのベクトルに変換し、低次元に投影しています。

![HuBERTによって4秒の音声を200フレームのベクトルに変換](/images/zero-avsr-eval-hubert.gif)

評価の際は、ベクトルに変換した上で、さらに下流タスクごとに異なるヘッドを装着して推論を行います。[TransformersのHubertのドキュメント](https://huggingface.co/docs/transformers/en/model_doc/hubert)にも、HubertForCTCやHubertForSequenceClassificationといったタスク別のクラスが定義されています。

### Stage 1: Cascaded Zero-AVSR

Zero-AVSRには2種類あります。初めにご紹介するのは、よりシンプルなCascaded Zero-AVSRです。

ごく簡単に言えば、音声と口パク映像から発音をローマ字として出力し、それをgpt-4o-miniに投げて文章化する、という構成です。

![](/images/zero-avsr-image1.png)[^JeongHun0716_2025]
[^JeongHun0716_2025]: J. H. Yeo, M. Kim, C. W. Kim, S. Petridis, and Y. M. Ro, “Zero-AVSR: Zero-Shot Audio-Visual Speech Recognition with LLMs by Learning Language-Agnostic Speech Representations,” July 21, 2025, arXiv: arXiv:2503.06273. doi: 10.48550/arXiv.2503.06273.

本構成における論文著者らの貢献は、AV-HuBERTに改変と追加学習を施してAV-Romanizerへと仕立て直したことです。具体的には、[AV-HuBERTの離散クラスタを推定する分類ヘッドを削除し](https://github.com/JeongHun0716/zero-avsr/blob/1a0b7a921a8d7e29e640e583ea252d1d11ecd1ca/stage1/model.py#L260)、代わりに[ローマ字列を生成するCTCヘッドを付与しています](https://github.com/JeongHun0716/zero-avsr/blob/1a0b7a921a8d7e29e640e583ea252d1d11ecd1ca/stage1/model.py#L278)。

また、追加学習にあたって、後述する多言語データセットMARCを新設しています。追加学習にあたってはAV-HuBERTの全層が学習対象であり、実際に視覚・音声の特徴抽出のレイヤーのパラメータのL2ノルムが変化していることを確認できました。

### Stage 2: (Directly Integrated) Zero-AVSR

間にローマ字を挟む `Cascated Zero-AVSR` とは異なる、音声・読唇から抽出した特徴をそのままLLMに統合するアーキテクチャが提案されています。論文ではこちらを単に `Zero-AVSR` とよんでいます。本記事では、特に `Cascaded Zero-AVSR` と区別したい場合に `Directly Integrated Zero AVSR` と呼ぶことにします。次のようなアーキテクチャです。


```mermaid
flowchart LR

```




```mermaid
flowchart TB
  subgraph AVRomanizer["AV-Romanizer Pretraining (AV-HuBERT)"]
    audio["Audio waveform"] -->|log-mel fbank| audEnc["Audio encoder (linear layer)"]
    video["Mouth ROI video"] --> visEnc["Visual encoder (3D ResNet-18)"]
    audEnc --> fuse["Concat + linear W"]
    visEnc --> fuse
    fuse --> transformer["Transformer encoder (24 layers, d=1024, heads=16)"]
    transformer --> romanHead["Linear projection to romanized tokens"]
    romanHead --> romanOut["Romanized token sequence"]
    romanOut --> ctc["CTC loss"]
  end

  transformer --> fav["Fused AV feature f_av"]

  subgraph Task1["Task 1: Align AV features with LLM"]
    fav --> lenComp["Length compressor (1D conv stride 2)"]
    lenComp --> adapter["Adapter to LLM embedding space"]
    instruction["Instruction prompt tokens"] --> instrEmb["Instruction embeddings"]
    adapter --> concat1["Concatenate embeddings"]
    instrEmb --> concat1
    concat1 --> llm1["LLM (Llama3.2-3B, frozen base) with QLoRA"]
    llm1 --> output1["Language-specific transcript"]
    output1 --> loss1["Language modeling loss"]
  end

  subgraph Task2["Task 2: Text-only de-romanization"]
    romanCorpus["Romanized transcripts (MARC text only)"] --> concat2["Tokenized prompt"]
    concat2 --> llm2["LLM (shared QLoRA adapters)"]
    llm2 --> output2["Language-specific transcript"]
    output2 --> loss2["Language modeling loss"]
  end

  llm1 --- shared["Shared QLoRA weights"]
  shared --- llm2
```

## Zero-AVSRを評価する

### MARC を用意する

### Cascaded Zero-AVSR を評価する

スクリプトを実行して、`Cascaded Zero-AVSR`の性能を評価します。

MARCのドイツ語コーパスを対象にAVSRタスクを実行すると、`Cascaded Zero-AVSR` 全体でそれぞれWER: 29%, CER: 17%程度の性能が得られます。また、`AV-Romanizer` が出力するローマ字＆単語区切り文字を単語列と見做した場合、WER: 42%程度の性能が得られます。

実行手順については、私がフォークした[`Zero-AVSR`](https://github.com/xhiroga/zero-avsr/)のリポジトリをご覧ください。

https://github.com/xhiroga/zero-avsr

実行の様子は次のとおりです。

```log
% bash scripts/stage1/eval.sh
[2025-10-08 14:56:01,014][__main__][INFO] - /home/hiroga/Documents/GitHub/zero-avsr/pretrained_models/av-romanizer/all/checkpoint_best.pt

.........

[2025-10-08 15:03:54,743][__main__][INFO] - ---------------------
[2025-10-08 15:03:54,744][__main__][INFO] - HYPO: gibts su
[2025-10-08 15:03:54,745][__main__][INFO] - REF: gebt es zu
[2025-10-08 15:03:54,745][__main__][INFO] - ---------------------
[2025-10-08 15:03:54,746][__main__][INFO] - HYPO: warum dat
[2025-10-08 15:03:54,746][__main__][INFO] - REF: warum das denn
[2025-10-08 15:03:54,746][__main__][INFO] - ---------------------
[2025-10-08 15:03:54,746][__main__][INFO] - Processed 735 sentences (0 tokens) in 0.0s 735000000.00 sentences per second, 1000000.00 tokens per second)
[2025-10-08 15:03:54,748][__main__][INFO] - Word error rate: 41.5876
100%|██████████████████████████████████████████████████████████████████████████████████████████████████████| 735/735 [00:00<00:00, 11753.95it/s]
Zero shot language WER, CER: 29.161822217524573, 17.073924509642328
```

スクリプトを実行すると、`zero-avsr/evaluation/clean/stage1/${lang}` 以下に推論結果が出力されます。`AV-Romanizer`の出力結果と、`Cacaded Zero-AVSR`の出力結果をそれぞれ確認することができます。

`hypo.units` はAV-Romanizerが出力した文字列です。次の推論結果はドイツ語を元にしていますが、出力がローマ字なのでウムラウト(ü など)が見当たりませんね。

```zero-avsr/evaluation/clean/stage1/deu/hypo.units
e i n e | g a n z | k l a s s i s c h e | a r t | a c h t s a m k e i t | z u | u e b e n | d a z u | m o e c h t e | i c h | s i e | g e r n e | j e t z t | e i n l a d e n | i n d e m | s i e | d i e | a u g e n | s c h l i e s s e n | s i c h | s o | b e q u e n e n | w i e | e s | g e r a d e | g e h t | h i n s a e t z e n | u n d | d i e | a u f m e r k s a m k e i t | n a c h | i n n e n | l e n k e n | z u | i h r e m | a t e m | (None-545)
d a s s | d a s | g a n z e | n a t u e r l i c h | a u c h | m a n c h m a l | i n | f e h e n s c h l a g | i s t | o d e r | a u c h | n a c h | w i e | v o r | n a c h | h i n t e n | g e h t | i s t | m i r | j e t z t | v o r | e i l e n | d i g e n | m o c h e n | g e g a n g k e n | d a | h a t t e | i f u r | m i c h | a u s g e s e h e n | e i n e n | s u p e r j o b | a n g e b o t | f u e r | e i n e r | d e r | g r o s t e n | a u t o m o b i e l h e r s t e l l e r | d i e | k o m p l e t t e | d i g i t a l i s i e r u n g | u n d | d i e | a u t o n o m e n | a u t o s | z u | l e i t e n | a l s o | w i r k l i c h | e i n | r i e s e n j o b | a b e r | i c h | h a b e | h i c r | n i c h t | b e k o m m e n | d a | w a r | n o c h | (None-292)
...
```

`avsr.json` はGPT-4oによって補正された推論結果とGround Truthのペアです。

```zero-avsr/evaluation/clean/stage1/deu/avsr.json
[
    {
        "Ground truth": "und genau diese frage mussten wir kollegen eines großen europäischen flugzeugherstellers beantworten\n",
        "prediction": "Und genau diese Frage mussten wir Kollegen eines großen europäischen Flugzeugherstellers beantworten."
    },
    {
        "Ground truth": "das ist eine stadt in nordspanien eine alte arbeiterstadt sehr dreckige bergarbeiterstadt\n",
        "prediction": "Das in Nordspanien, dass ein alter Arbeiter statt sehr, sehr dreckige Bergarbeiter statt."
    },
    ...
]
```

### (Directly Integrated) Zero-AVSR を評価する

```log
bash scripts/stage2/eval.sh

......

[2025-10-08 19:31:24,737][__main__][INFO] -
INST:Given romanized transcriptions extracted from audio-visual materials, back-transliterate them into the original script of German, Standard. Input:
REF:gebt es zu
HYP:gibt es zu

[2025-10-08 19:31:24,737][__main__][INFO] -
INST:Given romanized transcriptions extracted from audio-visual materials, back-transliterate them into the original script of German, Standard. Input:
REF:warum das denn
HYP:warum tat er das

......

[2025-10-08 19:31:24,748][__main__][INFO] -
German WER: 27.95071335927367%
German CER: 16.331652991921707%
```

## 自撮りで試してみる


