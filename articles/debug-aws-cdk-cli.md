---
title: "AWS CDK CLIをデバッグする"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","aws-cli","CDK"]
published: true
---
GitHubからCloneした[AWS CDK CLI](https://github.com/aws/aws-cdk)をローカルでデバッグする手順をまとめました。
もっと効率いい方法があったら教えて下さい！

# Overview
1. `tsc`, `tslint` をインストール
2. `pkglint`, `cdk-build-tools`, `awslint`, `cdk`をビルド
3. jsファイルにdebuggerを仕込む
4. ビルドしたCDK CLIで任意のCDKアプリケーションを実行

toolsとCDK CLI本体の依存関係はこんな感じです。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/aa2e8e95-0f94-30d9-ca8c-f9123b022950.png)

# 1. `tsc`, `tslint` をインストール

```
npm install -g tsc
npm install -g tslint
```

# 2. `pkglint`, `cdk-build-tools`, `awslint`, `cdk`をビルド

※パスは適宜読み替えてください。

```
cd aws-cdk/tools/pkglint
npm run build
```

```
cd aws-cdk/tools/cdk-build-tools
PATH=$PATH:~/.ghq/github.com/aws/aws-cdk/tools/pkglint/bin
npm run build
```

```
cd ~/.ghq/github.com/aws/aws-cdk/tools/awslint
npm run build
```

```
cd aws-cdk/packages/aws-cdk
PATH=$PATH:~/.ghq/github.com/aws/aws-cdk/tools/cdk-build-tools/bin
PATH=$PATH:~/.ghq/github.com/aws/aws-cdk/tools/awslint/bin
npm run build
```

# 3. jsファイルにdebuggerを仕込む

nodeアプリケーションのデバッグ方法は以下を参照してください。
https://nodejs.org/api/debugger.html

```
vi aws-cdk/packages/aws-cdk/bin/cdk.js
```

結果として、こんな感じになります。

```cdk.js
    async function cliSynthesize(stackNames, exclusively) {
        // 省略......
        debugger; // debuggerを仕込む。
        appStacks.processMetadata(stacks);
```


# 4. ビルドしたCDK CLIで任意のCDKアプリケーションを実行

```
cd ${APPLICATION_DIR}/cdk
node inspect ~/.ghq/github.com/aws/aws-cdk/packages/aws-cdk/bin/cdk synth -v
```

以上です。

