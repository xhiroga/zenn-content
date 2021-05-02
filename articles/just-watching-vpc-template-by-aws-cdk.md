---
title: "AWS CDKで構築したVPCのテンプレートを眺めるだけの記事"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","vpc","CDK"]
published: true
---
早速眺めていきます。

# ソース
```cdk_stack.py
from aws_cdk import (
    core,
    aws_ec2 as ec2
)

class CdkStack(core.Stack):

    def __init__(self, scope: core.Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        ec2.Vpc(self,"CDK_TEST_HIROGA")
```

`app.synth()` するところは省略。

# Overview
（アイコンが小さくて分かりづらい．．．）
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/3ee2fa4b-5dc6-4d38-a307-8e8ab16e1d47.png)

* VPC: 1つ
* Subnet: Public2つ、Private2つ
* InternetGateway: VPCに対して1つ
* NatGateway: PublicSubnetごとに1つ、合計2つ
* EIP: NatGatewayごとに1つ、合計2つ
* ほか、RouteTable, Route, SubnetRouteTableAssociation


# VPC
* デフォルトのCIDR Blockは`10.0.0.0/16`。
* Nameの命名規則は、指定しない限り `スタック名`/`リソースのID`になるようだ。

```
  CDKTESTHIROGA2CBE18D7:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      EnableDnsSupport: true
      InstanceTenancy: default
      Tags:
        - Key: Name
          Value: cdk/CDK_TEST_HIROGA
    Metadata:
      aws:cdk:path: cdk/CDK_TEST_HIROGA/Resource
```

# Public Subnet
* ネットワークアドレスはデフォルトで18ビット。
* AZ IDはデフォルトで `apne1-az4` と `apne1-az1`[^1]
* `aws-cdk:subnet-name` は、既存のSubnetをCDKでimportする際に参照するための値。何も指定しないと `aws-cdk:subnet-type` とおなじになるらしい。[^2]
* `aws-cdk:subnet-type` はISOLATED, PRIVATE, PUBLICのいずれか。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/52cef5d2-467b-251f-7001-5ec65134fcef.png)

[^1]: こういう記事ではAZ IDを使ったほうがいいみたいですね。[クラメソさんブログ](https://dev.classmethod.jp/cloud/use-az-id-for-identify-az-crossing-over-account/)
[^2]: [Importing an existing VPC](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-ec2-readme.html#importing-an-existing-vpc)

```
  CDKTESTHIROGAPublicSubnet1SubnetEDE92B34:
    Type: AWS::EC2::Subnet
    Properties:
      CidrBlock: 10.0.0.0/18
      VpcId:
        Ref: CDKTESTHIROGA2CBE18D7
      AvailabilityZone:
        Fn::Select:
          - 0
          - Fn::GetAZs: ""
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: cdk/CDK_TEST_HIROGA/PublicSubnet1
        - Key: aws-cdk:subnet-name
          Value: Public
        - Key: aws-cdk:subnet-type
          Value: Public
    Metadata:
      aws:cdk:path: cdk/CDK_TEST_HIROGA/PublicSubnet1/Subnet

```

# おまけ
今回作成したテンプレート

https://gist.github.com/hiroga-cc/5d5d33e2a677efce1a82c2f2d7cc9484


