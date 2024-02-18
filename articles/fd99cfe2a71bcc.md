---
title: "【DDD】Eloquentモデルからエンティティへの変換【Laravel】"
emoji: "🏭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [DDD,ドメイン駆動設計, 設計, Laravel]
published: true
---
# これは何？
LaravelでDDDの設計パターンで実装する際に、Eloquentモデルからドメインモデルへの詰め替えをどうしようか悩みました。
試行錯誤した結果の自分なりの結論を述べます。
同じように悩んでいる方に参考にしてもらえれば嬉しいです。
# 結論
- Eloquentモデルにエンティティへの変換メソッドを作っておく
- Eloquentモデルはそのままドメインモデルにはしない
# 前提
RepositoryクラスでLaravelのEloquentを使ってDBからUserを取得し、エンティティへ変換するアーキテクチャを想定しています。
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
以下はEloquentモデルのUserクラスになります。
```php
final class User extends Model
{
    public function createUserDomain(): UserDomain
    {
        return new UserDomain(
            id: $this->id,
            name: $this->name,
            age: $this->age,
        );
    }
}
```
ここではDBから取得して生成したEloquentモデル内の値をドメインに移し替える処理を実装しました。
こうすることで、移し替えが発生する箇所で上記のメソッドを呼び、コードの重複を防いでいます。
# Repositoryでの変換メソッドの使用
データを取り出すRepositoryクラスでドメインクラスへの変換メソッドを使います。
```php
final class UserRepository Implements UserRepositoryInterface
{
    public function getById(UserId $userId): ?User
    {
        $user = User::find($userId->value);
        if (empty($user)) {
            return null;
        }

        return $user->createUserDomain();
    }
}
```
上記のように得られたUserモデルで`createUserDomain`を実行し、エンティティを返します。
めちゃくちゃ単純な例でしたが、このようにしてEloquentモデルからEntityへ変換を行いました。
# Eloquentモデルはエンティティとはしない
今回のように、DDDで使われるエンティティにLaravelのEloquentモデルを使用しない方がいいと個人的に考えています。
理由としては以下です。
- エンティティがフレームワークに依存してしまう
- データ構造がテーブルと同じになる 
## エンティティがフレームワークに依存してしまう
1つ目の理由がエンティティがフレームワークに依存してしまうためです。
これにより、エンティティに実装するべきビジネスロジックと、Eloquentモデル固有のクエリビルダ等のコードが混在することになります。
そうすると、クラスが果たすべき責任が曖昧になってしまいます。
また、エンティティがフレームワークに依存すると、フレームワークの移行が難しくなります。
そのため、フレームワークはあくまでツールとして使うべきだと考えています。
## データ構造がテーブルと同じになる
2つ目の理由がデータ構造がテーブルと同じになるためです。
これにより、エンティティに値オブジェクトなどの固有のフィールドを持つことが難しくなります。
# 最後に
DDDの考え方で設計するときに、EloquentモデルのようなORMを使わずに、エンティティをどう表現するかを非常に悩んだので今回の記事を書きました。
参考になれば幸いです。