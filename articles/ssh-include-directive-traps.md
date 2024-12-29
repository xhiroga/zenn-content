---
title: "あなたの知らないSSHのIncludeディレクティブの落とし穴"
emoji: "🐘"
type: "tech"
topics: ["ssh", "linux"]
published: true
---

## はじめに

SSHの設定を分割管理できる**Include** ディレクティブをご存じでしょうか。  
これは、`~/.ssh/config` を複数のファイルに分割して書ける機能です。  
SSHクライアントでは、OpenSSH 7.3 (2016年8月リリース)[^openssh_7.3] 以降でサポートされています。
[^openssh_7.3]: <https://www.openssh.com/txt/release-7.3>

このIncludeは便利な半面、ちょっとした落とし穴もあります。  
筆者は今回、[SSHクライアントのターミナル『Tabby』](https://github.com/Eugeny/tabby)を[Include機能に対応させる](https://github.com/Eugeny/tabby/pull/10105/files)過程で、内部ロジックを調べて痛感しました。

そこで、ハマらないために予め知っておきたい落とし穴をまとめました。

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

## 落とし穴

### (1) パスの扱い

Includeディレクティブは相対パスで書くことができます。では、二重にIncludeする場合の基準はどこになるでしょうか？

```conf
# ~/.ssh/config.d/dev.conf
Include temp.conf
```

正解は、「`~/.ssh`または`/etc/ssh`が基準になる」です。Includeを宣言したファイルからの相対パスではないんですね。

`man ssh_config`にも次のように記載してあります。

> Files without absolute paths are assumed to be in ~/.ssh if included in a user configuration file or /etc/ssh if included from the system configuration file.

IncludeをJavaScriptのImportのように考えているとハマりそうです。  
Includeは、あくまで巨大なssh_configファイルを作るための機能ということを意識しておくと良いかもしれません。

### (2) Host や Match ブロック内でもIncludeが書けてしまう

```conf
# ~/.ssh/config
Host example.com
  User user

Include ~/.ssh/config.d/stg.conf

# ~/.ssh/config.d/stg.conf
HostName 1.2.3.4

Host test.com
  User user
  ...
```

`~/.ssh/config.d/stg.conf`が`HostName`から始まっていますね。なんとこれは有効なssh_configです。  
実行すると次のようになります。

```conf
% ssh -G example.com | grep hostname
hostName 1.2.3.4
```

ハマらないためには、Includeはファイルの先頭に書き、文中のIncludeや二重のIncludeを避けるのが良さそうです。

### (3) いきなりHost設定を継ぎ足せる

(2) と似ていますが、実は呼び出す順番さえ連続していればHost設定を途中から書くことができます。

```conf
# ~/.ssh/config
Include config.d/config1
Include config.d/config2

# ~/.ssh/config.d/config1
Host example.com
  User user

# ~/.ssh/config.d/config2
HostName 1.2.3.4
```

```shell
$ ssh -G example.com | grep hostname
hostname 1.2.3.4
```

結果として `example.com` が `1.2.3.4` に向かうため、意図しない接続先になってしまうことも。  
トラブルを避けるため、被Include対象の`ssh_config`はHostブロックから書き始めることを意識したいですね。

## まとめ

SSHのIncludeディレクティブは、設定を分割して管理するうえで非常に便利です。  
しかし、あくまで巨大なファイルを分割するだけの機能ということを忘れると、ハマる原因になります。本記事が皆さんをトラブルから遠ざけてくれますように。
