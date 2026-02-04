# Phase 7: DB・JPA 実装（Infrastructure）

> **目的**: 2つのテーブルを持つ DB 実装を作る。Phase 6 で定義した Repository インターフェース（契約書）を、JPA を使って実装する。Domain Entity と JPA Entity を分離し、Repository 実装クラスで橋渡しする。

---

## シニア × ジュニアのやり取り記録

### 課題の提示（シニア）

> - Step 1: JPA Entity を作る（infrastructure/jpa/）
> - Step 2: Repository 実装クラスを作る（infrastructure/repository/）

---

## Step 1: JPA Entity

### CategoryJpaEntity（3回で合格）

#### 1回目の提出

```java
@Entity
@Table(name = "categories")
public class CategoryJpaEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 50)
    private CategoryName name;  // ← VO を使ってしまっている

    public CategoryJpaEntity(Long id, CategoryName name) {
        // デフォルトコンストラクタがない
    }
}
```

#### シニアの指摘

| # | 問題 | 詳細 |
|---|------|------|
| 1 | `name` の型が `CategoryName`（VO） | JPA Entity では VO を使わず、プリミティブ型（`String`）を使う |
| 2 | デフォルトコンストラクタがない | JPA はインスタンス生成に引数なしコンストラクタが必須 |

### 詰まりポイント: Domain Entity と JPA Entity の違いが分からない

**ジュニアの疑問:**
> TodoJpaEntity だと VO もあるよ？

**シニアの回答:**
このロードマップでは「Domain Entity と JPA Entity を分離する」方針。変換メソッド（`toJpaEntity`, `toDomainEntity`）が必要ということは、2つの Entity は型が違う前提。

| | Domain Entity | JPA Entity |
|---|--------------|------------|
| フィールドの型 | **VO を使う**（CategoryName） | **プリミティブ型**（String） |
| 目的 | ビジネスルールを守る | DB とのマッピング |
| コンストラクタ | 有効な状態で生成 | JPA 用にデフォルトコンストラクタが必要 |

**なぜ分離するのか：**
- JPA Entity は DB の都合（`@Column`, `@Table` など）に縛られる
- Domain Entity はビジネスルールに集中する
- 混ぜると、DB の変更がドメインに影響してしまう

#### 2回目の提出

`String` に変更。デフォルトコンストラクタがまだない。

#### 3回目の提出（合格）

```java
@Entity
@Table(name = "categories")
public class CategoryJpaEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 50)
    private String name;

    public CategoryJpaEntity() {}  // JPA 用
    public CategoryJpaEntity(Long id, String name) { ... }
    // getter, setter
}
```

---

### TransactionJpaEntity（3回で合格）

#### 1回目の提出

```java
@Entity
@Table(name = "categories")  // ← コピペミス
public class TransactionJpaEntity {
    @ManyToOne
    @JoinColumn(name = "category_id")
    private Long category_id;      // ← スネークケース

    private LocalDate transaction_date;  // ← スネークケース
    private LocalDateTime created_at;    // ← スネークケース
    // コンストラクタに amount がない
}
```

#### シニアの指摘

| # | 問題 | 詳細 |
|---|------|------|
| 1 | `@Table(name = "categories")` | コピペで残っている。`"transactions"` に変更 |
| 2 | `@ManyToOne` + `@JoinColumn` と `Long` | 単純に `Long` で持つなら `@Column(name = "category_id")` でOK |
| 3 | フィールド名がスネークケース | Java はキャメルケース。`@Column(name = "...")` で DB カラム名と対応させる |
| 4 | コンストラクタに `amount` がない | 引数が足りない |

**学び: Java の命名規則と DB カラム名の対応**

| Java フィールド名 | DB カラム名 | アノテーション |
|-------------------|-----------|--------------|
| `categoryId` | `category_id` | `@Column(name = "category_id")` |
| `transactionDate` | `transaction_date` | `@Column(name = "transaction_date")` |
| `createdAt` | `created_at` | `@Column(name = "created_at")` |

#### 3回目の提出（合格）

```java
@Entity
@Table(name = "transactions")
public class TransactionJpaEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 20)
    @Enumerated(EnumType.STRING)  // Enum を文字列で保存
    private TransactionType type;

    @Column(nullable = false)
    private int amount;

    @Column(nullable = false, name = "category_id")
    private Long categoryId;

    @Column(nullable = true)
    private String description;

    @Column(nullable = false, name = "transaction_date")
    private LocalDate transactionDate;

    @Column(nullable = true, name = "created_at")
    private LocalDateTime createdAt;

    public TransactionJpaEntity() {}
    public TransactionJpaEntity(Long id, TransactionType type, Long categoryId,
            int amount, String description, LocalDate transactionDate,
            LocalDateTime createdAt) { ... }
    // getter, setter
}
```

**良かった点:** `@Enumerated(EnumType.STRING)` を自力で使えた。コメント「これがないと0,1の数字になる」も正しい理解。

---

## Step 2: Repository 実装

### CategoryRepositoryImpl（めちゃくちゃ躓いた）

#### 1回目の提出（変換メソッドは書けたが中身が TODO）

```java
@Repository
public class CategoryRepositoryImpl implements CategoryRepository {
    // save, findById, findAll, deleteById, existsByName が全部 TODO

    private CategoryJpaEntity toJpaEntity(Category category) {
        // ← これは正しく書けていた！
    }
    private Category toDomaiEntity(CategoryJpaEntity jpaEntity) {
        // ← typo はあるが変換ロジックは正しい
    }
}
```

**良かった点:** 変換メソッドの考え方は理解できていた。

#### 2回目の提出（各メソッド実装 + existsByName の問題）

| メソッド | 実装 | 結果 |
|---------|------|------|
| `save` | Domain → JPA → save → Domain | 合格 |
| `findById` | `.map(this::toDomainEntity)` | 合格 |
| `findAll` | `.stream().map().collect()` | 合格 |
| `deleteById` | `id.getValue()` | 合格 |
| `existsByName` | `jpaRepository.existsByName(name)` | NG: `.getValue()` が抜けている |

### 詰まりポイント: VO とプリミティブの変換を忘れる

**何が起きたか：**
`existsByName` で `CategoryName`（VO）をそのまま JPA Repository に渡してしまった。

**学び:**
Impl の全メソッドで「VO → プリミティブ」の変換が必要。`.getValue()` を忘れない。

### 大混乱: Impl のメソッド引数をプリミティブに変えてしまった

**何が起きたか：**
「JPA にはプリミティブを使う」という指摘を受けて、Impl のメソッド引数まで全部プリミティブに変えてしまった。

```java
// こう変えてしまった（間違い）
public Optional<Category> findById(Long id) { ... }
public void deleteById(Long id) { ... }
public boolean existsByName(String name) { ... }
```

**結果:** `implements CategoryRepository` と引数の型が合わなくなりコンパイルエラー。

```
The type CategoryRepositoryImpl must implement the inherited abstract method
CategoryRepository.existsByName(CategoryName)
```

**シニアの解説:**

```
domain の世界（VO）  →  Impl（ここで変換）  →  JPA の世界（プリミティブ）

Impl のメソッド引数は domain と同じ VO を使う（implements だから）
Impl のメソッド内部で .getValue() して JPA に渡す
```

| 場所 | 型 | 理由 |
|------|-----|------|
| domain インターフェース | VO（CategoryName） | ビジネスルールを表現 |
| Impl のメソッド引数 | VO（CategoryName） | implements だから domain と同じ |
| Impl のメソッド内部 | `.getValue()` で変換 | JPA に渡すため |
| JPA Repository | プリミティブ（String） | DB が理解できる型 |

**教訓:** 「プリミティブを使う」のは JPA 側だけ。Impl は domain のインターフェースを実装するので、引数は VO のまま。

### ジュニアの疑問と学び

**疑問1:** CategoryRepositoryImpl で VO を使っているけどいいの？

> Impl は2つの世界の橋渡し。domain 側（引数）は VO、JPA 側（内部）はプリミティブ。Impl が `.getValue()` で変換する。

**疑問2:** なぜ domain のインターフェースは VO を使うの？

> domain 層はビジネスルールの世界。VO を使うことで型安全になる（Long の categoryId と Long の transactionId を間違えて渡すミスを防げる）。

---

### TransactionRepositoryImpl（めちゃくちゃ躓いた）

#### 1回目の提出

| # | 問題 | 詳細 |
|---|------|------|
| 1 | `CategoryJpaRepository` を注入している | Transaction の Impl なので `TransactionJpaRepository` を使う |
| 2 | `toDomainEntity` の引数が足りない | Transaction のコンストラクタは7引数だが6個しか渡していない |
| 3 | `categoryId` が Long のまま | `new CategoryId()` で VO に変換が必要 |
| 4 | `transactionDate` が抜けている | 引数に含まれていない |

#### 2回目の提出（JpaRepository の型引数ミス）

```java
// TransactionJpaRepository
interface TransactionJpaRepository extends JpaRepository<TransactionJpaRepository, Long> {
//                                                        ↑ 自分自身を指定してしまった
}
```

**エラー:**
```
The method save(S) in the type CrudRepository<TransactionJpaRepository,Long>
is not applicable for the arguments (TransactionJpaEntity)
```

**原因:** `JpaRepository` の1つ目の型引数に自分自身（`TransactionJpaRepository`）を指定してしまった。正しくは `TransactionJpaEntity`。

**比較:**
```java
// Category（正しい）
interface CategoryJpaRepository extends JpaRepository<CategoryJpaEntity, Long>

// Transaction（間違い）
interface TransactionJpaRepository extends JpaRepository<TransactionJpaRepository, Long>

// Transaction（修正後）
interface TransactionJpaRepository extends JpaRepository<TransactionJpaEntity, Long>
```

#### 3回目の提出（合格）+ BUILD SUCCESS

```java
@Repository
public class TransactionRepositoryImpl implements TransactionRepository {
    private final TransactionJpaRepository jpaRepository;

    // save: Domain → JPA → save → Domain
    @Override
    public Transaction save(Transaction transaction) {
        TransactionJpaEntity jpaEntity = toJpaEntity(transaction);
        TransactionJpaEntity savedEntity = jpaRepository.save(jpaEntity);
        return toDomainEntity(savedEntity);
    }

    // findById: JPA → Domain に変換
    @Override
    public Optional<Transaction> findById(TransactionId id) {
        return jpaRepository.findById(id.getValue()).map(this::toDomainEntity);
    }

    // findAll: stream で全件変換
    @Override
    public List<Transaction> findAll() {
        return jpaRepository.findAll().stream()
                .map(this::toDomainEntity)
                .collect(Collectors.toList());
    }

    // deleteById: VO → Long に変換
    @Override
    public void deleteById(TransactionId id) {
        jpaRepository.deleteById(id.getValue());
    }

    // existsByCategoryId: VO → Long に変換
    @Override
    public boolean existsByCategoryId(CategoryId id) {
        return jpaRepository.existsById(id.getValue());
    }

    // Domain → JPA 変換
    private TransactionJpaEntity toJpaEntity(Transaction transaction) {
        TransactionJpaEntity entity = new TransactionJpaEntity();
        if(transaction.getId() != null) {
            entity.setId(transaction.getId().getValue());
        }
        entity.setType(transaction.getType());
        entity.setAmount(transaction.getAmount().getValue());
        entity.setCategoryId(transaction.getCategoryId().getValue());
        entity.setDescription(transaction.getDescription());
        entity.setTransactionDate(transaction.getTransactionDate());
        entity.setCreatedAt(transaction.getCreatedAt());
        return entity;
    }

    // JPA → Domain 変換
    private Transaction toDomainEntity(TransactionJpaEntity jpaEntity) {
        return new Transaction(
            new TransactionId(jpaEntity.getId()),
            new Amount(jpaEntity.getAmount()),
            jpaEntity.getType(),
            new CategoryId(jpaEntity.getCategoryId()),
            jpaEntity.getDescription(),
            jpaEntity.getTransactionDate(),
            jpaEntity.getCreatedAt()
        );
    }
}
```

---

## 詰まりポイントまとめ

### 1. Domain Entity と JPA Entity の違いが分からない

**何が起きたか:** JPA Entity に VO（CategoryName）を使ってしまった。

**学び:**

| | Domain Entity | JPA Entity |
|---|--------------|------------|
| 型 | VO を使う | プリミティブ型を使う |
| 目的 | ビジネスルール | DB マッピング |
| 場所 | domain/entity/ | infrastructure/jpa/ |

### 2. Impl のメソッド引数までプリミティブに変えてしまう

**何が起きたか:** 「JPA にはプリミティブ」と言われて、Impl の引数まで全部変えてコンパイルエラー。

**学び:** Impl は `implements` なので引数は domain のインターフェースと同じ VO。内部で `.getValue()` して JPA に渡す。

### 3. コピペによるミスが多発

| ミス | 詳細 |
|------|------|
| テーブル名 | `@Table(name = "categories")` のまま transactions に使った |
| JpaRepository の型引数 | `TransactionJpaRepository` を自分自身に指定 |
| 注入するリポジトリ | Transaction の Impl に `CategoryJpaRepository` を注入 |

**学び:** コピペしたら以下を必ず確認する：
1. クラス名・テーブル名は正しいか
2. 型引数は正しいか
3. 注入するリポジトリは正しいか

### 4. toDomainEntity の引数の数・型が合わない

**何が起きたか:** Transaction のコンストラクタは7引数だが、6個しか渡さなかった。`categoryId` の VO 変換も漏れた。

**学び:** `toDomainEntity` を書くときは、Domain Entity のコンストラクタを見て引数の数と型を1つずつ確認する。

---

## 完成した infrastructure パッケージ構成

```
infrastructure/
├── jpa/
│   ├── CategoryJpaEntity.java       ← categories テーブル対応
│   └── TransactionJpaEntity.java    ← transactions テーブル対応
└── repository/
    ├── CategoryJpaRepository.java    ← Spring Data JPA（interface）
    ├── CategoryRepositoryImpl.java   ← CategoryRepository の実装
    ├── TransactionJpaRepository.java ← Spring Data JPA（interface）
    └── TransactionRepositoryImpl.java ← TransactionRepository の実装
```

## 3つのファイルの関係（最重要）

```
domain/repository/CategoryRepository      ← インターフェース（VO を使う）
        ↑ implements
infrastructure/repository/CategoryRepositoryImpl  ← 橋渡し（VO ↔ プリミティブ変換）
        ↓ 内部で使う
infrastructure/repository/CategoryJpaRepository   ← Spring Data JPA（プリミティブを使う）
        ↓ マッピング
infrastructure/jpa/CategoryJpaEntity              ← DB テーブル対応（プリミティブを使う）
```

**VO を使うのは domain 側だけ。infrastructure 側はプリミティブ。Impl が変換する。**

---

## 自己チェック

| チェック | 内容 | 結果 |
|---------|------|------|
| [x] | JPA Entity を作成できた | Category, Transaction 両方 |
| [x] | Domain Entity と JPA Entity を分離できた | VO vs プリミティブの違いを理解 |
| [x] | Repository 実装クラスを作成できた | implements + 変換メソッド |
| [△] | コピペミスなく作れた | テーブル名、型引数、注入リポジトリで複数ミス |
| [x] | BUILD SUCCESS | コンパイルエラーなし |

### ToDo を参照した箇所

| 箇所 | 何を確認したか |
|------|---------------|
| Repository 実装全体 | Impl の書き方、変換メソッドのパターン |

---

## Phase 7 で最も大事な学び

**「Impl は2つの世界の橋渡し」**

```
domain の世界（VO）  ←→  Impl（変換）  ←→  DB の世界（プリミティブ）
```

- domain 側のメソッド引数: VO のまま（`CategoryId`, `CategoryName`）
- JPA 側のメソッド引数: プリミティブ（`Long`, `String`）
- Impl が `.getValue()` や `new CategoryId()` で双方向に変換する

これにより **DB を変えても domain は一切触らない** という DDD の原則が実現される。

---

## 次の Phase へ

Phase 7 完了。次は **Phase 8: API 実装（基本 CRUD）** へ進む。
- Controller の実装（TransactionController, CategoryController）
- Request / Response DTO の作成
- 例外ハンドリング（@RestControllerAdvice）
- PowerShell での動作確認

> **Phase 8 のポイント**: Controller は薄く。HTTP リクエストの受け取りと UseCase の呼び出し、レスポンスの返却だけ。業務ロジックは UseCase に任せる。
