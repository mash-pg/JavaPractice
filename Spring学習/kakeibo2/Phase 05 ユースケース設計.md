# Phase 05 ユースケース設計

> **目的**: 業務ロジックを UseCase（Service）に集約し、再利用可能にする

---

## 実装内容

### UseCase クラス

| UseCase | メソッド | ファイル |
|---------|---------|---------|
| `CategoryUseCase` | findAll, create, delete | `application/usecase/CategoryUseCase.java` |
| `TransactionUseCase` | findAll, create, update, deleteById | `application/usecase/TransactionUseCase.java` |

### アノテーション

| アノテーション | 用途 |
|--------------|------|
| `@Service` | Spring に Service として登録（DI 可能にする） |
| `@Transactional` | 変更系メソッドにトランザクション管理を適用 |

### TransactionType の改善

| 追加内容 | 説明 |
|---------|------|
| `from(String value)` | 文字列から Enum に変換する static メソッド |
| `InvalidTransactionTypeException` | 不正な取引種別のドメイン例外 |

---

## 理解ポイント

### なぜ UseCase を作る？Controller に書かないの？

**業務ロジックを再利用可能にするため**

```
Controller A ──→ ┌──────────────┐
                 │   UseCase    │ ←─ ロジックは1箇所
Controller B ──→ │  (Service)   │
                 └──────────────┘
バッチ処理   ──→
```

Controller に直接書くと、同じ処理を使いたい場所が増えるたびにコピペが必要になる。

### UseCase は何を受け取って何を返す？

| 項目 | 内容 | 理由 |
|------|------|------|
| **引数** | プリミティブ（int, Long, String） | Controller がそのまま渡せる |
| **戻り値** | Entity / VO | 呼び出し側が必要な情報を選べる |

```java
// UseCase
public Transaction create(String type, int amount, Long categoryId, ...) {
    TransactionType txType = TransactionType.from(type);  // VO に変換
    // ...
    return transaction;  // Entity を返す
}
```

### @Transactional を UseCase に付ける理由

**業務処理の単位でトランザクションを管理するため**

- UseCase のメソッドが「業務処理の単位」
- 途中で失敗したら全部ロールバック

### Enum の from() メソッド

```java
public static TransactionType from(String value) {
    try {
        return valueOf(value.toUpperCase());
    } catch (IllegalArgumentException e) {
        throw new InvalidTransactionTypeException("不正な取引種別です : " + value);
    }
}
```

**メリット:**
1. ドメイン例外で統一できる（IllegalArgumentException ではなく）
2. 大文字小文字を吸収できる（"income" → INCOME）
3. 他の VO と一貫性がある

---

## DI（依存性注入）

### コンストラクタインジェクション

```java
@Service
public class TransactionUseCase {
    private final TransactionRepository transactionRepository;
    private final CategoryRepository categoryRepository;

    // @Autowired 不要（コンストラクタが1つなら自動）
    public TransactionUseCase(TransactionRepository transactionRepository,
                              CategoryRepository categoryRepository) {
        this.transactionRepository = transactionRepository;
        this.categoryRepository = categoryRepository;
    }
}
```

### 3つのインジェクション方式

| 方式 | final | 推奨度 |
|------|-------|--------|
| コンストラクタ | ✓ 使える | **推奨** |
| フィールド (`@Autowired`) | ✗ 使えない | 非推奨 |
| セッター | ✗ 使えない | オプショナルな依存に |

---

## 躓いたポイント

| 問題 | 解決 |
|------|------|
| `new TransactionType.from(type)` と書いた | static メソッドに `new` は不要。`TransactionType.from(type)` |
| `orElse()` で例外を投げようとした | `orElseThrow()` を使う |
| `jakarta.transaction.Transactional` を使った | `org.springframework.transaction.annotation.Transactional` を使う |

---

## Git 操作

```
ブランチ: feature/phase05-usecase
PR: #2
マージ: ✅ 完了
```

### 学んだ Git 操作

| 状況 | 対処法 |
|------|--------|
| 間違ったディレクトリでブランチを切った | 正しいディレクトリで切り直す |
| コミット前に変更を持ったままブランチ移動したい | `git checkout -b 新ブランチ` でOK |
| レビュー指摘を修正したい | 同じブランチで修正 → `git push` でPRに反映 |

---

## 関連ドキュメント

- [[Spring Boot 家計簿 API 学習ロードマップ v2]]
- [[Phase 04 ドメインモデル設計]]（前）
- [[Phase 06 Repository インターフェース]]（同時に実装）
- [[Phase 07 DB・JPA 実装]]（次）
