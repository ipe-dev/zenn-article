---
title: "【Laravel】Eloquentモデルからエンティティへの変換"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [DDD, 設計, Laravel]
published: false
---
# これは何？
LaravelでDDDの設計パターンで実装する際に、Eloquentモデルからドメインモデルへの詰め替えをどうしようか悩みました。
試行錯誤した結果の自分なりの結論を述べます。
同じように悩んでいる方に参考にしてもらえれば嬉しいです。

# 結論
- Eloquentモデルにエンティティへの変換メソッドを作っておく
- Eloquentモデルはそのままドメインモデルにはしない
# Eloquentモデルからエンティティへの変換メソッド
## エンティティの実装
```php
final class User
{
    public function __construct(
        public readonly $id,
        public readonly string $name,
        public readonly int $age,
    ) {}
}
```
だいぶドメイン貧血症ですがご了承ください。
## Eloquentモデルの実装

# Eloquentモデルはドメインモデルとはしない
まず前提として、DDDで使われるドメインモデルにLaravelのEloquentモデルを使用しないようにします。
理由としては以下があります。
- 