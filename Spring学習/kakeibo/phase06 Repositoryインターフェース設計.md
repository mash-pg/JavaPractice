# Phase 6: Repository インターフェース設計（Domain 層）

> **目的**: データアクセスを抽象化する Repository インターフェースを domain 層に設計する。DB 技術を domain に漏らさず、テスト可能な構造を作る。

---

## なぜ Repository インターフェースが必要なのか？

### もし UseCase が直接 DB を操作したら...

```java
@Service
public class CreateTransactionUseCase {
    @Autowired
    private JdbcTemplate jdbcTemplate;  // ← DB 技術に直接依存！

    public void execute(Transaction transaction) {
        jdbcTemplate.update(
            "INSERT INTO transactions (type, amount, ...) VALUES (?, ?, ...)",
            transaction.getType().name(),
            transaction.getAmount().getValue()
        );
    }
}
```

**問題**:
- DB を H2 → PostgreSQL に変えたら **UseCase を修正** しなければならない
- SQL を JPA に変えたら **UseCase を修正** しなければならない
- テスト時に実際の DB が必要（遅い、不安定）
- ビジネスロジックと DB 操作が混ざって読みにくい

### だから「インターフェース」で抽象化する

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  UseCase（application 層）                                       │
│    │                                                            │
│    │ 「保存して」「取得して」とお願いするだけ                         │
│    │ → HOW（どうやって保存するか）は知らない                         │
│    │                                                            │
│    ▼                                                            │
│  Repository Interface（domain 層）    ← 「契約書」               │
│    │                                                            │
│    │ save(), findById(), findAll() などのメソッドを定義             │
│    │ → 実装は書かない。「何ができるか」だけ宣言                      │
│    │                                                            │
│    ▼                                                            │
│  RepositoryImpl（infrastructure 層） ← 「契約の実行者」           │
│    │                                                            │
│    │ JPA / JDBC / ファイル保存 など、実際の保存方法を実装             │
│    ▼                                                            │
│  DB                                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 依存性逆転の原則（DIP: Dependency Inversion Principle）

### 通常の依存方向（やりがちだが NG）

```
UseCase → RepositoryImpl → DB
         （具体クラスに依存）
```

この場合、DB 技術が変わると UseCase まで影響が波及する。

### DIP を適用した依存方向（今回の設計）

```
UseCase → Repository Interface ← RepositoryImpl → DB
          （domain 層）           （infrastructure 層）

  ※ 矢印の向き = 依存の方向
  ※ infrastructure が domain に依存する（逆転している！）
```

**ポイント**:
- `UseCase` は `Repository`（インターフェース）だけを知っている
- `RepositoryImpl` は `Repository` を `implements` する
- domain 層は infrastructure 層を **一切知らない**
- Spring の DI コンテナが実行時に `RepositoryImpl` を自動で注入する

### なぜ「逆転」と呼ぶのか？

```
普通の考え方:
  「上位（UseCase）が下位（DB操作）に依存する」

逆転後:
  「下位（RepositoryImpl）が上位（Repository Interface）に依存する」
  → 上位が「こういうメソッドが欲しい」と定義し、下位がそれに従う
```

**たとえ話**:
- 普通: レストランの客が「厨房のコンロがIHだから、IHで作れる料理を注文する」 → おかしい
- DIP: 客が「カレーを注文する」→ 厨房がガスでもIHでも好きな方法で作る → 自然

---

## Repository パターンとは

### DAO パターンとの違い

実務では Repository パターンと DAO（Data Access Object）パターンを混同しがち。

| 観点 | DAO パターン | Repository パターン |
|------|------------|-------------------|
| **置く層** | infrastructure 層 | **domain 層**（Interface） |
| **メソッドの視点** | DB 操作の視点（`insert`, `update`, `select`） | **ドメインの視点**（`save`, `findById`） |
| **扱う型** | DB のエンティティ（JPA Entity） | **ドメインのエンティティ**（Domain Entity） |
| **目的** | DB アクセスをカプセル化 | **ドメインの永続化を抽象化** |

```java
// DAO 的な発想（DB 中心）
public interface TransactionDao {
    void insert(TransactionJpaEntity entity);         // ← JPA Entity
    TransactionJpaEntity selectById(Long id);         // ← Long（プリミティブ）
    List<TransactionJpaEntity> selectAll();            // ← JPA Entity のリスト
}

// Repository 的な発想（ドメイン中心）★今回の設計
public interface TransactionRepository {
    Transaction save(Transaction transaction);         // ← Domain Entity
    Optional<Transaction> findById(TransactionId id);  // ← VO（TransactionId）
    List<Transaction> findAll();                       // ← Domain Entity のリスト
}
```

**重要**: Repository のインターフェースには DB の都合（Long, String, JPA Entity）が一切出てこない。domain 層の型（VO, Domain Entity）だけで構成される。

---

## シニア × ジュニアのやり取り記録

### 課題の提示（シニア）

> Phase 5 で UseCase を書く前に、UseCase が使う Repository インターフェースを先に作る。
> - Step 1: TransactionRepository
> - Step 2: CategoryRepository

---

## Step 1: TransactionRepository

### 1回目の提出

```java
public interface TransactionRepository {
    Transaction save(Transaction transaction);
    Optional<Transaction> findById(TransactionId id);
    List<Transaction> findAll();
    void deleteById(TransactionId id);
    List<Transaction> existsByCategoryId(
            TransactionType type,
            CategoryId categoryId,
            LocalDate from, LocalDate to);
}
```

### シニアの指摘

| # | 問題 | 詳細 |
|---|------|------|
| 1 | `existsByCategoryId` に2つのメソッドが混ざっている | `existsByCategoryId`（boolean）と `findByConditions`（List）は別メソッド |

### 詰まりポイント: メソッド名と戻り値の不一致

**何が起きたか：**
`existsByCategoryId` という名前なのに、戻り値が `List<Transaction>` で、引数も4つある。1つのメソッドに2つの役割を詰め込んでしまった。

**学び: メソッド名の命名規則**

| プレフィックス | 戻り値 | 意味 |
|-------------|--------|------|
| `exists~` | `boolean` | 存在するかどうか（true/false） |
| `find~` | `Optional<T>` / `List<T>` | 取得する |
| `save` | `T` | 保存して返す |
| `delete~` | `void` | 削除する |
| `count~` | `long` / `int` | 件数を返す |

**ルール**: メソッド名と戻り値は一致させる。`exists` なら `boolean`、`find` なら `Optional` or `List`。

### 2回目の提出（合格）

```java
public interface TransactionRepository {
    Transaction save(Transaction transaction);
    Optional<Transaction> findById(TransactionId id);
    List<Transaction> findAll();
    void deleteById(TransactionId id);
    boolean existsByCategoryId(CategoryId id);
}
```

> `findByConditions` は Phase 9 で追加予定（YAGNI: 今必要ないメソッドは作らない）

---

## Step 2: CategoryRepository

### 1回目の提出

TransactionRepository をコピペしたため、全メソッドの型が `Transaction` / `TransactionId` になっていた。

```java
public interface CategoryRepository {
    Transaction save(Transaction transaction);          // ← 全部 Transaction!
    Optional<Transaction> findById(TransactionId id);
    List<Transaction> findAll();
    void deleteById(TransactionId id);
    boolean existsByName(CategoryName name);            // ← ここだけ正しい
}
```

### シニアの指摘

| # | 問題 |
|---|------|
| 1 | `save(Transaction transaction)` → `save(Category category)` |
| 2 | `findById(TransactionId id)` → `findById(CategoryId id)` |
| 3 | `Optional<Transaction>` → `Optional<Category>` |
| 4 | `List<Transaction>` → `List<Category>` |
| 5 | `deleteById(TransactionId id)` → `deleteById(CategoryId id)` |

### 詰まりポイント: コピペ後の型チェック漏れ

**何が起きたか：**
TransactionRepository をコピペして `existsByName` だけ変更し、他のメソッドの型を変え忘れた。

**学び:**
コピペしたら **全メソッドの引数型と戻り値型** を1つずつ確認する。
- CategoryRepository は **Category** を扱う → 全メソッドが Category 系の型を使う
- TransactionRepository は **Transaction** を扱う → 全メソッドが Transaction 系の型を使う

### 2回目の提出（合格）

```java
public interface CategoryRepository {
    Category save(Category category);
    Optional<Category> findById(CategoryId id);
    List<Category> findAll();
    void deleteById(CategoryId id);
    boolean existsByName(CategoryName name);
}
```

---

## 深堀り: Repository インターフェースの設計判断

### 1. なぜ引数に VO を使うのか？

```java
// ❌ プリミティブ型を使う場合
Optional<Transaction> findById(Long id);
void deleteById(Long id);

// ✅ VO を使う場合
Optional<Transaction> findById(TransactionId id);
void deleteById(TransactionId id);
```

**VO を使う理由: 型安全**

```java
// プリミティブだと間違えやすい
transactionRepository.findById(categoryId);  // ← Long 同士なのでコンパイルが通ってしまう！

// VO だとコンパイルエラーで気づける
transactionRepository.findById(categoryId);  // ← CategoryId は TransactionId ではない → コンパイルエラー！
```

**たとえ話**:
- プリミティブ = 「お金を裸で渡す」→ 1000円なのか1000ドルなのか分からない
- VO = 「封筒に入れてラベルを貼って渡す」→ 「交通費: 1000円」と明確

### 2. なぜ `findById` は `Optional` を返すのか？

```java
// パターン A: null を返す（❌ 避けたい）
Transaction findById(TransactionId id);  // → 見つからないとき null が返る
// 呼び出し側で null チェックを忘れると NullPointerException！

// パターン B: Optional を返す（✅ 推奨）
Optional<Transaction> findById(TransactionId id);
// 呼び出し側が「見つからない場合」を必ず考慮する
```

**UseCase での使い方:**

```java
// Optional を受け取って、見つからなければ例外を投げる
Transaction transaction = transactionRepository.findById(id)
    .orElseThrow(() -> new TransactionNotFoundException("収支がみつかりません"));
```

**なぜ Repository で例外を投げないのか？**
- Repository は「データの出し入れ」だけが責務
- 「見つからないときどうするか」は **業務判断** → UseCase の責務
- 場面によって対応が違う可能性がある：
  - 詳細取得 → 例外を投げる
  - 存在チェック → boolean で返す
  - 一覧の中の1件 → スキップする

### 3. `existsByCategoryId` はなぜ TransactionRepository にあるのか？

**ジュニアの疑問:**
> カテゴリの存在チェックなのに TransactionRepository にあるのは変じゃないですか？

**シニアの回答:**

このメソッドの正確な意味を分解すると：

```
existsByCategoryId
  exists     → 「存在するか」を調べる
  By         → 「〜で検索して」
  CategoryId → 「CategoryId を条件にして」

→ 「ある CategoryId を持つ **Transaction** が存在するか」
```

| メソッド | 所属 | 何を検索するか |
|---------|------|--------------|
| `transactionRepository.existsByCategoryId(id)` | TransactionRepository | **Transaction** テーブルを CategoryId で検索 |
| `categoryRepository.findById(id)` | CategoryRepository | **Category** テーブルを CategoryId で検索 |
| `categoryRepository.existsByName(name)` | CategoryRepository | **Category** テーブルを CategoryName で検索 |

**原則**: Repository のメソッドは、**そのリポジトリが管理するエンティティのテーブル** を検索する。
- TransactionRepository → Transaction テーブルを操作
- CategoryRepository → Category テーブルを操作

**使用場面**:
```java
// DeleteCategoryUseCase で使う
// 「このカテゴリを使っている収支があるか？」を確認する
boolean inUse = transactionRepository.existsByCategoryId(categoryId);
if (inUse) {
    throw new CategoryInUseException("使用中のカテゴリは削除できません");
}
```

### 4. YAGNI: findByConditions を先送りにした判断

**ロードマップの計画:**
- Phase 9 でフィルタリング機能を実装する
- Phase 6 の段階では基本 CRUD に必要なメソッドだけ定義する

**最初の TransactionRepository に入れようとしたメソッド:**
```java
List<Transaction> findByConditions(
    TransactionType type,
    CategoryId categoryId,
    LocalDate from, LocalDate to);
```

**先送りにした理由:**
1. Phase 5 の UseCase（基本 CRUD）では使わない
2. 引数の組み合わせが未確定（全て必須？一部 null OK？）
3. 使う段階で設計した方が、より適切なインターフェースになる

> **YAGNI = You Aren't Gonna Need It**
> 「今使わないものは作らない。必要になったら作る。」
> ただし「存在を忘れる」のではなく「Phase 9 で対応予定」と記録しておく。

### 5. Repository インターフェースがテストを可能にする

```java
// UseCase のテスト（Repository をモックに差し替え）
@Test
void カテゴリが存在しないとき例外が飛ぶ() {
    // Arrange: モックの Repository を用意
    TransactionRepository mockRepo = mock(TransactionRepository.class);
    CategoryRepository mockCategoryRepo = mock(CategoryRepository.class);

    when(mockCategoryRepo.findById(any()))
        .thenReturn(Optional.empty());  // ← 「見つからない」を再現

    CreateTransactionUseCase useCase =
        new CreateTransactionUseCase(mockRepo, mockCategoryRepo);

    // Act & Assert
    assertThrows(CategoryNotFoundException.class,
        () -> useCase.execute(testTransaction));
}
```

**なぜインターフェースだとテストできるのか？**
- UseCase は `TransactionRepository`（インターフェース）に依存している
- テスト時にはモック（偽の実装）を渡せる
- 本番では Spring が `TransactionRepositoryImpl`（本物の実装）を渡す
- UseCase のコードは **一切変更不要**

```
本番: UseCase → TransactionRepositoryImpl → DB
テスト: UseCase → MockRepository → メモリ上のダミーデータ
```

---

## Repository インターフェースとレイヤの関係（全体像）

```
┌─────────────────────────────────────────────────────────────────┐
│ presentation 層                                                  │
│   Controller → UseCase を呼ぶだけ                                │
└───────────────────────┬─────────────────────────────────────────┘
                        │ 呼び出す
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│ application 層                                                   │
│   UseCase → Repository（Interface）を使って業務ロジックを実行      │
│                                                                 │
│   例: CreateTransactionUseCase                                  │
│       private final TransactionRepository transactionRepository;│
│       private final CategoryRepository categoryRepository;      │
│       → 「インターフェースの型」でフィールドを持つ                   │
└───────────────────────┬─────────────────────────────────────────┘
                        │ 依存（インターフェースに対して）
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│ domain 層                                                        │
│                                                                 │
│   TransactionRepository（interface）  CategoryRepository（interface）│
│   ├─ save(Transaction)               ├─ save(Category)          │
│   ├─ findById(TransactionId)         ├─ findById(CategoryId)    │
│   ├─ findAll()                       ├─ findAll()               │
│   ├─ deleteById(TransactionId)       ├─ deleteById(CategoryId)  │
│   └─ existsByCategoryId(CategoryId)  └─ existsByName(CategoryName)│
│                                                                 │
│   ★ ここには「何ができるか」だけ。「どうやって」は書かない          │
└───────────────────────▲─────────────────────────────────────────┘
                        │ implements（依存方向が逆転！）
                        │
┌───────────────────────┴─────────────────────────────────────────┐
│ infrastructure 層                                                │
│                                                                 │
│   TransactionRepositoryImpl       CategoryRepositoryImpl         │
│   ├─ implements TransactionRepository                            │
│   ├─ 内部で TransactionJpaRepository を使う                       │
│   └─ Domain Entity ↔ JPA Entity の変換を担当                     │
│                                                                 │
│   → Phase 7 で実装する                                           │
└─────────────────────────────────────────────────────────────────┘
```

---

## Spring の DI（依存性注入）が自動で行うこと

```java
// UseCase のコード
@Service
public class CreateTransactionUseCase {
    private final TransactionRepository transactionRepository;  // ← インターフェース型

    // コンストラクタインジェクション
    public CreateTransactionUseCase(TransactionRepository transactionRepository) {
        this.transactionRepository = transactionRepository;
    }
}
```

**疑問**: インターフェースには実装がないのに、なぜ動くのか？

**答え**: Spring の DI コンテナが自動で解決する。

```
1. Spring が起動する
2. @Repository が付いた TransactionRepositoryImpl を見つける
3. TransactionRepositoryImpl が TransactionRepository を implements していることを認識
4. UseCase のコンストラクタに TransactionRepository 型の引数があることを認識
5. 自動で TransactionRepositoryImpl のインスタンスを注入する
```

**つまり**:
```java
// 開発者が書くコード（インターフェース型で宣言）
private final TransactionRepository transactionRepository;

// Spring が実行時にやってくれること
this.transactionRepository = new TransactionRepositoryImpl(jpaRepository);
```

---

## 詰まりポイントまとめ

### 1. メソッド名と戻り値の不一致

**何が起きたか：**
`existsByCategoryId` という名前で `List<Transaction>` を返すメソッドを書いた。

**学び：**
- `exists~` → `boolean`
- `find~` → `Optional<T>` or `List<T>`
- メソッド名が「契約」。名前から期待される戻り値を返す。

### 2. コピペ後の型チェック漏れ

**何が起きたか：**
TransactionRepository をコピペして CategoryRepository を作ったが、型を変え忘れた。

**学び：**
コピペ後は **全行の型** を確認する。「どのエンティティの Repository か」を意識する。

### 3. 1つのメソッドに複数の責務を入れてしまう

**何が起きたか：**
`existsByCategoryId` に検索条件（type, from, to）まで入れてしまった。

**学び：**
1つのメソッド = 1つの責務。複数のことをしたいなら、メソッドを分ける。

---

## 完成した Repository インターフェース

```
domain/repository/
├── TransactionRepository.java   ← Transaction の永続化契約
└── CategoryRepository.java      ← Category の永続化契約
```

### TransactionRepository

```java
public interface TransactionRepository {
    Transaction save(Transaction transaction);
    Optional<Transaction> findById(TransactionId id);
    List<Transaction> findAll();
    void deleteById(TransactionId id);
    boolean existsByCategoryId(CategoryId id);
}
```

### CategoryRepository

```java
public interface CategoryRepository {
    Category save(Category category);
    Optional<Category> findById(CategoryId id);
    List<Category> findAll();
    void deleteById(CategoryId id);
    boolean existsByName(CategoryName name);
}
```

---

## 自己チェック

| チェック | 内容 | 結果 |
|---------|------|------|
| [x] | Repository インターフェースを domain 層に作成できた | 2つとも作成 |
| [△] | メソッド名と戻り値を一致させられた | existsByCategoryId で失敗 → 修正 |
| [△] | コピペせず正しい型を使えた | Category で型の変え忘れ → 修正 |
| [x] | なぜ domain 層にインターフェースを置くか説明できる | DIP、テスタビリティ |
| [x] | YAGNI を意識してメソッドを絞れた | findByConditions を Phase 9 に先送り |

### ToDo を参照した箇所

| 箇所 | 何を確認したか |
|------|---------------|
| TodoRepository | メソッドのシグネチャパターン（save, findById, findAll, deleteById） |

---

## Phase 6 で最も大事な学び

### 「Repository インターフェースは domain 層の "契約書"」

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Repository インターフェース = 「こういう操作が必要」という宣言     │
│                                                                 │
│  ・domain 層に置く → ビジネスの都合で設計する                      │
│  ・引数は VO → 型安全                                            │
│  ・戻り値は Domain Entity → DB の都合を漏らさない                 │
│  ・実装は書かない → HOW は infrastructure に任せる                │
│                                                                 │
│  たとえ話:                                                       │
│    レストランのメニュー = Repository Interface                    │
│    ・「カレー」「パスタ」「サラダ」が注文できると書いてある          │
│    ・「ガスコンロで炒めて...」とは書いていない                      │
│    ・厨房（infrastructure）がどう作るかは客（UseCase）は知らない   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Repository インターフェースの設計手順（実務でも使える）

```
1. UseCase を先にイメージする
   → 「どんなデータ操作が必要か？」

2. 必要なメソッドだけ定義する（YAGNI）
   → save, findById, findAll, deleteById が基本セット

3. ドメイン固有のメソッドを追加する
   → existsByCategoryId, existsByName（業務ルールに必要なもの）

4. メソッド名の命名規則を守る
   → exists → boolean, find → Optional/List

5. 引数と戻り値は domain の型を使う
   → VO, Domain Entity（Long, String は使わない）
```

---

## 次の Phase へ

Phase 6 完了。次は **Phase 7: DB・JPA 実装（Infrastructure）** へ進む。
- JPA Entity（TransactionJpaEntity, CategoryJpaEntity）の作成
- JpaRepository（Spring Data JPA インターフェース）の作成
- Repository 実装クラス（TransactionRepositoryImpl, CategoryRepositoryImpl）の作成
- Domain Entity と JPA Entity の変換メソッド

> **Phase 7 のポイント**: Phase 6 で定義した「契約書」（インターフェース）を、JPA を使って「実装」する。Domain Entity ↔ JPA Entity の変換が最大の課題。
