---
title: "創発コミュニケーションについての自分用まとめ"
emoji: "👨‍⚕️"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false

# レビュー観点
# 1. 一般的なソフトウェアエンジニアが知らない用語には、具体的な値を伴う例が示されていること
# 2. 文体が統一されているか
# 3. 引用方法が統一されているか
# 4. 複数行のコメントアウトをしていないこと（zenn.devが対応していない）
# 5. 可能なら漢語ではなく和語を用いること（例: ✗測定する ◯測る）
# 6. 長文は分割すること
---



## この記事は？

2人以上のプレイヤーが、道案内のようなゲームに協力して取り組むとき、プレイヤー間で考えていることを伝えるために新たな意味を持った記号が生まれます。こうしたコミュニケーションを創発コミュニケーションと呼びます。創発コミュニケーションに関するサーベイを読んでまとめました。

## TL;DR

<!-- TODO -->

## 動機

もともと、私はイラストを描くAIに興味がありました。Stable Diffusionのような生成AIは、画像を教師データから学んでいます。プロが描いたようなイラストを生成できる一方で、学習には元となる画像が必要です。

そこで、もし「複数のエージェントが、自ら絵を描いて道案内をする中で、絵の訓練をする」ようなタスクを通して、ゼロからイラストを学習させることが可能かが気になりました。

調べると、この分野は創発コミュニケーション (EmCom, Emergent Communication)と呼ばれているようです。主に記号や言語を対象として研究が進んでいるほか、イラストを描く研究もあるようです。そこで、どのような実験や課題があるかをまとめました。

## 対象の論文

主に Brandizzi (2023)[^Brandizzi_2023] と Lazaridou & Baroni (2020)[^Lazaridou_Baroni_2020] を読みました。Brandizzi (2023)を選んだのは、書かれたのが最近であること、私にとって構成が分かりやすかったことが理由です。また、Lazaridou & Baroni (2020) を選んだのは、この分野で先駆的な実験であるLazaridou et al. (2017)[^Lazaridou_et_al_2017]の著者によって書かれていることが理由です。

ほか、Boldt & Mortensen (2024)[^Boldt_Mortensen_2024]とUeda et al.(2024)[^Ueda_et_al_2023]を参照しました。前者は特に日本で盛んな「記号創発ロボティクス」をEmComの文脈で紹介していたことが、後者は構成性（compositionality, 複雑な意味が、その要素の意味＋組み合わせのルールから生まれるような性質）に関する研究を俯瞰的に紹介していたことが理由です。

その他に参照した論文については、都度紹介します。

### コミュニティ

創発コミュニケーションについての研究は、主に次の学会で発表されているようです。[^Boldt_Mortensen_2024][^Okanohara_2020]

- ICLR: 深層学習に特化しており、新規性を求める
  - EmeCom WorkShop: ICLRのワークショップ
- NeurIPS: 神経科学や人工知能もカバーしており、強化学習にも重きをおいている
- ICML: 機械学習のアルゴリズムや理論に重点を置いている

## 代表的な実験

<!-- Brandizzi (2023): II. COMMON PROPRIETIES, Boldt & Mortensen (2024): 1.2 Scope -->

初めに、創発コミュニケーションの代表的な実験を紹介します。次の特徴が共通しています。

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

<!-- https://ja.wikipedia.org/wiki/シグナリングゲーム, 本来は Lewis, D. (1969) を引用すべきだが、時間があるときにする。 -->

創発コミュニケーションの実験で用いられる、次のようなゲームを**シグナリングゲーム(signaling game)**と言います。

- 送信者と受信者からなる
- 送信者はタイプを持つ。受信者は送信者のタイプを知らない
- 送信者は受信者にメッセージを送る。受信者はメッセージを元に行動する

#### コミュニケーション自体に焦点を当てたゲーム

シグナリングゲームの中でも、受信者が送信者のタイプを当てるシンプルなゲームを**参照ゲーム(referential game)**と言います。なお参照ゲームをシグナリングゲームと呼ぶこともあります。

参照ゲームの代表的な例として、Lazaridou et al. (2017)[^Lazaridou_et_al_2017]の実験があります。これは参照ゲームの中でも**識別ゲーム(discrimination game)**と呼ばれることがあります。
<!-- Brandizzi (2023) は識別ゲーム以外の参照ゲームの例を挙げているが、あくまで参考程度なので省略する。 -->

実験では、送信者が2枚の画像から1枚を選び、受信者に1つの記号(10または100の語彙から選ばれた1語)を送信します。受信者にはシャッフルされた2枚の画像と送信された記号が提示されるので、どちらの画像を指しているかを当てるタスクを行います。
<!-- https://claude.ai/chat/eb5f6938-d36d-42f3-af66-0f22a493cca6 -->

#### コミュニケーションを補助として用いるゲーム

コミュニケーションを補助的な手段として用いることで、道案内や交渉などを成功させる種類のゲームがあります。参照ゲームではないシグナリングゲームの一種で、**指示ゲーム**と呼ばれることがあります。
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
<!-- Havrylov & Titov (2017) https://chatgpt.com/c/66f53b2c-3680-8010-84de-fcdcd5e7557f, Brandizzi (2023) https://claude.ai/chat/74c875b5-af8e-4bb9-a25a-6778dd92f7e4 -->

エージェント間の通信を微分可能にする試みとしては、他にDIAL（Differentiable Inter-Agent Learning, 微分可能エージェント間学習）[^Foerster_2016]があります。

人間の言語が離散的であるのに対して、DIALでは訓練時に連続的なベクトルを渡すことができます。大雑把に言えば、訓練時は「66.6%の確率で数字の8」というやり取りをするのに対して、実行時は「おそらく数字の8」という伝え方をするようなものです。
実験では、訓練時に連続的なベクトルを渡すことは学習を促進するものの、連続的なべクトルに適切なノイズを加えることが離散的な値への移行を促すことが報告されています。
<!-- https://claude.ai/chat/d7a065b3-faa9-4a2c-80eb-23c0e5fa12ff -->

さらに一歩進んで、創発コミュニケーションを1つの表現学習と見なした上で、シグナリングゲームを変分推論として捉えた研究もあります。[^Ueda_Taniguchi_2023]

エージェントの言語を人間の言語に似せることを目的として、教師あり学習による事前学習が用いられることもあります。

### エージェント間の関係

複数のエージェントが協力してタスクに取り組むことが、コミュニケーションの創発を促します。しかし、複数のエージェントからなるチームを競争させることで、より情報量が多く構成的な創発言語が発現する可能性があります。

Liang et al. (2020)[^Liang_et_al_2020]の研究によれば、複数チーム間での参照ゲームにおいて、他チームの創発言語を盗み聞きした場合、他チームのタスクやインスタンスを知らない場合であっても、自チームのゲームの正解率がより早く収束するようになったことが報告されています。
<!-- https://claude.ai/chat/1c000ba6-8b01-403f-b6e6-e8d1fec05366 -->

## 課題

言語・記号創発の分野では、深層ニューラルネットワークによるエージェント同士のコミュニケーションを研究します。中でも、次のような課題があります。
<!-- https://aistudio.google.com/app/prompts/1B07r2OCU-duu9Py64sqTPmFHNRKTQyd6 -->

1. 創発言語の評価と分析
   1. コミュニケーションの効果
   2. 一般化と構成性の評価
   3. 一般化と構成性の分析
2. 自然言語を組み込んだ創発コミュニケーション

### 創発言語の評価と分析

#### コミュニケーションの効果

<!-- Lazaridou & Baroni (2020): 3.1 Measuring the Degree of Effective Communication, Brandizzi (2023): III. DICHOTOMY OF EMERGENT COMMUNICATION -> A. MACHINE-CENTERED EMCOM -> 1) CHARACTERISTICS -->

創発コミュニケーションの研究において、エージェントが実際にコミュニケーションを取れているのか、その効果をどのように測るかは重要な課題です。

最もシンプルな指標としては、**タスク成功率** が挙げられます。例えば、画像を説明するゲームで、聞き手が正しく画像を識別できた割合を見る方法です。しかし、タスク成功率は、エージェントが意図したとおりにコミュニケーションを取れているかどうかを必ずしも反映しません。

例えば、エージェントがメッセージの内容ではなく、メッセージを送信するターン数などのメタ的な情報に基づいてタスクをクリアしてしまう可能性があります。

もしメタ的な情報に基づいてタスクをクリアしているなら、言語が意味を持たない可能性があります。そこで、**相互理解可能性**や**話者一貫性**を測ることが考えられます。

**相互理解可能性 (Mutual Intelligibility)**[^Graesser_et_al_2020]は、エージェントが自分自身とコミュニケーションができるかどうかを測る指標です。実験では単純な参照ゲームを行います。ただし、送信者と受信者のネットワークは、実際に送信・受信に関わる部分を除いて共通の設計になっています。これによって訓練した送信者自身を受信者として使えます。そのような場合でもタスクが成功するなら、意味を持った言語が出現していると言えそうです。
<!-- https://claude.ai/chat/100c90b3-d33a-454c-870b-a970c8912adc -->

**話者一貫性 (Speaker Consistency)**は、エージェントが特定の行動をとる際に、どの程度一貫して同じメッセージを発信するかを測定します。[^Jaques_et_al_2018]例えば、「右に移動」という行動をとる際に、常に "move right" というメッセージを発信するエージェントは、話し手の一貫性が高いと言えます。

また、メッセージと行動の関係性を直接測定することも考えられます。Casual Inference of Communication[^Lowe_et_al_2019]では、メッセージとその後の行動の相互情報量を求め、その値が高いほどメッセージが行動に影響を及ぼしていると考えます。
<!-- https://claude.ai/chat/6301eaa9-2b8b-49e5-8bc5-af6961d3a693 -->

#### 一般化と構成性の評価

創発言語がコミュニケーションを可能にしている場合でも、訓練データやタスクに対して過学習してしまっては、自然言語から離れてしまうと考えられます。創発言語を一般化するための指標がいくつか提案されています。

**ゼロショット性能 (Zero-Shot Performance)**は、訓練データに含まれていない未知の入力に対して、エージェントがどのように対応できるかを評価する指標です。創発コミュニケーションにおいては、エージェントが未知の状況にも対応できる汎用的な言語を学習しているかどうかを判断するのに役立ちます。ただし、適切な評価データセットを用意することが難しいと考えられます。

**転移学習 (ETL, Ease and transfer learning)** は、創発言語が、異なるタスクを実行する新しい受信者にどの程度速く、そしてうまく伝達されるかを捉える指標です。[^Chaabouni_et_al_2021]はじめに、何らかのタスクを用いて創発言語を求め、その言語を固定します。次に異なるタスクと新たな受信者を用意し、タスクの正解率が特定の閾値を超えるまでのステップ数や、逆に特定のステップ数時点でどれだけの正解率が出るかを測定します。個別のタスクに依存しない能力を測ると言えますが、自然言語ほどの汎用性を求められているとは言えないかもしれません。
<!-- https://claude.ai/chat/480c664a-6c5d-421c-82a3-f4678266dfe1 -->

ただし、創発言語が一般化されているからといって、構成性を持つとは言えません。実際に、構成性を持たない言語が訓練時に含まれなかった属性についてのコミュニケーションを高い精度で成功させている実験があります。[^Chaabouni_et_al_2020]

<!-- Lazaridou & Baroni (2020): 3.2 Compositionality, Brandizzi (2023): III. DICHOTOMY OF EMERGENT COMMUNICATION -> A. MACHINE-CENTERED EMCOM -> 2) HUNT FOR GENERALIZATION -->

**構成性 (Compositionality)** は、複雑な意味が、その構成要素の意味とそれらを組み合わせる規則によって決定されるという性質です。自然言語では、単語の意味と文法規則から文の意味を理解できることが構成性の例として挙げられます。

創発言語の構成性を評価する方法の多くは、その分布に注目しています。例えば、自然言語において単語の出現頻度とその順位の間に反比例の関係がある、という**ジップの法則 (Zipf's law)**への適合度を測る方法があります。ジップの法則では、出現頻度が$k$番目の単語の頻度は、1位の頻度の$1/k$倍になるとされています。

言語だけでなく、意味の分布と似ている度合いを評価する方法として**トポグラフィック類似性 (TopSim, Topographic Similarity)**[^Brighton_Kirby_2006]が挙げられます。例えば、意味空間では「犬」と「猫」、「自動車」と「飛行機」は他の意味に比べて近くにあるはずです。このとき、創発言語が構成的なら、単語同士の距離も近くなるはずです。

このように、トポグラフィック類似性は意味と言語の似ている度合いを捉えることはできますが、具体的な組み合わせの規則（文法）を測ることはできません。
<!-- https://claude.ai/chat/a41ea4da-52c3-404a-b366-08a09a23502f -->

創発言語の構成を評価するための方法としては、**positional disentanglement (posdis)**と**bag-of-symbols disentanglement (bosdis)**[^Chaabouni_et_al_2020]が挙げられます。posdisは、特定の位置にある記号が特定の属性の値に一致する度合いを測ります。例えば、送信者のメッセージが「AX」や「BY」であり、Aが赤、Bが青を示している場合、1文字目であることが色を表している可能性が高いです。逆にbosdisは、位置関係にかかわらず、ある文字とある属性が同時に出現する度合いを見ます。

研究では、構成性を持った言語は新たなエージェントにとっても学習しやすい一方で、posdisやbosdisの理論値までは幅がある、つまり創発言語の構成性にはかなり改善の余地が残されていることが示唆されています。
<!-- https://claude.ai/chat/7e9cf0ab-9640-44ba-b409-4e3f73195701 -->

#### 一般化と構成性の分析

構成性の評価だけでなく、どのように構成性を発現させるかも課題となっています。創発コミュニケーションの実験の多くはシンプルな設定です。そのため、より複雑で大規模な設定が構成性を引き上げそうだと期待してしまいますが、必ずしもそうでないことが報告されています。

Chaabouni et al. (2021)[^Chaabouni_et_al_2021]は、データセットの規模、タスクの複雑さ、エージェントの数を引き上げた場合の一般化と構成性の度合いを調べました。データセットの規模やタスクの複雑さは一般化を促す一方で、トポグラフィック類似性は連動しなかったため、構成性の測り方を見直す必要が示唆されています。また、エージェントの数を増やす場合は、単に送信者と受信者を増やして訓練ステップごとにシャッフルするのではなく、複数の受信者の投票によって行動を決めたり、送信者を教師として他の送信者に模倣させるなどして、エージェント間の相互作用を明示的に設計することが一般化・構成性に繋がるそうです。
<!-- https://claude.ai/chat/26e4ac40-3c84-403d-8e65-f165eeba49dc -->

### 自然言語を組み込んだ創発コミュニケーション

<!-- Lazaridou & Baroni (2020): 4.3 Machines Cooperating with Humans, Brandizzi (2023): III. DICHOTOMY OF EMERGENT COMMUNICATION -> B. HUMAN-CENTERED EmCom -->

機械どうしのコミュニケーションでは、エージェントが独自の言語を創発させていました。一方で、人間とコミュニケーションを取るためには、エージェントが **人間の自然言語 (HNL)** を理解し、生成できるようになることが重要です。これは、AIが人間にとってより身近で、使いやすい存在になるために欠かせない要素です。

自然言語を組み込んだ創発コミュニケーションは、従来の画像キャプション生成タスクと似ていますが、学習方法が大きく異なります。

* **画像キャプション生成:**  大量の画像とキャプションのペアを用いた教師あり学習によって、画像の内容を説明する自然言語文を生成するモデルを学習します。
* **自然言語を組み込んだ創発コミュニケーション:**  エージェントは、強化学習を通して、特定のタスクを達成するために必要な自然言語を、人間とのやり取りを通して学習していきます。

#### 言語ドリフト：自然言語からのズレ

自然言語を組み込んだ創発コミュニケーションにおいても、**言語ドリフト**は大きな課題です。エージェントがタスクを効率的に解決することに集中するあまり、学習した言語が人間の自然言語から徐々に乖離してしまう現象です。

言語ドリフトは、AIエージェントが人間の言語の複雑さやニュアンスを完全に理解できていないために起こります。例えば、エージェントは、文法的に正しくない文章や、人間には理解できない単語の組み合わせを生成してしまうことがあります。

#### 言語ドリフトの種類と抑制

Brandizzi (2023) [^Brandizzi_2023] は、言語ドリフトを以下の3つのタイプに分類しています。

* **構造的ドリフト (Structural Drift):** 文法的な構造が単純化され、人間にとって不自然な文になる。例えば、「猫ですか？」が「猫？」になるなど。
* **意味的ドリフト (Semantic Drift):** 単語の意味が本来の意味から変化してしまう。例えば、「古い教え」が「老人」という意味になるなど。
* **機能的ドリフト (Functional Drift):**  言語が意図した行動を引き起こさなくなる。例えば、交渉で合意したにもかかわらず、エージェントが別の取引を行ってしまうなど。

言語ドリフトを抑制するために、以下のようないくつかの手法が提案されています。

* **教師あり学習による事前学習:** 大規模なテキストデータを用いて、エージェントに自然言語の文法や語彙に関する知識を事前に学習させます。
* **反復学習 (Iterated Learning):**  エージェントの世代交代をシミュレートすることで、言語の安定性を高め、ドリフトを抑制します。
* **言語モデルの正則化:**  エージェントが生成する言語が、事前学習された言語モデルの出力と大きく乖離しないように、正則化項を導入します。

#### 言語ドリフト：進化の可能性

一方で、言語ドリフトは必ずしも負の側面だけを持つわけではありません。言語ドリフトは、エージェントが新しい表現や概念を発明する **言語進化** のプロセスと捉えることもできます。

例えば、エージェントがタスクをより効率的に解決するために、新しい単語や文法規則を創発させることがあるかもしれません。これは、人間の言語進化の歴史においても、新しい単語や表現が生まれてきた過程と似ています。

#### 自然言語を組み込んだ創発コミュニケーション： 今後の展望

自己教師あり学習によって訓練された大規模言語モデルが、近年目覚ましい成果を上げています。この流れを踏まえると、自然言語を組み込んだ創発コミュニケーションの研究は、単に人間とコミュニケーションできるAIを作るという目標だけでなく、**言語進化のメカニズムを解明するためのツール** として、より重要な役割を担っていく可能性があります。

AIエージェントが創発する言語を分析することで、私たちは人間の言語の起源や進化、そして言語が思考に与える影響について、より深い理解を得ることができるかもしれません。

<!-- https://claude.ai/chat/f532a165-ad61-47ad-b628-bf1c4e7440d8 -->
<!-- Lu et al. (2020) https://claude.ai/chat/aacd8dae-b571-496f-add1-b262f88cf3a4 -->

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
[^Okanohara_2020]: Okanohara D., “《日経Robotics》AIトップ国際会議では何が起きているか,” 日経Robotics（日経ロボティクス）. Accessed: Sep. 24, 2024. [Online]. Available: <https://xtech.nikkei.com/atcl/nxt/mag/rob/18/00007/00022/>
[^Yuan_et_al_2021]: L. Yuan, Z. Fu, J. Shen, L. Xu, J. Shen, and S.-C. Zhu, “Emergence of Pragmatics from Referential Game between Theory of Mind Agents,” Sep. 30, 2021, arXiv: arXiv:2001.07752. doi: 10.48550/arXiv.2001.07752.
[^Ueda_Taniguchi_2023]: R. Ueda and T. Taniguchi, “Lewis’s Signaling Game as beta-VAE For Natural Word Lengths and Segments,” presented at the The Twelfth International Conference on Learning Representations, Oct. 2023. Accessed: Oct. 02, 2024. [Online]. Available: https://openreview.net/forum?id=HC0msxE3sf
[^Ueda_et_al_2023]: R. Ueda et al., “言語とコミュニケーションの創発に関する構成論的研究の展開,” Jun. 07, 2023, OSF. doi: 10.31234/osf.io/rz5ng.

<!-- 機械どうしのコミュニケーションについては、既に取り上げた内容と被るので省略する。 -->
<!-- Lazaridou & Baroni (2020): 4.1 Communication Facilitating Inter-Agent Coordination, 4.2 Beyond Cooperation: Self-interested and Competing Agents -->
<!-- Brandizzi (2023): III. DICHOTOMY OF EMERGENT COMMUNICATION -> A. MACHINE-CENTERED EMCOM -->
