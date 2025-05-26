---
title: "macOSからWSL2にRemote SSHして開発する備忘録"
emoji: "🐧"
type: "tech"
topics: ["wsl", "windows", "ubuntu", "vscode", "machinelearning"]
published: true
---

## TL;DR

- macOSからWindows機のGPUを活用して機械学習開発をするためのWSL2環境構築方法
- WSL2の機能アップデートとTailscale、VSCodeのRemote SSHの活用により凝った設定が不要に

## 動機

ゲーム用途で購入したWindows PCに搭載されている高性能GPUを、機械学習の開発にも活用したいと考えました。

普段macOSを使っている開発者が、Windows上のWSL2環境を活用して効率的に開発を行うための環境構築とTIPSをまとめています。

## すること

環境は次の通りです。

1. macOSがSSHクライアントとなる
2. Windowsホストマシンの上のWSL2 UbuntuがSSHサーバーとなる
3. WSL2の`networkingMode=mirrored`を有効化することで、Windowsホストの22番ポート（またはそれ以外）をWSL2に向けられる
4. Tailscaleを用いることで、LAN内外を問わずホスト名でWindowsホストに接続可能となる

## やり方

### 共通設定のポイント

- Tailscaleを使ってネットワーク接続します
  - WSL側にTailscaleをインストールする必要はないです
  - Windowsホスト側のみにインストールすることで、WSLからも利用できます
  - ただ、WSL2をGUIで利用した場合などはちょっと不明です。SSHしている限りは困っていません。
- VSCode Remote SSHで接続して開発します
- Docker for Windows と Docker for Ubuntu があります。おそらく後者の方が安定します。
  - 筆者はGUIが欲しかったので前者で利用していますが、WSLにボリュームをマウントしてDockerから参照する場合に、マウント後にDocker自体を再起動する必要があるなど、やや不便です。
- Bonjourを有効にすると `.local` でアクセスできますが、Tailscaleに任せた方が安定します

### Windows側の設定

Windows側では次の設定を行います：

- Remote Desktopの接続を許可します
  - WSLを再起動することがあるため、リモートからWindowsに接続できると便利です
  - [リモート デスクトップの使い方](https://support.microsoft.com/ja-jp/windows/5fe128d5-8fb1-7a23-3b8a-41e636865e8c)
- Windowsホスト上でもOpenssh Serverを有効化するとより安心です
- `.wslconfig` ファイルを次のように設定します

```shell
# https://github.com/xhiroga/homelab/blob/98979a39f3ecdadea1191ce3b469a8401e775efc/playbooks/files/windows/.wslconfig
[wsl2]

networkingMode=mirrored

[experimental]

hostAddressLoopback=true
```

`networkingMode=mirrored` は、WSLで立てたサーバーにWindowsのブラウザからアクセスできるようにし、Tailscaleを共有するための設定です。Windows 11 22H2でexperimentalから昇格した機能と思われます。これによりポート番号が共有され、WSL側にTailscaleをインストールしなくて済みます。

### WSL側の設定

繰り返しですが、WSL側にはTailscaleをインストールしません。以下はTailscaleの公式ドキュメントからの引用です

:::details Tailscaleの公式ドキュメントからの引用

> If you run Tailscale on both the Windows host and inside WSL 2 at the same time, Tailscale encrypted traffic that flows from WSL 2 over Tailscale on the Windows host will not work due to Tailscale packets not being able to fit in Tailscale packets. For this reason it is recommended that users run Tailscale on the Windows host only, and not inside WSL 2.
> https://tailscale.com/kb/1295/install-windows-wsl2

:::

`/etc/wsl.conf` を次のように設定します。

```shell
# https://github.com/xhiroga/homelab/blob/main/playbooks/roles/wsl/files/etc/wsl.conf
[automount]
options = "metadata"

[boot]
systemd=true
```

systemdを有効にすることで、OpenSSH Serverを管理するのが簡単になります。

### クライアント（macOS）側の設定

macOS側では次の設定を行います：

- Windowsアプリ（旧Remote Desktopアプリ）をインストールしておきます
  - WSLの再起動などが発生したときに、遠隔で操作を続けられるようにするためです
- SSH設定を次のように行います
  - 筆者はWindowsホスト側でもOpenssh Serverを有効にしているので、ポート被りを避けています。

```shell
Host hiroga-rtx4090
    HostName hiroga-rtx4090 # Tailscaleで設定した値
    User hiroga
    IdentityFile ~/.ssh/id_ed25519

Host wsl.hiroga-rtx4090
    HostName hiroga-rtx4090 # Windowsホスト側と同一の値
    User hiroga
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
```

より詳細な設定例は以下のリポジトリにあります：
https://github.com/xhiroga/homelab/blob/dcd53a62963327b6e7876cda3a1a782866f9f86e/playbooks/homelab.conf

## 開発上のTIPS

- WindowsマシンにSSDなどをホストする場合は、フォーマットをNTFSにするのがオススメです。
  - exFATだとWSLから参照した際にシンボリックリンクを貼れないので困ったことがありました。
- リポジトリはWSL2側にクローンするのが良いです。
  - Windows側にGitHub Desktopを入れて管理することもできますが、WSLからWindowsの管理するファイルシステムを参照した場合ハードリンクが貼れません
  - ハードリンクが貼れないことで、uvやpnpmでの開発が滞ります。

## トラブルシューティング

### SSH接続できない場合

SSH接続ができない場合は、次の手順で確認・対処します：

1. まずはWSL内部からlocalhostへのSSH接続を試してみます
   - 内部接続ができれば、外部からの接続問題を切り分けられます
2. sshdが立っているかを確認します

:::details sshdの状態確認コマンドと出力例

```shell
$ systemctl status sshd
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2025-03-27 08:30:42 JST; 20min ago
       Docs: man:sshd(8)
             man:sshd_config(5)
    Process: 1234 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
   Main PID: 1235 (sshd)
      Tasks: 1 (limit: 9447)
     Memory: 1.2M
        CPU: 42ms
     CGroup: /system.slice/ssh.service
             └─1235 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"
```

:::

3. sshdが立たない場合、WSL関連設定がポートを確保してしまっていることがあります。その場合はWindows側で次のコマンドを実行します：

```powershell
wsl --shutdown
sudo net stop winnat
sudo net start winnat
wsl
```

4. Windowsのファイアウォール設定も確認します
   - SSHポートが適切に開放されているか確認します

## まとめ

本記事では、Windows PCのGPUを活用して機械学習開発を行うためのWSL2環境構築方法を紹介しました。macOSからTailscaleとSSHを使ってWSL2環境にリモート接続し、VSCode Remote SSHを用いて快適な開発環境を構築する方法を解説しました。

Windows側の方がSSDの増設もしやすく、慣れると普段の開発環境をこちらに移したくなると思います。お役に立てば幸いです。
