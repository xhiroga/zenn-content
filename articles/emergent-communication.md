---
title: "創発コミュニケーションに関するサーベイの私的まとめ"
emoji: "🐘"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false

# レビュー観点
# 1. 一般的なソフトウェアエンジニアが知らない用語には、具体的な値を伴う例が示されていること
# 2. 文体が統一されているか
# 3. 引用方法が統一されているか
# 4. 複数行のコメントアウトをしていないこと（zenn.devが対応していない）
# 5. 可能なら漢語ではなく和語を用いること（例: ✗測定する ◯測る）

# レビュー方法
# 1. 
# . 
---

## TL;DR

2人以上のプレイヤーが、道案内のようなゲームに協力して取り組むとき、プレイヤー間で考えていることを伝えるために新たな意味を持った記号が生まれます。こうしたコミュニケーションを創発コミュニケーションと呼びます。創発コミュニケーションに関するサーベイを読んでまとめました。

## 動機

Stable Diffusionのような生成AIは、画像を教師データから学んでいます。見た目の良いイラストを生成できる一方で、学習には元になる画像が必要です。

そこで、もし2体のエージェントが「3Dオブジェクトで作られた環境で、絵で道案内をする」ようなタスクに取り組むことで、ゼロからイラストを学んだらどうなるかが気になりました。

調べると、イラストではなく記号や言語の分野で同じ発想の研究があり、創発コミュニケーション (EmCom, Emergent Communication)と呼ばれているようです。そこで、どのような実験や課題があるのかを調べました。

## 対象の論文

主に Brandizzi (2023)[^Brandizzi_2023] と Lazaridou & Baroni (2020)[^Lazaridou_Baroni_2020] を読みました。Brandizzi (2023)を選んだのは、最近書かれていること、私にとって構成が分かりやすかったことが理由です。また、Lazaridou & Baroni (2020) を選んだのは、この分野で先駆的な実験であるLazaridou et al. (2017)[^Lazaridou_et_al_2017]の著者によって書かれていることが理由です。

ほか、Boldt & Mortensen (2024)[^Boldt_Mortensen_2024]とUeda et al.(2024)を参照しました。前者は特に日本で盛んな「記号創発ロボティクス」をEmComの文脈で紹介していたことが、後者は構成性（compositionality, 複雑な意味が、その要素の意味＋組み合わせのルールから生まれるような性質）に関する研究を俯瞰的に紹介していたことが理由です。

その他に参照した論文については、都度紹介します。

### コミュニティ

創発コミュニケーションについての研究は、主に次の学会で発表されているようです。[^Boldt_Mortensen_2024][^Okanohara_2020]

- ICLR: 深層学習に特化しており、新規性を求める
  - EmeCom WorkShop: ICLRのワークショップ
- NeurIPS: 神経科学や人工知能もカバーしており、強化学習にも重きをおいている
- ICML: 機械学習のアルゴリズムや理論に重点を置いている

## 代表的な実験

<!-- 参考 -->
<!-- Brandizzi (2023): II. COMMON PROPRIETIES -->
<!-- Boldt & Mortensen (2024): 1.2 Scope -->

創発コミュニケーションの分野の実験を紹介します。次の特徴が共通しています。

- 強化学習を用いることが多い
- 複数のエージェントが記号などを送信し合う
- ゲームの開始時、エージェントが送信する記号は意味を持たない（ゲームを通じて意味や構文が生まれる）

一方で、次のような観点で違いが見られます。続けて詳細を取り上げます。

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

その他、メッセージの長さ、単一ターンか複数ターンか、エージェントの数、など様々な違いがあります。

### ゲームの種類

<!-- https://ja.wikipedia.org/wiki/シグナリングゲーム -->
<!-- 本来は Lewis, D. (1969) を引用すべきだが、時間があるときにする。 -->

創発コミュニケーションの実験で用いられる、次のようなゲームをシグナリングゲーム(signaling game)と言います。

- 送信者と受信者からなる
- 送信者はタイプを持つ。受信者は送信者のタイプを知らない
- 送信者は受信者にメッセージを送る。受信者はメッセージを元に行動する

#### コミュニケーション自体に焦点を当てたゲーム

シグナリングゲームの中でも、受信者が送信者のタイプを当てるシンプルなゲームを参照ゲーム(referential game)と言います。なお参照ゲームをシグナリングゲームと呼ぶこともあります。

参照ゲームの代表的な例として、Lazaridou et al. (2017)[^Lazaridou_et_al_2017]の実験があります。これは参照ゲームの中でも識別ゲーム(discrimination game)と呼ばれることがあります。

実験では、送信者が2枚の画像から1枚を選び、受信者に1つの記号(10または100の語彙から選ばれた1語)を送信します。受信者にはシャッフルされた2枚の画像と送信された記号が提示されるので、どちらの画像を指しているかを当てるタスクを行います。

<!-- Brandizzi (2023) は識別ゲーム以外の参照ゲームの例を挙げているが、あくまで参考程度なので省略する。 -->
<!-- https://claude.ai/chat/eb5f6938-d36d-42f3-af66-0f22a493cca6 -->

#### コミュニケーションを補助として用いるゲーム

コミュニケーションを補助的な手段として用いることで、道案内や交渉などを成功させる種類のゲームがあります。参照ゲームではないシグナリングゲームの一種で、指示ゲームと呼ばれることがあります。

<!-- Ueda et al. (2023) などに登場する「指示ゲーム」の英語表現を見つけることができなかった。 -->

例えば、Das et al. (2019)[^Das_et_al_2024]の研究では、エージェントが協力して3D環境内で道案内などを実施します。研究では、中央集権的なアーキテクチャなどに比べても優れたパフォーマンスを示すことが報告されています。

<!-- https://claude.ai/chat/0ab5ff3c-e112-4539-a30c-3202629c12b6 -->

もっとも、創発言語の構成性や人間の言語との類似性を発現させるにあたって、参照ゲームよりも指示ゲームのような複雑なゲームの方が効果的といった報告は見つけられませんでした。指示ゲームについてはより研究が必要な状況のようです。

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

複数のエージェントが協力してタスクに取り組むことが、コミュニケーションの創発を促します。しかし、複数のエージェントからなるチームを競争させることで、より情報量が多く構成的な創発言語が発現する可能性があります。

Liang et al. (2020)[^Liang_et_al_2020]の研究によれば、複数チーム間での参照ゲームにおいて、他チームの創発言語を盗み聞きした場合、他チームのタスクやインスタンスを知らない場合であっても、自チームのゲームの正解率がより早く収束するようになったことが報告されています。

<!-- https://claude.ai/chat/1c000ba6-8b01-403f-b6e6-e8d1fec05366 -->

## 課題

言語・記号創発の分野では、深層ニューラルネットワークによるエージェント同士のコミュニケーションを研究します。中でも、次のような課題があります。

<!-- 1. Lazaridou & Baroni (2020): 3. 創発的な言葉を理解する -->
<!-- 2. Lazaridou & Baroni (2020): 4. より良いAIのための創発的コミュニケーション -->
<!-- 3. 数理的な理解: https://www.anlp.jp/proceedings/annual_meeting/2024/pdf_dir/E7-5.pdf など？ -->

1. 創発言語の分析と評価
   1. コミュニケーションの効果
   2. 構成性
2. 創発言語の応用
   1. 機械どうしのコミュニケーション
   2. 機械と人間のコミュニケーション
3. 数学的な解釈

<!-- https://aistudio.google.com/app/prompts/1B07r2OCU-duu9Py64sqTPmFHNRKTQyd6 -->

### 創発言語の分析と評価

#### コミュニケーションの効果

<!-- 参考 -->
<!-- Lazaridou & Baroni (2020): 3.1 Measuring the Degree of Effective Communication -->
<!-- Brandizzi (2023): III. DICHOTOMY OF EMERGENT COMMUNICATION -> A. MACHINE-CENTERED EMCOM -> 1) CHARACTERISTICS -->

創発コミュニケーションの研究において、エージェントが実際にコミュニケーションを取れているのか、その効果をどのように測るかは重要な課題です。

最もシンプルな指標としては、**タスク成功率** が挙げられます。例えば、画像を説明するゲームで、聞き手が正しく画像を識別できた割合を見る方法です。しかし、タスク成功率は、エージェントが意図したとおりにコミュニケーションを取れているかどうかを必ずしも反映しません。

例えば、エージェントがメッセージの内容ではなく、メッセージを送信するターン数などのメタ的な情報に基づいてタスクをクリアしてしまう可能性があります。

もしメタ的な情報に基づいてタスクをクリアしているなら、言語が意味を持たない可能性があります。そこで、**相互理解可能性**や**話者一貫性**を測ることが考えられます。

**相互理解可能性 (Mutual Intelligibility)**[^Graesser_et_al_2020]は、エージェントが自分自身とコミュニケーションができるかどうかを測る指標です。実験では単純な参照ゲームを行います。ただし、送信者と受信者のネットワークは、実際に送信・受信に関わる部分を除いて共通の設計になっています。これによって訓練した送信者自身を受信者として使えます。そのような場合でもタスクが成功するなら、意味を持った言語が出現していると言えそうです。

<!-- https://claude.ai/chat/100c90b3-d33a-454c-870b-a970c8912adc -->

**話者一貫性 (Speaker Consistency)**は、エージェントが特定の行動をとる際に、どの程度一貫して同じメッセージを発信するかを測定します。[^Jaques_et_al_2018]例えば、「右に移動」という行動をとる際に、常に "move right" というメッセージを発信するエージェントは、話し手の一貫性が高いと言えます。

また、メッセージと行動の関係性を直接測定することも考えられます。Casual Inference of Communication[^Lowe_et_al_2019]では、メッセージとその後の行動の相互情報量を求め、その値が高いほどメッセージが行動に影響を及ぼしていると考えます。

<!-- https://claude.ai/chat/6301eaa9-2b8b-49e5-8bc5-af6961d3a693 -->

#### 構成性

<!-- 参考 -->
<!-- Lazaridou & Baroni (2020): 3.2 Compositionality -->
<!-- Brandizzi (2023): III. DICHOTOMY OF EMERGENT COMMUNICATION -> A. MACHINE-CENTERED EMCOM -> 2) HUNT FOR GENERALIZATION -->

**構成性 (Compositionality)** は、複雑な表現の意味が、その構成要素の意味とそれらを組み合わせる規則によって決定されるという性質です。自然言語においては、単語の意味と文法規則から文の意味を理解できることが構成性の例として挙げられます。

これらの分析・評価方法は、創発コミュニケーション研究において重要な役割を果たしていますが、未解決の課題も存在します。今後、これらの課題を克服し、より洗練された評価指標を開発することで、人間のようにコミュニケーションできるAIの実現に近づくことができると期待されます。

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

#### ゼロショット性能 (Zero-Shot Performance)

**ゼロショット性能**は、訓練データに含まれていない未知の入力に対して、エージェントがどのように対応できるかを評価する指標です。[^Brandizzi_2023]  創発コミュニケーションにおいては、エージェントが未知の状況にも対応できる汎用的な言語を学習しているかどうかを判断するのに役立ちます。

* **課題:**  適切な評価データセットを準備することが難しい。また、タスクやドメインに特化した評価になりがちで、言語の汎用的な能力を測ることが難しい。

#### 転移学習 (Ease and Transfer Learning / ETL)

**転移学習 (ETL)** は、創発言語が、異なるタスクを実行する新しいリスナーにどの程度速く、そしてうまく伝達されるかを捉える指標です。[^Chaabouni_et_al_2021] これは、エージェントが学習した言語が、特定のタスクに特化したものではなく、より汎用的なコミュニケーション能力を獲得しているかどうかを評価するのに役立ちます。

* **課題:** 特定のタスクにおける有用性しか評価できないため、言語の汎用性や人間とのコミュニケーションにおける有用性については、さらなる分析が必要となる。

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

しかし、Chaabouni et al. (2021) [^Chaabouni_et_al_2021] は、大きな母集団が高い汎化性能を持つ頑健なプロトコルを誘因「しない」ことを報告しています。

このように、どのような建築的バイアスや環境的圧力が構成性の出現を促すかは、まだ完全には解明されていません。今後の研究によって、これらの要因がどのように相互作用し、構成性の発現に影響を与えるのかを明らかにすることが期待されます。

### 創発言語の応用

#### 機械どうしのコミュニケーション

<!-- 参考 -->
<!-- Lazaridou & Baroni (2020): 4.1 Communication Facilitating Inter-Agent Coordination, 4.2 Beyond Cooperation: Self-interested and Competing Agents -->
<!-- Brandizzi (2023): III. DICHOTOMY OF EMERGENT COMMUNICATION -> A. MACHINE-CENTERED EMCOM -->

<!-- 
TODO:
- コミュニケーションそのものではなく、様々なタスクでより良い成果を達成するにはどうしたら良いか？（コミュニケーションはその補助）という課題の問いかけ
- 学習を促進するアプローチ。DIALなど。しかし非現実的。
- 環境の複雑さについて
- 言語外のチャンネルを使い出すことについて。パフォーマンスは上がるかもしれないがパフォーマンスは上がるかもしれないが、総発言後には寄与しないかも

MEMO:
- 
 -->

#### 機械と人間のコミュニケーション

<!-- 参考 -->
<!-- Lazaridou & Baroni (2020): 4.3 Machines Cooperating with Humans -->
<!-- Brandizzi (2023): III. DICHOTOMY OF EMERGENT COMMUNICATION -> B. HUMAN-CENTERED EmCom -->

工知能（AI）が私たちの日常生活に浸透するにつれて、AIと人間が自然言語で円滑にコミュニケーションをとれるようになる ことは、AI技術の真価を発揮するために非常に重要です。この分野の研究は、AIエージェントが人間の言語を理解し、人間と協力してタスクを解決したり、人間に役立つ情報を提供したりすることを目指しています。

人間中心型EmCom： 人間との対話を目指して
創発コミュニケーションの研究分野において、人間中心型EmCom (Human-centered EmCom) は、まさにAIと人間の自然な対話を実現することを目標としています。人間中心型EmComでは、AIエージェントに人間の自然言語 (HNL) を理解させ、それを用いて人間とコミュニケーションをとる能力を学習させます。

言語ドリフト：人間言語からの乖離という課題
人間中心型EmComにおいて、大きな課題の一つとして 言語ドリフト (Language Drift) が挙げられます。言語ドリフトとは、AIエージェントがタスクを効率的に解決しようとするあまり、学習した言語が人間の言語から徐々に乖離していく現象です。これは、エージェントが人間の言語の複雑さやニュアンスを完全に理解できていないために起こると考えられています。

言語ドリフトには、大きく分けて以下の3つのタイプがあります。1

構造的ドリフト (Structural Drift): 文法的な構造が単純化され、人間にとって不自然な文になる。例えば、「猫ですか？」が「猫？」になるなど。

意味的ドリフト (Semantic Drift): 単語の意味が本来の意味から変化してしまう。例えば、「古い教え」が「老人」という意味になるなど。

機能的ドリフト (Functional Drift): 言語が意図した行動を引き起こさなくなる。例えば、交渉で合意したにもかかわらず、エージェントが別の取引を行ってしまうなど。

教師あり学習と強化学習のバランス
言語ドリフトを防ぎ、人間が理解しやすい言語を学習させるために、人間中心型EmComでは、教師あり学習 (Supervised Learning) と 強化学習 (Reinforcement Learning) を組み合わせる方法が広く用いられています。

教師あり学習: 大規模なテキストデータを用いて、人間の言語の文法や語彙を学習させます。

強化学習: 特定のタスクを通して、人間とコミュニケーションをとるための適切な言語の使い方を学習させます。

しかし、この二つの学習方法のバランスをどのように取るかが課題となっています。教師あり学習に偏ると、エージェントは一般的な言語能力は獲得できるものの、特定のタスクに合わせた柔軟なコミュニケーションが難しくなります。一方、強化学習に偏ると、言語ドリフトが発生しやすくなります。

Seeded Iterated Learning (SIL)
Lu et al. (2020) 2 は、言語ドリフトに対抗するために、Seeded Iterated Learning (SIL) と呼ばれる手法を提案しています。SILは、人間の言語データで事前学習されたエージェントを初期状態とし、教師エージェントと生徒エージェントを交互に学習させることで、言語ドリフトを防ぎながらタスクの性能を向上させることを目指します。

言語進化とタスク性能の両立
人間の子どもが言語を学習する過程では、言語能力を向上させながら、同時に周りの世界を理解し、様々なタスクをこなせるようにもなります。同様に、AIエージェントも、言語を進化させながらタスクの性能を向上させる ことができれば、より人間らしいコミュニケーション能力を獲得できると考えられます。

Chaabouni et al. (2022) 3 は、タスクのスコアと中間表現の英語としての BLEUスコア を用いて、言語進化とタスク性能の両立を評価しています。これは、エージェントが学習した言語が、タスクを効率的に解決できるだけでなく、人間にとっても理解しやすいものであることを保証するための指標となります。

機械と人間のコミュニケーション： 今後の展望
AIと人間が自然にコミュニケーションをとれるようになることは、AI研究における究極の目標の一つと言えるでしょう。そのためには、言語ドリフトの問題を克服し、人間のように文脈を理解し、意図を読み取り、適切な表現を選択できるAIエージェントを開発することが不可欠です。

<!-- https://claude.ai/chat/f532a165-ad61-47ad-b628-bf1c4e7440d8 -->
<!-- Lu et al. (2020) https://claude.ai/chat/aacd8dae-b571-496f-add1-b262f88cf3a4 -->

### 数理

<!-- [1] “指示ゲームの生成モデル的な再解釈”. -->

## まとめ

<!-- TODO -->

[^Boldt_Mortensen_2024]: B. Boldt and D. R. Mortensen, “A Review of the Applications of Deep Learning-Based Emergent Communication,” Transactions on Machine Learning Research, Aug. 2023, Accessed: Sep. 24, 2024. [Online]. Available: <https://openreview.net/forum?id=jesKcQxQ7j>
[^Brandizzi_2023]: N. Brandizzi, “Toward More Human-Like AI Communication: A Review of Emergent Communication Research,” IEEE Access, vol. 11, pp. 142317–142340, 2023, doi: 10.1109/ACCESS.2023.3339656.
[^Brighton_Kirby_2006]: H. Brighton and S. Kirby, “Understanding Linguistic Evolution by Visualizing the Emergence of Topographic Mappings”.
[^Chaabouni_et_al_2020]: R. Chaabouni, E. Kharitonov, D. Bouchacourt, E. Dupoux, and M. Baroni, “Compositionality and Generalization In Emergent Languages,” in Proceedings of the 58th Annual Meeting of the Association for Computational Linguistics, D. Jurafsky, J. Chai, N. Schluter, and J. Tetreault, Eds., Online: Association for Computational Linguistics, Jul. 2020, pp. 4427–4442. doi: 10.18653/v1/2020.acl-main.407.
[^Chaabouni_et_al_2021]: R. Chaabouni et al., “Emergent Communication at Scale,” presented at the International Conference on Learning Representations, Oct. 2021. Accessed: Sep. 24, 2024. [Online]. Available: <https://openreview.net/forum?id=AUGBfDIV9rL>
[^Das_et_al_2024]: A. Das et al., “TarMAC: Targeted Multi-Agent Communication,” in Proceedings of the 36th International Conference on Machine Learning, PMLR, May 2019, pp. 1538–1546. Accessed: Sep. 26, 2024. [Online]. Available: <https://proceedings.mlr.press/v97/das19a.html>
[^Foerster_2016]: J. N. Foerster, Y. M. Assael, N. de Freitas, and S. Whiteson, “Learning to Communicate with Deep Multi-Agent Reinforcement Learning,” May 24, 2016, arXiv: arXiv:1605.06676. doi: 10.48550/arXiv.1605.06676.
[^Graesser_et_al_2020]: L. Graesser, K. Cho, and D. Kiela, “Emergent Linguistic Phenomena in Multi-Agent Communication Games,” Feb. 28, 2020, arXiv: arXiv:1901.08706. Accessed: Oct. 01, 2024. [Online]. Available: <http://arxiv.org/abs/1901.08706>
[^Jaques_et_al_2018]: N. Jaques et al., “Intrinsic Social Motivation via Causal Influence in Multi-Agent RL,” Sep. 2018, Accessed: Oct. 01, 2024. [Online]. Available: <https://openreview.net/forum?id=B1lG42C9Km>
[^Lazaridou_et_al_2017]: A. Lazaridou, A. Peysakhovich, and M. Baroni, “Multi-Agent Cooperation and the Emergence of (Natural) Language,” arXiv.org. Accessed: Sep. 11, 2024. [Online]. Available: <https://arxiv.org/abs/1612.07182v2>
[^Lazaridou_Baroni_2020]: A. Lazaridou and M. Baroni, “Emergent Multi-Agent Communication in the Deep Learning Era,” Jul. 14, 2020, arXiv: arXiv:2006.02419. Accessed: Sep. 19, 2024. [Online]. Available: <http://arxiv.org/abs/2006.02419>
[^Lowe_et_al_2019] R. Lowe, J. Foerster, Y.-L. Boureau, J. Pineau, and Y. Dauphin, “On the Pitfalls of Measuring Emergent Communication,” arXiv.org. Accessed: Oct. 01, 2024. [Online]. Available: <https://arxiv.org/abs/1903.05168v1>
[^Liang_et_al_2020]: P. P. Liang, J. Chen, R. Salakhutdinov, L.-P. Morency, and S. Kottur, “On Emergent Communication in Competitive Multi-Agent Teams,” Jul. 16, 2020, arXiv: arXiv:2003.01848. doi: 10.48550/arXiv.2003.01848.
[^Lu_et_al_2020]: Y. Lu, S. Singhal, F. Strub, A. Courville, and O. Pietquin, “Countering Language Drift with Seeded Iterated Learning,” in Proceedings of the 37th International Conference on Machine Learning, PMLR, Nov. 2020, pp. 6437–6447. Accessed: Sep. 26, 2024. [Online]. Available: https://proceedings.mlr.press/v119/lu20c.html
[^Yuan_et_al_2021]: L. Yuan, Z. Fu, J. Shen, L. Xu, J. Shen, and S.-C. Zhu, “Emergence of Pragmatics from Referential Game between Theory of Mind Agents,” Sep. 30, 2021, arXiv: arXiv:2001.07752. doi: 10.48550/arXiv.2001.07752.



[^Okanohara_2020]: Okanohara D., “《日経Robotics》AIトップ国際会議では何が起きているか,” 日経Robotics（日経ロボティクス）. Accessed: Sep. 24, 2024. [Online]. Available: <https://xtech.nikkei.com/atcl/nxt/mag/rob/18/00007/00022/>
[^Ueda_et_al_2023]: R. Ueda et al., “言語とコミュニケーションの創発に関する構成論的研究の展開,” Jun. 07, 2023, OSF. doi: 10.31234/osf.io/rz5ng.
