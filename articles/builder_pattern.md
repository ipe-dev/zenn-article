---
title: "【Go】Builderパターンで実装してみた"
emoji: "👷🏼"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Golang]
published: false
---
# これは何
[ 前回の記事 ](https://zenn.dev/ippe1/articles/functional_option_pattern)ではGoのFunctional Option Patternについて書きました。
今回はそれと対比される実装パターンである、**Builderパターン**について書きたいと思います。
# この記事のゴール
- builderパターンによる実装方法を理解できること
# 結論
- Builderとなる構造体にメソッドを実装する
- そのメソッドを通して元になる構造体にパラメータをセットする
- Builderを呼び出す側はセットするかしないかを自由に選択できる
# Builder Patternの具体例
以下、Builder Patternの実装例です。
## UserとUserParam
以下のように`User`と`UserParam`を用意します。
UserParamという構造体を介して、Userにパラメータをセットしていくことになります。
また、petName,height,weightは任意でつけるようなパラメータだと想定します。
```go:user.go
type User struct {
	param UserParam
}
type UserParam struct {
	name    string
	age     int
	petName string // 任意のパラメータ
	height  int // 任意のパラメータ
	weight  int // 任意のパラメータ
}
```
## Builderを作成する関数
次に、`UserParam`を初期化する関数を実装します。

```go:user.go
func NewBuilder(name string, age int) *UserParam {
	return &UserParam{
		name: name,
		age:  age,
	}
}
```
## UserParamにメソッドを実装
`UserParam`に対してパラメータをセットして、自身を返すメソッドを実装します。
```go:user.go
func (up *UserParam) PetName(petName string) *UserParam {
	up.petName = petName
	return up
}

func (up *UserParam) Height(height int) *UserParam {
	up.height = height
	return up
}

func (up *UserParam) Weight(weight int) *UserParam {
	up.weight = weight
	return up
}
```
# 最後に