# Phase 8: API 実装

> **目的**: HTTP リクエストを受け取り、レスポンスを返す入口を作る。

---

## 学んだこと（まとめ）

### 1. DTO と Entity の分離

| 層 | 使うもの | 理由 |
|----|---------|------|
| Controller | DTO | JSON との変換が自然 |
| UseCase | Entity | ビジネスロジックを扱う |
| Repository | Entity ↔ JPA Entity | DB との変換 |

**なぜ Entity を直接返さない？**
```java
// ❌ NG - Entity を直接返す
@GetMapping("/{id}")
public Todo getTodo(@PathVariable Long id) {
    return getTodoUseCase.execute(id);
}

// ✅ OK - DTO に変換して返す
@GetMapping("/{id}")
public TodoResponse getTodo(@PathVariable Long id) {
    Todo todo = getTodoUseCase.execute(id);
    return TodoResponse.from(todo);
}
```

### 2. VO を使う場所・使わない場所

| 型 | Domain Entity | DTO | 理由 |
|----|--------------|-----|------|
| `TodoId` | ✅ 使う | ❌ → `Long` | VO は JSON で `{"value": 1}` になる |
| `TodoTitle` | ✅ 使う | ❌ → `String` | 同上 |
| `TodoStatus` (enum) | ✅ 使う | ✅ 使う | enum は `"OPEN"` とそのまま出力 |
| `TodoPriority` (enum) | ✅ 使う | ✅ 使う | 同上 |

**覚え方**:
- **VO（クラス）**: Domain 層だけで使う
- **enum**: どこでも使える

### 3. UseCase の呼び方

```java
// Controller から UseCase を呼ぶ = execute メソッドを呼ぶだけ
Todo todo = createTodoUseCase.execute(request.getTitle());
TodoResponse response = TodoResponse.from(todo);
```

### 4. リストの変換（stream）

```java
// List<Todo> → List<TodoResponse> に変換
List<TodoResponse> responses = todos.stream()
    .map(TodoResponse::from)
    .collect(Collectors.toList());
```

### 5. バリデーションアノテーション

| アノテーション | 意味 |
|--------------|------|
| `@NotBlank` | null・空文字・空白のみ禁止 |
| `@NotNull` | null 禁止 |
| `@Size(max = 100)` | 最大文字数 |
| `@Valid` | Controller でバリデーション実行 |

---

## 実装したクラス

### DTO（presentation/dto）

| クラス | 役割 |
|--------|------|
| `TodoResponse` | レスポンス用（`from(Todo)` で変換） |
| `CreateTodoRequest` | 作成リクエスト用（バリデーション付き） |
| `UpdateTodoRequest` | 更新リクエスト用（バリデーション付き） |
| `ErrorResponse` | エラーレスポンス用 |

### Controller（presentation/controller）

| メソッド | エンドポイント | 処理 | ステータス |
|---------|---------------|------|-----------|
| `POST` | `/api/todos` | 作成 | 201 Created |
| `GET` | `/api/todos` | 一覧 | 200 OK |
| `GET` | `/api/todos/{id}` | 詳細 | 200 OK |
| `PUT` | `/api/todos/{id}` | 更新 | 200 OK |
| `DELETE` | `/api/todos/{id}` | 削除 | 204 No Content |
| `POST` | `/api/todos/{id}/complete` | 完了 | 200 OK |

### 例外ハンドラ（presentation/exception）

| クラス | 役割 |
|--------|------|
| `GlobalExceptionHandler` | 全 Controller の例外を捕まえて HTTP ステータスに変換 |

---

## 課題と解答

### 課題 1: TodoResponse の作成

**課題**: Entity を DTO に変換するクラスを作成

**私の実装**:
```java
public class TodoResponse {
    private Long id;
    private String title;
    private TodoStatus status;      // enum はそのまま使える
    private TodoPriority priority;  // enum はそのまま使える
    private String description;
    private LocalDate duedate;
    private LocalDateTime created_at;
    private LocalDateTime updated_at;

    public static TodoResponse from(Todo todo) {
        TodoResponse response = new TodoResponse();
        response.id = todo.getId().getValue();  // VO から値を取り出す
        response.title = todo.getTitle().getValue();
        response.status = todo.getStatus();
        // ...
        return response;
    }
}
```

**学び**:
- VO（TodoId, TodoTitle）は `.getValue()` で中身を取り出す
- enum（TodoStatus, TodoPriority）はそのまま使える

---

### 課題 2: CreateTodoRequest の作成

**課題**: バリデーション付きのリクエスト DTO を作成

**私の実装**:
```java
public class CreateTodoRequest {
    @NotBlank(message = "タイトルは必須です")
    @Size(max = 100, message = "タイトルは100文字以内です")
    private String title;

    private String description;      // 任意
    private TodoPriority priority;   // 任意
    private LocalDate duedate;       // 任意

    // Getter / Setter
}
```

**指摘された点**:
- `status` は不要（新規作成時は常に OPEN）
- `title` の Getter/Setter を忘れていた

---

### 課題 3: TodoController の作成

**課題**: REST API の入口を作成

**私の疑問**:
> 「UseCase を呼ぶってなにをどうやって呼ぶの？」
> 「Entity → DTO 変換は別のメソッドを作る？」

**シニアの回答**:
```java
@RestController
@RequestMapping("/api/todos")
public class TodoController {

    private final CreateTodoUseCase createTodoUseCase;

    // コンストラクタで DI
    public TodoController(CreateTodoUseCase createTodoUseCase) {
        this.createTodoUseCase = createTodoUseCase;
    }

    @PostMapping
    public ResponseEntity<TodoResponse> create(@Valid @RequestBody CreateTodoRequest request) {
        // ① UseCase を呼ぶ = execute() を呼ぶだけ
        Todo todo = createTodoUseCase.execute(request.getTitle());

        // ② Entity → DTO 変換 = TodoResponse.from() を使う
        TodoResponse response = TodoResponse.from(todo);

        // ③ レスポンスを返す
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }
}
```

**指摘された点**:
- `@RestController` と `@RequestMapping` を忘れていた
- これがないと Spring が Controller として認識しない

---

### 課題 4: 残りのエンドポイント実装

**課題**: GET/{id}, PUT, DELETE, complete を追加

**私の実装**: 全6エンドポイント完成

**指摘された点**:
- `DeleteTodoUseCase` に `@Service` `@Transactional` がなかった
- `delete` メソッドの戻り値が `ResponseEntity<TodoResponse>` だったが `ResponseEntity<Void>` が正しい

---

### 課題 5: 例外ハンドリング

**課題**: エラー時に統一された JSON を返す

**私の実装**:
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(TodoNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(TodoNotFoundException e) {
        ErrorResponse error = new ErrorResponse("Not Found", e.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(AlreadyCompletedException.class)
    public ResponseEntity<ErrorResponse> handleConflict(AlreadyCompletedException e) {
        ErrorResponse error = new ErrorResponse("Conflict", e.getMessage());
        return ResponseEntity.status(HttpStatus.CONFLICT).body(error);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException e) {
        String message = e.getBindingResult().getFieldErrors().stream()
            .map(error -> error.getField() + ": " + error.getDefaultMessage())
            .collect(Collectors.joining(", "));
        ErrorResponse error = new ErrorResponse("Bad Request", message);
        return ResponseEntity.badRequest().body(error);
    }
}
```

**指摘された点**:
- 最初 `domain/exception` に配置 → `presentation/exception` に移動
- HTTP ステータスを扱うのは presentation 層の責務

---

## 設計判断ログ

| 判断ポイント | 判断内容 | 理由 |
|-------------|---------|------|
| DTO で VO を使わない | `Long`, `String` を使用 | JSON 変換で自然な形になる |
| DTO で enum は使う | `TodoStatus`, `TodoPriority` | enum は `"OPEN"` とそのまま出力 |
| 例外ハンドラの配置 | `presentation/exception` | HTTP を扱うのは presentation 層 |
| `@Transactional` の配置 | UseCase に付ける | 業務処理の単位でトランザクション管理 |

---

## アノテーション一覧

### Controller 系

| アノテーション | 意味 |
|--------------|------|
| `@RestController` | REST API の Controller（JSON を返す） |
| `@RequestMapping` | 共通のパスを設定 |
| `@GetMapping` | GET リクエスト |
| `@PostMapping` | POST リクエスト |
| `@PutMapping` | PUT リクエスト |
| `@DeleteMapping` | DELETE リクエスト |
| `@PathVariable` | URL の `{id}` 部分を取得 |
| `@RequestBody` | JSON を DTO に変換 |
| `@Valid` | バリデーションを実行 |

### 例外ハンドリング系

| アノテーション | 意味 |
|--------------|------|
| `@RestControllerAdvice` | 全 Controller の例外を捕まえる |
| `@ExceptionHandler` | 特定の例外を処理する |

### バリデーション系

| アノテーション | 意味 |
|--------------|------|
| `@NotBlank` | null・空文字・空白のみ禁止 |
| `@NotNull` | null 禁止 |
| `@Size(max = n)` | 最大文字数 |

---

## 動作確認コマンド

### Windows (PowerShell)

```powershell
# 一覧取得
Invoke-RestMethod -Uri "http://localhost:8080/api/todos" -Method GET

# 作成
$body = @{ title = "買い物に行く" } | ConvertTo-Json
Invoke-RestMethod -Uri "http://localhost:8080/api/todos" -Method POST -Body $body -ContentType "application/json"

# 詳細取得
Invoke-RestMethod -Uri "http://localhost:8080/api/todos/1" -Method GET

# 更新
$body = @{ title = "買い物に行く（更新）" } | ConvertTo-Json
Invoke-RestMethod -Uri "http://localhost:8080/api/todos/1" -Method PUT -Body $body -ContentType "application/json"

# 完了
Invoke-RestMethod -Uri "http://localhost:8080/api/todos/1/complete" -Method POST

# 削除
Invoke-RestMethod -Uri "http://localhost:8080/api/todos/1" -Method DELETE

# エラー確認（空タイトル → 400）
$body = @{ title = "" } | ConvertTo-Json
Invoke-RestMethod -Uri "http://localhost:8080/api/todos" -Method POST -Body $body -ContentType "application/json"

# エラー確認（存在しないID → 404）
Invoke-RestMethod -Uri "http://localhost:8080/api/todos/999" -Method GET
```

### curl（Git Bash / WSL / Mac / Linux）

```bash
# 一覧取得
curl http://localhost:8080/api/todos

# 作成
curl -X POST http://localhost:8080/api/todos \
  -H "Content-Type: application/json" \
  -d '{"title": "買い物に行く"}'

# 詳細取得
curl http://localhost:8080/api/todos/1

# 更新
curl -X PUT http://localhost:8080/api/todos/1 \
  -H "Content-Type: application/json" \
  -d '{"title": "買い物に行く（更新）"}'

# 完了
curl -X POST http://localhost:8080/api/todos/1/complete

# 削除
curl -X DELETE http://localhost:8080/api/todos/1
```

### ブラウザ

- http://localhost:8080/api/todos - 一覧
- http://localhost:8080/api/todos/1 - 詳細
- http://localhost:8080/h2-console - DB 確認

---

## 最終チェックリスト

- [x] `@RestController` を使用
- [x] `@RequestMapping("/api/todos")` で共通パス設定
- [x] Request DTO にバリデーションを付ける
- [x] Response DTO を固定（Entity を返さない）
- [x] 全6エンドポイント実装
- [x] 例外は API レイヤで HTTP ステータスに変換
- [x] `@RestControllerAdvice` で共通ハンドリング
- [x] H2 データベース設定

---

## 現場レベルか？

**はい、現場で使う基本パターンそのもの！**

| 学んだこと | 現場での重要度 |
|-----------|---------------|
| レイヤ分離（Controller → UseCase → Repository） | ⭐⭐⭐ 必須 |
| DTO と Entity の分離 | ⭐⭐⭐ 必須 |
| バリデーション（@Valid, @NotBlank） | ⭐⭐⭐ 必須 |
| DI（コンストラクタインジェクション） | ⭐⭐⭐ 必須 |
| 例外ハンドリング | ⭐⭐⭐ 必須 |
| REST API 設計（HTTP メソッド、ステータスコード） | ⭐⭐⭐ 必須 |

---

## API テストツール（現場での使い分け）

### ツール別の使用頻度

| ツール | 使用頻度 | 主な場面 |
|--------|---------|---------|
| **curl** | ⭐⭐⭐ | Linux サーバー、CI/CD、ドキュメント例 |
| **Postman** | ⭐⭐⭐ | 開発中のテスト、チーム共有 |
| **Invoke-RestMethod** | ⭐⭐ | Windows 環境での自動化 |
| **Swagger UI** | ⭐⭐ | API ドキュメント兼テスト |
| **HTTPie** | ⭐ | curl の使いやすい版（好みで） |

### curl - 最も汎用的（必須スキル）

どこでも使える（Linux / Mac / Git Bash / WSL）

**覚えておくべきオプション:**

| オプション | 意味 | 例 |
|-----------|------|-----|
| `-X` | HTTP メソッド指定 | `-X POST`, `-X PUT`, `-X DELETE` |
| `-H` | ヘッダー追加 | `-H "Content-Type: application/json"` |
| `-d` | リクエストボディ | `-d '{"title": "test"}'` |
| `-i` | レスポンスヘッダーも表示 | デバッグ時に便利 |
| `-v` | 詳細表示（verbose） | 通信内容を全部見たい時 |
| `-s` | 進捗表示を消す（silent） | スクリプトで使う時 |
| `-o` | ファイルに出力 | `-o response.json` |

**使う場面:**
- サーバー上での API 確認
- CI/CD パイプラインでの自動テスト
- README やドキュメントの例示
- 障害調査時のクイック確認

**実践例:**
```bash
# 認証付きリクエスト
curl -X GET http://api.example.com/users \
  -H "Authorization: Bearer your-token-here"

# レスポンスヘッダーも確認
curl -i http://localhost:8080/api/todos

# 詳細なデバッグ
curl -v http://localhost:8080/api/todos

# JSON を整形して表示（jq と組み合わせ）
curl -s http://localhost:8080/api/todos | jq
```

### Postman - 開発中のメインツール（必須スキル）

**使う場面:**
- API 開発中のテスト
- 複雑なリクエスト（認証、ファイルアップロード）
- チームで API コレクションを共有
- 環境変数（本番 / ステージング / ローカル）切り替え

> **面接で「API テストどうやってる？」と聞かれたら「Postman 使ってます」と答えられるように。**

### Swagger UI - API ドキュメント兼テスト

Spring Boot に `springdoc-openapi` を追加すると自動生成される。

```
http://localhost:8080/swagger-ui.html
```

**使う場面:**
- フロントエンドエンジニアへの API 共有
- 仕様書代わり
- 簡単なテスト

---

## テスト結果（実際に動作確認済み）

| テスト | 期待値 | 結果 |
|-------|--------|------|
| 作成 | 201 + id:1 | ✅ |
| 一覧取得 | 1件 | ✅ |
| 詳細取得 | id:1 の情報 | ✅ |
| 更新 | title が変更 | ✅ |
| 完了 | status が DONE | ✅ |
| 二重完了 | 409 Conflict | ✅ |
| 削除 | 204 No Content | ✅ |
| 削除確認 | 404 Not Found | ✅ |
| バリデーション | 400 Bad Request | ✅ |

---

*Phase 8 完了！API が動く状態になった。*
