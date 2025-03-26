WSL開発TIPS

動機

- ゲーム用途のWindowsに積んであるGPUを活かして機械学習の開発がしたい

アーキテクチャ

(フローチャートにする)
- macOSがクライアント
- Windowsホストマシン上のWSL2のUbuntuがサーバー
- Remote SSHを用いてVSCodeから接続

## 設定

### 共通

- tailscaleで接続するが、WSL側にはTailscaleをインストールしない
- VSCodeで接続する
- docker for windows と docker for ubuntuは択

### Windows

- こちらにはリポジトリを入れない
  - ハードリンクが貼れないため、uvやpnpmのパフォーマンスが下がる
- SSDなどがある場合、DNFSでマウントするのが望ましい
  - WSLからシンボリックリンクを作成できると嬉しいこともあるため
- WindowsホストにもSSHできるようにしておくと嬉しいこともある
- 次の通り設定する

```.wslconfig
# https://github.com/xhiroga/homelab/blob/98979a39f3ecdadea1191ce3b469a8401e775efc/playbooks/files/windows/.wslconfig
[wsl2]

networkingMode=mirrored

[experimental]

hostAddressLoopback=true
```

### WSL

- WSLにはtailscaleをインストールしない
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
