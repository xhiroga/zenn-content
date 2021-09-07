---
title: "API GatewayのHTTPプロキシ統合のURL・リクエストヘッダーを眺める"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","APIGateway"]
published: true
---
API GatewayのHTTPプロキシ統合の設定ごとのリクエストヘッダー・URLの値を眺めます。

# パターン
1. リソースパスが `/` (HTTP統合)
3. リソースパスが `/{myProxy+}`
4. リソースパスが `/{myProxy+}` & CORSを有効に設定


# 準備
API Gatewayの裏側はVPC_LINK → NLB　→ Flaskのアプリ@ECSの構成です。  
Flaskのアプリは[こちら](https://github.com/hiroga-cc/flask-repeat-after-me)。リクエストヘッダーを返すだけの簡単な構成です。  


# 1. リソースパスが `/` (HTTP統合)
![](https://i.imgur.com/xc892Ip.png)

```
http://example.com/
X-Amzn-Apigateway-Api-Id: fc5vwzacp2
X-Amzn-Trace-Id: Root=1-5c5fb0f2-843387cadeadbf4c9d3c9036
User-Agent: AmazonAPIGateway_fc5vwzacp2
Accept: application/json
Host: example.com
Connection: Keep-Alive
```

次に登場するHTTPプロキシ統合よりヘッダーの要素数が少ないです。


ちなみに `/`しか指定していない状態で `/hoge` にアクセスすると、↓のようなエラーで失敗します。`/`≠ワイルドカードなんですね。  
`{"message":"Missing Authentication Token"}`

# 2. リソースパスが `{myProxy+}`
![](https://i.imgur.com/Ji3zEX9.png)

```
http://example.com/hoge
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate, br
Accept-Language: en,en-US;q=0.9,ja;q=0.8
Cache-Control: max-age=0
Cookie: s_pers=%20s_vnum%3D1546597042049%2526vn%253D1%7C1546597042049%3B%20s_invisit%3Dtrue%7C1544007119025%3B%20s_nr%3D1544005319028-Repeat%7C1551781319028%3B
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36
X-Amzn-Trace-Id: Root=1-5c5fb581-c2979900f0d0ce0267df3340
X-Forwarded-For: 114.190.145.219
X-Forwarded-Port: 443
X-Forwarded-Proto: https
X-Amzn-Apigateway-Api-Id: fc5vwzacp2
Host: example.com
Connection: Keep-Alive
```

EndpoinrtURLの値が興味深いです。API Gatewayのリソースの階層構造とは無関係に、メソッドの統合リクエストの設定時に指定したURLに対してリクエストがなされなます。（他の階層構造でも試しました）  


# 3. リソースパスが `/{myProxy+}` & CORSを有効に設定
リクエストヘッダーは特に変わらないみたいですね。

```
http://exmaple.com/hoge
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate, br
Accept-Language: en,en-US;q=0.9,ja;q=0.8
Cookie: s_pers=%20s_vnum%3D1546597042049%2526vn%253D1%7C1546597042049%3B%20s_invisit%3Dtrue%7C1544007119025%3B%20s_nr%3D1544005319028-Repeat%7C1551781319028%3B
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36
X-Amzn-Trace-Id: Root=1-5c5fc5bd-ffb8781faee31572026e8616
X-Forwarded-For: 114.190.145.219
X-Forwarded-Port: 443
X-Forwarded-Proto: https
X-Amzn-Apigateway-Api-Id: mtwqhaxbhe
Host: exmaple.com
Connection: Keep-Alive
```

# おまけ
## EndpointURLの置換文字列は必ずしもリソース名と同じでなくて良い
Endpoint URLの置換文字列はパスパラメータからマッピングされたリクエストパラメータに過ぎないので、別途リクエストパラメータをマッピングすれば置換文字列の値を変えることが可能。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/96286/49494f15-ed94-74f6-ac44-847c02a6d745.png)



## リソースの設定されていないパスにアクセス
`{"message":"Missing Authentication Token"}`  
リソースの設定されていないパスにアクセスするとこうなります。  

## 貪欲パスのフォーマット
![](https://i.imgur.com/gNHJ5MB.png)
{}のなかに文字列と+マークで指定したリソースを`Greedy Path`というそうです。
HTTPプロキシ統合にも関わらず指定しない場合、エラーが発生。  
日本語だと貪欲パス、ってところでしょうか。  

## 貪欲パスの重複制限
![](https://i.imgur.com/5ae3G5R.png)
貪欲パスは２個以上持てないみたいです。それはそうか。  


# 参考
https://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/api-gateway-set-up-simple-proxy.html

