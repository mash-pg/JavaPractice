# Phase 10: 品質ルール

> **目的**: チーム開発を想定したルールを学ぶ。ログ出力・レイヤ境界・SOLID原則を実践する。

---

## 概要

| トピック | 内容 |
|---------|------|
| **ログ出力** | SLF4J でログを出す |
| **レイヤ境界** | Entity を外部に漏らさない |
| **SOLID原則** | 設計の5原則を理解する |

---

## 課題1: ログ出力の追加

### 要件

| レベル | 場所 | 出力内容 |
|--------|------|---------|
| `INFO` | `CreateTodoUseCase` | ToDo 作成時 |
| `INFO` | `CompleteTodoUseCase` | ToDo 完了時 |
| `ERROR` | `GlobalExceptionHandler` | 例外発生時 |

### ログの書き方

**方法1: 手動で Logger を定義**

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class CompleteTodoUseCase {
    private static final Logger log = LoggerFactory.getLogger(CompleteTodoUseCase.class);

    public Todo execute(Long id) {
        // ...
        log.info("Todo completed: {}", savedTodo.getId());
        return savedTodo;
    }
}
```

**方法2: Lombok の @Slf4j を使う**

```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class CreateTodoUseCase {
    public Todo execute(String title) {
        // ...
        log.info("Todo created: {}, title: {}", savedTodo.getId(), title);
        return savedTodo;
    }
}
```

---

### 詰まったところ

#### Q: ログに `TodoId@75fe0a3f` と表示される

**ジュニア**: ログを追加したら、ID が `TodoId@75fe0a3f` って表示されました。

**シニア**: それは `TodoId` オブジェクトの `toString()` が呼ばれているからだね。`toString()` をオーバーライドしていないと、デフォルトで `クラス名@ハッシュ` が出力される。

**ジュニア**: どうすれば「1」とか「2」って表示されますか？

**シニア**: 2つ方法がある：

```java
// 方法1: ログ出力時に値を取り出す
log.info("Todo created: {}", savedTodo.getId().getValue());

// 方法2: TodoId に toString() を追加
public class TodoId {
    @Override
    public String toString() {
        return String.valueOf(value);
    }
}
```

**ジュニア**: なるほど、Value Object をそのままログに出すときは注意が必要なんですね。

---

#### Q: ログのタイミングはいつがいい？

**ジュニア**: `save()` の前と後、どっちでログを出すべきですか？

**シニア**: **保存後**がベター。理由は？

**ジュニア**: うーん...

**シニア**: 保存前だと ID がまだ確定していない場合がある。DB が自動採番する `AUTO_INCREMENT` の場合、`save()` 後に初めて ID が決まる。

```java
// 悪い例
log.info("Todo created: {}", todo.getId());  // ← null の可能性
todoRepository.save(todo);

// 良い例
Todo savedTodo = todoRepository.save(todo);
log.info("Todo created: {}", savedTodo.getId());  // ← 確定した ID
```

---

#### Q: 文字列連結 vs プレースホルダー

**ジュニア**: `"id: " + id` と `"id: {}", id` はどう違いますか？

**シニア**: パフォーマンスが違う。

```java
// 文字列連結（常に連結処理が走る）
log.debug("id: " + id + ", title: " + title);

// プレースホルダー（ログレベルが無効なら連結処理をスキップ）
log.debug("id: {}, title: {}", id, title);
```

本番環境で DEBUG ログを無効にしている場合、プレースホルダー版は何も処理しない。文字列連結版は無駄に連結処理が走る。

---

## 課題2: レイヤ境界ルール

### ルール

| ルール | 意味 |
|--------|------|
| 入力は DTO で受け取る | Controller は `CreateTodoRequest` を受け取る |
| 出力は DTO で返す | Controller は `TodoResponse` を返す |
| Entity を外部に漏らさない | `Todo` を直接 JSON で返さない |

### 確認ポイント

```java
@PostMapping
public ResponseEntity<TodoResponse> create(@RequestBody CreateTodoRequest request) {
    //                  ↑DTO                              ↑DTO
    Todo todo = createTodoUseCase.execute(request.getTitle());
    //   ↑Entity（内部でのみ使用）
    TodoResponse response = TodoResponse.from(todo);
    //           ↑DTO に変換
    return ResponseEntity.status(HttpStatus.CREATED).body(response);
    //                                                    ↑DTO を返す
}
```

### 流れ図

```
[クライアント]
     │
     │  JSON: {"title": "買い物"}
     ▼
[Controller]
     │  CreateTodoRequest (DTO) で受け取る
     ▼
[UseCase]
     │  Todo (Entity) を返す
     ▼
[Controller]
     │  TodoResponse.from(todo) で変換
     ▼
[クライアント]
     │  JSON: {"id": 1, "title": "買い物", ...}
```

---

### 詰まったところ

#### Q: なぜ Entity を直接返してはダメ？

**ジュニア**: `Todo` をそのまま返せば変換処理が不要で楽なのに、なぜ DTO に変換するんですか？

**シニア**: 3つ理由がある：

| 理由 | 説明 |
|------|------|
| **API の安定性** | Entity に `password` フィールドを追加したら、API にも漏れる |
| **柔軟性** | レスポンスの形式を API の都合で変えられる |
| **責務の分離** | Entity は DB の構造、DTO は API の構造 |

**ジュニア**: Entity を変更したら API も変わっちゃうってことですね。

**シニア**: そう。Entity は「DB に何を保存するか」、DTO は「API で何を返すか」。別の関心事だから分ける。

---

## 課題3: SOLID 原則

### SOLID とは

ソフトウェア設計の5つの基本原則の頭文字。

| 原則 | 名前 | 一言で言うと |
|------|------|-------------|
| **S** | Single Responsibility Principle | 1クラス1責任 |
| **O** | Open/Closed Principle | 拡張に開き、修正に閉じる |
| **L** | Liskov Substitution Principle | 子クラスは親クラスと置換可能 |
| **I** | Interface Segregation Principle | インターフェースは小さく分ける |
| **D** | Dependency Inversion Principle | 抽象に依存する |

---

### S: Single Responsibility Principle（単一責任の原則）

> **クラスを変更する理由は1つだけであるべき**

```java
// 悪い例：1つのクラスが複数の責務を持つ
public class TodoService {
    public void createTodo() { ... }      // 作成
    public void sendEmail() { ... }        // メール送信
    public void generateReport() { ... }   // レポート生成
}

// 良い例：責務ごとにクラスを分ける
public class CreateTodoUseCase { ... }     // 作成だけ
public class EmailService { ... }           // メール送信だけ
public class ReportService { ... }          // レポート生成だけ
```

**ToDo アプリでの例:**
- `CreateTodoUseCase` → 作成だけ
- `CompleteTodoUseCase` → 完了だけ
- `TodoFactory` → 生成だけ

---

### O: Open/Closed Principle（開放閉鎖の原則）

> **拡張に対して開いていて、修正に対して閉じている**

```java
// 悪い例：新しいソート方法を追加するたびに修正が必要
public List<Todo> sort(String sortType, List<Todo> todos) {
    if (sortType.equals("createdAt")) {
        // 作成日ソート
    } else if (sortType.equals("priority")) {
        // 優先度ソート
    } else if (sortType.equals("dueDate")) {  // ← 追加するたびにここを修正
        // 期限ソート
    }
}

// 良い例：新しいクラスを追加するだけ
interface SortStrategy {
    List<Todo> sort(List<Todo> todos);
}

class CreatedAtSortStrategy implements SortStrategy { ... }
class PrioritySortStrategy implements SortStrategy { ... }
class TitleSortStrategy implements SortStrategy { ... }  // ← 追加するだけ
```

**ToDo アプリでの例:**
- `SortStrategy` インターフェース
- 新しいソート方法 = 新しいクラスを追加

---

### L: Liskov Substitution Principle（リスコフの置換原則）

> **子クラスは親クラスと置き換えても正しく動作すべき**

```java
// インターフェースを実装したクラスは、同じ振る舞いを保証する
SortStrategy strategy = new CreatedAtSortStrategy();
strategy = new PrioritySortStrategy();  // ← 置き換えても sort() が正しく動く
strategy = new DueDateSortStrategy();   // ← 置き換えても sort() が正しく動く
```

---

### I: Interface Segregation Principle（インターフェース分離の原則）

> **クライアントが使わないメソッドに依存させない**

```java
// 悪い例：巨大なインターフェース
interface TodoRepository {
    Todo save(Todo todo);
    Todo findById(TodoId id);
    List<Todo> findAll();
    void deleteById(TodoId id);
    void backup();           // ← 使わないクライアントもある
    void migrate();          // ← 使わないクライアントもある
}

// 良い例：必要なものだけ
interface TodoRepository {
    Todo save(Todo todo);
    Optional<Todo> findById(TodoId id);
    List<Todo> findAll();
    void deleteById(TodoId id);
}
```

---

### D: Dependency Inversion Principle（依存性逆転の原則）

> **具体ではなく抽象に依存する**

```java
// 悪い例：具体クラスに依存
public class ListTodosUseCase {
    private final CreatedAtSortStrategy strategy;  // ← 具体クラス
}

// 良い例：インターフェースに依存
public class ListTodosUseCase {
    private final SortStrategy strategy;  // ← インターフェース
}
```

**ToDo アプリでの例:**
- `UseCase` は `TodoRepository`（interface）に依存
- `UseCase` は `SortStrategy`（interface）に依存

---

### 詰まったところ

#### Q: Strategy パターンはどの原則を満たしている？

**ジュニア**: Phase 9 で作った Strategy パターンは SOLID のどれを満たしていますか？

**シニア**: 主に3つ満たしている：

| 原則 | どう満たしているか |
|------|-------------------|
| **S** | 各 Strategy は1つのソート方法だけ担当 |
| **O** | 新しいソートを追加しても既存コード変更不要 |
| **D** | UseCase は `SortStrategy`（interface）に依存 |

**ジュニア**: 「拡張に開き、修正に閉じる」ってそういう意味だったんですね。

**シニア**: そう。if-else で分岐を増やすのは「修正」、新しいクラスを追加するのは「拡張」。後者の方が既存コードを壊すリスクが低い。

---

## まとめ

### ログ出力

| ポイント | 内容 |
|---------|------|
| Logger 定義 | `@Slf4j` または `LoggerFactory.getLogger()` |
| フォーマット | プレースホルダー `{}` を使う |
| タイミング | 保存後（ID 確定後）にログ出力 |
| VO の注意 | `toString()` か `getValue()` で値を取り出す |

### レイヤ境界

| ルール | 守り方 |
|--------|--------|
| 入力は DTO | `@RequestBody CreateTodoRequest` |
| 出力は DTO | `TodoResponse.from(entity)` |
| Entity 隠蔽 | Controller 内で変換 |

### SOLID

| 原則 | ToDo アプリでの例 |
|------|------------------|
| S | UseCase を機能ごとに分割 |
| O | Strategy パターンで拡張可能 |
| L | SortStrategy の実装は置換可能 |
| I | 必要最小限の Repository interface |
| D | UseCase は interface に依存 |

---

## 修正したファイル

```
src/main/java/com/example/todo/
├── application/
│   └── usecase/
│       ├── CreateTodoUseCase.java     ← ログ追加
│       └── CompleteTodoUseCase.java   ← ログ追加
└── presentation/
    └── exception/
        └── GlobalExceptionHandler.java ← ログ追加
```

---

## 次のステップ

- [ ] Phase 11: テスト へ進む

---

*このドキュメントは学習中に詰まった点と解決方法を記録したものです。*
