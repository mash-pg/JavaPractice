# Phase 8: API 実装（基本 CRUD）

## 概要

- **目的**: HTTP リクエストを受け取り、レスポンスを返す Presentation 層を実装する
- **期間**: 2026-02-05
- **結果**: 完了 ✅

---

## シニア × ジュニア やり取り記録

### 1. DTO で VO を使っていいか？

**ジュニアの質問:**
> DTO で VO を使っていいですか？

**シニアの回答:**
DTO では VO を使わない方が良い。

**理由:**
1. DTO は Presentation 層、VO は Domain 層
2. JSON デシリアライズ時に VO のコンストラクタでバリデーションが走り、エラーハンドリングが複雑になる
3. バリデーションのタイミングを制御できない

**正しいアプローチ:**
```
JSON → DTO（Bean Validationでチェック）→ UseCase → VO生成（ドメインルールでチェック）
```

| レイヤ | バリデーション | 目的 |
|--------|---------------|------|
| Presentation（DTO） | Bean Validation | 入力形式のチェック |
| Domain（VO） | コンストラクタ | ビジネスルールの保証 |

**学び:** DTO ではプリミティブ型（Integer, String, LocalDate など）を使い、UseCase 内で VO に変換する。

---

### 2. UseCase の引数は VO？String？

**ジュニアの質問:**
> CreateCategoryUseCase の execute() メソッドの引数が VO ですが、Controller からは String を渡したい。どっちに合わせるべき？

**シニアの選択肢提示:**
- 選択肢 A: Controller で VO に変換してから UseCase に渡す
- 選択肢 B: UseCase の引数を String に変更し、UseCase 内で VO に変換

**ジュニアの反論:**
> UseCase はドメインじゃないですよね？Controller もドメインじゃないですよね？

**シニアの回答:**
その通り。どちらも Domain 層ではない。VO の生成自体はどちらからでも可能。
ただし「Controller は薄く」の原則に従い、**UseCase の引数をプリミティブ型にする（選択肢 B）** が推奨。

| 観点 | Controller で変換 | UseCase で変換 |
|------|------------------|----------------|
| Controller の責務 | VO 生成まで担当（やや厚い） | HTTP の受け渡しだけ（薄い） |
| UseCase の引数 | VO（Domain 型） | プリミティブ型 |
| テストのしやすさ | VO を作る必要あり | String を渡すだけ |

**学び:** UseCase が「外の世界（プリミティブ型）」と「Domain の世界（VO）」の橋渡しをする。

---

### 3. UseCase をプリミティブ型にするなら Repository も？

**ジュニアの質問:**
> UseCase の引数を String にするなら、Repository の引数もプリミティブ型でいいの？

**シニアの回答:**
**Repository は VO を使うべき。**

```
Controller  →  UseCase  →  Repository  →  DB
  String        String      CategoryName    JpaEntity
               （変換）↓
             CategoryName
```

| 境界 | 型 | 理由 |
|------|-----|------|
| Controller → UseCase | プリミティブ型 | 外部との接点は薄く |
| UseCase → Repository | VO | Domain 内は型安全に |
| Repository → DB | JpaEntity | インフラの都合 |

**なぜ Repository は VO を使うのか？**
```java
// ❌ プリミティブ型だと間違えやすい
repository.findById(1L);  // TransactionId? CategoryId?

// ✅ VO だとコンパイル時にミスを防げる
repository.findById(new TransactionId(1L));  // 明確
```

**学び:** UseCase の境界（外部向け）はプリミティブ型、Repository（Domain 内）は VO を使う。

---

### 4. DTO を UseCase に渡してはダメな理由

**ジュニアの質問:**
> Transaction の場合、フィールドが複数ある。Request DTO をそのまま UseCase に渡した方が間違いが少ないし安全な気がする。

**シニアの質問返し:**
> もし UseCase が DTO を引数に取ると、依存方向はどうなりますか？

**ジュニアの回答:**
> DTO はプレゼンテーション層、UseCase はアプリケーション層。ドメインではないので許される依存ではないでしょうか？

**シニアの説明:**

許される依存方向:
```
Presentation → Application → Domain
```

| 依存 | 可否 |
|------|------|
| Presentation → Application | ✅ OK |
| Application → Domain | ✅ OK |
| **Application → Presentation** | ❌ NG |

UseCase が DTO を引数に取ると、UseCase は DTO を **import** する。
```java
import com.example.kakeibo.presentation.dto.CreateTransactionRequest;  // ← この依存が問題
```

これは **Application → Presentation** の依存になり、**依存方向が逆**。

**ジュニアの追加質問:**
> アプリケーション層とプレゼンテーション層の役割をもう一度教えてください。なぜダメなのか詳しく教えてください。

**シニアの詳細説明:**

**Presentation層（表現層）の役割:**
- HTTP リクエストを受け取る
- 入力形式をチェックする
- レスポンスを返す
- **「外部の世界（HTTP、JSON）」を扱うレイヤ**

**Application層の役割:**
- ユースケースの実行
- トランザクション管理
- Domain オブジェクトを使って処理
- **「ビジネスの流れ」を扱うレイヤ。HTTP や JSON のことは知らない。**

**なぜ Application → Presentation の依存がダメなのか:**

1. **Presentation層は「変わりやすい」**
   - REST API → GraphQL に変えたい
   - 別のクライアント（CLI、バッチ）から呼び出したい
   - UseCase が DTO に依存していると、REST API 専用になってしまう

2. **変更の影響範囲**
   - DTO を変更 → UseCase も変更が必要 → テストも変更が必要
   - 依存が多いほど、変更の影響が広がる

**学び:** 「下のレイヤは上のレイヤを知らない」が原則。

---

### 5. UpdateTransactionRequest に id を追加していいか？

**ジュニアの質問:**
> UpdateTransactionRequest に id を追加してもいいですか？

**シニアの回答:**
**追加しない方が良い。**

更新 API のパス: `PUT /api/transactions/{id}`

id は URL のパスパラメータで渡される。

```java
@PutMapping("/{id}")
public ResponseEntity<TransactionResponse> update(
        @PathVariable Long id,  // ← URL から取得
        @Valid @RequestBody UpdateTransactionRequest request  // ← ボディには id 以外
)
```

もし Request にも id を入れると:
```
URL:  PUT /api/transactions/1
Body: { "id": 2, "amount": 1000, ... }
```
**どちらの id を信用する？** → 混乱の元

**学び:** id はパスから、それ以外はボディから。

---

### 6. 例外ハンドリングで getBindingResult() が使えない

**ジュニアの問題:**
`InvalidAmountException` と `InvalidCategoryException` で `getBindingResult()` を使おうとしてエラー。

```java
// ❌ エラーになるコード
@ExceptionHandler(InvalidAmountException.class)
public ResponseEntity<ErrorResponse> handleValidation(InvalidAmountException e){
    String message = e.getBindingResult().getFieldErrors().stream()  // ← getBindingResult() がない！
        ...
}
```

**シニアの説明:**
`getBindingResult()` は `MethodArgumentNotValidException`（Spring の Bean Validation 例外）専用のメソッド。

自作の Domain Exception（`InvalidAmountException`, `InvalidCategoryException`）は単純な `RuntimeException` を継承しているので、`getMessage()` だけで OK。

```java
// ✅ 正しいコード
@ExceptionHandler(InvalidAmountException.class)
public ResponseEntity<ErrorResponse> handleInvalidAmount(InvalidAmountException e) {
    ErrorResponse error = new ErrorResponse("Bad Request", e.getMessage());
    return ResponseEntity.badRequest().body(error);
}
```

| 例外 | `getBindingResult()` | 理由 |
|------|---------------------|------|
| `MethodArgumentNotValidException` | ✅ 使える | Spring が提供する例外 |
| `InvalidAmountException` | ❌ 使えない | 自作の Domain 例外 |

**ジュニアの追加質問:**
> 使えるメソッドの一覧があるドキュメントってあるの？

**シニアの回答:**
**Javadoc を見る。**

| クラス | Javadoc URL |
|--------|-------------|
| `MethodArgumentNotValidException` | https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/MethodArgumentNotValidException.html |
| `Exception`（Java標準） | https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Exception.html |

**実務での調べ方:**
1. IDE で `e.` と打つと候補が出る（一番早い）
2. Javadoc を見る（クラス名 + "javadoc" で検索）
3. 継承関係を確認する

**学び:** 分からなくなったら Javadoc を見る習慣をつける。

---

## コードレビューで指摘された点

### タイポ

| ファイル | 間違い | 正しい |
|---------|--------|--------|
| `CreateTransactionRequest` | `amoutn` | `amount` |
| `UpdateTransactionRequest` | `amoutn` | `amount` |
| `TransactionResponse` | `transaction.getId().getValue()` | `transaction.getCategoryId().getValue()` |

### 不足していたもの

| ファイル | 不足 |
|---------|------|
| `CreateTransactionRequest` | Bean Validation（`@NotNull`, `@Min`） |
| `CategoryController` | `@RestController`, `@RequestMapping` |
| `CategoryController` | `@DeleteMapping("/{id}")` のパス |

### 間違ったステータスコード

| メソッド | 間違い | 正しい |
|---------|--------|--------|
| `detail`（GET） | 201 Created | 200 OK |
| `edit`（PUT） | 201 Created | 200 OK |

---

## 作成したファイル一覧

### DTO（presentation/dto）

| ファイル | 役割 |
|---------|------|
| `CreateTransactionRequest.java` | 収支登録リクエスト |
| `UpdateTransactionRequest.java` | 収支更新リクエスト |
| `TransactionResponse.java` | 収支レスポンス |
| `CreateCategoryRequest.java` | カテゴリ作成リクエスト |
| `CategoryResponse.java` | カテゴリレスポンス |
| `ErrorResponse.java` | エラーレスポンス |

### Controller（presentation/controller）

| ファイル | エンドポイント |
|---------|---------------|
| `CategoryController.java` | `/api/categories` |
| `TransactionController.java` | `/api/transactions` |

### 例外ハンドリング（presentation/exception）

| ファイル | 役割 |
|---------|------|
| `GlobalExceptionHandler.java` | 例外 → HTTP ステータス変換 |

### 修正した UseCase（application/usecase）

| ファイル | 修正内容 |
|---------|---------|
| `CreateCategoryUseCase.java` | 引数を String に変更、戻り値を Category に変更 |
| `CreateTransactionUseCase.java` | 引数をプリミティブ型に変更 |
| `UpdateTransactionUseCase.java` | 引数をプリミティブ型に変更 |

---

## 動作確認結果

### 正常系

| API | メソッド | 結果 |
|-----|---------|------|
| カテゴリ作成 | POST /api/categories | ✅ 201 |
| カテゴリ一覧 | GET /api/categories | ✅ 200 |
| 収支登録 | POST /api/transactions | ✅ 201 |
| 収支一覧 | GET /api/transactions | ✅ 200 |
| 収支詳細 | GET /api/transactions/{id} | ✅ 200 |
| 収支更新 | PUT /api/transactions/{id} | ✅ 200 |
| 収支削除 | DELETE /api/transactions/{id} | ✅ 204 |

### 異常系

| ケース | 結果 |
|--------|------|
| 存在しない収支を取得 | ✅ 404 Not Found |
| バリデーションエラー（amount=0） | ✅ 400 Bad Request |

---

## Phase 8 で学んだことまとめ

| トピック | 学んだこと |
|---------|-----------|
| DTO と VO の使い分け | DTO は Presentation 層でプリミティブ型、VO は Domain 層 |
| UseCase の引数 | プリミティブ型で受け取り、UseCase 内で VO に変換 |
| レイヤの依存方向 | Application → Presentation は NG |
| id の渡し方 | パスパラメータから取得、Request Body には含めない |
| 例外ハンドリング | 自作例外は `getMessage()` のみ、Spring 例外は `getBindingResult()` も使える |
| 調べ方 | 分からないメソッドは Javadoc を見る |

---

## 自己評価

| チェック | 内容 |
|---------|------|
| [x] | Controller を自力で書けた（ToDo 参照あり） |
| [x] | レイヤの依存方向を理解した |
| [x] | UseCase の引数設計の考え方を理解した |
| [x] | 例外ハンドリングを実装できた |
| [x] | 動作確認で全 API が動いた |

---

## 次のフェーズ

Phase 9: パターン適用（集計・フィルタ）
- Factory パターン
- Strategy パターン（集計戦略）
- フィルタリング実装
