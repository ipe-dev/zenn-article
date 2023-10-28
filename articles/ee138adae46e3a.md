---
title: "【Linux】環境変数とシェル変数の違い"
emoji: "🐧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [linux]
published: true
---
# 結論
- 環境変数はシェル全体の設定で別のシェルを起動しても引き継がれる変数
- シェル変数は別のシェルを起動しても引き継がれない変数

つまり影響するスコープが違うため、
- 環境変数　→ グローバル変数
- シェル変数　→ ローカル変数

という感じに捉えてます。
# 環境変数
## 設定方法
試しに`HOGE`という環境変数を設定します。
```bash
export HOGE="hoge"
```
`export`をつけることによって環境変数にセットすることができます。
`=`の左右にスペースを入れないように注意です。
## 確認方法
### 現在のシェルで確認
`env`コマンドで環境変数を確認できます。
今回は`HOGE`のみを出力してみます。
```bash
env | grep HOGE
```
`HOGE=hoge`と出力されます。
### 別のシェルを開いてから確認
次に、別のシェルを起動してから再確認します
```bash
bash
env | grep HOGE
```
これでも`HOGE=hoge`と出力されます。
# シェル変数
設定前に先ほど起動したbashを`exit`で終了します。
## 設定方法
シェル変数として`FUGA`という変数を設定します。
```bash
FUGA='fuga'
```
## 確認方法
### 現在のシェルで確認
シェル変数を確認するには`set`コマンドを使います。
```bash
set | grep FUGA
```
`FUGA=fuga`と出力されます。
ちなみに先ほど設定した`HOGE`もsetで確認できます。
```bash
set | grep -e FUGA -e HOGE
```
`HOGE=hoge`と`FUGA=fuga`が両方出力されます。
### 別のシェルを開いてから確認
次に、別のシェルを起動させます。
```bash
bash
```
`ps -fx`で確認すると以下のようになってます。
```bash
 PID TTY      STAT   TIME COMMAND
  549 pts/1    Ss     0:00 bash
 3362 pts/1    S      0:00  \_ bash
 3365 pts/1    R+     0:00      \_ ps -fx
    1 pts/0    Ss+    0:00 /bin/bash
```
この状態でシェル変数を確認します。
```bash
set | grep FUGA
```
今度は何も出力されません。
次に以下のコマンドを実行します。
```bash
set | grep -e FUGA -e HOGE
```
すると環境変数である`HOGE=hoge`のみが出力されます。
`exit`してから再度確認すると`FUGA=fuga`が出力されます。
```bash
exit
set | grep FUGA
```
以上のように、環境変数は別のシェルを開いても引き継がれ、シェル変数は現在のシェルでしか使用できない変数になります。