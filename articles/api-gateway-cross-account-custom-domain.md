---
title: "API Gatewayに別アカウントで所有しているドメインを使ってカスタムドメイン名を設定する"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","APIGateway"]
published: true
---
# Steps
1. 別アカウントで所有しているドメインを利用した証明書の取得
2. 証明書をセットしたカスタムドメイン名の作成
3. カスタムドメイン名をRoute53のレコードに設定

# 1. 別アカウントで所有しているドメインを利用した証明書の取得
証明書の発行自体はACMからすぐにできましたが、ドメインを持っていること自体の証明が必要でした。  
DNS証明という方法があって、これから証明書を取得するドメイン名のサブドメインに対して、自分が指定したドメイン名（=証明書を申請した側のAWSアカウントで作成された、`******.acm-validations.aws/`のようなアドレス）　の組み合わせを作成し、それをドメインを所有している側のアカウントでレコードに登録することで証明しています。  

![](https://i.imgur.com/VCKkX3y.png)


参考:
 - [AWS Certificate Manager で証明書の準備をする](https://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/how-to-custom-domains-prerequisites.html)
 - [DNS を使用したドメインの所有権の検証](
https://docs.aws.amazon.com/ja_jp/acm/latest/userguide/gs-acm-validate-dns.html)

# 2. 証明書をセットしたカスタムドメイン名の作成
API Gatewayの　Custom Domain Nameから、こんな感じでドメイン名を設定します。  
![](https://i.imgur.com/7TtNTXu.png)


なお、ドメイン名に関する用語が紛らわしいので整理すると...  
"デフォルトのベースURL": `https://api-id.execute-api.region.amazonaws.com/stage`  
"リージョンのドメイン名"(またはターゲットドメイン名）: `https://api-id.execute-api.region.amazonaws.com`
"カスタムドメイン名": `https://api.example.com/myservice`  

ガイドにも書いてありますが、リージョンAPIのカスタムドメイン名を作成する場合にAPI Gatewayから返ってくるのは"リージョンのドメイン名"なんですね。これをDNSレコードに設定することで、カスタムドメイン名を使うための手続きが完了します。  

参考: [API Gateway で API のカスタムドメイン名を設定する](https://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/how-to-custom-domains.html)


# 3. カスタムドメイン名をRoute53のレコードに設定
ドキュメント [API Gateway でリージョン別 API 用のカスタムドメイン名をセットアップする](https://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/apigateway-regional-api-custom-domain-create.html)ではCLIのコマンドが示されている箇所です。  
![](https://i.imgur.com/3EXk1ln.png)

Aliasとして設定します。

# その他
* 疎通確認の際、 `http` でアクセスしないように気をつけてください。Chromeでドメインだけ（`test.example.com`）打ち込むと`http` として解釈されます。  


