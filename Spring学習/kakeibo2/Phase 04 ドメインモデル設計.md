# Phase 04 ドメインモデル設計

> **目的**: 家計簿のビジネスルールをコードで表現する

---

## 実装内容

### Value Object（値オブジェクト）

| VO | ルール | ファイル |
|----|--------|---------|
| `Amount` | 0より大きい | `domain/valueobject/Amount.java` |
| `TransactionId` | null禁止 | `domain/valueobject/TransactionId.java` |
| `CategoryId` | null禁止 | `domain/valueobject/CategoryId.java` |
| `CategoryName` | 空禁止、最大50文字 | `domain/valueobject/CategoryName.java` |
| `TransactionType` | INCOME / EXPENSE | `domain/valueobject/TransactionType.java` |

### Entity（エンティティ）

| Entity | フィールド | ファイル |
|--------|-----------|---------|
| `Transaction` | id, type, amount, categoryId, description, transactionDate, createdAt | `domain/entity/Transaction.java` |
| `Category` | id, categoryName | `domain/entity/Category.java` |

### Exception（例外）

| 例外 | 用途 |
|------|------|
| `InvalidAmountException` | 金額が不正（0以下） |
| `InvalidCategoryNameException` | カテゴリ名が不正（空、50文字超） |
| `TransactionNotFoundException` | 収支が見つからない |
| `CategoryNotFoundException` | カテゴリが見つからない |
| `CategoryInUseException` | 使用中のカテゴリを削除しようとした |

---

## 理解ポイント

### VO と Entity の違い

| | VO | Entity |
|--|-----|--------|
| **比較基準** | 値 | ID |
| **例** | Amount(1000) = Amount(1000) | Transaction(id=1) = Transaction(id=1) |
| **特徴** | 不変（immutable） | 状態が変わりうる |

### なぜコンストラクタでバリデーション？

**不正な状態のオブジェクトを作らせないため**

```java
// これは作れない（例外が飛ぶ）
new Amount(0);      // InvalidAmountException
new Amount(-100);   // InvalidAmountException

// これは作れる
new Amount(1000);   // OK
```

### isBlank() vs equals("")

| メソッド | `""` | `"   "` | `"食費"` |
|---------|------|---------|---------|
| `equals("")` | true | false | false |
| `isBlank()` | true | true | false |

空白のみも弾きたい場合は `isBlank()` を使う。

---

## 躓いたポイント

| 問題 | 解決 |
|------|------|
| `value >= 0` と書いてしまった | 「0以下を弾く」なら `value <= 0` |
| `NotFound` 例外を使おうとした | 値が不正なら `Invalid` を使う |
| `createdAt` を `createAt` とタイポ | 過去形は `created` |

---

## Git 操作

```
ブランチ: feature/phase04-domain
PR: #1
マージ: ✅ 完了
```

---

## 関連ドキュメント

- [[Spring Boot 家計簿 API 学習ロードマップ v2]]
- [[Phase 03 レイヤー]]
- [[Phase 05 ユースケース設計]]（次）
