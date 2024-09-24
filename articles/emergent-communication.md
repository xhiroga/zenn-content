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

## 論文を探した方法

次の2つのサーベイ論文を入口として引用の追跡を行いました。

- [A. Lazaridou and M. Baroni, “Emergent Multi-Agent Communication in the Deep Learning Era,”](http://arxiv.org/abs/2006.02419)
- [R. Ueda et al., “言語とコミュニケーションの創発に関する構成論的研究の展開,”](https://osf.io/preprints/psyarxiv/rz5ng)

## 言語・記号創発の課題

言語・記号創発の分野では、深層ニューラルネットワークによるエージェント同士のコミュニケーションを研究します。中でも、次のような課題があります。

1. 創発言語の分析と評価
   1. 創発プロトコルの一般的特徴の理解
   2. 構成性をどのように発現させるか・評価するか
   3. 人間の言語との類似性をどのように分析・評価するか
2. 創発言語の応用
   1. 機械-機械間コミュニケーションの改善
   2. 機械-人間間コミュニケーションの改善
   3. LLMと創発言語の組み合わせ

## 代表的な実験

言語・記号創発の実験は強化学習の枠組みで捉えられます。実験は次のように分類できます。

- コミュニケーションの形式
  - 連続ベクトルを用いたコミュニケーション: DIALなど
  - 離散的シンボルを用いたコミュニケーション: RIALなど
- メッセージの長さ
  - 固定長: Lazaridou et al. (2017) など
  - 可変長: Havrylov & Titov など
  - ほか、単一か複数かといった切り口もある
- ゲームの種類
  - シグナリングゲーム / 参照ゲーム
  - 指示ゲーム
  - 交渉ゲーム
- ターン
  - 単一ターン
  - マルチターン: Evtimova...
- 環境と入力
  - 抽象的環境
  - 2D視覚環境・画像
  - 3D環境・マルチモーダル入力: Das et al (2019)など
- エージェント間の関係
  - 完全協力
  - 部分的協力/競争
- エージェント数
  - 2エージェント
  - 多エージェント

続けて、代表的な実験をいくつか取り上げます。

### 連続ベクトルを用いたコミュニケーション

連続ベクトルを用いたコミュニケーションの例としては、同じくFoerster et al. (2016)のDIAL（Differentiable Inter-Agent Learning）システムがあります[8]。DIALでは、エージェント間の通信チャネルが連続的であり、システム全体を通じて勾配を逆伝播することができます。これにより、エージェントは互いの内部状態にアクセスでき、より効率的な学習が可能になります。

### 離散的シンボルを用いたコミュニケーション

離散的シンボルを用いたコミュニケーションの代表的な例としては、Foerster et al. (2016)のRIAL（Reinforced Inter-Agent Learning）システムが挙げられます[8]。RIALでは、エージェントは離散的なシンボルを用いてコミュニケーションを行い、タスク報酬のみを用いて学習します。この方法では、エージェント間でエラー情報を連続的に伝播することはできませんが、自然言語のような離散的な構造を模倣することができます。

### シグナリングゲーム / 参照ゲーム

シグナリングゲームとは、送信者が受信者に対して、複数の選択肢の中から特定のターゲットを伝えるゲームです。シグナリングゲームと参照ゲームは密接に関連しており、しばしば同じ文脈で使用されます。シグナリングゲームは、送信者と受信者の間の一般的な情報伝達を扱う広義の概念です。一方、参照ゲームは特定の対象物（参照物）についてコミュニケーションを行うシグナリングゲームの一種と考えることができます。

典型的な例としては、Lazaridou et al. (2017)の実験が挙げられます[1]。この実験では、送信者は2つの画像を見て、そのうちの1つをターゲットとして選び、1つのシンボルを発信します。受信者は、そのシンボルと2つの画像（ランダムな順序で提示される）を見て、ターゲットの画像を当てるというタスクを行います。

### 指示ゲーム

指示ゲームもシグナリングゲームの一種です。シグナリングゲームが主に情報の伝達に焦点を当てているのに対し、指示ゲームではその情報に基づいて具体的な行動を取ることが求められます。Das et al. (2019)の研究[2]では、言語指示に基づいてナビゲーションを行うタスクを通じて、より複雑な状況での言語の使用と理解を調査しています。

指示ゲームとは、一方のエージェントが他方のエージェントに特定のタスクや行動を指示するゲームです。例えば、Das et al. (2019)の研究では、3D環境内でエージェントが協力してナビゲーションタスクを解決する実験を行っています[2]。この実験では、一方のエージェントが環境を観察し、もう一方のエージェントに移動の指示を与えます。

<!-- TODO: 指示ゲームになったことで、より複雑だったり構成性を持った言語が発現したのか？ -->

### 交渉ゲーム

交渉ゲームとは、エージェント間で限られたリソースの分配を交渉するゲームです。Cao et al. (2018)の研究では、2つのエージェントが複数のオブジェクトの価値について交渉を行う実験を実施しています[3]。各エージェントには各オブジェクトの隠された価値が割り当てられ、エージェントはメッセージを交換しながら、オブジェクトの分割方法を提案し合います。

### 3D環境・マルチモーダル入力

3D環境を用いた実験では、エージェントはより現実的な状況でコミュニケーションを行います。Das et al. (2019)の研究では、エージェントは3D家屋環境内を移動しながら、視覚情報と言語情報を組み合わせてタスクを解決します[2]。このような環境では、エージェントは空間的な関係や物体の属性など、より豊富な情報を扱う必要があります。

<!-- TODO: 実論文を読んでブラッシュアップ -->

## 創発言語の分析と評価に関する取り組みと課題

創発言語の分析と評価について、主な取り組みと課題は以下の通り。

1. コミュニケーションの効果
2. トポグラフィック類似性 & ダイエンタングルメント & 言語の圧縮性
3. 人間の言語との比較

### コミュニケーションの効果

送信者と受信者それぞれの分析手段がある。

実験の形式に注意しないと、メッセージではなくターン数などのメタ的な要素でコミュニケーションを取る可能性がある。

### トポグラフィック類似性 & ダイエンタングルメント & 言語の圧縮性

Brighton & Kirby (2006)によって導入された指標で、意味空間と形式空間の間の構造的類似性を測定します[4]。この指標は言語の構成性を間接的に評価しますが、具体的な構成過程を明らかにすることはできません。

ダイエンタングルメントは、表現学習において、データの生成過程における独立した要因を分離して表現することを目指す概念です。Andreas (2019)[6]は、この概念を創発言語の分析に適用しました。具体的には、言語の異なる側面（例えば、文法的機能と意味内容）を独立した表現として分離することを試みています。これにより、創発言語のどの部分がどのような機能を担っているかをより詳細に分析することが可能になります。
なので予め文法が分かっていないと性質が分からない。

Chaabouni et al. (2020)は、創発言語の構成性を評価するために、表現の圧縮性を利用する方法を提案しています[5]。構成的な言語はより効率的に圧縮できるという考えに基づいています。

課題: どのような建築的バイアスや環境的圧力が構成性の出現を促すかはかなり大雑把な理解。世代を経たり、話者数が多いと構成的になる傾向があるらしい。

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

## まとめ

<!-- TODO -->

## 参考文献

[1] A. Lazaridou, A. Peysakhovich, and M. Baroni, "Multi-Agent Cooperation and the Emergence of (Natural) Language," in International Conference on Learning Representations, 2017.
[2] A. Das et al., "TarMAC: Targeted Multi-Agent Communication," in Proceedings of ICML, pp. 1538-1546, 2019.
[3] K. Cao et al., "Emergent Communication through Negotiation," in Proceedings of ICLR Conference Track, 2018.
[4] H. Brighton and S. Kirby, "Understanding Linguistic Evolution by Visualizing the Emergence of Topographic Mappings," Artificial Life, vol. 12, no. 2, pp. 229-242, 2006.
[5] R. Chaabouni et al., "Compositionality and Generalization In Emergent Languages," in Proceedings of the 58th Annual Meeting of the Association for Computational Linguistics, pp. 4427-4442, 2020.
[6] J. Andreas, "Measuring Compositionality in Representation Learning," in International Conference on Learning Representations, 2019.
[7] A. Lazaridou, A. Potapenko, and O. Tieleman, "Multi-agent Communication Meets Natural Language: Synergies Between Functional and Structural Language Learning," in Association of Computational Linguistics, 2020.
[8] J. Foerster et al., "Learning to Communicate with Deep Multi-Agent Reinforcement Learning," in Advances in Neural Information Processing Systems, pp. 2137-2145, 2016.
[9] J. Lee, K. Cho, and D. Kiela, "Countering Language Drift via Visual Grounding," in Proceedings of the 2019 Conference on Empirical Methods in Natural Language Processing and the 9th International Joint Conference on Natural Language Processing (EMNLP-IJCNLP), pp. 4376-4386, 2019.
[10] Y. Lu et al., "Countering Language Drift with Seeded Iterated Learning," in International Conference on Machine Learning, 2020.
