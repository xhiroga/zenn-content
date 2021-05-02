---
title: "GCP・Terraform・CircleCIでゼロからテックブログ更新通知botを作ったので解説（GitHubリポジトリあり）"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["JavaScript","CircleCI","gcp","Terraform","cloudfunctions"]
published: true
---
テックブログの更新通知をRSSから取得してDiscordに投稿するbotをGCPで作ったので解説します！
GCP・Terraformをしっかり触ったのはこれが初めてなので、ご指摘などあればぜひコメントいただきたいです。

## botの動作イメージ
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/e9c28571-d6aa-f674-7a0b-558ee49ffa32.png)

## ソースコード（★ください！）
https://github.com/hiroga-cc/gcp-rss-webhook

# 全体像
## アーキテクチャ
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/941a37cb-8d0b-682e-733b-fc5949bc26e7.png)

## デプロイフロー
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/2f9258ed-fb1c-0522-c6f8-4b6c8ad45f20.png)

一応上記のアーキテクチャの元ファイル
https://docs.google.com/presentation/d/1bPJmY_O2kh6mIyC_UyGhKS9TbcKhMVfcbCO-gQau200/edit?usp=sharing

# 手順
## 1. リポジトリの作成
手元にリポジトリを作成します。↓をフォークしても良いと思います（★ください！）
https://github.com/hiroga-cc/gcp-rss-webhook

また、GitHubリポジトリをCircleCIに接続しておいてください。この時点のビルドは失敗しても大丈夫です。

## 2. プロジェクトの作成
メニューからプロジェクトを作成します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/ed070c61-1419-438b-bdba-6b56c24bb948.png)

## 3. サービスアカウントの作成
IAMからサービスアカウントを作成します。  
権限はとりあえずプロジェクト編集者、としています。本当は利用するリソースにしぼった方が良いのでしょうけど、デプロイ時に変なところでハマりたくなかったので。
キーをJSONで作成し、ダウンロードします。

## 4. 鍵のローカル・CircleCIへの配置
ダウンロードしたJSONファイルを、ローカルとリモートの両方に配置します。  
ローカルへの配置は、Terraform・CircleCIの挙動確認で必要になります。  

### ローカルに配置
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/c113527f-bebe-95e7-44f7-48fd2808891b.png)

### CircleCIに配置
CircleCIのcontextを適当に作成します。作成したcontextの中に、 `GCLOUD_SERVICE_KEY` という名前でJSONファイルの中身を保存します。もっと良いやり方をご存知の方は教えてください...。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/3b0fd7a8-7fc8-7b91-5e05-7c57e48d6bb4.png)

## 5. CloudStorageを作成
Terraformのstate管理のファイルやソースコードをアップロードするバケットを作成します。リポジトリ内の`mb.sh`を使ってください。  

```sh
gsutil mb -l asia gs://${BUCKET_NAME}/
```

なお、リポジトリ内の `main.tf` にCloudStorageの名前を直接書いているところがあります。*これを直さないと動かない*のでご注意ください。

## 6. まずはローカルからTerraformを起動！
この段階で、必要なAPIをGCPのコンソール上から有効にします。`Cloud Pub/Sub`と`Cloud Scheduler`のAPIを有効にする必要があります。それぞれメニューから開く、ジョブを途中まで作る、などしてみて下さい。
続いて、環境変数をセットした上で、ターミナルからTerraformを実行します。`tf`ファイルはリポジトリ内を参照してください。  

```sh
export GOOGLE_APPLICATION_CREDENTIALS=./key.json
export TF_VAR_GOOGLE_APPLICATION_CREDENTIALS=./key.json
export TF_VAR_project_id="${PROJECT_ID}"
export TF_VAR_bucket_name="${BUCKET_NAME}"
# export TF_LOG=DEBUG
```

## ポイント
### クレデンシャルの渡し方
`GOOGLE_APPLICATION_CREDENTIALS` を2通りの方法で渡す必要があることです（これもなんとかなりませんか？）
１行目のexportではTerraformがCloudStorageにアクセスするためのクレデンシャルを設定し、２行目のexportではTerraformがデプロイする際に必要なクレデンシャルを設定しています。

### ソースコードの指定
ソースコードのファイル名を毎回変えているのにお気付きですか？
2019-05-19時点でTerraformは、CloudFunctionsのソースコードのパスがそのままだと、変更を検知できないようです。そのため、毎回ファイルのパスを変更しています。
参考: https://github.com/terraform-providers/terraform-provider-google/issues/1938

```
resource "google_cloudfunctions_function" "function" {
  name                = "rss-webhook-function"
  entry_point         = "handler"
  available_memory_mb = 256
  project             = "${var.project_id}"
  runtime             = "nodejs8"

  environment_variables = {
    BUCKET_NAME = "${var.bucket_name}"
  }

  event_trigger {
    event_type = "google.pubsub.topic.publish"
    resource   = "${google_pubsub_topic.topic.name}"
  }

  source_archive_bucket = "${var.bucket_name}"
  source_archive_object = "src/${data.archive_file.function_src.output_base64sha256}.zip"
}


~~

resource "google_storage_bucket_object" "archive" {
  name       = "src/${data.archive_file.function_src.output_base64sha256}.zip"
  bucket     = "${var.bucket_name}"
  source     = "function_src.zip"
  depends_on = ["data.archive_file.function_src"]
}
```

### node_modulesを同梱しない
CloudFunctionsでjavascriptの関数を作る場合、node_modulesを同梱しないでもpackage.jsonがあれば向こうでなんとかしてくれます。
この辺Lambdaとの違いですね。


## 7. ローカルからCircleCIを起動！
適切に環境変数をセットしたのち、CircleCIをローカルから起動して設定をチェックします。プロジェクト名、バケット名は自分で指定して下さい。

```
circleci config validate

export GCLOUD_SERVICE_KEY=$(cat ./key.json)

circleci local execute --job test \
    -e GCLOUD_SERVICE_KEY="$GCLOUD_SERVICE_KEY" \
    -e TF_VAR_project_id="${PROJECT_ID}" \
    -e TF_VAR_bucket_name="${BUCKET_NAME}"

circleci local execute --job build \
    -e GCLOUD_SERVICE_KEY="$GCLOUD_SERVICE_KEY" \
    -e TF_VAR_project_id="${PROJECT_ID}" \
    -e TF_VAR_bucket_name="${BUCKET_NAME}"
```

### 8. リモートのCircleCIのcontextを指定
必要なcontextを設定します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/84369dd6-58b0-9a9c-af9e-b456edc517c9.png)

### 9. コンソールから実行
取得するrssとwebhookの設定をまとめたjsonファイルをCloudStorageにアップロードし、CloudFunctionsを起動します。
webhookアドレスを公開すると私のbotが暴れまわってしまうのでこのような構成にしました。職場のリポジトリとかでやる場合は普通にコミットしていいと思います。

### 設定ファイルのアップロード
以下のようなファイルを作成します。

```rss_webhooks.json
[
    {
        "label": "クックパッド開発者ブログ",
        "rss": "http://techlife.cookpad.com/rss",
        "webhook": "https://discordapp.com/api/webhooks/***/***"
    },
    {
        "label": "hnapp",
        "rss": "http://hnapp.com/rss?q=score%3E100",
        "webhook": "https://discordapp.com/api/webhooks/***/***"
    }
]
```

その後、CloudStorageにアップロードします。

```
gsutil cp -r rss_webhooks.json gs://${BUCKET_NAME}/
```

### 実行
ここまでの準備ができたら、Cloud Schedulerから`今すぐ実行`を押してサービスを起動します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/24ac3eb7-7eab-8452-221d-66e17878dfc8.png)

呼び出しが発生していればOKです。お疲れ様でした！
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/96286/7ad27719-c2c7-d959-ef24-5c0e8d1efb59.png)

# まとめ
TerraformでGCPのリソースを作成し、それをCircleCIでデプロイできる環境を整えました。
やって思ったのは、素直にDeployment Manager使えばよかった...普段AWSを利用しているので、GCPを単体で使うことはないと判断してTerraformに挑戦してみましたが、なにぶん外部サービスなのでクレデンシャルの扱いに苦労しました。
ただ、CloudFormationのYAMLよりは読みやすいですね！


# 今後の課題
* CircleCIへのサービスアカウントのクレデンシャルの渡し方は、本当にcontext経由でいいのでしょうか？
https://circleci.com/docs/2.0/google-auth/
* terraformの実行時、`-lock=false`にしないと動かなかったのが悔しいです。
* tfファイル内でバケット名を変数にする方法が分かりませんでした。
* TerraformをCIで回す場合（このシチュエーションがそもそもまれ？）、planをtestとして実行するのは一般的なのか。私は理にかなっていると思いますが。

