---
title: "WSL2でHomebrewを入れ直す"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["wsl2", "homebrew", "linux", "ubuntu"]
published: true
---

## TL;DR

wsl2にデフォルトとは異なる`HOMEBREW_PREFIX`でHomebrewをインストールしていたため、再インストールしました。

## 動機

WSL2にHomebrewを入れて運用していたところ、設定がおかしくなっていたことが分かりました。

```shell
# 診断用コマンドのアウトプット（抜粋）
/mnt/c/Users/hiroga$ brew doctor

# ...省略...
Warning: Your Homebrew's prefix is not /home/linuxbrew/.linuxbrew.
```

LinuxのHomebrewでは `HOMEBREW_PREFIX` が `/home/linuxbrew/.linuxbrew` であるべきです。完全に自業自得なのですが、macOS用のAnsible RoleでHomebrewを設定したことが原因かもしれません。

WSL2環境のHomebrewを再インストールすることにしました。

## 手順

### 削除前の状態の記録

`brew list`, `brew config` および `brew doctor` の出力を控えておきます。

<details>
<summary>アウトプット全体</summary>

```shell
/mnt/c/Users/hiroga$ brew config
HOMEBREW_VERSION: 4.4.0
ORIGIN: https://github.com/Homebrew/brew
HEAD: 84c31175f11860129a9aaed40a13c549625e2db1
Last commit: 2 weeks ago
Core tap JSON: 15 Oct 22:13 UTC
HOMEBREW_PREFIX: /usr/local
HOMEBREW_REPOSITORY: /usr/local/Homebrew
HOMEBREW_CELLAR: /usr/local/Cellar
HOMEBREW_CASK_OPTS: []
HOMEBREW_DISPLAY: :0
HOMEBREW_MAKE_JOBS: 20
Homebrew Ruby: 3.3.5 => /usr/local/Homebrew/Library/Homebrew/vendor/portable-ruby/3.3.5/bin/ruby
CPU: 20-core 64-bit unknown_0x6_0xbf
Clang: N/A
Git: 2.34.1 => /bin/git
Curl: 7.81.0 => /bin/curl
Kernel: Linux 5.15.153.1-microsoft-standard-WSL2 x86_64 GNU/Linux
OS: Ubuntu 22.04.5 LTS (jammy)
WSL: 2 (Microsoft Store)
Host glibc: 2.35
/usr/bin/gcc: 11.4.0
/usr/bin/ruby: N/A
glibc: N/A
gcc@11: N/A
gcc: N/A
xorg: N/A
---
**NOTE: I am confident that the warning regarding Homebrew's prefix is unrelated to the current issue.**

/mnt/c/Users/hiroga$ brew doctor
Please note that these warnings are just used to help the Homebrew maintainers
with debugging if you file an issue. If everything you use Homebrew for is
working fine: please don't worry or file an issue; just ignore this. Thanks!

Warning: Your Homebrew's prefix is not /home/linuxbrew/.linuxbrew.

Many of Homebrew's bottles (binary packages) can only be used with the default prefix.
Consider uninstalling Homebrew and reinstalling into the default prefix.
It is expected behaviour that some formulae will fail to build in this unsupported configuration.
It is expected behaviour that Homebrew will be buggy and slow.
Do not create any issues about this on Homebrew's GitHub repositories.
Do not create any issues even if you think this message is unrelated.
Any opened issues will be immediately closed without response.
Do not ask for help from Homebrew or its maintainers on social media.
You may ask for help in Homebrew's discussions but are unlikely to receive a response.
Try to figure out the problem yourself and submit a fix as a pull request.
We will review it but may or may not accept it.
```

</details>

## カスタムプレフィックスにインストールしたHomebrewの削除

[公式のガイド](https://github.com/homebrew/install?tab=readme-ov-file#uninstall-homebrew)に従ってHomebrewをアンインストールします。ただし、カスタムプレフィックスのパスによってはスクリプトが失敗します。

というのも、私がHomebrewをインストールしていた`/usr/local`は、NVIDIAのツールも入ってくるディレクトリなのですよね。ディレクトリが空ではないため、削除に失敗します。

ここでは、`uninstall.sh`の実行後に`sudo rm -rf /usr/local/Homebrew`を実行するに留めました。

<details>
<summary>アウトプット全体</summary>

```shell
curl -fsSLO https://raw.githubusercontent.com/Homebrew/install/HEAD/uninstall.sh
/bin/bash uninstall.sh --path /usr/local

/bin/bash uninstall.sh --path /usr/local
Warning: This script will remove:
/home/hiroga/.cache/Homebrew/
/usr/local/Caskroom/
/usr/local/Cellar/
/usr/local/Homebrew/
/usr/local/Homebrew/.devcontainer/
/usr/local/Homebrew/.dockerignore
/usr/local/Homebrew/.editorconfig
/usr/local/Homebrew/.github/
/usr/local/Homebrew/.gitignore
/usr/local/Homebrew/.shellcheckrc
/usr/local/Homebrew/.sublime/
/usr/local/Homebrew/.vale.ini
/usr/local/Homebrew/.vscode/
/usr/local/Homebrew/CHANGELOG.md
/usr/local/Homebrew/CONTRIBUTING.md
/usr/local/Homebrew/Dockerfile
/usr/local/Homebrew/LICENSE.txt
/usr/local/Homebrew/Library/
/usr/local/Homebrew/README.md
/usr/local/Homebrew/bin/brew
/usr/local/Homebrew/completions/
/usr/local/Homebrew/docs/
/usr/local/Homebrew/manpages/
/usr/local/Homebrew/package/
/usr/local/bin/brew -> /usr/local/Homebrew/bin/brew
/usr/local/etc/bash_completion.d/brew -> /usr/local/Homebrew/completions/bash/brew
/usr/local/share/doc/homebrew -> /usr/local/Homebrew/docs
/usr/local/share/fish/vendor_completions.d/brew.fish -> /usr/local/Homebrew/completions/fish/brew.fish
/usr/local/share/man/man1/README.md -> /usr/local/Homebrew/manpages/README.md
/usr/local/share/man/man1/brew.1 -> /usr/local/Homebrew/manpages/brew.1
/usr/local/share/zsh/site-functions/_brew -> /usr/local/Homebrew/completions/zsh/_brew
/usr/local/var/homebrew/
Are you sure you want to uninstall Homebrew? This will remove your installed packages! [y/N] Y
==> Removing Homebrew installation...
Warning: Failed to delete /usr/local/Caskroom
rm: cannot remove '/usr/local/Caskroom': Permission denied
Warning: Failed to delete /usr/local/Cellar
rm: cannot remove '/usr/local/Cellar': Permission denied
Warning: Failed to delete /usr/local/Homebrew
rm: cannot remove '/usr/local/Homebrew/.git/describe-cache/84c31175f11860129a9aaed40a13c549625e2db1': Permission denied
==> Removing empty directories...
[sudo] password for hiroga: 
==> /usr/bin/sudo /usr/bin/find /usr/local/bin /usr/local/etc /usr/local/include /usr/local/lib /usr/local/opt /usr/local/sbin /usr/local/share /usr/local/var /usr/local/Caskroom /usr/local/Cellar /usr/local/Homebrew /usr/local/Frameworks -depth -type d -empty -exec rmdir {} ;
==> /usr/bin/sudo rmdir /usr/local
rmdir: failed to remove '/usr/local': Directory not empty
Warning: Failed during: /usr/bin/sudo rmdir /usr/local
==> /usr/bin/sudo rmdir /usr/local/Homebrew
rmdir: failed to remove '/usr/local/Homebrew': Directory not empty
Warning: Failed during: /usr/bin/sudo rmdir /usr/local/Homebrew
Warning: Homebrew partially uninstalled (but there were steps that failed)!
To finish uninstalling rerun this script with `sudo`.
The following possible Homebrew files were not deleted:
/usr/local/Homebrew/
/usr/local/Homebrew/.git/
/usr/local/bin/
/usr/local/cuda -> /usr/local/cuda-12.6
/usr/local/cuda-12 -> /usr/local/cuda-12.6
/usr/local/cuda-12.6/
/usr/local/etc/
/usr/local/games/
/usr/local/include/
/usr/local/lib/
/usr/local/man -> /usr/local/share/man
/usr/local/opt/
/usr/local/sbin/
/usr/local/share/
/usr/local/src/
You may wish to remove them yourself.
```

</details>

## デフォルトのプレフィックスに対してHomebrewを再インストール

念のためにWSL2を再起動してから、公式のガイドに従ってHomebrewを再インストールします。

```shell
sudo reboot
wsl
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

<details>
<summary>アウトプット全体</summary>

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
==> Checking for `sudo` access (which may request your password)...
[sudo] password for hiroga:
==> This script will install:
/home/linuxbrew/.linuxbrew/bin/brew
/home/linuxbrew/.linuxbrew/share/doc/homebrew
/home/linuxbrew/.linuxbrew/share/man/man1/brew.1
/home/linuxbrew/.linuxbrew/share/zsh/site-functions/_brew
/home/linuxbrew/.linuxbrew/etc/bash_completion.d/brew
/home/linuxbrew/.linuxbrew/Homebrew
==> The following new directories will be created:
/home/linuxbrew/.linuxbrew/bin
/home/linuxbrew/.linuxbrew/etc
/home/linuxbrew/.linuxbrew/include
/home/linuxbrew/.linuxbrew/lib
/home/linuxbrew/.linuxbrew/sbin
/home/linuxbrew/.linuxbrew/share
/home/linuxbrew/.linuxbrew/var
/home/linuxbrew/.linuxbrew/opt
/home/linuxbrew/.linuxbrew/share/zsh
/home/linuxbrew/.linuxbrew/share/zsh/site-functions
/home/linuxbrew/.linuxbrew/var/homebrew
/home/linuxbrew/.linuxbrew/var/homebrew/linked
/home/linuxbrew/.linuxbrew/Cellar
/home/linuxbrew/.linuxbrew/Caskroom
/home/linuxbrew/.linuxbrew/Frameworks

Press RETURN/ENTER to continue or any other key to abort:
==> /usr/bin/sudo /usr/bin/install -d -o hiroga -g hiroga -m 0755 /home/linuxbrew/.linuxbrew
==> /usr/bin/sudo /bin/mkdir -p /home/linuxbrew/.linuxbrew/bin /home/linuxbrew/.linuxbrew/etc /home/linuxbrew/.linuxbrew/include /home/linuxbrew/.linuxbrew/lib /home/linuxbrew/.linuxbrew/sbin /home/linuxbrew/.linuxbrew/share /home/linuxbrew/.linuxbrew/var /home/linuxbrew/.linuxbrew/opt /home/linuxbrew/.linuxbrew/share/zsh /home/linuxbrew/.linuxbrew/share/zsh/site-functions /home/linuxbrew/.linuxbrew/var/homebrew /home/linuxbrew/.linuxbrew/var/homebrew/linked /home/linuxbrew/.linuxbrew/Cellar /home/linuxbrew/.linuxbrew/Caskroom /home/linuxbrew/.linuxbrew/Frameworks
==> /usr/bin/sudo /bin/chmod ug=rwx /home/linuxbrew/.linuxbrew/bin /home/linuxbrew/.linuxbrew/etc /home/linuxbrew/.linuxbrew/include /home/linuxbrew/.linuxbrew/lib /home/linuxbrew/.linuxbrew/sbin /home/linuxbrew/.linuxbrew/share /home/linuxbrew/.linuxbrew/var /home/linuxbrew/.linuxbrew/opt /home/linuxbrew/.linuxbrew/share/zsh /home/linuxbrew/.linuxbrew/share/zsh/site-functions /home/linuxbrew/.linuxbrew/var/homebrew /home/linuxbrew/.linuxbrew/var/homebrew/linked /home/linuxbrew/.linuxbrew/Cellar /home/linuxbrew/.linuxbrew/Caskroom /home/linuxbrew/.linuxbrew/Frameworks 
==> /usr/bin/sudo /bin/chmod go-w /home/linuxbrew/.linuxbrew/share/zsh /home/linuxbrew/.linuxbrew/share/zsh/site-functions
==> /usr/bin/sudo /bin/chown hiroga /home/linuxbrew/.linuxbrew/bin /home/linuxbrew/.linuxbrew/etc /home/linuxbrew/.linuxbrew/include /home/linuxbrew/.linuxbrew/lib /home/linuxbrew/.linuxbrew/sbin /home/linuxbrew/.linuxbrew/share /home/linuxbrew/.linuxbrew/var /home/linuxbrew/.linuxbrew/opt /home/linuxbrew/.linuxbrew/share/zsh /home/linuxbrew/.linuxbrew/share/zsh/site-functions /home/linuxbrew/.linuxbrew/var/homebrew /home/linuxbrew/.linuxbrew/var/homebrew/linked /home/linuxbrew/.linuxbrew/Cellar /home/linuxbrew/.linuxbrew/Caskroom /home/linuxbrew/.linuxbrew/Frameworks 
==> /usr/bin/sudo /bin/chgrp hiroga /home/linuxbrew/.linuxbrew/bin /home/linuxbrew/.linuxbrew/etc /home/linuxbrew/.linuxbrew/include /home/linuxbrew/.linuxbrew/lib /home/linuxbrew/.linuxbrew/sbin /home/linuxbrew/.linuxbrew/share /home/linuxbrew/.linuxbrew/var /home/linuxbrew/.linuxbrew/opt /home/linuxbrew/.linuxbrew/share/zsh /home/linuxbrew/.linuxbrew/share/zsh/site-functions /home/linuxbrew/.linuxbrew/var/homebrew /home/linuxbrew/.linuxbrew/var/homebrew/linked /home/linuxbrew/.linuxbrew/Cellar /home/linuxbrew/.linuxbrew/Caskroom /home/linuxbrew/.linuxbrew/Frameworks 
==> /usr/bin/sudo /bin/mkdir -p /home/linuxbrew/.linuxbrew/Homebrew
==> /usr/bin/sudo /bin/chown -R hiroga:hiroga /home/linuxbrew/.linuxbrew/Homebrew
==> Downloading and installing Homebrew...
remote: Enumerating objects: 284019, done.
remote: Counting objects: 100% (404/404), done.
remote: Compressing objects: 100% (239/239), done.
remote: Total 284019 (delta 173), reused 359 (delta 152), pack-reused 283615 (from 1)
remote: Enumerating objects: 55, done.
remote: Counting objects: 100% (33/33), done.
remote: Total 55 (delta 33), reused 33 (delta 33), pack-reused 22 (from 1)
==> Updating Homebrew...
==> Downloading https://ghcr.io/v2/homebrew/portable-ruby/portable-ruby/blobs/sha256:bd08b92d6725f9216fc3c2458ffd174d5468d43d47dd0fcaeb5109e3008fd40b
############################################################################################################ 100.0%
==> Pouring portable-ruby-3.3.5.x86_64_linux.bottle.tar.gz
Warning: /home/linuxbrew/.linuxbrew/bin is not in your PATH.
  Instructions on how to configure your shell for Homebrew
  can be found in the 'Next steps' section below.
==> Installation successful!

==> Homebrew has enabled anonymous aggregate formulae and cask analytics.
Read the analytics documentation (and how to opt-out) here:
  https://docs.brew.sh/Analytics
No analytics data has been sent yet (nor will any be during this install run).

==> Homebrew is run entirely by unpaid volunteers. Please consider donating:
  https://github.com/Homebrew/brew#donations

==> Next steps:
- Run these commands in your terminal to add Homebrew to your PATH:
    echo >> /home/hiroga/.bashrc
    echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/hiroga/.bashrc
    eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
- Install Homebrew's dependencies if you have sudo access:
    sudo apt-get install build-essential
  For more information, see:
    https://docs.brew.sh/Homebrew-on-Linux
- We recommend that you install GCC:
    brew install gcc
- Run brew help to get started
- Further documentation:
    https://docs.brew.sh
```

</details>

インストール後に`brew config`と`brew doctor`を実行し、警告が出ていないことを確認します。

```shell
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
brew config
brew doctor
```

<details>
<summary>アウトプット全体</summary>

```shell
$ eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
$ brew
Example usage:
  brew search TEXT|/REGEX/
  brew info [FORMULA|CASK...]
  brew install FORMULA|CASK...
  brew update
  brew upgrade [FORMULA|CASK...]
  brew uninstall FORMULA|CASK...
  brew list [FORMULA|CASK...]

Troubleshooting:
  brew config
  brew doctor
  brew install --verbose --debug FORMULA|CASK

Contributing:
  brew create URL [--no-fetch]
  brew edit [FORMULA|CASK...]

Further help:
  brew commands
  brew help [COMMAND]
  man brew
  https://docs.brew.sh
$ brew config
HOMEBREW_VERSION: 4.4.1
ORIGIN: https://github.com/Homebrew/brew
HEAD: 70672606c6ac07a5e8f75abeda7b8736e8285dbe
Last commit: 2 days ago
Core tap JSON: 16 Oct 02:08 UTC
HOMEBREW_PREFIX: /home/linuxbrew/.linuxbrew
HOMEBREW_CASK_OPTS: []
HOMEBREW_DISPLAY: :0
HOMEBREW_MAKE_JOBS: 20
Homebrew Ruby: 3.3.5 => /home/linuxbrew/.linuxbrew/Homebrew/Library/Homebrew/vendor/portable-ruby/3.3.5/bin/ruby   
CPU: 20-core 64-bit unknown_0x6_0xbf
Clang: N/A
Git: 2.34.1 => /bin/git
Curl: 7.81.0 => /bin/curl
Kernel: Linux 5.15.153.1-microsoft-standard-WSL2 x86_64 GNU/Linux
OS: Ubuntu 22.04.5 LTS (jammy)
WSL: 2 (Microsoft Store)
Host glibc: 2.35
/usr/bin/gcc: 11.4.0
/usr/bin/ruby: N/A
glibc: N/A
gcc@11: N/A
gcc: N/A
xorg: N/A
$ brew doctor
Your system is ready to brew.
```

</details>

次に、Next stepsに従って次の3アクションを実施します。

1. .bashrc（などのインタラクティブシェル用設定）に対する環境変数等の設定
2. `sudo apt-get install build-essential`
3. `brew install gcc`

ただし、私はdotfilesをすべての環境で共有しているため（これが諸悪の根源な気もしますが...）、環境変数等の設定は次の通り書き換えています。

```bash
# 変更前
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"

# 変更後
# https://github.com/Homebrew/install/blob/511002fcae8217680de7ac194b17fb45a7856d56/uninstall.sh#L66-L87
un="$(uname)"
case "${un}" in
  Linux)
    ostype=linux
    homebrew_prefix_default=/home/linuxbrew/.linuxbrew
    ;;
  Darwin)
    ostype=macos
    if [[ "$(uname -m)" == "arm64" ]]
    then
      homebrew_prefix_default=/opt/homebrew
    else
      homebrew_prefix_default=/usr/local
    fi
    realpath() {
      cd "$(dirname "$1")" && echo "$(pwd -P)/$(basename "$1")"
    }
    ;;
  *)
    abort "Unsupported system type '${un}'"
    ;;
esac
eval "$(${homebrew_prefix_default}/bin/brew shellenv)"
```

## まとめ

wsl2にHomebrewを再インストールできました。
