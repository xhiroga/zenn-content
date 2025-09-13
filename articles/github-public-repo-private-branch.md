---
title: "GitHubで公開リポジトリにプライベートなブランチを生やす"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GitHub", "git"]
published: true
---

## TL;DR

次のとおりです。個人的に「二重リポ戦略」と呼んでいます。

1. 同じ内容のリポジトリを2つ作成し、片方をプライベートにする
2. `private/main`ブランチを作成し、プライベート側のリポジトリにのみPushする
3. 誤って`private/main`ブランチを公開リポジトリ/ブランチにPush/Mergeしないよう、commit hookを設定する。

## Motivation

最近、機械学習を利用した個人開発サービス「[Turntable](https://turntable.sawara.dev/)」をリリースしました。

TurntableではBlender拡張もあるのですが、そちらはオープンソースで公開しています。

https://github.com/xhiroga/blender-mcp-senpai

一方で、キャラクターを360度回転させるためのモデル（具体的にはFramePackのLoRA）は非公開です。どのようなビジネスの機会があるか分からないので、そうした判断をしています。

しかし、開発自体はモノリポの方が捗るのも事実です。そこで、前述の「**二重リポ戦略**」を用いて、**一部のコードをOSSにしつつ、モデルや学習データ・推論コードをクローズド**に保っています。

それ以外にも、例えば「フロントエンドはOSSにしたいが、バックエンドは非公開にしたい」のような場合でも役に立つと思います。　

## やり方

まずリポジトリを2つ、公開と非公開で新規作成します。私の場合、非公開の方は頭に`private`を付けています。

次に、ローカルのリポジトリに`private/main`ブランチを作成します。この時単に`private`ブランチを作成すると、あとあと`private/hogehoge`が作成できなくなるので注意です。

それが完了したら、`private/main`のremoteを`private`という名前で非公開のリポジトリに設定します。また、誤って公開リポジトリ側にPush / 公開ブランチ側にMergeしないように、commit hookを設定します。

## Commit Hook

まず、`.githooks`を作成します。[私のリポジトリ](https://github.com/xhiroga/blender-mcp-senpai/tree/main/.githooks)からコピーするのが手っ取り早いかもです。
（コードはClaudeで書きました）

なお、`~/.git-template/hooks/` を活用する方法もあります。非公開リポジトリの存在さえ明かせない場合は、そちらを検討してもいいかもしれません。私の場合は機械学習モデルの訓練のためにGPUサーバーにリポジトリを`clone`する機会が多いため、`.githooks`を使っています。

```console
% chmod +x .githooks/*

% git config core.hooksPath .githooks

% git config core.hooksPath
.githooks
```

### 動作確認

`pre-push`は次のように動作します。

```console
% git push                  
Error: Attempting to push private branch 'private/test' to remote 'origin'
Private branches can only be pushed to the 'private' remote
Use: git push private private/test
error: failed to push some refs to 'ssh://github.com/xhiroga/blender-mcp-senpai'
```

`post-merge`は次のように動作します。なお、**Mergeを防ぐ効果はないのでご注意ください（警告のみ）**

```console
% git merge private/test
Merge made by the 'ort' strategy.
 TEST.md | 0
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 TEST.md

⚠️  WARNING: You just merged a private branch into public branch 'feat/hooks'
This may expose private code to the public repository!

To undo this merge, run:
  git reset --hard HEAD^
```

## 別の環境にCloneする場合

まず公開リポジトリをCloneし、次に非公開リポジトリから`private/`ブランチをPullするのがおすすめです。

```console
% git clone git@github.com:owner/my-repo.git
% cd my-repo

# 非公開リポジトリを remote として追加
% git remote add private git@github.com:owner/private-my-repo.git

# 非公開リポジトリの private/* ブランチを取得
% git fetch private

# 例: private/main をローカルに作成して追跡
% git switch -c private/main --track private/private/main
```


## 注意事項

Commit Hookで予防を試みているものの、開発効率 > 公開リスク である場合（研究・小規模サービス）のみ適用するようにしてください。

企業で開発しているプロダクトや、個人情報・課金情報を扱うようなプロダクトでは、素直に別リポジトリに分けるのが良いと思います。

## まとめ

モノリポの一部機能を隠したまま、OSSにすることができます。皆さんもぜひOSS活動をしてください！
