---
title: "SDモデルからCLIPを取り出して使う"
emoji: "📎"
type: "tech"
topics: ["CLIP", "StableDiffusion"]
published: false
---

## 


## 動機

Stable Diffusionモデルの特性を調べるうえで、CLIPScoreという指標があることを知りました。

特に継続学習したモデルを調べようと思ったのですが、訓練に使用された・埋め込まれているCLIPモデルを特定する必要があると思いました。また、LoRAなどの学習ではText Encoderも学習させることができます。

そこで、Stable DiffusionモデルからCLIPを取り出して使うことができるのかを検証しました。

## モデルの構造

CLIPは、画像のキャプションのペアを用いて、TextのEncoderと画像のEncoderが近い埋め込みを返すように訓練したモデルです。

![CLIP](https://github.com/openai/CLIP/blob/main/CLIP.png?raw=true)

CLIPのうち、Stable Diffusionに埋め込まれているのはフローチャート中にピンクで示した部分です。勉強中なので、作図は誤っている可能性があります。

```mermaid
flowchart TB

text_input([テキスト入力<br/>A photo of a dog]):::simple
image_input([画像入力<br/>犬の画像]):::simple

subgraph text_encoder["Text Encoder"]
direction LR
token_embedding[Token Embedding<br/>49408 x 768]:::sd
positional_embedding[Positional Embedding<br/>77 x 768]:::sd
resblocks_text[Transformer Blocks<br/>12層]:::sd
eos_token(["Extract [CLS] Token<br/>1 x 768"]):::simple
text_projection[Text Projection<br/>768 x 768]:::simple
text_input --> token_embedding
token_embedding --> positional_embedding
positional_embedding --> resblocks_text --> eos_token --> text_projection
end

subgraph vision_encoder["Vision Encoder"]
direction LR
patch_embedding[Patch Embedding<br/>49 x 768]:::simple
positional_embedding_vision[Positional Embedding<br/>50 x 768]:::simple
resblocks_vision[Transformer Blocks<br/>12層]:::simple
cls_token(["Extract [CLS] Token<br/>1 x 768"]):::simple
visual_projection[Visual Projection<br/>1024 x 768]:::simple
image_input --> patch_embedding
patch_embedding --> positional_embedding_vision
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
<!-- https://huggingface.co/blog/stable_diffusion -->

したがって、実はStable Diffusionから純粋なCLIPのText Encoderを取り出して使うことができませんでした。

## 実装方針

はじめにCLIPの実装ですが、OpenAI CLIPを用いました。今から考えると、Huggingface TransformersのCLIPのCLIPTextModelなどを使ったほうが楽だったかもしれませんが...

リサーチ

OpenAI CLIPを使った方法はあった。しかしライブラリのコードをいじる。これは再現性が低くなる。
別の方法を検討したい（でもよく考えたらコードをフォークしたらそれで済んだのでは）
https://github.com/openai/CLIP/issues/113

CLIPTextModel使ったほうが絶対楽だったぞ...
https://huggingface.co/docs/transformers/en/model_doc/clip#transformers.CLIPTextModel

実装の大まかな方針

無いレイヤーはCLIPから取ってくる
CLIPのstate_dictからのロード機能を使う

TODO

- [ ] state_dictとは？（なんか共通規格っぽいけど誰が定義したの？Torch？）
- [ ] 実際のレイヤーは階層構造になっているけど、state_dictでは開いている

ソース

https://github.com/xhiroga/til/tree/main/software-engineering/openai/clip/_src/text-encoder-only


評価

ViT/L-14と、同じテキストを埋め込みにしてコサイン類似度を測る

結果

<!-- https://claude.ai/chat/19b185aa-b5be-49f1-a73b-9a88a38fab48, https://claude.ai/chat/8134061b-e048-424f-96f1-a224ce43e764 -->
