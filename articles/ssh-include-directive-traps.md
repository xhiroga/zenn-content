---
title: "あなたの知らないSSH Client Includeディレクティブの落とし穴"
emoji: "🐘"
type: "tech"
topics: ["ssh", "linux"]
published: false
---

## はじめに

SSHの設定を分割管理できる**Include** ディレクティブをご存じでしょうか。  
これは、`~/.ssh/config` を複数のファイルに分割して書ける機能です。  
SSHクライアントでは、OpenSSH 7.3 (2016年8月リリース)[^openssh_7.3] 以降でサポートされています。
[^openssh_7.3]: <https://www.openssh.com/txt/release-7.3>

このIncludeは便利な半面、ちょっとした落とし穴もあります。  
筆者は今回、[SSHクライアントのターミナル『Tabby』](https://github.com/Eugeny/tabby)に[このInclude機能を対応させる](https://github.com/Eugeny/tabby/pull/10105/files)過程で、内部ロジックを調べて痛感しました。

そこで、**「SSHのIncludeディレクティブにはどんなメリットがあるか」**、そして**「どこに注意すると良いか」**をまとめます。

## TL;DR

- **Include** を使うと、SSHクライアント設定を複数ファイルに分割できます。  
- たとえば `~/.ssh/config.d/` のようなフォルダを丸ごと読み込むので、サーバ追加・削除のたびにメインの `config` を修正しなくても済みます。  
- ただし、**Host/Matchブロック内でもIncludeが書ける**、**相対パスの扱い**、**同じキーの再定義時の上書き順** など、仕様がややこしい面があります。

## そもそもIncludeディレクティブとは？

SSHの設定が増えてくると、`~/.ssh/config` が巨大になりがちです。  
自宅サーバや共有サーバ、テスト環境などをまとめて管理しようとすると、さらに大変です。

```shell
$ cat ~/.ssh/config
Host home-server
  HostName 192.168.1.10
  User me

Host staging-server
  HostName 192.168.2.15
  User me
  ...
# (この先100行以上ある)
```

これを分割しようとしたときに役立つのが、**Includeディレクティブ** です。  
次のように書けば `~/.ssh/config.d/*.conf` にある設定ファイルをすべて読み込んでくれます。

```conf
# ~/.ssh/config
Include ~/.ssh/config.d/*.conf
```

### 嬉しいポイント

1. **設定の分割管理**  
   自宅用 / 共有サーバ用 / 社内サーバ用など、ファイルごとに分割可能です。  
2. **glob が使える**  
   新しいサーバ用の設定ファイルを `~/.ssh/config.d/` にポンと置くだけで完了します。

## 意外と多い罠

### (1) パスの扱い

原則、**絶対パス** で書くと素直に解決されます。  
相対パスの場合、`~/.ssh/` を基準にして展開されると考えてください。  
ただし、OS や一部のOpenSSH実装では細かい差異があるため、トラブルを避けるなら絶対パスが無難です。

### (2) Host や Match ブロック内でもIncludeが書けてしまう

```conf
# ~/.ssh/config
Match User boss
  Include ~/.ssh/config.d/boss.conf
```

このようにブロック内に書くと、「合致した場合にさらに別ファイルを読み込む」などの複雑な挙動が可能になります。  
しかし可読性を損なうため、あまり推奨されていません。

### (3) いきなりHost設定を継ぎ足せる

ssh_configでは、分割ファイルの読み込み順も含めて最終的に1つの設定に統合されるのがポイントです。

下記の例では、**誤って** `HostName` を別ファイルにだけ書いてしまっても、最後に見つかった `HostName` が適用されます。

```conf
# ~/.ssh/config.d/config1
Host example.com
  User user
```

```conf
# ~/.ssh/config.d/config2
HostName 5.6.7.8
```

:::details 実行結果

```shell
$ ssh -G example.com | grep hostname
hostname 5.6.7.8
```

:::

結果として `example.com` が `5.6.7.8` に向かうため、意図しない接続先になってしまうことも。  
複数ファイルに分割するときは、重複定義に注意しましょう。

## まとめ

SSHのIncludeディレクティブは、設定を分割して管理するうえで非常に便利です。  
しかし、**相対パスの解釈** や **Hostブロック内におけるInclude**、**キー上書きの優先順位** といった点で混乱しやすいです。

- **大規模な環境** では、分割で管理しやすくなる反面、意図しない上書きが起きないように設計しましょう。
- **自宅や個人の環境** でも、サーバが増えてきたらIncludeを活用すると快適になります。

筆者はTabby（Electronベースのターミナル）に対してInclude機能を実装する過程で、こうした挙動を把握し、苦労しました。  
みなさんのSSH運用でも、一度Includeを試してみてください。
