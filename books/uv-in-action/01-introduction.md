---
title: "はじめに"
references:
- https://chatgpt.com/c/68ea0bdb-f878-8320-aeed-24ca730a9428    # npmやRustについて
- https://discuss.python.org/t/installing-multiple-versions-of-a-package/4862
---

（執筆中です...）

## pip install にはどんなリスクがあるか？


:::message そもそもPythonは、どうしてパッケージごとに異なるバージョンの依存関係を持たせてあげないの？

よく分かります。実は`JavaScript`のパッケージ管理ツールの`npm`では、各モジュール（パッケージに相当）独自の依存関係を持つことができます。よく`Dependency Hell`と呼ばれるように、パッケージの数が膨大になる理由でもあります。

また、Rustも同一クレート（パッケージに相当）の複数のバージョンを同時に利用することができます。Rustの場合は、バージョン名まで含めて宣言をします。

```toml
[dependencies]
serde_v1 = { package = "serde", version = "1" }
serde_v2 = { package = "serde", version = "2" }
```

```rs
use serde_v1 as serde1;
use serde_v2 as serde2;

fn main() {
    println!("serde v1 version: {}", serde1::VERSION);
    println!("serde v2 version: {}", serde2::VERSION);
}
```

一方、Pythonは同じライブラリの異なるバージョンを同時にインストールすることができません。構文の変更をはじめとした非常に大規模な変更が必要になるため、今日に至るまで実現していないのですね。

:::

