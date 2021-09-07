---
title: "REDASH_SECRET_KEY が誤っており、気合でDataSourcesを復旧する"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["EC2","redash"]
published: true
---

RedashでDataSourcesの接続設定を暗号化するために用いられるキー、 `REDASH_SECRET_KEY`の値が誤っており、かつ正しい値が分からなくなりました。  
DataSourcesを気合で復旧した際の手順書です。

## TL;DR

前提: [Redash公式から提供されている EC2 AMI](https://redash.io/help/open-source/setup)を利用してホスティングしていることを想定しています。

1. バックアップの取得
2. ポート5432の開放
3. data_sourcesの削除および、シーケンスのリセット
4. data_sourcesを気合で復元
5. 動作確認・ポートの閉鎖

なお、[RedashのData Sourcesを解除してQueriesが吹き飛んだときの対処 - VTRyo](https://blog.vtryo.me/entry/redash-queries-restore)をかなり参考にしました。ありがとうございます。

## 1. バックアップの取得

AMIをバックアップします。
AWS Backupで普段からバックアップしていると安心です。

## 2. ポート5432の開放

RedashインスタンスにSSHした後、以下の通り`docker-compose.yml`を書き換えます。

```shell
ssh -i /tmp/id_rsa ubuntu@ip-**-**-**-**.ap-northeast-1.compute.internal
ubuntu@ip-**-**-**-**:~$ sudo vi /opt/redash/docker-compose.yml
```

```docker-compose.yml
~~~
  postgres:
    image: postgres:9.6-alpine
    env_file: /opt/redash/env
    volumes:
      - /opt/redash/postgres-data:/var/lib/postgresql/data
    restart: always
    ports:
      - "5432:5432"
~~~
```

## 3. data_sourcesの削除および、シーケンスのリセット

RedashにPostgreSQLクライアントを接続した上で、以下の通りSQLを実行します。

```sql
ALTER TABLE data_sources DISABLE TRIGGER ALL;

ALTER SEQUENCE data_sources_id_seq RESTART WITH 1; -- ポイント: 

DELETE FROM public.data_sources;

ALTER TABLE data_sources ENABLE TRIGGER ALL;
```

## 4. data_sourcesを気合で復元

かつて登録してあったDataSourceを、登録した順に正確に再現します。  
登録してからアーカイブしたものがあれば、それも再現します...w

## 5. 動作確認・ポートの閉鎖

特記事項なし

## まとめ

DataSourcesが全部使えなくなったら、手作業でデータを復元するしかない...という話でした。  
将来同じ事態に陥った人が、他にもつらい思いをした先駆者がいる、と思って慰められれば幸いです。

（一番いいのは `REDASH_SECRET_KEY` の正しい値をなんとかして突き止めることですけどね...!）
