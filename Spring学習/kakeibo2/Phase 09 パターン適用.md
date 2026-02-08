# Phase 09: パターン適用（Factory / Strategy）

## 概要

デザインパターンを適用して、コードの保守性・拡張性を向上させる。

---

## 作成するファイル

| パッケージ | ファイル | 役割 |
|-----------|---------|------|
| `domain/factory` | `TransactionFactory` | Transaction の生成ロジックを集約 |
| `domain/strategy` | `SummaryStrategy` | 集計処理のインターフェース |
| `domain/strategy` | `TotalSummaryStrategy` | 全取引の合計 |
| `domain/strategy` | `TypeSummaryStrategy` | 収入/支出ごとの合計 |
| `domain/strategy` | `SummaryStrategyProvider` | 適切な Strategy を選択して返す |

---

## 理解ポイント

### Factory パターンとは？

**「オブジェクトの作り方を一箇所にまとめる」**

#### Before（Factory なし）

```java
// UseCase に VO 変換が散らばる
public Transaction create(...) {
    TransactionType tranType = TransactionType.from(type);
    Amount amt = new Amount(amount);
    CategoryId catId = new CategoryId(categoryId);
    Transaction transaction = new Transaction(null, tranType, amt, ...);
    return transactionRepository.save(transaction);
}

public Transaction update(...) {
    // また同じ VO 変換を書く...
    TransactionType tranType = TransactionType.from(type);
    Amount amt = new Amount(amount);
    // ...
}
```

問題点：
- 同じ生成ロジックが複数箇所に存在
- 生成ルールを変えるとき、全箇所を修正する必要がある
- 修正漏れのリスク

#### After（Factory あり）

```java
// UseCase は Factory に任せる
public Transaction create(...) {
    categoryRepository.findById(...).orElseThrow(...);
    Transaction transaction = transactionFactory.createNew(type, amount, ...);
    return transactionRepository.save(transaction);
}
```

```java
// Factory が生成ロジックを一手に引き受ける
@Component
public class TransactionFactory {
    public Transaction createNew(...) {
        TransactionType tranType = TransactionType.from(type);
        Amount amt = new Amount(amount);
        CategoryId catId = new CategoryId(categoryId);
        return new Transaction(null, tranType, amt, catId, ...);
    }
}
```

メリット：
- 生成ロジックが一箇所に集約
- ルールを変えるときは Factory だけ修正
- UseCase がシンプルになる

---

### Factory の責務

| Factory がやること | Factory がやらないこと |
|-------------------|----------------------|
| オブジェクトを生成する | 保存する（save） |
| VO への変換 | 存在チェック |
| 生成時のログ出力 | ビジネスルールの判断 |

**Factory は「作る」だけ。保存は UseCase の責務。**

---

### Strategy パターンとは？

**「処理のやり方を切り替え可能にする」**

#### Before（if-else 地獄）

```java
public int summarize(String type, List<Transaction> transactions) {
    if (type.equals("total")) {
        // 全合計ロジック...
    } else if (type.equals("income")) {
        // 収入合計ロジック...
    } else if (type.equals("expense")) {
        // 支出合計ロジック...
    }
    // 新しい集計方法が増えるたびに else if 追加...
}
```

問題点：
- メソッドがどんどん長くなる
- 新しい処理を追加するたびに既存コードを修正
- テストしにくい

#### After（Strategy パターン）

```java
// インターフェース
public interface SummaryStrategy {
    int summarize(List<Transaction> transactions);
}

// 実装クラス（全合計）
public class TotalSummaryStrategy implements SummaryStrategy {
    @Override
    public int summarize(List<Transaction> transactions) {
        return transactions.stream()
            .mapToInt(t -> t.getAmount().getValue())
            .sum();
    }
}

// 実装クラス（タイプ別）
public class TypeSummaryStrategy implements SummaryStrategy {
    private final TransactionType targetType;

    public TypeSummaryStrategy(TransactionType targetType) {
        this.targetType = targetType;
    }

    @Override
    public int summarize(List<Transaction> transactions) {
        return transactions.stream()
            .filter(t -> t.getType() == targetType)
            .mapToInt(t -> t.getAmount().getValue())
            .sum();
    }
}
```

---

### Provider（Strategy を選ぶ役割）

```java
@Component
public class SummaryStrategyProvider {
    public SummaryStrategy getStrategy(String type) {
        return switch (type) {
            case "total" -> new TotalSummaryStrategy();
            case "income" -> new TypeSummaryStrategy(TransactionType.INCOME);
            case "expense" -> new TypeSummaryStrategy(TransactionType.EXPENSE);
            default -> throw new IllegalArgumentException("Unknown type: " + type);
        };
    }
}
```

**文字列で「どの集計方法を使うか」を指定できる。**

---

### OCP（開放閉鎖原則）との関係

Strategy パターンは **OCP** を実現する。

**Open-Closed Principle = 拡張に対して開いていて、修正に対して閉じている**

| やりたいこと | どうする？ |
|-------------|-----------|
| 新しい集計方法を追加 | 新しい Strategy クラスを **追加** |
| 既存コードを修正 | **しない** |

例：「月別集計」を追加したい場合

```java
// 新しいクラスを追加するだけ
public class MonthlySummaryStrategy implements SummaryStrategy {
    private final YearMonth targetMonth;
    // ...
}
```

既存の `TotalSummaryStrategy` や `TypeSummaryStrategy` は触らない！

---

### 拡張 for 文での書き方（参考）

Stream API が使えない古い現場向け：

```java
// Stream API（モダン）
return transactions.stream()
    .filter(t -> t.getType() == targetType)
    .mapToInt(t -> t.getAmount().getValue())
    .sum();

// 拡張 for 文（クラシック）
int total = 0;
for (Transaction t : transactions) {
    if (t.getType() == targetType) {
        total += t.getAmount().getValue();
    }
}
return total;
```

**どちらも書けるようにしておく。**

---

## DI と @Component の関係

| アノテーション | 役割 |
|---------------|------|
| `@Component` | Spring に「このクラスを管理してね」と伝える |
| `@Service` | UseCase 用（Component の一種） |
| `@Repository` | Repository 実装用（Component の一種） |

Factory や Provider に `@Component` を付けることで、UseCase から DI で受け取れる。

```java
@Service
public class TransactionUseCase {
    private final TransactionFactory factory;  // ← DI で受け取る

    public TransactionUseCase(TransactionFactory factory) {
        this.factory = factory;
    }
}
```

---

## 集計 API

### エンドポイント

```
GET /api/transactions/summary?type=total
GET /api/transactions/summary?type=income
GET /api/transactions/summary?type=expense
```

### レスポンス

```json
6000
```

---

## まとめ

| パターン | 目的 | 適用場面 |
|---------|------|---------|
| Factory | 生成ロジックを一箇所に集約 | オブジェクト生成が複雑なとき |
| Strategy | 処理を切り替え可能にする | 条件分岐が増えそうなとき |

**共通点**: どちらも「変更に強いコード」を作るためのパターン。

---

## 次のステップ

Phase 10: テスト
- Domain テスト（VO のバリデーション）
- UseCase テスト（モック使用）
- Controller テスト（MockMvc 使用）
