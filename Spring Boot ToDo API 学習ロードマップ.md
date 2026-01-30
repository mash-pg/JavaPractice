# Spring Boot ToDo API 学習ロードマップ（DDD軽量版）

> このロードマップは、Spring Boot で REST API を作りながら、実務レベルの設計・テストを一通り経験することを目的としています。

---

## 前提・ゴール

| 項目 | 内容 |
|------|------|
| **題材** | ToDo アプリ（CRUD + 状態遷移） |
| **学ぶこと** | REST API / DDD（軽量）/ DI / Repository / Strategy / Factory |
| **設計原則** | SOLID / DRY / YAGNI |
| **記録** | Obsidian で設計判断ログを残す |

---

## 学習スタイル

> **シニアエンジニア（Claude）× ジュニアエンジニア（自分）形式**

| 役割 | 担当 |
|------|------|
| **シニア（Claude）** | 課題を出す、ヒントを与える、コードレビュー、間違いを指摘 |
| **ジュニア（自分）** | 課題を解く、コードを書く、質問する |

### 進め方

1. シニアが課題と要件を提示
2. ジュニアが実装を試みる
3. シニアがコードレビュー（良い点・改善点）
4. 修正 → 再レビュー → 合格まで繰り返す
5. 動作確認して次の課題へ

> **ポイント**: 答えを見る前にまず自分で考える。間違えた箇所は記録して復習する。

---

## 進捗管理

- [x] Phase 1: 環境構築
- [x] Phase 2: API 設計
- [x] Phase 3: レイヤ設計
- [x] Phase 4: ドメイン設計
- [x] Phase 5: ユースケース
- [x] Phase 6: Repository
- [x] Phase 7: DB 実装
- [x] Phase 8: API 実装
- [x] Phase 9: パターン適用
- [ ] Phase 10: 品質ルール ← **次はここ**
- [ ] Phase 11: テスト
- [ ] Phase 12: 仕上げ

---

## 学習フェーズ一覧

```
Phase 1: 環境構築        → 開発の土台を整える
Phase 2: API 設計        → 何を作るか決める
Phase 3: レイヤ設計      → コードの置き場所を決める
Phase 4: ドメイン設計    → ビジネスルールを表現する
Phase 5: ユースケース    → 処理の流れを実装する
Phase 6: Repository     → データの出し入れを抽象化する
Phase 7: DB 実装        → 実際のデータ保存処理を作る
Phase 8: API 実装       → HTTP の入口を作る
Phase 9: パターン適用   → 設計パターンで拡張性を持たせる
Phase 10: 品質ルール    → チームで守るルールを決める
Phase 11: テスト        → 品質を担保する
Phase 12: 仕上げ        → ドキュメント・振り返り
```

---

## Phase 1: 環境構築 ✓

> **目的**: 開発を始められる状態を作る

### 1.1 開発環境

- [x] JDK 17 をインストール
- [x] IDE（Eclipse）の設定
- [x] Build Tool: Maven

### 1.2 Spring Initializr でプロジェクト作成

https://start.spring.io/ で以下を設定：

| 設定項目 | 値 |
|---------|-----|
| Project | Maven |
| Language | Java |
| Spring Boot | 3.5.10 |
| Packaging | Jar |
| Java | 17 |

**Dependencies（依存ライブラリ）:**

| ライブラリ | 用途 |
|-----------|------|
| Spring Web | REST API を作るため |
| Validation | 入力値チェック |
| Spring Data JPA | DB 操作を簡単に |
| H2 Database | インメモリ DB（開発用） |
| Lombok | ボイラープレート削減 |

### 1.3 動作確認

- [x] アプリ起動確認
- [x] http://localhost:8080 にアクセス（Whitelabel Error Page が出れば OK）

---

## Phase 2: API 設計 ✓

> **目的**: 「何を作るか」を先に決めておく。実装中にブレないための指針。

### 2.1 エンドポイント一覧

| メソッド | エンドポイント | 説明 | ステータス |
|---------|---------------|------|-----------|
| `POST` | `/api/todos` | ToDo を作成 | 201 |
| `GET` | `/api/todos` | ToDo 一覧を取得 | 200 |
| `GET` | `/api/todos/{id}` | ToDo 詳細を取得 | 200 |
| `PUT` | `/api/todos/{id}` | ToDo を更新 | 200 |
| `DELETE` | `/api/todos/{id}` | ToDo を削除 | 204 |
| `POST` | `/api/todos/{id}/complete` | ToDo を完了にする | 200 |

### 2.2 ステータスコード方針

| コード | 意味 | 使う場面 |
|--------|------|---------|
| `201` | Created | 作成成功 |
| `200` | OK | 取得・更新成功 |
| `204` | No Content | 削除成功 |
| `400` | Bad Request | バリデーションエラー |
| `404` | Not Found | リソースが存在しない |
| `409` | Conflict | ビジネスルール違反 |

### 2.3 設計ルール

- [x] Request / Response に **DTO** を使う
- [x] Entity を Controller から直接返さない

> **なぜ？**: Entity は DB の構造に依存している。API のレスポンス形式は別で管理した方が変更に強い。

---

## Phase 3: レイヤ設計（DDD 軽量版） ✓

> **目的**: コードの「置き場所」を決める。責務を分離して変更に強くする。

### 3.1 パッケージ構成

```
src/main/java/com/example/todo/
├── presentation/          ← HTTP の入口
│   ├── controller/
│   ├── dto/
│   └── exception/
├── application/           ← ビジネスロジックの流れ
│   └── usecase/
├── domain/                ← ビジネスルール（最も重要）
│   ├── entity/
│   ├── valueobject/
│   ├── repository/
│   └── exception/
└── infrastructure/        ← 外部技術との接続
    ├── jpa/
    └── repository/
```

### 3.2 依存の方向（重要）

```
presentation → application → domain
                    ↑
infrastructure ─────┘
```

> **ポイント**: `domain` は他のレイヤに依存しない。最も安定したレイヤ。

---

## Phase 4: ドメインモデル設計 ✓

> **目的**: ビジネスルールをコードで表現する。バグの温床になりやすい部分をここに閉じ込める。

### 4.1 Entity

| Entity | 説明 |
|--------|------|
| `Todo` | ToDo のメインエンティティ |

### 4.2 Value Object（値オブジェクト）

| VO | ルール |
|----|--------|
| `TodoId` | null 禁止 |
| `TodoTitle` | 空禁止、最大100文字 |
| `TodoStatus` | OPEN / DONE |
| `TodoPriority` | HIGH / MEDIUM / LOW |

### 4.3 Exception

| 例外 | 用途 |
|------|------|
| `InvalidTodoTitleException` | タイトル不正時 |
| `AlreadyCompletedException` | 二重完了時 |
| `TodoNotFoundException` | ToDo が見つからない時 |

### 4.4 ドメインルール

- [x] ビジネスルールは Entity のメソッドに閉じ込める

```java
// 例: 完了処理のルールを Todo に持たせる
public void complete() {
    if (this.status == TodoStatus.DONE) {
        throw new AlreadyCompletedException("すでに完了しています");
    }
    this.status = TodoStatus.DONE;
}
```

---

## Phase 5: ユースケース設計（Application 層） ✓

> **目的**: 「この API では何をするか」という処理の流れを定義する。

### 5.1 ユースケース一覧

| クラス名 | 責務 |
|---------|------|
| `CreateTodoUseCase` | ToDo を作成する |
| `ListTodosUseCase` | ToDo 一覧を取得する |
| `GetTodoUseCase` | ToDo 詳細を取得する |
| `UpdateTodoUseCase` | ToDo を更新する |
| `DeleteTodoUseCase` | ToDo を削除する |
| `CompleteTodoUseCase` | ToDo を完了にする |

### 5.2 設計原則

- [x] **Controller は薄く**: HTTP の受け取りと返却だけ
- [x] **業務判断は UseCase に集約**: if 文はここに書く
- [x] **@Transactional は UseCase に付ける**

### 5.3 学んだこと

- UseCase は「処理の流れ」を記述する（取得 → 変更 → 保存）
- ビジネスルールは Entity に任せる（`complete()`, `changeTitle()` など）
- Entity のメソッドでフィールドデータを書き換える

---

## Phase 6: Repository（ポート）設計 ✓

> **目的**: データアクセスを抽象化する。DB を変えても domain / application は変更不要にする。

### 6.1 Repository Interface

```java
public interface TodoRepository {
    Todo save(Todo todo);
    Optional<Todo> findById(TodoId id);
    List<Todo> findAll();
    void deleteById(TodoId id);
}
```

### 6.2 設計原則

- [x] Interface は `domain` パッケージに置く
- [x] JPA / SQL などの DB 技術を domain に漏らさない

> **なぜ Interface を切る？**: H2 → MySQL への切り替え、テスト時のモック差し替えが容易になる。

### 6.3 実装したクラス

| クラス | 役割 |
|-------|------|
| `TodoJpaEntity` | DBテーブルと1:1で対応（JPA用） |
| `TodoJpaRepository` | Spring Data JPA の機能を使う |
| `TodoRepositoryImpl` | Domain ↔ JPA の橋渡し役 |

### 6.4 学んだこと

- `implements TodoRepository` と `使う TodoJpaRepository` は別物
- 変換メソッド（`toJpaEntity`, `toDomainEntity`）は `TodoRepositoryImpl` 内に private で書く
- Value Object は「包む・取り出す」で変換する

---

## Phase 7: DB・JPA 実装（Infrastructure） ✓

> **目的**: 実際のデータ保存処理を実装する。

### 7.1 テーブル設計

**テーブル: `todos`**

| カラム | 型 | 説明 |
|--------|-----|------|
| id | BIGINT | 主キー（AUTO_INCREMENT） |
| title | VARCHAR(100) | タイトル（NOT NULL） |
| description | TEXT | 説明（NULL OK） |
| status | VARCHAR(20) | OPEN / DONE |
| priority | VARCHAR(20) | HIGH / MEDIUM / LOW |
| due_date | DATE | 期限（NULL OK） |
| created_at | TIMESTAMP | 作成日時 |
| updated_at | TIMESTAMP | 更新日時 |

### 7.2 実装チェックリスト

**JPA Entity:**
- [x] 永続化専用の Entity を作成（`TodoJpaEntity`）
- [x] Domain Entity と分離する

**Mapper:**
- [x] Domain ↔ JPA の変換は `TodoRepositoryImpl` 内で実施
- [x] Repository 実装の中で変換する

**Repository 実装:**
- [x] `TodoRepositoryImpl` が `TodoRepository` を実装
- [x] Spring Data JPA の利用はここに閉じ込める

**Transaction:**
- [x] `@Transactional` は **Application 層（UseCase）** に付ける
- [x] Repository にトランザクション境界を持たせない

### 7.3 H2 データベース設定

**application.properties:**

```properties
spring.application.name=todo

# H2 Database 設定
spring.datasource.url=jdbc:h2:mem:tododb
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# JPA 設定
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true

# H2 Console（ブラウザでDBを見れる）
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

### 7.4 学んだこと（YAGNI の実践）

| 項目 | 判断 | 理由 |
|------|------|------|
| `TodoPriority` | enum で作成 | 値が固定（3値） |
| `description` | String で作成 | ビジネスルールなし |
| `dueDate` | LocalDate で作成 | 型で保証される、任意項目 |
| `createdAt/updatedAt` | LocalDateTime で作成 | ビジネスルールなし |

> **YAGNI**: 今ルールがないなら VO は作らない。必要になったら追加する。

---

## Phase 8: API 実装 ✓

> **目的**: HTTP リクエストを受け取り、レスポンスを返す入口を作る。

### 8.1 Controller 実装

- [x] `@RestController` を使用
- [x] `@RequestMapping("/api/todos")` で共通パス設定
- [x] Request DTO にバリデーションを付ける
  - `@NotBlank` - 空文字禁止
  - `@Size(max = 100)` - 最大長
- [x] Response DTO を固定（Entity を返さない）

### 8.2 実装した DTO

| クラス | 役割 |
|--------|------|
| `TodoResponse` | レスポンス用（Entity → DTO 変換） |
| `CreateTodoRequest` | 作成リクエスト用 |
| `UpdateTodoRequest` | 更新リクエスト用 |
| `ErrorResponse` | エラーレスポンス用 |

### 8.3 実装したエンドポイント

| メソッド | エンドポイント | Controller メソッド |
|---------|---------------|-------------------|
| `POST` | `/api/todos` | `create()` |
| `GET` | `/api/todos` | `list()` |
| `GET` | `/api/todos/{id}` | `get()` |
| `PUT` | `/api/todos/{id}` | `update()` |
| `DELETE` | `/api/todos/{id}` | `delete()` |
| `POST` | `/api/todos/{id}/complete` | `complete()` |

### 8.4 例外ハンドリング

**方針:**
- [x] 例外は API レイヤで HTTP ステータスに変換
- [x] JSON エラーレスポンスの形式を統一
- [x] `@RestControllerAdvice` で共通ハンドリング

**マッピング:**

| 例外 | HTTP ステータス |
|------|----------------|
| `TodoNotFoundException` | 404 Not Found |
| `AlreadyCompletedException` | 409 Conflict |
| `MethodArgumentNotValidException` | 400 Bad Request |

### 8.5 学んだこと

**VO を使う場所・使わない場所:**

| 型 | Domain Entity | DTO | 理由 |
|----|--------------|-----|------|
| `TodoId` | ✅ 使う | ❌ → `Long` | VO は JSON で `{"value": 1}` になる |
| `TodoTitle` | ✅ 使う | ❌ → `String` | 同上 |
| `TodoStatus` (enum) | ✅ 使う | ✅ 使う | enum は `"OPEN"` とそのまま出力される |
| `TodoPriority` (enum) | ✅ 使う | ✅ 使う | 同上 |

**UseCase の呼び方:**
```java
// Controller から UseCase を呼ぶ
Todo todo = createTodoUseCase.execute(request.getTitle());
TodoResponse response = TodoResponse.from(todo);
```

**Entity → DTO 変換（リスト）:**
```java
List<TodoResponse> responses = todos.stream()
    .map(TodoResponse::from)
    .collect(Collectors.toList());
```

---

## 動作確認方法

### アプリ起動

**Eclipse の場合:**
1. `TodoApplication.java` を右クリック
2. Run As → Spring Boot App

**コマンドラインの場合:**

```bash
# Windows (PowerShell)
cd C:\dev\todo
.\mvnw.cmd spring-boot:run

# Windows (コマンドプロンプト)
cd C:\dev\todo
mvnw.cmd spring-boot:run

# Mac / Linux
cd /path/to/todo
./mvnw spring-boot:run
```

### H2 Console 確認

1. ブラウザで http://localhost:8080/h2-console を開く
2. 以下を入力:
   - JDBC URL: `jdbc:h2:mem:tododb`
   - User Name: `sa`
   - Password: （空欄）
3. Connect をクリック
4. TODOS テーブルがあれば OK

### API 動作確認

**Windows (PowerShell):**

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

# バリデーションエラー確認（空タイトル）
$body = @{ title = "" } | ConvertTo-Json
Invoke-RestMethod -Uri "http://localhost:8080/api/todos" -Method POST -Body $body -ContentType "application/json"

# 404 エラー確認（存在しないID）
Invoke-RestMethod -Uri "http://localhost:8080/api/todos/999" -Method GET
```

**Windows (コマンドプロンプト) / Mac / Linux (curl):**

```bash
# 一覧取得
curl http://localhost:8080/api/todos

# 作成
curl -X POST http://localhost:8080/api/todos \
  -H "Content-Type: application/json" \
  -d "{\"title\": \"買い物に行く\"}"

# 詳細取得
curl http://localhost:8080/api/todos/1

# 更新
curl -X PUT http://localhost:8080/api/todos/1 \
  -H "Content-Type: application/json" \
  -d "{\"title\": \"買い物に行く（更新）\"}"

# 完了
curl -X POST http://localhost:8080/api/todos/1/complete

# 削除
curl -X DELETE http://localhost:8080/api/todos/1

# バリデーションエラー確認
curl -X POST http://localhost:8080/api/todos \
  -H "Content-Type: application/json" \
  -d "{\"title\": \"\"}"

# 404 エラー確認
curl http://localhost:8080/api/todos/999
```

**ブラウザで確認（GET のみ）:**
- http://localhost:8080/api/todos - 一覧
- http://localhost:8080/api/todos/1 - 詳細

---

## Phase 9: Strategy / Factory パターンの適用 ✓

> **目的**: 拡張ポイントを設計パターンで整理する。ただし YAGNI を意識。

### 9.1 Factory パターン

**TodoFactory:**
- [x] title の正規化（前後の空白除去 `trim()`）
- [x] createdAt の自動付与（`LocalDateTime.now()`）
- [x] 初期ステータス設定（OPEN）

> **目的**: 生成ルールを一箇所に集約し、UseCase をシンプルに保つ。

**役割分担:**

| クラス | 責務 |
|--------|------|
| **Factory** | オブジェクトを「作る」 |
| **UseCase** | 作ったオブジェクトを「保存する」 |

### 9.2 Strategy パターン

**SortStrategy（ソート戦略）:**
- [x] `CreatedAtSortStrategy` - 作成日順
- [x] `PrioritySortStrategy` - 優先度順（HIGH → MEDIUM → LOW）
- [x] `DueDateSortStrategy` - 期限順（null は最後）

**SortStrategyFactory:**
- [x] sortType に応じて適切な Strategy を選択

### 9.3 API の変更

```
GET /api/todos              → デフォルト（作成日順）
GET /api/todos?sort=priority   → 優先度順
GET /api/todos?sort=dueDate    → 期限順
```

### 9.4 学んだこと

**Java のソート処理:**
```java
// 基本形
list.sort(Comparator.comparing(Todo::getCreatedAt));

// 降順
list.sort(Comparator.comparing(Todo::getCreatedAt).reversed());

// null 対応
list.sort(Comparator.comparing(Todo::getDueDate, Comparator.nullsLast(Comparator.naturalOrder())));
```

**@RequestParam の使い方:**
```java
@GetMapping
public ResponseEntity<List<TodoResponse>> list(
    @RequestParam(defaultValue = "createdAt") String sort) { ... }
```

> **詳細**: [[Phase9 パターン適用]] を参照

---

## Phase 10: 品質ルール

> **目的**: チーム開発を想定したルールを決めておく。

### 10.1 ログ方針

| レベル | 使う場面 |
|--------|---------|
| `ERROR` | 例外発生時 |
| `INFO` | 業務イベント（作成・完了など） |
| `DEBUG` | 開発時のデバッグ |

### 10.2 レイヤ境界ルール

- [ ] 入力: DTO で受け取る
- [ ] 出力: DTO で返す
- [ ] Entity を外部に漏らさない

### 10.3 設計チェックリスト

- [ ] SOLID 原則を意識しているか
- [ ] UseCase が肥大化していないか（100行超えたら分割検討）
- [ ] DRY しすぎて読みにくくなっていないか

> **可読性 ＞ 抽象化**: 3回同じコードが出てきたら共通化を検討。

---

## Phase 11: テスト戦略

> **目的**: 品質を担保する。どこで何をテストするかを明確にする。

### 11.1 Domain Unit Test（最優先）

**テスト対象:**
- [ ] VO の境界値テスト（空文字、最大長、日付）
- [ ] Entity の状態遷移（OPEN → DONE）
- [ ] Factory / Strategy のテスト

```java
@Test
void タイトルが空の場合は例外() {
    assertThrows(InvalidTitleException.class,
        () -> new TodoTitle(""));
}
```

### 11.2 Application Unit Test

- [ ] Repository をモックして UseCase をテスト
- [ ] 正常系: 期待通りの結果が返る
- [ ] 異常系: NotFound / BusinessRule 例外

### 11.3 Slice Test

**Controller:**
- [ ] `@WebMvcTest` を使用
- [ ] 400 / 404 / 409 のレスポンス確認
- [ ] JSON レスポンスの形式確認

**Repository:**
- [ ] `@DataJpaTest` を使用
- [ ] JPA マッピングの確認
- [ ] クエリが正しく動作するか

### 11.4 Integration Test

- [ ] `@SpringBootTest` を使用
- [ ] CRUD 一連の動作確認
- [ ] H2 インメモリ DB を使用

---

## Phase 12: 仕上げ

> **目的**: ドキュメントを整え、学びを言語化する。

### 12.1 オプション機能

- [ ] OpenAPI / Swagger で API ドキュメント自動生成
- [ ] Actuator でヘルスチェック

### 12.2 README

- [ ] 起動方法
- [ ] テスト実行方法
- [ ] API 一覧

### 12.3 設計判断ログ（重要）

以下の判断理由を記録しておく：

| 判断ポイント | 記録すること |
|-------------|-------------|
| **DTO 分離** | なぜ Entity を直接返さなかったか |
| **Repository ポート化** | なぜ Interface を切ったか、DB 切り替えをどう意識したか |
| **Strategy / Factory** | 使った理由、使わなかった理由、YAGNI をどう判断したか |

> **このログの価値**: 面接や職務経歴書で「なぜそう設計したか」を語れるようになる。

---

## 困ったときは

| 状況 | 対処 |
|------|------|
| 何から手をつけていいか分からない | Phase 1 から順番にやる |
| 設計に迷う | YAGNI を思い出す。今必要なものだけ作る |
| コードが汚くなってきた | 動くものを作ってからリファクタリング |
| テストが書けない | まず Domain のテストだけ書く |

---

## 関連ドキュメント

- [[Phase3 レイヤ設計（DDD 軽量版）]]
- [[Phase4 ドメインモデル設計]]
- [[Phase5 ユースケース]]
- [[Phase6 Repository]]
- [[Phase7 DB実装]]
- [[Phase8 API実装]]
- [[Phase9 パターン適用]]

---

## 現在のプロジェクト構造

```
src/main/java/com/example/todo/
├── TodoApplication.java
├── presentation/
│   ├── controller/
│   │   └── TodoController.java          ← Phase9で修正（@RequestParam追加）
│   ├── dto/
│   │   ├── TodoResponse.java
│   │   ├── CreateTodoRequest.java
│   │   ├── UpdateTodoRequest.java
│   │   └── ErrorResponse.java
│   └── exception/
│       └── GlobalExceptionHandler.java
├── application/
│   └── usecase/
│       ├── CreateTodoUseCase.java        ← Phase9で修正（Factory使用）
│       ├── ListTodosUseCase.java         ← Phase9で修正（Strategy使用）
│       ├── GetTodoUseCase.java
│       ├── UpdateTodoUseCase.java
│       ├── DeleteTodoUseCase.java
│       └── CompleteTodoUseCase.java
├── domain/
│   ├── entity/
│   │   └── Todo.java                     ← Phase9で修正（changeCreatedAt追加）
│   ├── valueobject/
│   │   ├── TodoId.java
│   │   ├── TodoTitle.java
│   │   ├── TodoStatus.java
│   │   └── TodoPriority.java
│   ├── repository/
│   │   └── TodoRepository.java
│   ├── factory/                          ← Phase9で追加
│   │   └── TodoFactory.java
│   ├── strategy/                         ← Phase9で追加
│   │   ├── SortStrategy.java
│   │   ├── CreatedAtSortStrategy.java
│   │   ├── PrioritySortStrategy.java
│   │   ├── DueDateSortStrategy.java
│   │   └── SortStrategyFactory.java
│   └── exception/
│       ├── TodoNotFoundException.java
│       ├── AlreadyCompletedException.java
│       └── InvalidTodoTitleException.java
└── infrastructure/
    ├── jpa/
    │   └── TodoJpaEntity.java
    └── repository/
        ├── TodoJpaRepository.java
        └── TodoRepositoryImpl.java
```

---

*このロードマップは実装しながら更新していく。判断に迷ったらログを残すこと。*

---

## Phase 12 終了後の学習計画

> **目的**: Spring Boot と DDD 軽量版の設計パターンを「手が覚える」レベルまで定着させる。

### 学習順序

```
Phase 9〜12完了（今のToDoアプリを仕上げる）
        ↓
    ②別ドメインで再実装（2〜3個）← ここが一番大事
        ↓
    ①拡張（Docker、認証など）
        ↓
    ③DDD深掘り
```

### なぜこの順番か

| 順番 | 内容 | 理由 |
|------|------|------|
| **1st** | 別ドメインで再実装 | 同じパターンを3回繰り返すと手が覚える。1個だけだと「写経しただけ」になりがち |
| **2nd** | 拡張（Docker/認証） | アプリが作れる前提の話。先にやると設計よりインフラに意識が向く |
| **3rd** | DDD深掘り | 手が動かないうちに理論を深めても身につかない。疑問が出てからの方が吸収が早い |

### 再実装の題材候補

| 題材 | 複雑度 | 学べるポイント |
|------|--------|---------------|
| **家計簿アプリ** | ★★☆ | 集計、カテゴリ分類 |
| **予約システム** | ★★★ | 日時の扱い、状態遷移が複雑 |
| **ECサイト（注文管理）** | ★★★★ | 複数エンティティの関連、在庫管理 |

### 拡張で学ぶこと

| 拡張内容 | 学べること |
|---------|-----------|
| **Spring Security** | JWT、認証・認可 |
| **Docker化** | コンテナ、環境構築の自動化 |
| **MySQL/PostgreSQL** | 本番DB運用 |
| **GitHub Actions** | CI/CD、自動テスト |
| **AWS/GCPデプロイ** | クラウド運用 |

### DDD深掘りトピック

| トピック | 内容 |
|---------|------|
| **集約（Aggregate）** | 整合性の境界を設計 |
| **ドメインイベント** | 非同期処理、イベント駆動 |
| **境界づけられたコンテキスト** | マイクロサービスへの布石 |

> **ポイント**: 「手が慣れるまでガンガン作る」が最優先。理論は後からついてくる。
