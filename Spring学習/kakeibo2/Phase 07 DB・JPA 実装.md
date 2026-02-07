# Phase 07: DB・JPA 実装

## 概要

Domain Entity を実際のデータベースに接続するための JPA 層を実装した。

---

## 作成したファイル

| パッケージ | ファイル | 役割 |
|-----------|---------|------|
| `infrastructure/jpa` | `CategoryJpaEntity` | DB用 Entity |
| `infrastructure/jpa` | `TransactionJpaEntity` | DB用 Entity |
| `infrastructure/repository` | `CategoryJpaRepository` | Spring Data JPA インターフェース |
| `infrastructure/repository` | `TransactionJpaRepository` | Spring Data JPA インターフェース |
| `infrastructure/repository` | `CategoryRepositoryImpl` | domain Repository の実装 |
| `infrastructure/repository` | `TransactionRepositoryImpl` | domain Repository の実装 |

---

## 理解ポイント

### なぜ Domain Entity と JPA Entity を分けるのか？

| Domain Entity | JPA Entity |
|---------------|------------|
| VO を使う（Amount, CategoryId など） | プリミティブ型を使う（int, Long など） |
| ビジネスルールを持つ | DB の都合（カラム名、型変換）を持つ |

**DB はプリミティブ型しか保存できない** ため、変換が必要。
分離することで、お互い独立して変更できる。

---

### JPA Entity に必要なアノテーション

```java
@Entity                                          // JPA の Entity であることを示す
@Table(name = "categories")                      // テーブル名を指定
public class CategoryJpaEntity {
    @Id                                          // 主キー
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // 自動採番
    private Long id;

    private String name;

    public CategoryJpaEntity() {}                // JPA 用のデフォルトコンストラクタ（必須）
}
```

---

### 変換メソッド

```java
// JPA Entity → Domain Entity
public Category toDomainEntity() {
    return new Category(
        new CategoryId(this.id),
        new CategoryName(this.name)
    );
}

// Domain Entity → JPA Entity
public static CategoryJpaEntity fromDomainEntity(Category category) {
    Long id = category.getId().getValue();
    String name = category.getCategoryName().getValue();
    return new CategoryJpaEntity(name, id);
}
```

---

### enum → String 変換

```java
// enum から String を取得
TransactionType.INCOME.name()  // → "INCOME"

// String から enum を取得
TransactionType.from("INCOME")  // → TransactionType.INCOME
```

---

### Spring Data JPA

```java
public interface CategoryJpaRepository extends JpaRepository<CategoryJpaEntity, Long> {
}
```

これだけで以下のメソッドが自動で使える：
- `save()` - 保存
- `findById()` - ID で検索
- `findAll()` - 全件取得
- `deleteById()` - 削除

---

### Repository 実装クラス

```java
@Repository
public class CategoryRepositoryImpl implements CategoryRepository {

    private final CategoryJpaRepository jpaRepository;

    @Override
    public List<Category> findAll() {
        return jpaRepository.findAll()
                .stream()
                .map(jpa -> jpa.toDomainEntity())  // Stream で変換
                .toList();
    }

    @Override
    public Optional<Category> findById(CategoryId id) {
        return jpaRepository.findById(id.getValue())
                .map(jpa -> jpa.toDomainEntity());  // Optional の map で変換
    }
}
```

---

### Optional.map() のポイント

```java
Optional<CategoryJpaEntity>  // findById の戻り値
    .map(jpa -> jpa.toDomainEntity())  // 中身を変換
// → Optional<Category>  // 型が変わる！
```

- 中身がある → 変換して `Optional<Category>` を返す
- 中身がない → `Optional.empty()` を返す

---

### Repository で例外を投げない理由

Repository の役割は **「データの取得」だけ**。

「見つからない → どうする？」は **ビジネスの判断** なので UseCase が行う。

- ある UseCase では「見つからなかったらエラー」
- 別の UseCase では「見つからなかったら新規作成」

だから Repository は Optional をそのまま返す。

---

## 次のステップ

Phase 8: API 実装へ進む
