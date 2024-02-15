---
title: "【Laravel】Eloquentモデルからドメインモデルへの変換"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [DDD, 設計]
published: false
---
# これは何？
LaravelでDDDの設計パターンで実装する際に、Eloquentモデルからドメインモデルへの変換方法で試行錯誤した結果の自分なりの結論を述べます。

# 結論
- Eloquentモデルはそのままドメインモデルにはしない
- Eloquentモデルにドメインモデルへの変換メソッドを作っておく
# 前提:Eloquentモデルはドメインモデルとはしない
まず前提として、DDDで使われるドメインモデルにLaravelのEloquentモデルを使用しないようにします。
理由としては以下があります。
- 