---
title: "GitHubのOrganizationにおいて、ThirdPartyのアプリを制限する際のTips"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GitHub"]
published: false
---




## TL;DR

ThirdParty Applicationの制限を有効にすると、それまで使っていたアプリケーションがが使えなくなります。  
メンバーに再び有効化してもらう必要があります。


## Detail

`Third-party application access policy` はデフォルトでOFFだが、メンバーが誤って危険なアプリをインストールしないように設定したほうがよい。  
ただし、設定に伴ってこれまでインストールしていたアプリが利用できなくなる。

例:
- Gitクライアントが使えなくなる（ex. GitKraken, Fork)
- AWS Amplifyでビルドが失敗するようになり、結果プレビューやデプロイが失敗する。
- CircleCIにログインできなくなる

## How to

Onにすると同時に、チームメンバーに自分が利用している ThirdParty Appの認可を求めるようにしてもらってください。  
具体的には、[Setting - Applications](https://github.com/settings/applications)

