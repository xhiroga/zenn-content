WSL開発TIPS

動機

- ゲーム用途のWindowsに積んであるGPUを活かして機械学習の開発がしたい

アーキテクチャ

(フローチャートにする)
- macOSがクライアント
- Windowsホストマシン上のWSL2のUbuntuがサーバー
  - WSL2がOpenssh Serverを立てて、そのポートをmirroredでホスト側に共有し、それにmacOSから接続
- Remote SSHを用いてVSCodeから接続

## 設定

### 共通

- tailscaleで接続するが、WSL側にはTailscaleをインストールしない
- VSCodeで接続する
- docker for windows と docker for ubuntuは択
- Bonjour を有効にすると .local でアクセスできるものの、Tailscaleに任せた方が安定する

### Windows

- こちらにはリポジトリを入れない
  - ハードリンクが貼れないため、uvやpnpmのパフォーマンスが下がる
- SSDなどがある場合、DNFSでマウントするのが望ましい
  - WSLからシンボリックリンクを作成できると嬉しいこともあるため
- WindowsホストにもSSHできるようにしておくと嬉しいこともある
- Remote Desktopのサーバー側の設定を有効化しておく
- 次の通り設定する

```.wslconfig
# https://github.com/xhiroga/homelab/blob/98979a39f3ecdadea1191ce3b469a8401e775efc/playbooks/files/windows/.wslconfig
[wsl2]

networkingMode=mirrored

[experimental]

hostAddressLoopback=true
```

networkingMode: WSLで立てたサーバーにWindowsのブラウザからアクセスできる、tailascaleを共有する、など
Windows１１ 22H2でexperimentalから昇格したと思わっる機能
ポート番号が共有される。これによってWSL側にTailscaleをインストールしなくて良い

### WSL

- WSLにはtailscaleをインストールしない

> If you run Tailscale on both the Windows host and inside WSL 2 at the same time, Tailscale encrypted traffic that flows from WSL 2 over Tailscale on the Windows host will not work due to Tailscale packets not being able to fit in Tailscale packets. For this reason it is recommended that users run Tailscale on the Windows host only, and not inside WSL 2.
https://tailscale.com/kb/1295/install-windows-wsl2


- WindowsにもSSHする場合もある。WSLは22番とは別ポートでSSHできるようにしておくと吉
- 次の通り設定する
- systemd

```/etc/wsl.conf
# https://github.com/xhiroga/homelab/blob/main/playbooks/roles/wsl/files/etc/wsl.conf
[automount]
options = "metadata"

[boot]
systemd=true

```

### クライアント

- macOSを想定
  - Windowsアプリ（旧Remote Desktopアプリ）入れておく
  - WSLの再起動などが発生したときに、遠隔で操作を続けられるように

```
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

（実ファイルではもうちょっと凝った設定にしてます）
https://github.com/xhiroga/homelab/blob/dcd53a62963327b6e7876cda3a1a782866f9f86e/playbooks/homelab.conf


## 開発上のTIPS

- WindowとUbuntuのパスはそれぞれ相互参照できる
  - Cドライブは /mnt/c/... で参照できる
- SSH接続が切れることは割とあるので、他の接続方法を確保しておく
  - Remote Desktop, Rustdeskなど


### トラブルシュート

- SSH接続できない
  - sshdが立っているかを確認 systemctl status sshd
    - そもそも立たない場合、WSL関連設定がポートを確保してしまっていることがあるのでキルする

```powershell
net stop winnat
net stop winnat
```

  - まずはWSL自体からWSL
  - Firewarllも疑う
  - だいたい1～2ヶ月で全パターンのトラブルを踏んで強くなれると思う
