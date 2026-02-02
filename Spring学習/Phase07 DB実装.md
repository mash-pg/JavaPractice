# Phase 7: DB・JPA 実装

> **目的**: 実際のデータ保存処理を実装する。テーブル設計に基づいてカラムを追加し、トランザクション境界を正しく設定する。

---

## 学んだこと（まとめ）

### 1. VO（Value Object）を作る判断基準

| 判断ポイント | VO が必要 | VO 不要 |
|-------------|----------|---------|
| ビジネスルールがある | `TodoTitle`（空禁止、100文字制限） | `description`（制限なし） |
| 型で保証される | - | `LocalDate`, `LocalDateTime` |
| 任意項目（null OK） | 複雑になる | シンプルに String/LocalDate |

**YAGNI の原則**: 今ルールがないなら VO は作らない。必要になったら追加する。

### 2. JPA Entity には VO を直接持たせない

```java
// ❌ NG - JPA は TodoDueDate を保存できない
private TodoDueDate duedate;

// ✅ OK - JPA が理解できる型を使う
private LocalDate dueDate;
```

変換は `TodoRepositoryImpl` の中で行う。

### 3. @Transactional は UseCase に付ける

```java
@Service
@Transactional  // ← ここに付ける
public class UpdateTodoUseCase {
    // findById → 変更 → save が1つのトランザクション
}
```

**理由**: UseCase が「業務処理の単位」だから。

### 4. 命名規則

| DB カラム名 | Java フィールド名 |
|------------|------------------|
| `created_at` | `createdAt` |
| `due_date` | `dueDate` |

Java はキャメルケース、DB はスネークケース。JPA が自動でマッピングしてくれる。

---

## 実装したクラス

### Value Object（追加）

| クラス | 役割 |
|--------|------|
| `TodoPriority` | 優先度（HIGH / MEDIUM / LOW） |

### 変更したクラス

| クラス | 追加したフィールド |
|--------|-------------------|
| `Todo.java` | priority, description, dueDate, createdAt, updatedAt |
| `TodoJpaEntity.java` | 同上 |
| `TodoRepositoryImpl.java` | 変換メソッドを更新 |

### @Transactional を追加

| UseCase | 理由 |
|---------|------|
| `CreateTodoUseCase` | 書き込み処理 |
| `UpdateTodoUseCase` | 取得→変更→保存 |
| `CompleteTodoUseCase` | 取得→変更→保存 |

---

## 課題と解答

### 課題 1: Priority の追加

**課題**: `TodoPriority` enum を作成し、Todo に追加する

**私の解答**:
```java
// TodoPriority.java
public enum TodoPriority {
    HIGH,
    MEDIUM,
    LOW
}
```

**レビュー結果**:
- 最初 `HIGT` とタイポしていた → `HIGH` に修正
- `toJpaEntity` で priority のセットを忘れていた → 追加

**学び**: 変換メソッドは `toJpaEntity` と `toDomainEntity` の両方を確認する！

---

### 課題 2: description の追加

**課題**: description カラムを追加する。VO は必要か？

**私の判断**:
> どんな値も入れることが可能なため、ビジネスルールはそのままでよい。
> よって VO を作る必要はない。
> 今後、説明欄に何かしらルールが必要になった場合は YAGNI に則って VO を新規で作ればいい。

**シニアの評価**: 「これが正しい考え方。実務でもこの判断ができると強い。」

**指摘された点**:
```java
// ❌ 意図が不明確
this.description = description;  // 引数にないのに代入

// ✅ 明示的
this.description = null;
```

---

### 課題 3: dueDate の追加

**課題**: 期限（日付のみ）を追加する

**私の最初の実装**: VO を作って null チェックをした

**問題点**:
1. JPA Entity に VO を直接持たせていた → JPA は保存できない
2. `nullable = false` にしていた → 期限は任意項目
3. VO の null チェックが複雑になる

**修正後の判断**:
> VO をやめる！YAGNI でしたね。まずは全部 OK にしてそこから継ぎ足してやります。

**シニアの評価**: 「これが YAGNI の正しい使い方。最初から完璧を目指さず、必要になったら追加する。」

---

### 課題 4: created_at / updated_at の追加

**課題**: 監査情報を追加する。VO は必要か？

**私の判断**: VO なしで作成

**シニアの評価**: 「VO 不要の判断は正解！created_at / updated_at には特別なビジネスルールがない。単なる記録用の日時だから。」

**軽微な指摘**: 命名規則
- `created_at` → `createdAt`（Java 標準はキャメルケース）
- 動作には影響しないので任意で修正

---

### 課題 5: @Transactional の追加

**課題**: 書き込み系 UseCase に @Transactional を付ける

**私の実装**:
```java
import jakarta.transaction.Transactional;

@Service
@Transactional
public class CreateTodoUseCase { ... }
```

**軽微な指摘**: import 先
- `jakarta.transaction.Transactional` → 基本機能のみ
- `org.springframework.transaction.annotation.Transactional` → 推奨（readOnly などオプション豊富）

どちらでも動くが、Spring プロジェクトでは Spring の方が一般的。

---

## 設計判断ログ

| 判断ポイント | 判断内容 | 理由 |
|-------------|---------|------|
| `TodoPriority` を enum で作成 | VO として作成 | `TodoStatus` と同じパターン、値が固定（3値） |
| `description` を String で作成 | VO 不要 | ビジネスルールがない、YAGNI |
| `dueDate` を LocalDate で作成 | VO 不要 | 型で保証される、任意項目で複雑になる |
| `createdAt/updatedAt` を LocalDateTime で作成 | VO 不要 | ビジネスルールがない |
| `@Transactional` を UseCase に付ける | 書き込み系のみ | 業務処理の単位でトランザクションを管理 |

---

## 最終チェックリスト

- [x] `TodoJpaEntity` 作成
- [x] Domain Entity と分離
- [x] priority 追加
- [x] description 追加
- [x] dueDate 追加
- [x] createdAt / updatedAt 追加
- [x] `@Transactional` を UseCase に付ける
- [x] Repository にトランザクション境界を持たせない

---

## 現在のテーブル構造

```
todos テーブル
├── id (BIGINT, PK, AUTO_INCREMENT)
├── title (VARCHAR(100), NOT NULL)
├── status (VARCHAR(20), NOT NULL) - OPEN / DONE
├── priority (VARCHAR(20), NOT NULL) - HIGH / MEDIUM / LOW
├── description (TEXT, NULL)
├── due_date (DATE, NULL)
├── created_at (TIMESTAMP, NULL)
└── updated_at (TIMESTAMP, NULL)
```

---

## 次のフェーズ

**Phase 8: API 実装** へ進む
- Controller の作成
- DTO の作成
- 例外ハンドリング

---

*Phase 7 完了！YAGNI を意識した設計判断ができるようになった。*
