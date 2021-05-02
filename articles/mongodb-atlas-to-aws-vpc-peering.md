---
title: "MongoDB Atlas → AWSへのVPC Peering"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["MongoDB","AWS"]
published: true
---


# 概用
1. Atlas側のVPCからマイVPCにVPC Peeringのリクエスト
2. マイVPC側でAccept
3. マイVPC側のセキュリティグループにAtlasのVPCを追加

※基本的に[Atlasの公式ドキュメント](https://docs.atlas.mongodb.com/security-vpc-peering/)通りに作業します。

# 1. Atlas側のVPCからマイVPCにVPC Peeringをリクエスト

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/13d53d0e-3f0d-f077-6550-f0bda4dd10c8.png)

2020-05-12 追記: `Organization Owner` の権限が必要です。

# 2. マイVPC側でAccept

マネジメントコンソールからVPC Peeringのメニューを開き、VPC PeeringのリクエストをAcceptします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/9149a6b5-277a-df58-5743-92a5174f3fb8.png)


# 3. マイVPC側のセキュリティグループにAtlasのVPCを追加

ここがポイント。手動でRoute Tableの設定を修正してもいいですが、CFnで冪等に管理します。

スクリプト

```
#!/bin/sh -e

###
# VPC PeeringをAcceptした側のVPCに、VPC Peeringが存在する場合のみRouteを追加するスクリプト。
###

REQUESTER_VPC_ID=$1
REQUESTER_FRIENDLY_NAME=$2
DESCRIBE_VPC_PEER="aws ec2 describe-vpc-peering-connections --filters Name=status-code,Values=active Name=requester-vpc-info.vpc-id,Values=${REQUESTER_VPC_ID}"
echo ${DESCRIBE_VPC_PEER}

export VpcPeeringConnectionId=$(${DESCRIBE_VPC_PEER} \
    --query VpcPeeringConnections[0].VpcPeeringConnectionId \
    --output text)
export RequesterVpcCidr=$(${DESCRIBE_VPC_PEER} \
    --query VpcPeeringConnections[0].RequesterVpcInfo.CidrBlock \
    --output text)

if [ ${VpcPeeringConnectionId} != None ]; then
    aws ec2 modify-vpc-peering-connection-options \
        --vpc-peering-connection-id ${VpcPeeringConnectionId} \
        --accepter-peering-connection-options '{"AllowDnsResolutionFromRemoteVpc":true}' \
        --region ${AWS_DEFAULT_REGION}

    aws cloudformation deploy \
        --template-file cfn/vpc-peer-acceptor-config.yml \
        --stack-name vpc-peer-with-${REQUESTER_FRIENDLY_NAME} \
        --parameter-overrides \
            VpcPeeringConnectionId=${VpcPeeringConnectionId} \
            RequesterVpcCidr=${RequesterVpcCidr} \
        --tags date="$(date '+%Y%m%d%H%M%S')"
fi
```


CloudFormationテンプレート

```
AWSTemplateFormatVersion: '2010-09-09'
Description: VPC Peering Accepter Config

Parameters:
  VpcPeeringConnectionId:
    Type: String
  RequesterVpcCidr:
    Type: String

Resources:
  XXXRouteToVPCPeer:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId: { "Fn::ImportValue": !Sub "XXXRouteTable-Id" }
      DestinationCidrBlock: !Ref  RequesterVpcCidr
      VpcPeeringConnectionId: !Ref VpcPeeringConnectionId
```


# 4. 疎通チェック

今回はAmazon Linuxにmongoコマンドをインストールします。

ダウンロードリンクはここから。
https://www.mongodb.com/download-center/enterprise?jmp=docs

```
# 絶対もっと効率いいやり方あります。
MONGO_SHELL=https://downloads.mongodb.org/linux/mongodb-shell-linux-x86_64-amazon-4.2.1.tgz
curl -O $MONGO_SHELL
tar -zxvf mongodb-shell-linux-x86_64-amazon-4.2.1.tgz
sudo cp mongodb-linux-x86_64-amazon-4.2.1/bin/mongo /usr/bin/mongo
```

```
# 接続確認
mongo "mongodb+srv://<your_mongodb_host>.mongodb.net/test" --username <username>
...
MongoDB Enterprise <your_mongodb_host>-0:PRIMARY> 
```




