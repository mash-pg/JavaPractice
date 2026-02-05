# Phase 9: パターン適用（集計・フィルタ）

> **目的**: Strategy / Factory パターンを家計簿ドメインに適用する。

---

## 実装したファイル一覧

### Factory パターン

| ファイル | 場所 | 役割 |
|---------|------|------|
| `TransactionFactory.java` | domain/factory/ | Transaction 生成ルールを集約 |

### Strategy パターン

| ファイル | 場所 | 役割 |
|---------|------|------|
| `SummaryStrategy.java` | domain/strategy/ | 集計戦略のインターフェース |
| `MonthlySummaryStrategy.java` | domain/strategy/ | 月次集計（収入・支出・残高） |
| `CategorySummaryStrategy.java` | domain/strategy/ | カテゴリ別支出集計 |

### 型安全な Result クラス

| ファイル | 場所 | 役割 |
|---------|------|------|
| `SummaryResult.java` | domain/strategy/ | マーカーインターフェース |
| `MonthlySummaryResult.java` | domain/strategy/ | 月次集計の結果 |
| `CategorySummaryResult.java` | domain/strategy/ | カテゴリ別集計の結果（内部クラス CategoryTotal を含む） |

### UseCase

| ファイル | 場所 | 役割 |
|---------|------|------|
| `GetMonthlySummaryUseCase.java` | application/usecase/ | type パラメータで Strategy を切り替え |

### DTO

| ファイル | 場所 | 役割 |
|---------|------|------|
| `MonthlySummaryResponse.java` | presentation/dto/ | 月次集計レスポンス |
| `CategorySummaryResponse.java` | presentation/dto/ | カテゴリ別集計レスポンス |

---

## 学んだこと

### 1. Factory パターン

| トピック | 学んだこと |
|---------|-----------|
| 目的 | オブジェクト生成ルールを一箇所に集約 |
| 配置 | `domain/factory/` パッケージ |
| static vs instance | Factory は `static` メソッド、UseCase は instance（DI が必要） |
| デフォルト値 | transactionDate が null なら `LocalDate.now()` |
| 自動付与 | createdAt は常に `LocalDateTime.now()` |

**コード例:**

```java
public class TransactionFactory {
    public static Transaction create(
            TransactionType type,
            Amount amount,
            CategoryId categoryId,
            String description,
            LocalDate transactionDate
    ) {
        LocalDate date = (transactionDate != null) ? transactionDate : LocalDate.now();
        return new Transaction(
            null, amount, type, categoryId, description, date, LocalDateTime.now()
        );
    }
}
```

### 2. Strategy パターン

| トピック | 学んだこと |
|---------|-----------|
| 目的 | アルゴリズムを実行時に切り替え可能にする |
| ToDo との比較 | ToDo はソート戦略、家計簿は集計戦略 |
| インターフェース | `SummaryStrategy` で `calculate()` を定義 |
| 具象クラス | `MonthlySummaryStrategy`, `CategorySummaryStrategy` |
| 選択ロジック | UseCase で `type` パラメータに応じて new する |

**コード例:**

```java
// インターフェース
public interface SummaryStrategy {
    SummaryResult calculate(List<Transaction> transactions, int year, int month);
}

// UseCase での選択
SummaryStrategy strategy;
if ("category".equals(type)) {
    strategy = new CategorySummaryStrategy();
} else {
    strategy = new MonthlySummaryStrategy();
}
return strategy.calculate(transactions, year, month);
```

### 3. 型安全な設計（マーカーインターフェース）

| トピック | 学んだこと |
|---------|-----------|
| 問題 | `Object` を返すと `@SuppressWarnings("unchecked")` が必要 |
| 解決策 | マーカーインターフェース `SummaryResult` を導入 |
| メリット | `instanceof` で安全にキャスト、IDE の補完が効く |
| Controller | `SummaryResult` を受け取り、`instanceof` で分岐 |

**Before（型安全でない）:**

```java
@SuppressWarnings("unchecked")
Map<Long, Integer> map = (Map<Long, Integer>) result;
```

**After（型安全）:**

```java
if (result instanceof CategorySummaryResult) {
    CategorySummaryResult categoryResult = (CategorySummaryResult) result;
    // getter でアクセス
}
```

### 4. Stream API の活用

| メソッド | 用途 |
|---------|------|
| `filter()` | 年月でフィルタリング |
| `mapToInt()` | Amount から int に変換 |
| `sum()` | 合計を計算 |
| `Collectors.groupingBy()` | カテゴリ ID でグルーピング |
| `Collectors.summingInt()` | グループごとに合計 |

**コード例:**

```java
// カテゴリ別集計
Map<Long, Integer> grouped = filtered.stream()
    .filter(t -> t.getType() == TransactionType.EXPENSE)
    .collect(Collectors.groupingBy(
        t -> t.getCategoryId().getValue(),
        Collectors.summingInt(t -> t.getAmount().getValue())
    ));
```

### 5. JSON キー名の決まり方

| トピック | 学んだこと |
|---------|-----------|
| Jackson | フィールド名ではなく getter からキー名を生成 |
| 例 | `getCategoryId()` → `"categoryId"` |
| 命名規則 | Java は camelCase → JSON も camelCase |

---

## 躓いたポイント

| 問題 | 原因 | 解決策 |
|------|------|--------|
| インターフェースと実装の戻り値型不一致 | `Map<String, Object>` vs `Object` | インターフェースを柔軟にし、後で型安全に修正 |
| UseCase を static で呼ぼうとした | Factory と UseCase の違いを理解していなかった | UseCase は DI で注入し instance メソッドで呼ぶ |
| `@SuppressWarnings` の警告 | Object を直接キャストしていた | マーカーインターフェース + instanceof で解決 |
| JSON キー名が想定と違った | フィールド名が `CategoryId`（大文字）だった | `categoryId`（小文字）に修正 |

---

## ToDo との比較

| 観点 | ToDo アプリ | 家計簿アプリ |
|------|-----------|------------|
| Factory | `TodoFactory` | `TransactionFactory` |
| Strategy 対象 | ソート（並び替え） | 集計（計算） |
| Strategy 数 | 2（CreatedAt, Id） | 2（Monthly, Category） |
| 戻り値 | `List<Todo>`（同じ型） | `SummaryResult`（異なる型） |
| 型安全性 | 同じ型を返すので問題なし | マーカーインターフェースで対応 |

---

## 設計の流れ（層の依存関係）

```
Controller (Presentation)
    ↓ SummaryResult を受け取る
    ↓ instanceof で分岐して DTO に変換
UseCase (Application)
    ↓ type パラメータで Strategy を選択
    ↓ strategy.calculate() を呼ぶ
Strategy (Domain)
    ↓ 計算ロジックを実行
    ↓ SummaryResult を返す
Result (Domain)
```

---

## 自己チェック

| チェック | 内容 |
|---------|------|
| [x] | Factory を自力で設計・実装できた |
| [x] | Strategy パターンを集計に適用できた |
| [x] | マーカーインターフェースで型安全にできた |
| [x] | Stream API（groupingBy, summingInt）を使えた |
| [x] | `@SuppressWarnings` なしでキャストできた |

---

## API 動作確認

```
GET /api/transactions/summary?year=2026&month=1
→ 月次サマリー（totalIncome, totalExpense, balance）

GET /api/transactions/summary?year=2026&month=1&type=category
→ カテゴリ別サマリー（categoryId, totalAmount のリスト）
```

---

*Phase 9 完了。次は Phase 10: テスト へ進む。*
