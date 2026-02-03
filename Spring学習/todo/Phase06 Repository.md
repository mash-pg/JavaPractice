# Phase 6: Repository 実装（Infrastructure層）

> **目的**: データアクセスを抽象化する。DB を変えても domain / application は変更不要にする。

---

## なぜこんな構造が必要？

### もし Domain Entity に直接 @Entity を付けたら...

```java
@Entity  // ← JPA（DB技術）に依存してしまう！
public class Todo {
    @Id
    private TodoId id;  // ← JPAは TodoId を理解できない
}
```

**問題**:
- DB を変えたら Domain も変更が必要
- ビジネスルールと DB の都合が混ざる
- テストしにくい

### だから分離する

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   Domain層（ビジネスルール）     Infrastructure層（DB技術）       │
│                                                                 │
│   Todo                         TodoJpaEntity                    │
│   ├─ TodoId id                 ├─ Long id        ← シンプルな型 │
│   ├─ TodoTitle title           ├─ String title   ← シンプルな型 │
│   └─ complete() ←ルール        └─ @Entity        ← JPAの都合    │
│                                                                 │
│   「何をするか」に集中           「どう保存するか」に集中          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 全体像

```
┌─────────────────────────────────────────────────────────────────┐
│  Domain層                       Infrastructure層                │
│                                                                 │
│  TodoRepository ◀─implements── TodoRepositoryImpl              │
│  (interface)                    │                               │
│                                 │ 使う（フィールドに持つ）        │
│                                 ▼                               │
│                           TodoJpaRepository ───▶ DB             │
│                           (Spring Data JPA)                     │
│                                 ▲                               │
│                                 │ 操作対象                       │
│                           TodoJpaEntity                         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 課題 6-1: TodoJpaEntity

**場所**: `com.example.todo.infrastructure.jpa.TodoJpaEntity`

**役割**: DBテーブルと1:1で対応する箱

### 重要なアノテーション

| アノテーション | 意味 |
|--------------|------|
| `@Entity` | 「これはDBテーブルに対応するクラス」 |
| `@Table(name="todos")` | テーブル名を指定 |
| `@Id` | 主キー |
| `@GeneratedValue(strategy=IDENTITY)` | DBが自動でIDを振る |
| `@Enumerated(EnumType.STRING)` | Enumを文字列で保存（"OPEN","DONE"） |

### 実装

```java
@Entity
@Table(name = "todos")
public class TodoJpaEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String title;

    @Column(nullable = false, length = 20)
    @Enumerated(EnumType.STRING)  // これがないと0,1の数字になる
    private TodoStatus status;

    // 引数なしコンストラクタ（JPA必須）
    public TodoJpaEntity() {}

    // 全引数コンストラクタ
    public TodoJpaEntity(Long id, String title, TodoStatus status) {
        this.id = id;
        this.title = title;
        this.status = status;
    }

    // Getter / Setter
}
```

---

## 課題 6-2: TodoJpaRepository

**場所**: `com.example.todo.infrastructure.repository.TodoJpaRepository`

**役割**: Spring Data JPA の魔法（継承するだけでCRUDが使える）

```java
public interface TodoJpaRepository extends JpaRepository<TodoJpaEntity, Long> {
    // 書かなくても以下が自動で使える！
    // - save(entity)
    // - findById(id)
    // - findAll()
    // - deleteById(id)
}
```

---

## 課題 6-3: TodoRepositoryImpl

**場所**: `com.example.todo.infrastructure.repository.TodoRepositoryImpl`

**役割**: 橋渡し役・翻訳者（Domain ↔ Infrastructure をつなぐ）

### 処理の流れ（save の例）

```
UseCase が todoRepository.save(todo) を呼ぶ
    │
    ▼
┌─────────────────────────────────────────────────────────────────┐
│ TodoRepositoryImpl.save(todo)                                   │
│                                                                 │
│   Step 1: toJpaEntity(todo)  ← Domain → JPA 変換               │
│           Todo { id=null, title="買い物", status=OPEN }          │
│                          ↓                                      │
│           TodoJpaEntity { id=null, title="買い物", status=OPEN } │
│                                                                 │
│   Step 2: jpaRepository.save(jpaEntity)  ← DB保存              │
│           → ID が自動採番される（例: id=1）                       │
│                                                                 │
│   Step 3: toDomainEntity(savedEntity)  ← JPA → Domain 変換     │
│           TodoJpaEntity { id=1, ... }                           │
│                          ↓                                      │
│           Todo { id=TodoId(1), title=TodoTitle("買い物"), ... }  │
│                                                                 │
│   return todo;  ← IDが付いた Domain Entity を返す                │
└─────────────────────────────────────────────────────────────────┘
```

---

## つまづいたポイント

### 1. `implements` を間違えた

**間違い**:
```java
public class TodoRepositoryImpl implements TodoJpaRepository  // NG!
```

**正しい**:
```java
public class TodoRepositoryImpl implements TodoRepository  // OK!
```

**理由**:
- `TodoJpaRepository` は Spring Data JPA のインターフェース（継承するだけ）
- `TodoRepository` は Domain層 のインターフェース（これを実装する）

```
TodoRepositoryImpl は:
- implements TodoRepository   → Domain層のルールに従う
- 使う: TodoJpaRepository     → 実際のDB操作はこっちに任せる

両方使うけど、役割が違う！
```

### 2. 変換メソッドをどこに書くか分からなかった

**質問**: `toJpaEntity` と `toDomainEntity` はどのクラスに書く？

**答え**: `TodoRepositoryImpl` の中に `private` メソッドとして書く

**理由**:
- 変換処理はこのクラスの中だけで使う
- 他のクラスから呼ぶ必要がない
- Repository実装の責務として Domain ↔ JPA の変換を担当

### 3. `getValue()` が見つからない

**エラー**: `entity.setId(todo.getId().getValue())` で getValue() がない

**原因**: TodoId のメソッド名は `getId()` だった（`getValue()` ではない）

```java
// TodoId.java
public Long getId() {  // getValue() ではなく getId()
    return id;
}
```

**教訓**: Value Object のメソッド名は統一しておくと良い（`getValue()` が一般的）

---

## 変換メソッドの解説

```java
// Domain → JPA（保存する前に呼ぶ）
// Value Object から中身を取り出す
private TodoJpaEntity toJpaEntity(Todo todo) {
    TodoJpaEntity entity = new TodoJpaEntity();
    if (todo.getId() != null) {
        entity.setId(todo.getId().getId());  // TodoId → Long
    }
    entity.setTitle(todo.getTitle().getValue());  // TodoTitle → String
    entity.setStatus(todo.getStatus());
    return entity;
}

// JPA → Domain（取得した後に呼ぶ）
// シンプルな型を Value Object で包む
private Todo toDomainEntity(TodoJpaEntity jpaEntity) {
    return new Todo(
        new TodoId(jpaEntity.getId()),       // Long → TodoId
        new TodoTitle(jpaEntity.getTitle()), // String → TodoTitle
        jpaEntity.getStatus()
    );
}
```

**ポイント**: 包む・取り出す

```
保存時: TodoTitle("買い物") → .getValue() → "買い物"（String）
取得時: "買い物"（String）  → new TodoTitle() → TodoTitle("買い物")
```

---

## Phase 6 の学び

```
┌────────────────────────────────────────────────────────────────┐
│ Infrastructure層の役割                                          │
│                                                                │
│  ・Domain層 と DB をつなぐ「翻訳者」                              │
│  ・Domain Entity ↔ JPA Entity の変換を担当                      │
│  ・DB技術（JPA）を Domain層 に漏らさない                         │
│                                                                │
│ 重要な気づき:                                                   │
│  ・implements と 使う は別物！                                  │
│    - implements TodoRepository（Domain層のルールに従う）         │
│    - 使う TodoJpaRepository（DB操作を委譲する）                  │
└────────────────────────────────────────────────────────────────┘
```

---

## 作成したファイル一覧

```
src/main/java/com/example/todo/infrastructure/
├── jpa/
│   └── TodoJpaEntity.java       ← DBテーブルと対応（課題6-1）
└── repository/
    ├── TodoJpaRepository.java   ← Spring Data JPA（課題6-2）
    └── TodoRepositoryImpl.java  ← 橋渡し役（課題6-3）
```

---

## 次のフェーズ

**Phase 7: DB 実装** または **Phase 8: API 実装** へ進む
