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

また、特に技術的な課題については Chaabouni (2024)[^Chaabouni_et_al_2024]を参考にしました。

### コミュニティ

創発コミュニケーションについての研究は、主に次の学会で発表されているようです。[^Boldt_Mortensen_2024][^Okanohara_2020]

- ICLR: 深層学習に特化しており、新規性を求める
  - EmeCom WorkShop: ICLRのワークショップ
- NeurIPS: 神経科学や人工知能もカバーしており、強化学習にも重きをおいている
- ICML: 機械学習のアルゴリズムや理論に重点を置いている

EmeCom WorkShopの名称から分かるように、この分野はEmergent Communication = EmeCom と呼ばれることがあります。

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

## 課題

言語・記号創発の分野では、深層ニューラルネットワークによるエージェント同士のコミュニケーションを研究します。中でも、次のような課題があります。

<!-- 1. Lazaridou & Baroni (2020): 3. 創発的な言葉を理解する -->
<!-- 2. Lazaridou & Baroni (2020): 4. より良いAIのための創発的コミュニケーション -->
<!-- 3. 数理的な理解: https://www.anlp.jp/proceedings/annual_meeting/2024/pdf_dir/E7-5.pdf など？ -->

1. 創発言語の分析と評価
   1. 効果的なコミュニケーションの度合い
   2. 構成性
2. 創発言語の応用
   1. 機械どうしのコミュニケーション
   2. 機械と人間のコミュニケーション

<!-- https://aistudio.google.com/app/prompts/1B07r2OCU-duu9Py64sqTPmFHNRKTQyd6 -->

### 創発言語の分析と評価

<!-- TODO: 簡単に説明 -->

#### コミュニケーションの効果

<!-- Lazaridou & Baroni (2020): 3.1 Measuring the Degree of Effective Communication および 2) Brandizzi (2023): INPUT REPRESENTATION -> C) OPEN CHALLENGES を参考に -->

創発コミュニケーションの研究において、エージェントが実際にコミュニケーションを取れているのか、その効果をどのように測るかは重要な課題です。

最もシンプルな指標としては、**タスク成功率** が挙げられます。例えば、画像を説明するゲームで、聞き手が正しく画像を識別できた割合を見る方法です。しかし、タスク成功率は、エージェントが意図したとおりにコミュニケーションを取れているかどうかを必ずしも反映しません。

例えば、エージェントがメッセージの内容ではなく、メッセージを送信するターン数などのメタ的な情報に基づいてタスクをクリアしてしまう可能性があります。[^Brandizzi_2023] これは、私たち人間がコミュニケーションにおいて想定するような、意味に基づいたやり取りが行われていないことを意味します。

このような問題に対処するために、様々な分析・評価方法が提案されていますが、それぞれにまだ未解決の課題も残されています。

#### 相互理解度 (Mutual Intelligibility)

**相互理解度**は、エージェントが自分自身とコミュニケーションを取ることができるかどうかを測定する指標です。[^Brandizzi_2023] もしエージェントが共有されたコミュニケーションプロトコルを学習していれば、自分自身とゲームをプレイしても問題なくタスクをこなせるはずです。これは、エージェントが学習した言語が一貫性を持っているかどうかを評価するのに役立ちます。

* **課題:** 送信者と受信者の役割が固定されているエージェントにしか適用できない。また、言語の複雑さ（例えば、文法や語彙の豊かさ）を捉えきれていない可能性がある。

#### ゼロショット性能 (Zero-Shot Performance)

**ゼロショット性能**は、訓練データに含まれていない未知の入力に対して、エージェントがどのように対応できるかを評価する指標です。[^Brandizzi_2023]  創発コミュニケーションにおいては、エージェントが未知の状況にも対応できる汎用的な言語を学習しているかどうかを判断するのに役立ちます。

* **課題:**  適切な評価データセットを準備することが難しい。また、タスクやドメインに特化した評価になりがちで、言語の汎用的な能力を測ることが難しい。

#### 話し手の一貫性 (Speaker Consistency)

**話し手の一貫性**は、エージェントが特定の行動をとる際に、どの程度一貫して同じメッセージを発信するかを測定します。[^Brandizzi_2023] 例えば、「右に移動」という行動をとる際に、常に "move right" というメッセージを発信するエージェントは、話し手の一貫性が高いと言えます。

単純なタスク成功率では、このようなメッセージと行動の一貫性を評価することができません。話し手の一貫性を測定することで、エージェントが意味のある方法で言語を使用しているかどうかをより深く理解することができます。

* **課題:**  聞き手の行動を考慮していないため、コミュニケーション全体における一貫性を捉えきれていない。

同様に、**話し手とメッセージの一貫性**や**聞き手とメッセージの一貫性**を測定することで、コミュニケーションにおけるそれぞれの役割において、エージェントがどの程度一貫して言語を使用しているかを評価できます。

#### 因果的影響 (Causal Influence)

**因果的影響**は、送信者のメッセージが受信者の行動にどの程度影響を与えているかを測定します。[^Lazaridou_Baroni_2020] この指標は、エージェント間のコミュニケーションが実際にタスクの成功に貢献しているかどうかを判断するのに役立ちます。

* **課題:** 計算コストが高く、実装が複雑であるため、大規模な実験への適用が難しい。

#### 転移学習 (Ease and Transfer Learning / ETL)

**転移学習 (ETL)** は、創発言語が、異なるタスクを実行する新しいリスナーにどの程度速く、そしてうまく伝達されるかを捉える指標です。[^Chaabouni_2022] これは、エージェントが学習した言語が、特定のタスクに特化したものではなく、より汎用的なコミュニケーション能力を獲得しているかどうかを評価するのに役立ちます。

* **課題:** 特定のタスクにおける有用性しか評価できないため、言語の汎用性や人間とのコミュニケーションにおける有用性については、さらなる分析が必要となる。

これらの分析・評価方法は、創発コミュニケーション研究において重要な役割を果たしていますが、未解決の課題も存在します。今後、これらの課題を克服し、より洗練された評価指標を開発することで、人間のようにコミュニケーションできるAIの実現に近づくことができると期待されます。

#### 構成性

<!-- Lazaridou & Baroni (2020): 3.2 Compositionality を参考に -->

**構成性 (Compositionality)** は、複雑な表現の意味が、その構成要素の意味とそれらを組み合わせる規則によって決定されるという性質です。自然言語においては、単語の意味と文法規則から文の意味を理解できることが構成性の例として挙げられます。

創発コミュニケーションにおいても、エージェントが学習した言語が構成的であるかどうかは重要な関心事です。構成的な言語は、新しい概念やアイデアを表現する能力が高く、未知の状況にも柔軟に対応できる可能性を秘めているからです。

構成性を評価するための手法として、以下のようなものが提案されています。

構成性を評価する指標として、Zipfの法則 1 への適合度が挙げられます。Zipfの法則は、自然言語において、単語の出現頻度とその順位の間には、逆比例の関係があることを示す法則です。具体的には、出現頻度が $r$ 番目に高い単語の出現頻度は、最も頻度の高い単語の出現頻度の $\frac{1}{r^\alpha}$ に比例します。

創発言語においても、Zipfの法則に従うような単語の出現頻度分布が見られる場合、その言語が自然言語と似た統計的な構造を持っていることを示唆し、構成性を持つ可能性を高めます。いくつかの研究 2 では、創発言語がどのような条件下でZipfの法則に近づくのかを検証しており、構成性の発現要因を探る手がかりとして注目されています。

課題: Zipfの法則は、あくまで単語の出現頻度分布という巨視的な側面を捉えたものであり、構成性そのものを直接評価するものではありません。また、Zipfの法則に従わない言語でも構成性を持つ可能性は否定できません。

#### トポグラフィック類似性 (Topographic Similarity)

**トポグラフィック類似性**[^Brighton_Kirby_2006] は、意味空間における距離と、それに対応する信号空間における距離の相関関係を測定することで、言語の構成性を間接的に評価する指標です。

例えば、意味空間において「猫」と「犬」が近く、「車」と「飛行機」が近いとします。このとき、構成的な言語であれば、信号空間においても「猫」を表す信号と「犬」を表す信号が近く、「車」を表す信号と「飛行機」を表す信号が近いという関係が見られるはずです。

* **課題:** トポグラフィック類似性は、言語の全体的な構造を捉えることはできますが、具体的な構成プロセス（例えば、どのような規則で要素を組み合わせているか）を明らかにすることはできません。
* **課題:** Chaabouni et al. (2022) では、大規模な自然画像データセットを用いた実験において、トポグラフィック類似性とエージェントの汎化性能の間に相関が見られないことを報告しており、自然画像を扱う際には、トポグラフィック類似性だけでは構成性を十分に評価できない可能性が示唆されています。

#### ダイエンタングルメント (Disentanglement)

**ダイエンタングルメント**は、表現学習において、データの生成過程における独立した要因を分離して表現することを目指す概念です。創発言語の分析においては、言語の異なる側面（例えば、文法的機能と意味内容）を独立した表現として分離することで、言語の構成性を評価することができます。

Andreas (2019) [^Andreas_2019] は、**Tree Reconstruction Error (TRE)** と呼ばれる指標を提案しています。TREは、観測された表現を近似するように構成要素を最適化し、その近似の良さを評価することで構成性を測定します。例えば、あるオブジェクトが「赤い丸」と表現される場合、TREは「赤」と「丸」を表現するベクトルを学習し、それらを組み合わせて「赤い丸」の表現をどれだけ正確に再現できるかを評価します。

* **課題:** TREは構成要素と構成規則を事前に定義する必要があるため、あらかじめ言語の構造についての知識が必要です。これは、自然言語のように複雑な構造を持つ言語に対しては、適用が難しい場合があります。

#### 言語の圧縮性 (Language Compressibility)

Chaabouni et al. (2020) [^Chaabouni_2020] は、創発言語の構成性を評価するために、**言語の圧縮性**に着目しています。構成的な言語は、要素の意味と組み合わせ規則を明確に定義することで、より少ない情報量で表現することができるという考えに基づいています。

例えば、「赤い丸」と「青い三角」を表現する場合、構成的な言語であれば、「赤」、「青」、「丸」、「三角」という4つの要素と、色と形を組み合わせるという規則を定義するだけで済みます。一方、非構成的な言語では、それぞれのオブジェクトに対して個別の表現を定義する必要があるため、より多くの情報量が必要となります。

* **課題:** 言語の圧縮性は、構成性を評価する上で有用な指標となりえますが、圧縮率と構成性の関係は複雑であり、圧縮率だけで構成性を完全に評価することは難しい場合があります。


#### 構成性の発現

構成性の発現には、エージェントのアーキテクチャ、学習アルゴリズム、タスクの性質、環境の複雑さなど、様々な要因が影響を与えることが分かっています。

* **世代を超えた学習:** 世代を超えた言語の伝達をシミュレートする反復学習 (Iterated Learning) においては、世代が進むにつれて言語がより構成的になる傾向が見られることが報告されています。[^Kirby_2002]
* **話者数の増加:** 話者数が多いほど、より体系的で構成的な言語が進化する傾向があることが、人間の実験およびシミュレーションによって示されています。[^Raviv_2019a] [^Raviv_2019b] 
* **環境の複雑さ:**  単純なタスクや環境では、エージェントは構成的でない言語を使ってもタスクを達成できてしまうため、構成性が発現しにくい傾向があります。[^Kottur_2017] より複雑なタスクや環境を設定することで、エージェントは構成的な言語を学習する必要性に迫られ、構成性が発現しやすくなると考えられています。

しかし、Chaabouni et al. (2022) [^Chaabouni_2022] は、大きな母集団が高い汎化性能を持つ頑健なプロトコルを誘因「しない」ことを報告しています。

このように、どのような建築的バイアスや環境的圧力が構成性の出現を促すかは、まだ完全には解明されていません。今後の研究によって、これらの要因がどのように相互作用し、構成性の発現に影響を与えるのかを明らかにすることが期待されます。

### 創発言語の応用

#### 機械どうしのコミュニケーション

<!-- Lazaridou & Baroni (2020): 4.1 Communication Facilitating Inter-Agent Coordination および 4.2 Beyond Cooperation: Self-interested and Competing Agents を参考に -->

#### 機械と人間のコミュニケーション

<!-- Lazaridou & Baroni (2020): 4.3 Machines Cooperating with Humans を参考に -->

- Lu et al. (2020)[^Lu_et_al_2020]は、創発コミュニケーションと教師あり学習を組み合わせることで、人間の言語に近い創発言語を生成する方法を提案しています[9]。
- 言語ドリフトについて触れる

<!-- SIL successfully counters language drift in the translation game while optimizing for task-completion. -->

<!-- https://claude.ai/chat/f532a165-ad61-47ad-b628-bf1c4e7440d8 -->
<!-- Lu et al. (2020) https://claude.ai/chat/aacd8dae-b571-496f-add1-b262f88cf3a4 -->

TODO: 人間の子どもがそうであるように、言語を進化させつつスコアを引き上げられると良い。論文ではタスクのスコアと中間表現の英語としてのBLUEスコアで計測している。そのような、言語としての要求を満たしつつ言語を進化させるところに創造性がある、みたいなことを主張する


## まとめ

<!-- TODO -->

## 参考文献

[^Boldt_Mortensen_2024]: B. Boldt and D. R. Mortensen, “A Review of the Applications of Deep Learning-Based Emergent Communication,” Transactions on Machine Learning Research, Aug. 2023, Accessed: Sep. 24, 2024. [Online]. Available: <https://openreview.net/forum?id=jesKcQxQ7j>
[^Brandizzi_2023]: N. Brandizzi, “Toward More Human-Like AI Communication: A Review of Emergent Communication Research,” IEEE Access, vol. 11, pp. 142317–142340, 2023, doi: 10.1109/ACCESS.2023.3339656.
[^Brighton_Kirby_2006]: H. Brighton and S. Kirby, “Understanding Linguistic Evolution by Visualizing the Emergence of Topographic Mappings”.
[^Chaabouni_et_al_2020]: R. Chaabouni, E. Kharitonov, D. Bouchacourt, E. Dupoux, and M. Baroni, “Compositionality and Generalization In Emergent Languages,” in Proceedings of the 58th Annual Meeting of the Association for Computational Linguistics, D. Jurafsky, J. Chai, N. Schluter, and J. Tetreault, Eds., Online: Association for Computational Linguistics, Jul. 2020, pp. 4427–4442. doi: 10.18653/v1/2020.acl-main.407.
[^Chaabouni_et_al_2024]: R. Chaabouni et al., “Emergent Communication at Scale,” presented at the International Conference on Learning Representations, Oct. 2021. Accessed: Sep. 24, 2024. [Online]. Available: <https://openreview.net/forum?id=AUGBfDIV9rL>
[^Das_et_al_2024]: A. Das et al., “TarMAC: Targeted Multi-Agent Communication,” in Proceedings of the 36th International Conference on Machine Learning, PMLR, May 2019, pp. 1538–1546. Accessed: Sep. 26, 2024. [Online]. Available: <https://proceedings.mlr.press/v97/das19a.html>
[^Foerster_2016]: J. N. Foerster, Y. M. Assael, N. de Freitas, and S. Whiteson, “Learning to Communicate with Deep Multi-Agent Reinforcement Learning,” May 24, 2016, arXiv: arXiv:1605.06676. doi: 10.48550/arXiv.1605.06676.
[^Lazaridou_et_al_2017]: A. Lazaridou, A. Peysakhovich, and M. Baroni, “Multi-Agent Cooperation and the Emergence of (Natural) Language,” arXiv.org. Accessed: Sep. 11, 2024. [Online]. Available: <https://arxiv.org/abs/1612.07182v2>
[^Lazaridou_Baroni_2020]: A. Lazaridou and M. Baroni, “Emergent Multi-Agent Communication in the Deep Learning Era,” Jul. 14, 2020, arXiv: arXiv:2006.02419. Accessed: Sep. 19, 2024. [Online]. Available: <http://arxiv.org/abs/2006.02419>
[^Liang_et_al_2020]: P. P. Liang, J. Chen, R. Salakhutdinov, L.-P. Morency, and S. Kottur, “On Emergent Communication in Competitive Multi-Agent Teams,” Jul. 16, 2020, arXiv: arXiv:2003.01848. doi: 10.48550/arXiv.2003.01848.
[^Lu_et_al_2020]: Y. Lu, S. Singhal, F. Strub, A. Courville, and O. Pietquin, “Countering Language Drift with Seeded Iterated Learning,” in Proceedings of the 37th International Conference on Machine Learning, PMLR, Nov. 2020, pp. 6437–6447. Accessed: Sep. 26, 2024. [Online]. Available: https://proceedings.mlr.press/v119/lu20c.html
[^Yuan_et_al_2021]: L. Yuan, Z. Fu, J. Shen, L. Xu, J. Shen, and S.-C. Zhu, “Emergence of Pragmatics from Referential Game between Theory of Mind Agents,” Sep. 30, 2021, arXiv: arXiv:2001.07752. doi: 10.48550/arXiv.2001.07752.

[^Okanohara_2020]: Okanohara D., “《日経Robotics》AIトップ国際会議では何が起きているか,” 日経Robotics（日経ロボティクス）. Accessed: Sep. 24, 2024. [Online]. Available: <https://xtech.nikkei.com/atcl/nxt/mag/rob/18/00007/00022/>
[^Ueda_et_al_2023]: R. Ueda et al., “言語とコミュニケーションの創発に関する構成論的研究の展開,” Jun. 07, 2023, OSF. doi: 10.31234/osf.io/rz5ng.