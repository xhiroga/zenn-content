---
title: "SDモデルからCLIPを取り出して使う"
emoji: "📎"
type: "tech"
topics: ["CLIP", "StableDiffusion"]
published: true
chats:
- https://claude.ai/chat/19b185aa-b5be-49f1-a73b-9a88a38fab48
- https://claude.ai/chat/8134061b-e048-424f-96f1-a224ce43e764
---

## TL;DR

- Stable DiffusionのモデルからCLIP Textモデルを取り出そうとしました。
- Stable DiffusionはCLIP Textモデルのすべてを使っているわけではありません。
- Stable Diffusion由来の重みでViT/L-14の重みを差し替えても動くことを確認できました。

## 動機

Stable Diffusionモデルの特性を調べるうえで、CLIPScoreという指標があることを知りました。

そこで任意のモデルのCLIPScoreを調べるシステムを組もうと思ったのですが、モデルによってCLIPが異なる可能性に思い当たりました。そのため、正しい計測にはモデルごとのCLIPを特定する必要がありそうです。

その代わり、Stable DiffusionモデルからCLIPを取り出して使うことができるのかを検証しました。

## CLIPとは（おさらい）

CLIPは、画像のキャプションのペアを用いて、TextのEncoderと画像のEncoderが近い埋め込みを返すように訓練したモデルです。

![CLIP](https://github.com/openai/CLIP/blob/main/CLIP.png?raw=true)

CLIPのうち、Stable Diffusionに埋め込まれているのはフローチャート中にピンクで示した部分です。

（ViT/L-14をもとに作図。勉強中のため誤っている可能性があります）

```mermaid
flowchart TB

text_input([テキスト入力<br/>A photo of a dog]):::simple
image_input([画像入力<br/>犬の画像]):::simple

subgraph text_encoder["Text Encoder"]
direction LR
token_embedding[Token Embedding<br/>49408 x 768]:::sd
positional_embedding[Positional Embedding<br/>77 x 768]:::sd
resblocks_text[Transformer Blocks<br/>12層]:::sd
eos_token(["[CLS] Tokenの抽出<br/>1 x 768"]):::simple
text_projection[Text Projection<br/>768 x 768]:::simple
text_input --> token_embedding
token_embedding --> positional_embedding
positional_embedding --> resblocks_text --> eos_token --> text_projection
end

subgraph vision_encoder["Vision Encoder"]
direction LR
class_embedding[Class Embedding<br/>1024]:::simple
positional_embedding_vision[Positional Embedding<br/>257 x 1024]:::simple
resblocks_vision[Transformer Blocks<br/>23層]:::simple
cls_token(["[CLS] Tokenの抽出<br/>1 x 768"]):::simple
visual_projection[Visual Projection<br/>1024 x 768]:::simple
image_input --> class_embedding
class_embedding --> positional_embedding_vision
positional_embedding_vision --> resblocks_vision --> cls_token --> visual_projection
end

output([埋め込み表現<br/>768次元]):::simple
text_projection --> output
visual_projection --> output

classDef textEncoder fill:#DFD5E6,stroke:#B9A6C5
class text_encoder textEncoder
classDef visionEncoder fill:#D9E7D6,stroke:#B4CDA3
class vision_encoder visionEncoder
classDef simple fill:#fff,stroke:#000
classDef sd fill:#f9f
```

図の通り、Stable DiffusionはCLIPのText Encoderをすべて使っているわけではありません。CLIPはテキストの情報を分類のための[CLS]トークンに集約させて使っています。一方で、Stable Diffusionでは精度の高い推論のためにすべてのトークンに対する埋め込み表現を用いています。

> The stable diffusion model takes both a latent seed and a text prompt as an input. The latent seed is then used to generate random latent image representations of size 64×64 where as the text prompt is transformed to text embeddings of size 77×768 via CLIP's text encoder.[^huggingface_stable_diffusion]

[^huggingface_stable_diffusion]: <https://huggingface.co/blog/stable_diffusion>

引用にある**text embeddings of size 77×768**とは、Transformer Blockを経由した状態（[CLS]トークンの抽出直前）です。なので、CLIPとしての動作に必要なText Projection層は同梱されていないんですね。

したがって、実はStable Diffusionから純粋なCLIPのText Encoderを取り出して使うことができませんでした。

## 実装方針

はじめにCLIPの実装ですが、OpenAI CLIPを用いました。今から考えると、[Huggingface TransformersのCLIPのCLIPTextModel](https://huggingface.co/docs/transformers/en/model_doc/clip#transformers.CLIPTextModel)を使ったほうが楽だったかもしれません...

OpenAI CLIPを使ってテキストの重みのみをロードする[Issue](https://github.com/openai/CLIP/issues/113)があり、これを参考にしました。ただし、Issueとは異なりライブラリ側のコードを修正しなくても動くようにしました。

また、前述のとおりStable DiffusionモデルはCLIPのText Projectionにかかる重みをもっていません。ここは仕方なく、訓練済みCLIPから取ってくることにしました。

## 実装

ソースコードをGitHubに公開しました。

<!-- markdownlint-disable -->
https://github.com/xhiroga/til/blob/86afdf3aa5fffec038ecd40923775ee5a4138d58/software-engineering/openai/clip/_src/text-encoder-only/sd.py

## 評価

（比較的構造がシンプルな）Stable Diffusionモデルから取り出したCLIPとViT/L-14で、同じテキストを埋め込み表現にしてそのコサイン類似度を測りました。結果、非常に近い値になりました。

```shell
$ uv run python cos_sim.py
texts=['a diagram', 'a dog', 'a cat']
text_features_vitl14.shape=torch.Size([3, 768])
text_features.shape=torch.Size([3, 768])
text_features_sd_clip.shape=torch.Size([3, 768])
similarity=0.9999422497250121, distance=5.775027498788887e-05
```

なぜ完全一致ではないんだろう？と謎は残るものの、Stable Diffusion1.5でViT/L-14を使っていることが体感でき、良かったです。

## まとめ

CLIPモデルの重みをStable DiffusionモデルのCLIPの重みで差し替えても動作することが確認できました。
