---
title: "$20/MonthでRedashインスタンスを建てる（クレジットつき）"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["DigitalOcean","redash"]
published: true
---

DigitalOceanでインスタンスとロードバランサーを構築し、おひとり様Redashインスタンスの運用をはじめました。  
誰かの役に立ったらいいな＆間違いがあったら指摘してほしいな、という思いから記事にします。

**無料$100クレジットのリンク**

リファラルプログラムがあり、以下のリンクからサインアップすると $100 のクレジットが手に入ります（私にも$25が追加されます）。私は誰の紹介もなくサインアップして悔しい思いをしたので、よければ！

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.cdn.digitaloceanspaces.com/WWW/Badge%201.svg)](https://www.digitalocean.com/?refcode=140d95082e62&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)


## 免責

- DigitalOceanの初心者なので、誤っている可能性があります。
- $20/Monthにドメイン管理費用は含まれていません。

## 手順

1. Dropletsの作成
2. LoadBalancerの作成(HTTP)
3. LoadBalancerのHTTPS化

## Dropletsの作成

AWSでいうEC2に相当する概念だと思います。  
Maketplaceから設定済みのイメージが選べるので、サクッと建てられます。特にハマるポイントはなし。

[Setting up a Redash Instance](https://redash.io/help/open-source/setup)

インスタンスを作成したら、SSHで接続します。  
ポイントは ユーザー名が `root` であること。AWSのRedashのAMIでは `ubuntu` なので、この辺クラウドベンダーごとに違うんですかね。

```sh
ssh root@${IP_ADDRESS}
```

## LoadBalancerの作成(HTTP)

LoadBalancerを作成し、DNSレコードを登録します。  
DigitalOceanにもLoadBalancerはありますが、私はGCPのCloudDNSに登録しました。

インスタンスとロードバランサーのリージョンを合わせるのがポイントです。  
（私はどちらもCaliforniaにしてみました）

## LoadBalancerのHTTPS化

DigitalOceanの外部でDNSを管理してしまったので、CertBotで証明書を発行します。  
TXTレコードを手動で登録する方法を選びました。

```sh
docker run -it --rm --name certbot \
    -v "$(pwd)/etc/letsencrypt:/etc/letsencrypt"\
    certbot/certbot certonly --manual --preferred-challenges dns --agree-tos -d ${DOMAIN} -m ${EMAIL}
```

証明書が発行されたら、証明書を使ってForwarding rulesをHTTP → HTTPに切り替えます。

## まとめ

DigitalOceanでインスタンスとロードバランサーを構築することで、他のクラウドベンダーと比べて安価にRedashインスタンスをホストできました。

**無料$100クレジットのリンク（再掲）**

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.cdn.digitaloceanspaces.com/WWW/Badge%201.svg)](https://www.digitalocean.com/?refcode=140d95082e62&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)
