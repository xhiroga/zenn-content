---
title: "言語・記号創発に関するサーベイの私的まとめ"
emoji: "🐘"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## 動機

イラストを生成ではなく創発するエージェントについて考えています。

そのようなエージェントは、自らの判断で伝えたい内容に合わせて強調・省略したイラストを描く能力があるはずです。

考え方のヒントを求めて、言語・記号創発の分野の研究を初心者なりにサーベイしました。

## 対象の論文

主に Lazaridou & Baroni (2020)[^Lazaridou_Baroni_2020] をまとめました。
Lazaridou & Baroni (2020)[^Lazaridou_Baroni_2020] は代表的な実験である Lazaridou et al. (2017)[^Lazaridou_et_al_2017]の著者が執筆しているほか、他のサーベイでも言及されているため、最初に読むべき内容として適切だと考えています。
ほかに、キーワードを拾ったり個別の観点を補強するために、次のサーベイを簡単に読みました。

- N. Brandizzi, “Toward More Human-Like AI Communication: A Review of Emergent Communication Research,” IEEE Access, vol. 11, pp. 142317–142340, 2023, doi: 10.1109/ACCESS.2023.3339656.[^Brandizzi_2023]
- B. Boldt and D. R. Mortensen, “A Review of the Applications of Deep Learning-Based Emergent Communication,” Transactions on Machine Learning Research, Aug. 2023, Accessed: Sep. 24, 2024. [Online]. Available: <https://openreview.net/forum?id=jesKcQxQ7j>[^Boldt_Mortensen_2024]
- R. Ueda et al., “言語とコミュニケーションの創発に関する構成論的研究の展開,” Jun. 07, 2023, OSF. doi: 10.31234/osf.io/rz5ng.[^Ueda_et_al_2023]

<!-- Brandizzi (2023) はこの分野の初心者向けに書かれている印象がある。 -->
<!-- Boldt & Mortensen (2024) は Lazaridou(2024) を踏まえた上で、応用（ロボットを含む）を扱っている（その中で谷口らも取り上げている） -->
<!-- Ueda et al. (2023) は日本語のサーベイとして一読した。 -->

また、特に技術的な課題については Chaabouni (2024)[^Chaabouni_2024]を参考にしました。

### コミュニティ

創発コミュニケーションについての研究は、主に次の学会で発表されているようです。[^Boldt_Mortensen_2024][^Okanohara_2020]

- ICLR: 深層学習に特化しており、新規性を求める
  - EmeCom WorkShop: ICLRのワークショップ
- NeurIPS: 神経科学や人工知能もカバーしており、強化学習にも重きをおいている
- ICML: 機械学習のアルゴリズムや理論に重点を置いている

EmeCom WorkShopの名称から分かるように、この分野はEmergent Communication = EmeCom と呼ばれることがあります。

## 言語・記号創発の課題

言語・記号創発の分野では、深層ニューラルネットワークによるエージェント同士のコミュニケーションを研究します。中でも、次のような課題があります。

<!-- 1. Lazaridou & Baroni (2020): 3. 創発的な言葉を理解する -->
<!-- 2. Lazaridou & Baroni (2020): 4. より良いAIのための創発的コミュニケーション -->
<!-- 3-1. Chaabouni (2024) -->
<!-- 3-2. Lazaridou & Baroni (2020): その分析を高速化し一般化できる自動化ツールを開発すること -->
<!-- 4. 数理的な理解: https://www.anlp.jp/proceedings/annual_meeting/2024/pdf_dir/E7-5.pdf など？ -->

1. 創発言語の分析と評価
   1. 創発プロトコルの一般的特徴の理解
   2. 構成性をどのように発現させるか・評価するか
   3. 人間の言語との類似性をどのように分析・評価するか
2. 創発言語の応用
   1. 機械-機械間コミュニケーションの改善
   2. 機械-人間間コミュニケーションの改善
   3. LLMと創発言語の組み合わせ
3. 学習
   1. 学習のスケールアップ
   2. 学習の効率化・自動化
4. 数理的な理解

## 代表的な実験

創発コミュニケーションの課題の詳細の前に、それらの実験の共通の性質と違いについて確認します。

<!-- 主に Brandizzi (2023) から引用 -->
<!-- Boldt & Mortensen (2024): 1.2 Scope に相当 -->

共通の性質は次のとおりです。

- 強化学習を用いることが多い
- 複数のエージェントが記号などを送信し合う
- ゲームの開始時、エージェントが送信する記号は意味を持たない（ゲームを通じて意味や構文が生まれる）

一方で、研究によって次のような観点で違いが見られます。続けて詳細を取り上げます。

<!-- 構成: Lazaridou & Baroni (2020) が網羅的。しかし、メッセージの長さなど比較的軽い要素が含まれる。しかし影響が大きい観点に絞るため、 Brandizzi (2023) II. COMMON PROPERTIES に倣うのが良いかも。 -->
<!-- ゲームの種類: Lazaridou et al. (2020) および Brandizzi (2023) にならって2分した。 -->
<!-- 入力表現:  -->

- ゲームの種類
  - コミュニケーション自体に焦点を当てたゲーム
  - コミュニケーションを補助として用いるゲーム
- 入力表現
  - 数字
  - 画像
  - 3Dオブジェクト
- 学習パラダイム
  - 強化学習
  - 教師あり学習による事前学習
- エージェント間の関係
  - 完全協力
  - 部分的協力/競争
- メッセージの長さ
  - 固定長: Lazaridou et al. (2017) など
  - 可変長: Havrylov & Titov など
  - ほか、単一か複数かといった切り口もある
- ターン
  - 単一ターン
  - マルチターン: Evtimova...
- エージェント数
  - 2エージェント
  - 多エージェント
- コミュニティ数
  - 単一のコミュニティ
  - 複数のコミュニティ

### ゲームの種類

創発コミュニケーションの実験で用いられる、次のようなゲームをシグナリングゲーム(signaling game)と言います。

- 送信者と受信者からなる
- 送信者はタイプを持つ。受信者は送信者のタイプを知らない
- 送信者は受信者にメッセージを送る。受信者はメッセージを元に行動する

<!-- https://ja.wikipedia.org/wiki/シグナリングゲーム -->
<!-- 本来は Lewis, D. (1969) を引用すべきだが、時間があるときにする。 -->

#### コミュニケーション自体に焦点を当てたゲーム

シグナリングゲームの中でも、受信者が送信者のタイプを当てるシンプルなゲームを参照ゲーム(referential game)と言います。なお参照ゲームをシグナリングゲームと呼ぶこともあります。

参照ゲームの代表的な例として、Lazaridou et al. (2017)[^Lazaridou_et_al_2017]の実験があります。これは参照ゲームの中でも識別ゲーム(discrimination game)と呼ばれることがあります。

実験では、送信者が2枚の画像から1枚を選び、受信者に1つの記号(10または100の語彙から選ばれた1語)を送信します。受信者にはシャッフルされた2枚の画像と送信された記号が提示されるので、どちらの画像を指しているかを当てるタスクを行います。

<!-- https://claude.ai/chat/eb5f6938-d36d-42f3-af66-0f22a493cca6 -->
<!-- Brandizzi (2023) は識別ゲーム以外の参照ゲームの例を挙げているが、あくまで参考程度なので省略する。 -->

#### コミュニケーションを補助として用いるゲーム

コミュニケーションを補助的な手段として用いることで、道案内や交渉などを成功させる種類のゲームがあります。参照ゲームではないシグナリングゲームの一種で、指示ゲームと呼ばれることがあります。

<!-- Ueda et al. (2023) などに登場する「指示ゲーム」の英語表現を見つけることができなかった。 -->

例えば、Das et al. (2019)[^Das_et_al_2024]の研究では、エージェントが協力して3D環境内で道案内などを実施します。研究では、中央集権的なアーキテクチャなどに比べても優れたパフォーマンスを示すことが報告されています。

<!-- https://claude.ai/chat/0ab5ff3c-e112-4539-a30c-3202629c12b6 -->

もっとも、創発言語の構成性や人間の言語との類似性を発現させるにあたって、参照ゲームよりも指示ゲームのような複雑なゲームの方が有利といった報告は見つけられませんでした。指示ゲームについてはより研究が必要な状況のようです。

<!-- https://claude.ai/chat/08e448a1-e8d8-4624-9a0b-136b9cea62a5 -->

### 入力表現

創発コミュニケーションのためのシグナリングゲームにおいて、様々な入力表現が用いられます。単なる数字のセット、Bag of Featureとして特徴を抽出したセット、画像、3Dオブジェクトなどです。

<!-- https://claude.ai/chat/75748675-15c9-47c0-a791-1c533818624a -->

Yuan et al. (2021)[^Yuan_et_al_2021]の実験では、参照ゲームの入力表現として単なる数字のセットと3Dオブジェクトを比較しています。実験の結果、3Dオブジェクトを用いた方が学習の収束が早いことが分かっています。入力表現が色や形などの構造化された情報を持つことが、学習の収束につながったと考えられています。

また、Chaabouni et al. (2020)[^Chaabouni_et_al_2020]の実験では、入力空間の十分な広さが汎化性能、つまり新しい複合概念を参照する能力を発達させることが示されています。

### 学習パラダイム

創発コミュニケーションでは主に強化学習が用いられます。アルゴリズムとしてはREINFORCEが用いられるほか、学習を効率化する目的でGumbel Softmaxなどが用いられます。[^Brandizzi_2023]

<!-- Havrylov & Titov (2017) https://chatgpt.com/c/66f53b2c-3680-8010-84de-fcdcd5e7557f -->
<!-- Brandizzi (2023) https://claude.ai/chat/74c875b5-af8e-4bb9-a25a-6778dd92f7e4 -->

エージェント間の通信を微分可能にする試みとしては、他にDIAL（Differentiable Inter-Agent Learning, 微分可能エージェント間学習）[^Foerster_2016]があります。

人間の言語が離散的であるのに対して、DIALでは訓練時に連続的なベクトルを渡すことができます。大雑把に言えば、訓練時は「66.6%の確率で数字の8」というやり取りをするのに対して、実行時は「おそらく数字の8」という伝え方をするようなものです。
実験では、訓練時に連続的なベクトルを渡すことは学習を促進するものの、連続的なべクトルに適切なノイズを加えることが離散的な値への移行を促すことが報告されています。

<!-- https://claude.ai/chat/d7a065b3-faa9-4a2c-80eb-23c0e5fa12ff -->

エージェントの言語を人間の言語に似せることを目的として、教師あり学習による事前学習が用いられることもあります。

### エージェント間の関係

複数のエージェントが協力してタスクに取り組むことが、コミュニケーションの創発を促す。しかし、複数のエージェントからなるチームを競争させることで、より情報量が多く構成的な創発言語が発現する可能性がある。

Liang et al. (2020)[^Liang_et_al_2020]の研究によれば、複数チーム間での参照ゲームにおいて、他チームの創発言語を盗み聞きした場合、他チームのタスクやインスタンスを知らない場合であっても、自チームのゲームの正解率がより早く収束するようになったことが報告されている。

<!-- https://claude.ai/chat/1c000ba6-8b01-403f-b6e6-e8d1fec05366 -->

## 創発言語の分析と評価に関する取り組みと課題

創発言語の分析と評価について、主な取り組みと課題は以下の通り。

1. コミュニケーションの効果
2. トポグラフィック類似性 & ダイエンタングルメント & 言語の圧縮性
3. 人間の言語との比較

### コミュニケーションの効果

送信者と受信者それぞれの分析手段がある。

実験の形式に注意しないと、メッセージではなくターン数などのメタ的な要素でコミュニケーションを取る可能性がある。

<!-- Lazaridou & Baroni (2020): 今後の研究が取り組むべきさらに深刻な問題は、現在のシミュレーションの大部分において、訓練段階とテスト段階におけるエージェントのコミュニケーション・パートナーが同じであること、 すなわち、一緒に訓練されたエージェント間のコミュニケーションを評価することである。 -->

### トポグラフィック類似性 & ダイエンタングルメント & 言語の圧縮性

Brighton & Kirby (2006)によって導入された指標で、意味空間と形式空間の間の構造的類似性を測定します[4]。この指標は言語の構成性を間接的に評価しますが、具体的な構成過程を明らかにすることはできません。

ダイエンタングルメントは、表現学習において、データの生成過程における独立した要因を分離して表現することを目指す概念です。Andreas (2019)[6]は、この概念を創発言語の分析に適用しました。具体的には、言語の異なる側面（例えば、文法的機能と意味内容）を独立した表現として分離することを試みています。これにより、創発言語のどの部分がどのような機能を担っているかをより詳細に分析することが可能になります。
なので予め文法が分かっていないと性質が分からない。

Chaabouni et al. (2020)は、創発言語の構成性を評価するために、表現の圧縮性を利用する方法を提案しています[5]。構成的な言語はより効率的に圧縮できるという考えに基づいています。

課題: どのような建築的バイアスや環境的圧力が構成性の出現を促すかはかなり大雑把な理解。世代を経たり、話者数が多いと構成的になる傾向があるらしい。
また一般性も課題となっている
一方で Chaabouni et al. (2021)では、大きな母集団が高い汎化性能を持つ頑健なプロトコルを誘因「しない」ことを報告している。

### 人間の言語との比較

<!-- TODO: 出現した言語が直感に反する性質を持つケースと、その理由の分析 -->

Lazaridou et al. (2020)[7]は、事前学習された言語モデルを用いて創発言語と自然言語の類似性を評価する方法を提案しています。具体的には、以下のようなアプローチを取っています：

1. 大規模な自然言語コーパスで事前学習された言語モデルを用意する。
2. このモデルを創発言語のタスクに適応させる。
3. 適応後のモデルの性能や内部表現を分析し、創発言語がどの程度自然言語の特性を獲得しているかを評価する。

このアプローチにより、創発言語が自然言語のどのような特性を獲得し、どのような点で異なっているかを定量的に評価することが可能になります。ただし、この方法にはまだ課題があり、例えば使用する事前学習モデルの選択や、評価指標の設計などにさらなる研究が必要です。

## 創発言語の応用に関する取り組みと課題

創発言語の応用に関する取り組みと課題は次のとおりです。

機械-機械間コミュニケーションの改善：Foerster et al. (2016)のDIALシステムなど、連続的なコミュニケーションチャネルを用いることで、エージェント間の協調をより効果的に行う研究が進められています[8]。
機械-人間間コミュニケーションの改善：Lee et al. (2019)は、創発コミュニケーションと教師あり学習を組み合わせることで、人間の言語に近い創発言語を生成する方法を提案しています[9]。
LLMと創発言語の組み合わせ：Lu et al. (2020)は、反復学習のパラダイムを用いて、事前学習された言語モデルと創発言語学習を組み合わせる手法を提案しています[10]。これにより、タスク特化型の言語を生成しつつ、自然言語との類似性を保つことを目指しています。

TODO: 人間の子どもがそうであるように、言語を進化させつつスコアを引き上げられると良い。論文ではタスクのスコアと中間表現の英語としてのBLUEスコアで計測している。

<!-- SIL successfully counters language drift in the translation game while optimizing for task-completion. -->

<!-- https://claude.ai/chat/f532a165-ad61-47ad-b628-bf1c4e7440d8 -->
<!-- Lu et al. (2020) https://claude.ai/chat/aacd8dae-b571-496f-add1-b262f88cf3a4 -->

## まとめ

<!-- TODO -->

## 参考文献

[^Boldt_Mortensen_2024]: B. Boldt and D. R. Mortensen, “A Review of the Applications of Deep Learning-Based Emergent Communication,” Transactions on Machine Learning Research, Aug. 2023, Accessed: Sep. 24, 2024. [Online]. Available: <https://openreview.net/forum?id=jesKcQxQ7j>
[^Brandizzi_2023]: N. Brandizzi, “Toward More Human-Like AI Communication: A Review of Emergent Communication Research,” IEEE Access, vol. 11, pp. 142317–142340, 2023, doi: 10.1109/ACCESS.2023.3339656.
[^Chaabouni_et_al_2020]: R. Chaabouni, E. Kharitonov, D. Bouchacourt, E. Dupoux, and M. Baroni, “Compositionality and Generalization In Emergent Languages,” in Proceedings of the 58th Annual Meeting of the Association for Computational Linguistics, D. Jurafsky, J. Chai, N. Schluter, and J. Tetreault, Eds., Online: Association for Computational Linguistics, Jul. 2020, pp. 4427–4442. doi: 10.18653/v1/2020.acl-main.407.
[^Chaabouni_2024]: R. Chaabouni et al., “Emergent Communication at Scale,” presented at the International Conference on Learning Representations, Oct. 2021. Accessed: Sep. 24, 2024. [Online]. Available: <https://openreview.net/forum?id=AUGBfDIV9rL>
[^Das_et_al_2024]: A. Das et al., “TarMAC: Targeted Multi-Agent Communication,” in Proceedings of the 36th International Conference on Machine Learning, PMLR, May 2019, pp. 1538–1546. Accessed: Sep. 26, 2024. [Online]. Available: <https://proceedings.mlr.press/v97/das19a.html>
[^Foerster_2016]: J. N. Foerster, Y. M. Assael, N. de Freitas, and S. Whiteson, “Learning to Communicate with Deep Multi-Agent Reinforcement Learning,” May 24, 2016, arXiv: arXiv:1605.06676. doi: 10.48550/arXiv.1605.06676.
[^Lazaridou_et_al_2017]: A. Lazaridou, A. Peysakhovich, and M. Baroni, “Multi-Agent Cooperation and the Emergence of (Natural) Language,” arXiv.org. Accessed: Sep. 11, 2024. [Online]. Available: <https://arxiv.org/abs/1612.07182v2>
[^Lazaridou_Baroni_2020]: A. Lazaridou and M. Baroni, “Emergent Multi-Agent Communication in the Deep Learning Era,” Jul. 14, 2020, arXiv: arXiv:2006.02419. Accessed: Sep. 19, 2024. [Online]. Available: <http://arxiv.org/abs/2006.02419>
[^Liang_et_al_2020]: P. P. Liang, J. Chen, R. Salakhutdinov, L.-P. Morency, and S. Kottur, “On Emergent Communication in Competitive Multi-Agent Teams,” Jul. 16, 2020, arXiv: arXiv:2003.01848. doi: 10.48550/arXiv.2003.01848.
[^Yuan_et_al_2021]: L. Yuan, Z. Fu, J. Shen, L. Xu, J. Shen, and S.-C. Zhu, “Emergence of Pragmatics from Referential Game between Theory of Mind Agents,” Sep. 30, 2021, arXiv: arXiv:2001.07752. doi: 10.48550/arXiv.2001.07752.

[^Okanohara_2020]: Okanohara D., “《日経Robotics》AIトップ国際会議では何が起きているか,” 日経Robotics（日経ロボティクス）. Accessed: Sep. 24, 2024. [Online]. Available: <https://xtech.nikkei.com/atcl/nxt/mag/rob/18/00007/00022/>
[^Ueda_et_al_2023]: R. Ueda et al., “言語とコミュニケーションの創発に関する構成論的研究の展開,” Jun. 07, 2023, OSF. doi: 10.31234/osf.io/rz5ng.
