# Phase 06 Repository インターフェース

> **目的**: DIP（依存性逆転の原則）を適用し、domain が infrastructure に依存しない設計にする

---

## 実装内容

### Repository インターフェース

| Repository | メソッド | ファイル |
|------------|---------|---------|
| `CategoryRepository` | findAll, findById, save, deleteById | `domain/repository/CategoryRepository.java` |
| `TransactionRepository` | findAll, findById, save, deleteById | `domain/repository/TransactionRepository.java` |

### 配置場所

```
domain/
├── entity/
├── exception/
├── repository/          ← インターフェースはここ（domain 層）
│   ├── CategoryRepository.java
│   └── TransactionRepository.java
└── valueobject/

infrastructure/
└── repository/          ← 実装クラスはここ（Phase 7 で作成）
    ├── CategoryRepositoryImpl.java
    └── TransactionRepositoryImpl.java
```

---

## 理解ポイント

### なぜ Interface を domain に置く？

**DIP（依存性逆転の原則）を適用するため**

```
通常の依存:
  UseCase → Repository実装（infrastructure）
  ↑ DB が変わったら UseCase も影響を受ける

DIP を適用:
  UseCase → Repository Interface（domain）
                    ↑
            Repository実装（infrastructure）
  ↑ DB が変わっても UseCase は影響を受けない！
```

**依存の方向が逆転する:**
- infrastructure が domain に依存する（domain が infrastructure に依存しない）

### なぜ VO を引数にする？（Long ではなく CategoryId）

**型安全（Type Safety）のため**

```java
// Long を使った場合
void deleteCategory(Long id);
void deleteTransaction(Long id);

deleteCategory(transactionId);  // ← 間違えてもコンパイル通る！

// VO を使った場合
void deleteCategory(CategoryId id);
void deleteTransaction(TransactionId id);

deleteCategory(transactionId);  // ← コンパイルエラー！型が違う！
```

### Optional を返す理由

**「見つからない」ケースの判断を呼び出し側に委ねる**

```java
// Repository
Optional<Category> findById(CategoryId id);

// UseCase
categoryRepository.findById(categoryId)
    .orElseThrow(() -> new CategoryNotFoundException("カテゴリが存在しません"));
```

- `Optional.empty()` なら見つからなかった
- 呼び出し側が例外を投げるか、デフォルト値を使うか決められる

---

## インターフェースの書き方

```java
public interface CategoryRepository {
    List<Category> findAll();
    Optional<Category> findById(CategoryId id);  // VO を使う
    Category save(Category category);
    void deleteById(CategoryId id);              // VO を使う
}
```

**ポイント:**
- メソッドは自動的に `public abstract`（省略可）
- 実装は Phase 7 で `infrastructure` 層に作成

---

## 躓いたポイント

| 問題 | 解決 |
|------|------|
| Repository を infrastructure に置こうとした | インターフェースは domain に置く（DIP） |
| Long を引数にしようとした | VO（CategoryId, TransactionId）を使う（型安全） |

---

## Phase 5 と同時に実装した理由

UseCase が Repository を使うため、先に Repository インターフェースが必要だった。

```java
@Service
public class CategoryUseCase {
    private final CategoryRepository categoryRepository;  // ← これが必要
    // ...
}
```

---

## Git 操作

```
ブランチ: feature/phase05-usecase（Phase 5 と同時）
PR: #2
マージ: ✅ 完了
```

---

## 関連ドキュメント

- [[Spring Boot 家計簿 API 学習ロードマップ v2]]
- [[Phase 05 ユースケース設計]]（同時に実装）
- [[Phase 07 DB・JPA 実装]]（次：Repository の実装クラスを作成）
