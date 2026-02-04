# Phase 4: ドメインモデル設計

> **目的**: 家計簿ドメインのビジネスルールをコードで表現する。

---

## シニア × ジュニアのやり取り記録

### 課題の提示（シニア）

> 以下の3ステップで進めてください：
> 1. Value Object（5つ）を作成する
> 2. ドメイン例外を作成する
> 3. Entity（2つ）を作成する

---

## Step 1: Value Object

### TransactionId・Amount（1回目提出）

#### ジュニアの回答

```java
// TransactionId
public class TransactionId {
    private final Long transactionId;  // ← フィールド名が冗長
    public TransactionId(Long transactionId) {
        if(transactionId == null) {
            throw new IllegalArgumentException("transactionIdを設定してください");
        }
        this.transactionId = transactionId;
    }
}

// Amount
public class Amount {
    private final Long amount;  // ← Long は過剰
    public Amount(Long amount) {
        if(amount == null) {
            throw new IllegalArgumentException("amountを設定してください");
        }
        if(amount <= 0) {
            throw new InvalidAmountException("金額が０以下です。");
        }
        this.amount = amount;
    }
}
```

#### シニアのレビュー

| # | 指摘 | 理由 |
|---|------|------|
| 1 | フィールド名を `value` に統一すべき | VO クラスの中なので `transactionId` は冗長 |
| 2 | Amount の型は `Long` ではなく `Integer` が適切 | 家計簿の金額に Long（最大約920京）は不要 |

#### ジュニアの判断

- **Integer を採用**: `int` だと null チェックができないので `Integer` にした → 良い判断
- **IllegalArgumentException のまま**: 単純な null チェックにドメイン例外は過剰（YAGNI） → 良い判断

---

### TransactionType・CategoryId・CategoryName（まとめて提出）

#### ジュニアの回答（1回目）

```java
// TransactionType
public enum TransactionType {
    INCOME("収入"),
    EXPENSE("支出");
    private TransactionType(String value) {}  // ← value を保存していない
}

// CategoryId
public class CategoryId {
    public CategoryId(Long value) {
        if(value == null) {
            throw new IllegalArgumentException("収支が見つかりません");  // ← メッセージが違う
        }
    }
}

// CategoryName
public class CategoryName {
    public CategoryName(String value) {
        if(value.isEmpty()) {
            throw new CategoryNotFoundException("空文字が入っています");  // ← 例外の種類が違う
        }
        if(value.length() > 50) {
            throw new InvalidCategoryException("最大50文字までです");
        }
    }
}
```

#### シニアのレビュー

| # | 指摘 | 詳細 |
|---|------|------|
| 1 | TransactionType: value を保存していない | YAGNI で日本語ラベルなしのシンプル版を推奨 |
| 2 | CategoryId: メッセージが「収支が見つかりません」 | カテゴリなので「カテゴリIDを設定してください」が適切 |
| 3 | CategoryName: 空文字チェックに `CategoryNotFoundException` を使っている | 入力値の不正は `InvalidCategoryException`。NotFoundException は DB 検索用 |
| 4 | CategoryName: null チェックがない | `value.isEmpty()` の前に null が来ると NullPointerException |

### 詰まりポイント: 例外の使い分け

**何に詰まったか：**
`CategoryNotFoundException` と `InvalidCategoryException` の使い分けが曖昧だった。

**2回目の提出でも間違えた：**
null チェックに `InvalidCategoryException`（正しい）を使えたが、空文字チェックに `CategoryNotFoundException`（間違い）を使ってしまった。

**学んだこと：**

| 例外の種類 | いつ使う | どこで使う |
|-----------|---------|-----------|
| `Invalid〇〇Exception` | 入力値がルール違反 | **VO のコンストラクタ** |
| `〇〇NotFoundException` | DB にリソースが存在しない | **UseCase** |
| `〇〇InUseException` | 業務ルール違反（使用中など） | **UseCase** |

> **覚え方**: VO は「値が正しいか」をチェック → Invalid系。UseCase は「リソースが存在するか」「業務的にOKか」をチェック → NotFound系, InUse系

#### 最終版（合格）

```java
// TransactionType - YAGNI でシンプルに
public enum TransactionType {
    INCOME,
    EXPENSE;
}

// CategoryId - メッセージ修正
public CategoryId(Long value) {
    if(value == null) {
        throw new IllegalArgumentException("カテゴリIDを設定してください");
    }
    this.value = value;
}

// CategoryName - 例外を統一、null チェック追加
public CategoryName(String value) {
    if(value == null) {
        throw new InvalidCategoryException("空文字が入っています");
    }
    if(value.isEmpty() || value.length() > 50) {
        throw new InvalidCategoryException("最大50文字までまたは空文字です" + value);
    }
    this.value = value;
}
```

---

## Step 2: ドメイン例外

### 作成した例外クラス

| 例外 | 継承元 | 用途 |
|------|--------|------|
| `InvalidAmountException` | RuntimeException | 金額が 0 以下のとき |
| `InvalidCategoryException` | RuntimeException | カテゴリ名が不正なとき |
| `TransactionNotFoundException` | RuntimeException | 収支が見つからないとき（Phase 5 で使用） |
| `CategoryNotFoundException` | RuntimeException | カテゴリが見つからないとき（Phase 5 で使用） |

**Phase 5 で追加予定:**
- `CategoryInUseException` - 使用中のカテゴリを削除しようとしたとき

---

## Step 3: Entity

### Transaction（3回のレビューで合格）

#### 1回目の提出 → 不合格

| # | 問題 | 詳細 |
|---|------|------|
| 1 | デフォルトコンストラクタで全部 null | Entity は有効な状態で生成されるべき |
| 2 | `getId(TransactionId id)` | ゲッターに引数は不要 |
| 3 | `updatedAt` がある | ロードマップに定義なし（YAGNI） |
| 4 | `final` の使い方 | 不変にすべきフィールドを検討すべき |

#### 2回目の提出 → 惜しい

- 引数付きコンストラクタにできた
- ただし `amount`, `type`, `categoryId` の3つしか受け取っていなかった
- `id`, `description`, `transactionDate`, `createdAt` が null 固定

#### 3回目の提出 → 合格

```java
public class Transaction {
    private final TransactionId id;
    private TransactionType type;
    private Amount amount;
    private final CategoryId categoryId;
    private String description;
    private LocalDate transactionDate;
    private LocalDateTime createdAt;

    public Transaction(
            TransactionId id,
            Amount amount,
            TransactionType type,
            CategoryId categoryId,
            String description,
            LocalDate transactionDate,
            LocalDateTime createdAt) {
        this.id = id;
        this.type = type;
        this.amount = amount;
        this.categoryId = categoryId;
        this.description = description;
        this.transactionDate = transactionDate;
        this.createdAt = createdAt;
    }
    // ゲッター省略
}
```

### 詰まりポイント: コンストラクタの設計

**何に詰まったか：**
デフォルトコンストラクタで全フィールドを null にしてしまった。

**なぜダメか：**
VO でバリデーションしているのに、Entity のコンストラクタが全部 null を許したら VO の意味がなくなる。

**学んだこと：**
- Entity は「有効な状態」で生成されるべき
- 必要なフィールドはコンストラクタで受け取る
- 新規作成用コンストラクタ（idなし）は Phase 5 or 9 で必要になったら追加する（YAGNI）

### ToDo との比較: コンストラクタのパターン

```
ToDo: 2つのコンストラクタ
  1. 新規作成用（idなし、デフォルト値を設定）
  2. DB復元用（全フィールド）

家計簿: 今は1つだけ（DB復元用 = 全フィールド）
  → 新規作成用は Phase 5 or 9 で YAGNI 判断に基づき追加予定
```

---

### Category（1回のレビューで合格）

#### 1回目の提出 → 指摘1つ

フィールド名が `categoryId`, `categoryName` と冗長だった。Transaction では `id`, `type`, `amount` とシンプルにしていたので一貫性を持たせて `id`, `name` に修正。

#### 最終版

```java
public class Category {
    private final CategoryId id;
    private CategoryName name;

    public Category(CategoryId id, CategoryName name) {
        this.id = id;
        this.name = name;
    }
    // ゲッター省略
}
```

---

## 完成した domain パッケージ構成

```
domain/
├── entity/
│   ├── Transaction.java    ← 収支エンティティ（7フィールド）
│   └── Category.java       ← カテゴリエンティティ（2フィールド）
├── valueobject/
│   ├── TransactionId.java  ← null 禁止
│   ├── Amount.java         ← null 禁止、0 より大きい
│   ├── TransactionType.java ← enum: INCOME / EXPENSE
│   ├── CategoryId.java     ← null 禁止
│   └── CategoryName.java   ← null 禁止、空禁止、最大50文字
└── exception/
    ├── InvalidAmountException.java
    ├── InvalidCategoryException.java
    ├── TransactionNotFoundException.java
    └── CategoryNotFoundException.java
```

---

## YAGNI 判断の記録

| 項目 | 判断 | 理由 |
|------|------|------|
| TransactionType に日本語ラベル | 不要（後で追加） | 今は enum の値だけで十分 |
| Transaction の新規作成用コンストラクタ | 後で追加 | Phase 5 or 9 で必要になったとき |
| `updatedAt` フィールド | 不要 | ロードマップに定義なし |
| `CategoryInUseException` | 後で追加 | Phase 5 で必要になったとき |

---

## 自己チェック

| チェック | 内容 | 結果 |
|---------|------|------|
| ○ | VO のコンストラクタでバリデーションを書けた | 全5つで正しくバリデーション |
| ○ | Entity のフィールドに VO を使えた | プリミティブ型ではなく VO を使用 |
| ○ | ドメイン例外を自分で設計できた | Invalid系 と NotFound系の使い分けを理解 |

### ToDo を参照した箇所

| 箇所 | 何を確認したか |
|------|---------------|
| Todo.java のコンストラクタ | 新規作成用と DB 復元用の2パターンがあることを確認 |

---

## 次の Phase へ

Phase 4 完了。次は **Phase 5: ユースケース設計（Application 層）** へ進む。
- UseCase クラスの実装
- `CategoryInUseException` の追加
- Repository インターフェースとの連携

> **Phase 5 のポイント**: Controller は薄く、業務判断は UseCase に集約する。@Transactional は UseCase に付ける。
