# Draft

前提知識

CLIPの構成

CLIPの有名な図解乗せておく
https://github.com/openai/CLIP/blob/main/CLIP.png

ブロックごとに解説

- Text Encoder
- Image Encoder
- (アウトプットするやつ)

TODO

- [ ] 頭にtransformerともvisiualとも書いてないのは何者？以下6つが該当

k='positional_embedding', v.shape=torch.Size([77, 768])
k='text_projection', v.shape=torch.Size([768, 768])
k='logit_scale', v.shape=torch.Size([])

k='token_embedding.weight', v.shape=torch.Size([49408, 768])
k='ln_final.weight', v.shape=torch.Size([768])
k='ln_final.bias', v.shape=torch.Size([768])

Stable Diffusionの解説

この図解載せる
https://jalammar.github.io/images/stable-diffusion/stable-diffusion-unet-steps.png

Text Encoder, Image Information Creater, Image Decoder

TODO

- [ ] Tokenizerはどこに？というか、モデルと合致したTokenizerであることはどこで確認すれば良い？

リサーチ

OpenAI CLIPを使った方法はあった。しかしライブラリのコードをいじる。これは再現性が低くなる。
別の方法を検討したい（でもよく考えたらコードをフォークしたらそれで済んだのでは）
https://github.com/openai/CLIP/issues/113

TODO

- [ ] 実はOpenAI/CLIP以外のCLIPだとTextのみのロードはある？
- [ ] そもそもTextのみのCLIPのWeight&Biasesの共通規格あったりする？

実装の大まかな方針

CLIPのstate_dictからのロード機能を使う

TODO

- [ ] state_dictとは？（なんか共通規格っぽいけど誰が定義したの？Torch？）
- [ ] 実際のレイヤーは階層構造になっているけど、state_dictでは開いている

ソース

https://github.com/xhiroga/til/tree/main/software-engineering/openai/clip/_src/text-encoder-only

TODO

- [ ] 現状コサイン類似度がNaNなのでコードがおかしい


評価

ViT/L-14と、同じテキストを埋め込みにしてコサイン類似度を測る

TODO

- [ ] 全く同じCLIPで測ったり、サイズの違うモデルで測ったりしてもいいかも。じゃないと数字の大小がわからない
