---
title: "【Go】Functional Option Patternで引数を可変にしてみた"
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
以下のような構造体と構造体を返す`New`を用意します。
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
引数に`*User`を取り、パラメータをセットする無名関数を返しています。
これにより、`WithPetName`実行時にUserにPetNameがセットされます。
### New関数の引数に可変のoptionを追加する
```go:user.go
type User struct {
	name string
	age int
	petName string
}

type Option func(*User)

func New(name string, age int, options ...Option) *User {
	user := &User{
		name: name,
		age: age,
	}
	for _, option := range options {
		option(user) // ここでoptionで指定した値をuserにセット
	}
	return user
}
```
上記のように`Option型`を用意し、Newの引数として可変で渡します。
Newの中で`option`を実行し、userに値をセットしていきます。
次にこのNewを呼び出します。
```go:main.go
func main() {
	user := user.New("taro", 20, user.WithPetName("tom"))
	fmt.Println(*user) // {taro 20 tom}
}
```
Newの最後に`WithPetName`をつけることでペットの名前をセットできました。
もちろん、PetNameが不要な場合は渡さないことも可能です。
```go:main.go
func main() {
	user := user.New("jiro", 19)
	fmt.Println(*user) // {jiro 19}
}
```
# まとめ