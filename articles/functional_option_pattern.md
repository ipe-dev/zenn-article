---
title: "【Go】Functional Option Patternによる実装"
emoji: "👌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Golang]
published: false
---
# これは何
GoにはFunctional Option Patternという、structを生成するパターンがあります。
その実装方法について書いた記事です。
# 何が解決できるか
- structを生成するときのパラメータを可変にできる
# 結論
- 作りたいstructにパラメータを付与するような関数を用意する
- 呼び出す側はその関数に引数を渡してパラメータをセットする
- 必要な時だけ関数を呼び出せばいいので、structにセットするパラメータを可変にできる
# 具体例
## 元になる構造体
以下のような構造体があったとします。
この構造体を返すような`Newメソッド`を用意します。
```go:user.go
type User struct {
    name: string
    age: int
}

func New(name string, age int) *User {
	return &User{
		name: name,
		age: age,
	}
}
```
## 特定のパラメータをオプションで渡す
このUserに対して、ペットを飼っている人だけペットの名前を渡したいとします。
```go:user.go
type User struct {
	name string
	age int
	petName string // ペットを飼っている人だけ追加したい
}
```
しかし、Goは`func New(name string, age int, petName=nil)`のように引数のデフォルト値を設定することができません。
また、Newで作成するタイミングで`petName`を指定し、後から変更は出来ないようにしたいです。
そんな時に使うのが`Functional Option Pattern`です。
## Functional Option Pattern
### パラメータをセットできる関数を作る
```go:user.go
func WithPetName(petName string) func(*User) {
	return func(u *User) {
		u.petName = petName 
	}
}
```
引数に`*User`を取り、パラメータをセットする関数を返しています。

# まとめ