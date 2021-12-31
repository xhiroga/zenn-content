---
title: "日本語訳がないman pagesをブラウザから日本語で読む"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Linux", "debian", "man"]
published: true
---

## TL;DR

Web版のman pagesのGoogle翻訳版リンクを生成します（Debianに対応）

![](/images/2021-12-31-man-in-google-translate-ja.gif)

```shell
% gman () { man "$1" > /dev/null && echo "https://dyn-manpages-debian-org.translate.goog/jump?q=$1&_x_tr_sl=auto&_x_tr_tl=ja&_x_tr_hl=$2"; }
% gman interfaces ja
https://dyn-manpages-debian-org.translate.goog/jump?q=interfaces&_x_tr_sl=auto&_x_tr_tl=ja&_x_tr_hl=ja

% gman nothing ja
No manual entry for nothing
```

## How it works

Debianのmanpagesは、Webサイトでも公開されています。

[index — Debian Manpages](https://manpages.debian.org/)

また、Google翻訳はテキストボックスにURLを貼り付けるだけでまるごと翻訳できます。

[ウェブページやドキュメントを翻訳する \- パソコン \- Google Translate ヘルプ](https://support.google.com/translate/answer/2534559?hl=ja&co=GENIE.Platform%3DDesktop)

それらを組み合わせて、Google翻訳後のDebianのWebサイトを表示するための関数を作りました。


:::details 仕様について  
Debian ManpagesのWebサイトは、ありがたいことにクエリパラメータからのjumpで最新版のURLを参照できます。
- Before: https://manpages.debian.org/jump?q=interfaces
- After:  https://manpages.debian.org/bullseye/ifupdown2/interfaces.5.en.html

また、Google翻訳で貼り付けるURLはリダイレクト前のURLでもOKです。
- OK     : https://manpages-debian-org.translate.goog/bullseye/ifupdown2/ifdown.8.en.html?_x_tr_sl=auto&_x_tr_tl=ja&_x_tr_hl=ja
- これもOK: https://dyn-manpages-debian-org.translate.goog/jump?q=interfaces&_x_tr_sl=auto&_x_tr_tl=ja&_x_tr_hl=ja

そのため、コマンドのパッケージを特定するとか、リダイレクト後のURLを取得するとか、面倒な処理なしで簡単に作れました。細かい気遣いに感謝ですね。

:::

## まとめ

気軽にmanpagesを参照できるようになりました。


## References

- [linux \- How do I check if a man page exists? \- Stack Overflow](https://stackoverflow.com/questions/12241709/how-do-i-check-if-a-man-page-exists)
