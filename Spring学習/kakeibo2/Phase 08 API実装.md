# Phase 08: API 実装

## 概要

UseCase を HTTP API として公開するための Controller 層を実装する。

---

## 作成するファイル

| パッケージ | ファイル | 役割 |
|-----------|---------|------|
| `presentation/dto` | `TransactionRequest` | 取引の作成/更新リクエスト |
| `presentation/dto` | `TransactionResponse` | 取引のレスポンス |
| `presentation/dto` | `CategoryRequest` | カテゴリの作成/更新リクエスト |
| `presentation/dto` | `CategoryResponse` | カテゴリのレスポンス |
| `presentation/controller` | `TransactionController` | 取引 API |
| `presentation/controller` | `CategoryController` | カテゴリ API |
| `presentation/exception` | `GlobalExceptionHandler` | 例外ハンドリング |

---

## 理解ポイント

### なぜ DTO を使うのか？（Entity をそのまま返さない理由）

#### 問題1: 見せちゃダメなデータが漏れる

```java
public class User {
    private Long id;
    private String name;
    private String passwordHash;  // ← これが API で返されたら...？
}
```

Entity をそのまま返すと、**全フィールドが JSON に含まれる**。
セキュリティ上、見せてはいけないデータが漏れる危険がある。

#### 問題2: API に不要なデータまで返される

```java
public class Transaction {
    private TransactionId id;
    private Amount amount;
    private CreatedAt createdAt;  // ← 画面に出す必要ある？
}
```

`createdAt`（データ作成日時）は内部管理用。
ユーザーには不要なデータまで返してしまう。

#### 問題3: Entity の変更が API に影響する

Entity に新しいフィールドを追加したら、API のレスポンスも変わる。
**内部の変更が外部に漏れる** = 結合度が高い。

#### DTO で解決

| 問題 | DTO で解決 |
|------|-----------|
| 見せちゃダメなデータがある | **必要なフィールドだけ** 選んで返せる |
| API に不要なデータがある | **必要なフィールドだけ** 選んで返せる |
| Entity の構造が変わったら API も変わる | DTO が **クッション** になる |

**DTO = 「API 用の形に整えたデータ」**

---

### Request DTO と Response DTO

| 種類 | 役割 | 例 |
|------|------|-----|
| Request DTO | クライアントからの入力を受け取る | `TransactionRequest` |
| Response DTO | クライアントへの出力を整形する | `TransactionResponse` |

```java
// Request DTO（入力用）
public class TransactionRequest {
    private String type;      // "INCOME" or "EXPENSE"
    private int amount;
    private String description;
    private String date;
    private Long categoryId;
}

// Response DTO（出力用）
public class TransactionResponse {
    private Long id;
    private String type;
    private int amount;
    private String description;
    private String date;
    private Long categoryId;
    // createdAt は含めない！
}
```

---

### Controller の役割

Controller は **「HTTP ↔ UseCase の橋渡し」** だけを行う。

```java
@RestController
@RequestMapping("/api/transactions")
public class TransactionController {

    private final RegisterTransactionUseCase registerUseCase;

    @PostMapping
    public ResponseEntity<TransactionResponse> create(@RequestBody TransactionRequest request) {
        // 1. UseCase を呼ぶ
        Transaction transaction = registerUseCase.execute(
            request.getType(),
            request.getAmount(),
            request.getDescription(),
            request.getDate(),
            request.getCategoryId()
        );

        // 2. Entity → Response DTO に変換
        TransactionResponse response = TransactionResponse.fromEntity(transaction);

        // 3. HTTP レスポンスを返す
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }
}
```

**Controller にビジネスロジックは書かない！**

---

### @RestControllerAdvice とは？

全 Controller の例外を **一箇所で** ハンドリングする。

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(TransactionNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(TransactionNotFoundException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(new ErrorResponse(e.getMessage()));
    }

    @ExceptionHandler(InvalidAmountException.class)
    public ResponseEntity<ErrorResponse> handleInvalidAmount(InvalidAmountException e) {
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
                .body(new ErrorResponse(e.getMessage()));
    }
}
```

**メリット:**
- 例外処理が各 Controller に散らばらない
- 例外 → HTTP ステータスのマッピングが一目でわかる

---

### ResponseEntity を使う理由

```java
// ステータスコードを明示的に指定できる
return ResponseEntity.status(HttpStatus.CREATED).body(response);  // 201
return ResponseEntity.status(HttpStatus.NO_CONTENT).build();      // 204
return ResponseEntity.ok(response);                                // 200
```

Phase 2 で決めた API 設計に従って、適切なステータスコードを返せる。

---

### HTTP ステータスコード（復習）

| 操作 | 成功時 | 意味 |
|------|--------|------|
| GET（一覧・詳細） | 200 OK | 成功 |
| POST（作成） | 201 Created | リソース作成成功 |
| PUT（更新） | 200 OK | 更新成功 |
| DELETE（削除） | 204 No Content | 削除成功（返すボディなし） |

| エラー | コード | 意味 |
|--------|--------|------|
| リソースが見つからない | 404 Not Found | 指定した ID が存在しない |
| 入力値が不正 | 400 Bad Request | バリデーションエラー |
| ビジネスルール違反 | 409 Conflict | 使用中カテゴリの削除など |

---

## レイヤー間の流れ

```
[Client]
    ↓ HTTP Request (JSON)
[Controller]
    ↓ Request DTO → プリミティブ値
[UseCase]
    ↓ Domain Entity
[Controller]
    ↓ Domain Entity → Response DTO
[Client]
    ↓ HTTP Response (JSON)
```

---

## 次のステップ

1. feature/phase08-api ブランチを作成
2. DTO を実装
3. Controller を実装
4. GlobalExceptionHandler を実装
5. 動作確認
6. PR 作成 → レビュー → マージ
