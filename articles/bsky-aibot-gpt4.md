---
title: "フォローだけで誰でも使えるGPT-4のチャットボットを、Blueskyで開発しました"
emoji: "💙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["bluesky","atproto", "chatgpt", "openai"]
published: true
---

## TL;DR

- フォローするとフォロバして、タイムライン上のメンションとリプライに返信してくれるGPT-4
- スレッドの文脈も考慮
- 未来的な体験で面白いので、一般公開します（API利用料のハードリミットは設定済み💰）
- [@aibot.bsky.social](https://bsky.app/profile/hiroga.bsky.social) をフォローすれば誰でも利用OK!

![](/images/2023-07-03-8-18-28.png)

## 動機

私はChatGPTのヘビーユーザーを自認しています。仕事の開発・日常気になったことの質問など、1日体感30分~1時間は利用しています。  
興味深い話を聞いたとき、それをTwitterの友達に毎回シェアするのは手間だなと思いました。  
そこで、初めからSNS上でChatGPTと会話できるようにしました。

## 設計

ざっくり言えば、ポスト（ツイートに相当）の取得とリプライが必要です。

Blueskyではポストの取得方法が2つあります。XRPCとEvent Streamです。  
XRPCは一般的なHTTPSのリクエストで、GraphQLのように独自の文法があるというだけなので、親しみやすいです。  
Event Streamは、AWSユーザーならKinessis Firehorseといえば伝わるでしょうか。Websocketで接続して最新のポスト・Like・Unlike・Repost・etc...が爆速で流れ込んできます。これ一般ユーザーが無料で使えるの...!?

Event Streamだと処理する投稿の量が多すぎるので、まずはXRPCを使うことにしました。スクショの通り、すごい勢いで投稿が流れてきます。

![](/images/2023-07-03-8-27-00.png)

## 実装

リポジトリはこちらです。

https://github.com/xhiroga/bsky-aibot

一部抜粋すると次の通りで、タイムラインを取得し、投稿ごとにメンション or 自分の投稿へのリプライを判定し、TrueならChatGPTが返信します。  
ただ、通知から取得したほうが早いかもしれません。この辺は改善ポイントです。

```python
    timeline = get_timeline(client)
    # should check with latest reply, not last replied datetime
    last_replied_datetime = get_last_replied_datetime()
    new_feed = filter_and_sort_timeline(timeline.feed, last_replied_datetime)

    for feed_view in new_feed:
        mentioned = does_post_have_mention(feed_view.post, profile.did)
        reply_to_me = is_reply_to_me(feed_view, profile.did)
        if mentioned or reply_to_me:
            if reply_to_me: # TODO: スレッド内で急にメンションされたときに対応できないので後で直す
                thread = get_thread(client, feed_view.post.uri)
                post_messages = thread_to_messages(thread, profile.did)
            else:
                post_messages = posts_to_sorted_messages([feed_view.post], profile.did)
            reply = generate_reply(post_messages)
            client.send_post(text=f"{reply}", reply_to=reply_to(feed_view.post))
            update_last_replied_datetime(feed_view.post.record.createdAt)
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

![](/images/2023-07-03-8-34-04.png)

ChatGPTに比べて長文が返せないのが今のところネックですが、スレッドを使って返せるようにしたいところ。  
また、Botが返信をくれるまでアーキテクチャ上30秒くらいの遅延があるのですが、Botと2つの会話を並列して、その間別の返信をしていれば気にならないことも分かりました。

なかなか面白い体験なので、是非みなさん使ってみてください！
