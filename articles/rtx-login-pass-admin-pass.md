---
title: "RTX1210でログインパスワードと管理パスワードに同じものを使ったら"
emoji: "🐘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rtx", "rtx1210"]
published: false
---

## TL;DR

- RTX1210では、ダッシュボードにBasic認証が設定されている
- ログインパスワード（一般ユーザーのパスワード）を設定すれば一般ユーザーとして、管理パスワードを入力すれば管理ユーザーとしてログインされる
- ログインパスワードと管理パスワードに同じものを使うことは可能
- 同じパスワードが設定されている場合、Basic認証でパスワードを入力すると としてログインする

## 詳細

RTX1210では、ダッシュボードにログインする際にパスワードが要求されます。
このとき、一般ユーザーと管理ユーザーそれぞれにユーザー名が設定されておらず、かつパスワードが同じだとどうなるかを試します。

### 同じパスワードを設定

RTX1210にtelnetでログインし、、管理ユーザーに成ってからログインユーザーのパスワードを変更します。

```telnet
% telnet 192.168.100.1 23
Trying 192.168.100.1...
Connected to 192.168.100.1.
Escape character is '^]'.

Password: 

RTX1210 Rev.14.01.40 (Fri Apr 16 09:27:57 2021)
Copyright (c) 1994-2021 Yamaha Corporation. All Rights Reserved.
To display the software copyright statement, use 'show copyright' command.
(省略)
Memory 256Mbytes, 3LAN, 1BRI
> administrator 
Password: 
# login password 
Old_Password: 
New_Password: 
New_Password: 
Password Strength : Very strong
```

まず、ログインユーザーと管理ユーザーで同じパスワードを設定するのは可能なんですね。

## ダッシュボードにログイン

Basic認証でパスワードを入力すると、

![RTX1210 Basic認証](images/2022-03-20-09-15-03.png)

管理ユーザーとしてログインしていました。

![RTX1210 ダッシュボード（管理ユーザーとしてログイン）](/images/2022-03-20-09-08-42.png)

## まとめ

- 管理ユーザーとしてのログインが優先されるようです。
- この後パスワードを変えて、一般ユーザーにはユーザー名を設定しました。
