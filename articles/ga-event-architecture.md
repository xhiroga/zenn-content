---
title: "Universal Analytics・GA4の両方を考慮したイベントの設計・モデリング"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Google Analytics","BigQuery"]
published: true
---

Google Analyticsには、Universal Analytics(UA, GA3)とGA4の2つがあります。

2021年現在、以下の理由から両方を使うのが良いと考えています。

- 利用者やエコシステムがGA4に対応しきれていない（RedashはGA3のみ対応）
- GA4に接続することでBigQueryにイベント情報を送信できる

※その他分析に関する違いがあると思いますが、当方詳しくないので省略します。


## TL;DR

- GTMからUA/GA4のタグを起動する
- UAの event_category, event_action, event_label, event_value は、全てGA4のプロパティに充てる
- GA4のために event_name を設ける

## Google Tag Manager(GTM)の利用

BizDevメンバーが利用できるように、分析系のツールはGTMに寄せる方針で設計しています。

## イベントのモデリング

UAとGA4の両方に対応したモデリングで、注意すべきことは以下の5点と考えます。

1. UAとGA4のイベントの違い
2. 公式のヘルプ
3. ユニークイベント
4. GA4で自動的に収集されるイベント・拡張計測機能
5. BigQueryでのクエリの書きやすさ


### 1. UAとGA4のイベントの違い

UAとGA4ではイベントの形式・型が異なります。

UAではイベントに対して「イベントカテゴリ」「イベントアクション」「イベントラベル」「値」の4つの属性があるのに対して、
GA4では「イベント名」と「イベントパラメーター」（パラメーター名と値のペア）だけが存在します。

型にも違いがあり、UAの値は数字だけを扱える一方で、GA4のイベントパラメーターの値は文字列を扱うことができます。

ちなみに、イベントの送信数は[UA](https://developers.google.com/analytics/devguides/collection/analyticsjs/limits-quotas)/[GA4](https://support.google.com/analytics/answer/9267744)ともに1セッションあたり500件のようです。

### 2. 公式のヘルプ

UA・GA4のそれぞれに、イベントの設計に関するヘルプがあります。  
10分くらいで通読できるので、ご参照ください。

- [UA イベントについて](https://support.google.com/analytics/answer/1033068?hl=ja&ref_topic=1033067#See)
- [GA4 イベントについて](https://support.google.com/analytics/answer/9322688?hl=ja&ref_topic=9756175)

### 3. ユニークイベント

あるイベントが達成されたセッションが全体のセッションの何割か？という分析がしたいなら、そのユーザーの行動は「イベントアクション」に指定すべきです。

UAではレポートで、1セッションの中の同じイベントをまとめて集計することができます。  
例えば、セッションAは商品を2つ買い、セッションBは1つ、セッションCは0、とした場合、購入のあったセッション = 66.6%、と算出できます。  

そのためには、購入イベントの「イベントアクション」に「購入」を設定する必要があります。
ex. カテゴリ: 家電, アクション: 購入, ラベル: 扇風機

逆に、「イベントカテゴリ」に「購入」を設定してしまうと、全てのセッションでカテゴリが購入のイベントがいくつあったか？という計測しかできないようです。  
（迂回策があるかもしれませんが、詳しい方はぜひ教えて下さい）

### 4. GA4で自動的に収集されるイベント・拡張計測機能

GA4では、一部のイベントが自動的に収集されたり、拡張計測機能を有効にすることで収集されたりします。  
（ちなみに、これらのイベントはBigQueryにも送信されるようです）

具体的には、 `first_visit`, `session_start`, `scroll` などのイベントが記録されています。

GA4では Engagement > Event からイベント数を比較したグラフが見えるので、そのビューを活用するためにも自動で記録されるイベントと足並みを揃えたほうが良さそうです。  
例えば `session_start` が月間10,000件に対し、独自定義した `submit` が 月間5,000件だとしたら、だいたい半数の人がフォームなどの提出をしていることが分かりますね。


### 5. BigQueryでのクエリの書きやすさ

BigQueryでSQLを書く際に分かりやすくするため、イベントプロパティのキー名は固定のほうが良さそうです。


### 上記を踏まえたモデリング

上記を踏まえ、以下のようなモデリングを推奨します。エンジニア向けに分かりやすくするためJSONで書いています。  
Webサイトはオンライン購入にも対応している小売店などを想像してください。

GTMに送信するイベント

```ts
const gtm_event = {
    event_name: 'purchase__t_shirts',
    action: 'purchase',
    category: 'online_shop',
    label: 't_shirts',
    value: 1500,
    other_prop: 'some_value'
}
```

GTMのイベントとUniversal Analyticsの対応関係

```ts
const ua_event = {
    "EventCategory": gtm_event.category
    "EventAction": gtm_event.action
    "EventLabel": gtm_event.label
    "EventValue": gtm_event.value
}
```

GTMのイベントとGA4の対応関係

```ts
const ga4_event = {
    "EventName": gtm_event.event_name,
    "EventProperties": [
        {
            "Name": 'event_category',
            "Value": gtm_event.category
        },
        {
            "Name": 'event_action',
            "Value": gtm_event.action
        },
        {
            "Name": 'event_label',
            "Value": gtm_event.label
        },
        {
            "Name": 'event_value',
            "Value": gtm_event.value
        },
        {
            "Name": 'other_props',
            "Value": gtm_event.other_props
        }
    ]
}
```

## その他

GA4ではイベントに多くの情報を投入できますが、個人情報は絶対に投入しないように気をつけてください。

## まとめ

GTM経由でUA/GA4の両方にデータを送信しています。より良いモデリングがあれば是非アドバイスください！
