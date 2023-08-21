---
title: "Redash on EC2の障害と復旧作業"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["EC2","redash"]
published: true
---
EC２でホスティングしているRedashが停止しました。

ELBから告げられる `502 Bad Gateway` のメッセージ

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/0522a04d-7e28-023b-a833-341201e9ac0f.png)

原因特定から復旧までのログを残します。

## 調査

### ELBのログ調査

さっそく省略します。なぜなら、ALBのロギングが有効になっていなかったため。

次回以降に役立つよう、AWSの公式ドキュメントに従って、ALBのアクセスログをS3に保存するように設定しておきます。  
<https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-access-logs.html#enable-access-logging>

### EC2インスタンスへの直接アクセス

踏み台サーバー経由で、EC2インスタンスの80番ポートにアクセスします。すると接続できない。
SecurityGroupに変更を加えた覚えはないので、EC2の中で問題が起きている可能性が高そうです。

```terminal
curl -v  ip-**-**-**-**.ap-northeast-1.compute.internal:80
...
* Failed to connect to ip-**-**-**-**.ap-northeast-1.compute.internal port 80: Connection refused
* Closing connection 0
...
```

ちなみに、成功した場合のレスポンスはこんな感じです。

```terminal
curl -v  ip-**-**-**-**.ap-northeast-1.compute.internal:80

< HTTP/1.1 302 FOUND
< Server: nginx/1.9.10
< Date: Tue, 12 Jan 2021 01:21:12 GMT
......
```

### RedashへのSSHとプロセスの死活確認

Redashの入っているEC2インスタンスにSSHし、当該プロセスの死活確認をします。
ここで、そもそもRedash on EC2ってどうやって動いているんだっけ？という疑問に突き当たりました。

[公式ドキュメント](https://redash.io/help/open-source/setup)によれば docker-compose を利用しているとのこと。

> What the script does is:

> Install Docker and Docker Compose.
Download our recommended Docker Compose configuration and create initial configuration.
Start everything.

確認してみると...あれ、Dockerプロセスが死んでますね。

```terminal
/opt/redash$ sudo docker ps
sudo: unable to resolve host ip-**-**-**-**: Resource temporarily unavailable
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

## 原因特定・復旧作業

### 原因特定

docker-composeプロセスを再起動するため、[Redashのアップグレード手順](https://redash.io/help/open-source/admin-guide/how-to-upgrade)を参考に再起動を実行すると、エラーが発生しました。

```terminal

cd /opt/redash
docker-compose up -d

> INTERNAL ERROR: cannot create temporary directory
```

検索するとディスクの残量が少ないことが原因のよう。  
<https://stackoverflow.com/questions/40755494/docker-compose-internal-error-cannot-create-temporary-directory>

アプリケーションレベルでは、Redashがクエリの実行結果をキャッシュしていることが原因のような気がします。  
<https://discuss.redash.io/t/redash-taking-high-disk-space/6777>

そういえば limit句 なしでクエリを実行して、レスポンスがなかったのでブラウザのタブを閉じたような記憶が...
（PostgresDBの中を覗けば明らかになると思いますが、今回はそこまでしません）

### 復旧作業

[Qiita](https://qiita.com/mangano-ito/items/629b10ea5d1ab80f2cc6)、[AWSのドキュメント](https://aws.amazon.com/premiumsupport/knowledge-center/ebs-volume-size-increase/)に従い、EBSのボリュームの拡張とデバイス・ファイルシステムへの割り当ての増加を行いました。

## 反省

EC2のディスク・メモリの逼迫はデフォルトのメトリクスに出てこないので、必ず追加設定するような仕組み作りがしたいところですね。

