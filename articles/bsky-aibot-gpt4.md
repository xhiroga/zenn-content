---
title: "AIチャットボット(GPT-4)をBlueskyで公開しました！"
emoji: "💙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["bluesky","atproto", "chatgpt", "openai"]
published: true
---

## TL;DR

- メンションとリプライに返信してくれるBot
- スレッドの文脈も考慮
- **[@aibot.bsky.social](https://bsky.app/profile/hiroga.bsky.social) をフォローすれば誰でも利用OK!**

## 動機

私はChatGPTのヘビーユーザーを自認しています。仕事の開発・日常気になったことの質問など、1日体感30分~1時間は利用しています。  
興味深い話を聞いたとき、それをTwitterの友達に毎回シェアするのは手間だなと思いました。  
そこで、初めからSNS上でChatGPTと会話できるようにしました。

**利用例**

![](/images/2023-07-04-8-11-27.png)

## 設計

ざっくり言えば、ポスト（ツイートに相当）の取得とリプライが必要です。

Blueskyではポストの取得方法が2つあります。XRPCとEvent Streamです。  
XRPCは一般的なHTTPSのリクエストで、GraphQLのように独自の文法があるというだけなので、親しみやすいです。  
Event Streamは、AWSユーザーならKinessis Firehorseといえば伝わるでしょうか。Websocketで接続して最新のポスト・Like・Unlike・Repost・etc...が爆速で流れ込んできます。これ一般ユーザーが無料で使えるの...!?

スクショの通り、Event Streamだとすごい勢いで投稿が流れてきます。  
![](/images/2023-07-03-8-27-00.png)

まずはXRPCで取得することにしました。

## 実装

リポジトリはこちらです。

https://github.com/xhiroga/bsky-aibot

一部抜粋すると次の通りで、通知を取得し、投稿ごとにメンション or 自分の投稿へのリプライを判定し、TrueならChatGPTが返信します。

```python
def read_notifications_and_reply(client: Client, last_seen_at: datetime = None) -> datetime:
    logging.info(f"last_seen_at: {last_seen_at}")
    did = client.me.did

    # unread countで判断するアプローチは、たまたまbsky.appで既読をつけてしまった場合に弱い
    ns = get_notifications(client)
    seen_at = datetime.now(tz=timezone.utc)
    ns = filter_mentions_and_replies_from_notifications(ns)
    if last_seen_at is not None:
        ns = filter_unread_notifications(ns, last_seen_at)

    for notification in ns:
        thread = get_thread(client, notification.uri)
        if is_already_replied_to(thread, did):
            logging.info(f"Already replied to {notification.uri}")
            continue

        post_messages = thread_to_messages(thread, did)
        reply = generate_reply(post_messages)
        client.send_post(text=f"{reply}", reply_to=reply_to(notification))

    update_seen(client)
    return seen_at
```

プロンプトはかなりシンプルです。ただし、300文字を超えるとBlueskyに投稿できなくなるので、縮めてもらっています。  
また、メンションにはメンションで返したくなってしまうらしいので、その点も注意してもらっています。

```python
def generate_reply(post_messages: t.List[OpenAIMessage]):
    # <https://platform.openai.com/docs/api-reference/chat/create>
    messages = [{"role": "system", "content": "Reply friendly in 280 characters or less. No @mentions."}]
    messages.extend(post_messages)
    chat_completion = openai.ChatCompletion.create(
        model="gpt-4",
        messages=messages,
    )
    first = chat_completion.choices[0]
    return first.message.content
```

なお、現在は私の自宅のUbuntu Desktopで動かしています。PollingでもWebsocketでも、サーバーを公開しなくていいのは楽でいいすね。
　
## 感想

まず、文脈を考慮するだけでかなり自然に使えます。  
例えば具体例を知りたい場合、次のように使えます。

![](/images/2023-07-04-8-10-41.png)

ChatGPTに比べて長文が返せないのが今のところネックですが、スレッドを使って返せるようにしたいところ。  
また、Botが返信をくれるまでアーキテクチャ上30秒くらいの遅延があるのですが、Botと2つの会話を並列して、その間別の返信をしていれば気にならないことも分かりました。

なかなか面白い体験なので、是非みなさん使ってみてください！
なお、API利用料のハードリミットは設定済みです💰笑
