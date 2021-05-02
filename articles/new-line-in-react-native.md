---
title: "React Nativeで改行したい！やり方3選"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react-native"]
published: true
---
# やり方3種類
1. テンプレートリテラルを用いる
2. \nを{}で囲って用いる
3. <Text>コンポーネントを分ける

---
### 1. テンプレートリテラルを用いる
```
<Text>
{`
  1行目
  2行目
`}
</Text>
```

### 2. \nを{}で囲って用いる
```
<Text>1行目{"\n"}２行目</Text>
```


### 3. <Text>コンポーネントを分ける

```
<Text>1行目</Text>
<Text>2行目</Text>
```



### 実践編
```
import Text from 'react-native'; 

class Aiueo extends Component {

  render() {
    text = "あい  うえ  お"

    return(
      <Text>
        { text.replace(/\s\s/g, '\n') }
      </Text>
    )
  }
}
```
```
あい
うえ
お
```


---
他のやり方をご存知の方、コメントお待ちしております〜！

