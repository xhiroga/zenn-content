---
title: "リージョン丸ごと綺麗にしてみた"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS"]
published: true
---
環境作りの練習に使っていた環境を削除するので、その様子をメモ書きします。  

# やったこと
1. CloudFormationのスタックを削除する準備
2. CloudFormationのスタックを削除する
3. 残ったものを削除する


# 1. CloudFormationのスタックを削除する準備
* ECRを削除します。  
あらかじめ消さない場合、以下のようにCloudFormationのスタック削除が失敗します。

```
in registry with id '******' cannot be deleted because it still contains images (Service: AmazonECR; Status Code: 400; Error Code: RepositoryNotEmptyException; Request ID: cbe340fe-39e6-11e9-90f7-******)
```

* API GatewayのVPC Linkを削除します。  
VPC Linkが参照しているリソースは削除できないためです。

* RDSインスタンスを削除します。  


# 2. CloudFormationのスタックを削除する
普通にCloudFormationのコンソール画面からスタックを削除します。  
![](https://i.imgur.com/76vryy4.png)

# 3. 残ったものを削除する
* EC2のマネジメントコンソールからネットワークインターフェースの一覧を表示し、それを削除対象のVPCで絞り込み、利用しているリソースを削除します。  
![](https://i.imgur.com/fFxq55Z.png)

* ENIを利用しているリソースを削除したら、次はENIをでタッチします。

* 最後にVPCを削除します。
![](https://i.imgur.com/kp42kpF.png)

* Route53で登録されているドメイン名から、使わなくなったものを削除します。  

