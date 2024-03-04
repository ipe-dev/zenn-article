---
title: "【Go】Functional Option Patternによるコンストラクタの実装"
emoji: "👌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Go]
published: false
---
# これは何
GoにはFunctional Option Patternという、structを生成するパターンがあります。
そのパターンについて書いた記事です。
# 何が解決できるか
- structを生成するときのパラメータを可変にできる
# 結論
- 作りたいstructにパラメータを付与するような関数を用意する
- 呼び出す側はその関数に引数を渡してパラメータをセットする
- 必要な時だけ関数を呼び出せばいいので、structにセットするパラメータを可変にできる
# 具体例
## 構造体を準備
以下のような構造体があったとします
```go:user.go
    type User struct {
        Name: string
        Age: int
    }
```
この構造体を返すような`Newメソッド`を用意します
```go
func NewUser(name string, age int) User {
    return User {
        Name: name
    }
}
```
# まとめ