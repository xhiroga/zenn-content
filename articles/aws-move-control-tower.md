---
title: "AWS Control TowerとSSOを北バージニアから東京リージョンに引っ越す（廃止編）"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS"]
published: true
---

2021年3月に[AWS Control Tower が東京リージョンで利用できるようになりました。](https://aws.amazon.com/jp/blogs/news/aws-control-tower-tokyo/)  
また、2020年3月には[AWS Single Sign-Onが東京リージョンで利用できるようになっています。](https://aws.amazon.com/jp/blogs/news/aws-single-sign-on-tokyo/)
北バージニアに設定していたControl TowerとSSOを、東京リージョンに引っ越してみます。もちろん引っ越しオプションなどはないので、一度退役させることになります。  

先駆者様はこちら。

- [Control Towerを廃止して東京リージョンで再有効化してみた – 廃止編 \| DevelopersIO](https://dev.classmethod.jp/articles/decommission-control-tower/)
- [Control Towerを廃止して東京リージョンで再設定してみた \- Qiita](https://qiita.com/kazu_kazu/items/122d7dc6b6e34a23e081)

## 前提

- 作業用にAdmin権限を持つIAM Userを作成してあります。
- Service Catalogはノータッチです。
- Control Towerが作成するVPCはノータッチです。
- Organizationは削除せず、そのまま持ち越しました。
- OrganizationのControl Tower統合も削除せずに持ち越しました。

（というか、正直に言ってOrganizationにControl Tower統合があるとは知りませんでした...マネジメントコンソールだと分からないので...）

# 手順

- AWS Control Towerの退役
- AWS SSOの退役
- 手動クリーンアップ

## AWS Control Towerの退役

Lading zone setting から `Decommission your landing zone` を選択します。
完了したら、きれいに消えているかをチェックしましょう。

参考: 作成されるリソース(ただし結構残るので、[手動クリーンアップ](##手動クリーンアップ)で削除します。)  
[AWS Control Tower のしくみ \- AWS Control Tower](https://docs.aws.amazon.com/ja_jp/controltower/latest/userguide/how-control-tower-works.html#what-shared)

## AWS SSOの退役

特に難しいことはなくて、`Delete AWS SSO configuration` を選択します。  
この手順でディレクトリグループとアクセス権限セットを含む、SSOに関するすべての設定が削除されます。ご注意ください。

## 手動クリーンアップ

以下のチェックリストに対応します。

- [ ] Control Towerが作成したが削除されてないStackSetsの削除
- [ ] OrganizationのOUの削除
- [ ] CloudWatch Logグループの削除
- [ ] AWS Configが有効になっていないことの確認
- [ ] IAM RoleおよびIAM Policyの削除


### ピックアップ: CloudWatch Logグループの削除

OrganizationのRootアカウントのControl Towerを有効にしたリージョン（この記事の場合は北バージニア）に、`aws-controltower/CloudTrailLogs`という名前で存在します。

### ピックアップ: AWS Configが有効になっていないことの確認

こんな感じのシェルをOrganizationのメンバーアカウントに実行し、有効になっていないかを確認しました。

```shell
#!/bin/sh

for region in 'us-east-1' 'us-east-2' 'us-west-2' 'ap-northeast-1' # 必要に応じて足してください。
do
  aws configservice describe-delivery-channels --region $region
done
```


### ピックアップ: IAM RoleおよびIAM Policyの削除

このポイントだけ、先駆者の皆さんと手順が異なります。  
筆者の環境では Control Towerのバージョンが2.2だったためか、新たに2.7でLanding Zoneを再作成する際に同じIAM Roleを利用できませんでした。  
（権限が足りず、 `is not authorized to perform: organizations:DescribeAccount on resource:` のようなエラーになる）

したがって、以下のIAM Role, Policyをまとめて削除しました。
- AWSControlTowerAdmin
- AWSControlTowerCloudTrailRole
- AWSControlTowerStackSetRole
- AWSControlTowerConfigAggregatorRoleForOrganizations
- AWSControlTowerServiceRolePolicy
- AWSControlTowerAdminPolicy
- AWSControlTowerCloudTrailRolePolicy
- AWSControlTowerStackSetRolePolicy

削除されたRole, Policyは、Landing Zoneを再設定する際に再生成されるました。

## 参考資料

- [Walkthrough: Decommission a landing zone \- AWS Control Tower](https://docs.aws.amazon.com/controltower/latest/userguide/decommission-landing-zone.html)
- [AWS SSO Region availability \- AWS Single Sign\-On](https://docs.aws.amazon.com/singlesignon/latest/userguide/regions.html?icmpid=docs_sso_console)
- [AWS Control Tower でのアイデンティティベースのポリシー \(IAM ポリシー\) の使用 \- AWS Control Tower](https://docs.aws.amazon.com/ja_jp/controltower/latest/userguide/access-control-managing-permissions.html)
- [AWS Control Tower がロールと連携してアカウントを作成および管理する方法 \- AWS Control Tower](https://docs.aws.amazon.com/ja_jp/controltower/latest/userguide/roles-how.html)
