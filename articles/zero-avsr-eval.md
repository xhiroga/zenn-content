---
title: '多言語対応の読唇モデル "Zero-AVSR" を動かす'
emoji: "💄"
type: "tech"
topics: ["zero-avsr", "marc", "av-hubert", "hubert"]
published: false
---

## TL;DR


## 動機



## AVSRとは？

読唇タスクに関する研究をASR, 音声と読唇の両方の情報を使って発話内容を認識する研究を AVSR (Audio-Visual Speech Recognition) といいます。




## Zero-AVSRについて

Zero-AVSRは、2025年3月にarXivにて初版が公開された、多言語対応のAVSRフレームワークです。次のようなアーキテクチャから成り立ちます。

### Cascaded Zero-shot AVSR

```mermaid
flowchart LR

```

### Zero-shot AVSR with Multi-task Training

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




