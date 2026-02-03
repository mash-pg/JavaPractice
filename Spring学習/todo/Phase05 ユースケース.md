# Phase 5: ユースケース設計（Application層）

> **目的**: 「この API では何をするか」という処理の流れを定義する

---

## 事前知識

UseCaseを実装する前に、**Repositoryインターフェース** が必要になる。
UseCaseはデータの保存・取得をRepositoryに委譲するため。

```
UseCase → Repository(interface) ← RepositoryImpl
```

この設計により:
- UseCase は「何をするか」に集中できる
- DBの実装詳細を知らなくて済む
- テスト時にモックに差し替え可能

---

## 課題 5-1: TodoRepository インターフェースの作成

**場所**: `com.example.todo.domain.repository.TodoRepository`

| メソッド | 説明 |
|---------|------|
| `Todo save(Todo todo)` | 保存（新規/更新両方） |
| `Optional<Todo> findById(TodoId id)` | ID検索 |
| `List<Todo> findAll()` | 全件取得 |
| `void deleteById(TodoId id)` | 削除 |

**ポイント**:
- インターフェースは `domain` パッケージに置く（ドメインが定義するポート）
- JPA や SQL といった技術用語は一切使わない
- 戻り値・引数はすべてドメインオブジェクトを使う

### 実装結果

```java
package com.example.todo.domain.repository;

import java.util.List;
import java.util.Optional;

import com.example.todo.domain.entity.Todo;
import com.example.todo.domain.valueobject.TodoId;

public interface TodoRepository {
    Todo save(Todo todo);
    Optional<Todo> findById(TodoId id);
    List<Todo> findAll();
    void deleteById(TodoId id);
}
```

**レビュー**: 合格

---

## 課題 5-2: CreateTodoUseCase の作成

**場所**: `com.example.todo.application.usecase.CreateTodoUseCase`

**仕様**:
- タイトルを受け取り、新しい Todo を作成して保存する
- 保存した Todo を返す
- `@Service` アノテーションを付ける
- コンストラクタで `TodoRepository` を受け取る（コンストラクタインジェクション）

### 実装結果

```java
@Service
public class CreateTodoUseCase {
    private final TodoRepository todoRepository;

    public CreateTodoUseCase(TodoRepository todoRepository){
        this.todoRepository = todoRepository;
    }

    public Todo execute(String title) {
        TodoTitle todoTitle = new TodoTitle(title);
        Todo todo = new Todo(todoTitle);
        Todo todoResult = todoRepository.save(todo);
        return todoResult;
    }
}
```

**レビュー**: 合格（軽微な指摘あり）

**指摘**: 変数名のキャメルケース
```java
// 最初の実装（修正前）
private final TodoRepository todorepository;  // NG
Todo todoresult = todorepository.save(todo);  // NG

// 推奨（Javaの命名規則）
private final TodoRepository todoRepository;  // OK
Todo savedTodo = todoRepository.save(todo);   // OK
```

---

## 課題 5-3: 残りのユースケースを実装

| クラス名 | 責務 | 戻り値 |
|---------|------|--------|
| `GetTodoUseCase` | IDで1件取得 | `Todo`（見つからなければ例外） |
| `ListTodosUseCase` | 全件取得 | `List<Todo>` |
| `UpdateTodoUseCase` | タイトル更新 | `Todo` |
| `CompleteTodoUseCase` | 完了にする | `Todo` |

### GetTodoUseCase - 合格

```java
@Service
public class GetTodoUseCase {
    private final TodoRepository todoRepository;

    public GetTodoUseCase(TodoRepository todoRepository) {
        this.todoRepository = todoRepository;
    }

    public Todo execute(Long id) {
        TodoId todoId = new TodoId(id);
        return todoRepository.findById(todoId)
            .orElseThrow(()->new TodoNotFoundException("Todo not found: " + id));
    }
}
```

### ListTodosUseCase - 合格

```java
@Service
public class ListTodosUseCase {
    private final TodoRepository todoRepository;

    public ListTodosUseCase(TodoRepository todoRepository) {
        this.todoRepository = todoRepository;
    }

    public List<Todo> execute(){
        return todoRepository.findAll();
    }
}
```

---

## 詰まったポイント: UpdateTodoUseCase と CompleteTodoUseCase

### 最初の疑問

> クラスのGetTodoUseCaseとListTodosUseCaseまでは終了しましたが、
> UpdateTodoUseCaseとCompleteTodoUseCaseが判りません。
> saveで更新するのとchangeTitleでsave処理を実装する感じでしょうか？

### 最初に書いた（間違った）コード

```java
public Todo execute(Long id, TodoTitle title) {
    Todo todoTitle = new Todo(title);  // ← 新しいTodoを作ってしまっている（間違い）
    TodoId todoId = new TodoId(id);
    Todo updateTitle = todoRepository.findById(todoId)
        .orElseThrow(()->new TodoNotFoundException("Todo not found: " + id));

    updateTitle.changeTitle();  // ← 引数がない、returnもない（未完成）
}
```

### シニアからの解説

**混乱のポイントは「どこに何を書くか」**

```
┌─────────────────────────────────────────────────────────┐
│  UpdateTodoUseCase（処理の流れを書く）                    │
│                                                         │
│  1. Repository から既存の Todo を取得                    │
│  2. Todo.changeTitle(newTitle) を呼ぶ ←── Entityのメソッド│
│  3. Repository.save(todo) で保存                        │
│  4. 保存結果を返す                                       │
└─────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  Todo Entity（ビジネスルールを書く）                      │
│                                                         │
│  public void changeTitle(TodoTitle newTitle) {          │
│      this.title = newTitle;  // タイトルを変更           │
│  }                                                      │
└─────────────────────────────────────────────────────────┘
```

**ポイント**:
1. `changeTitle` は **Todo Entity に追加するメソッド**（UseCaseに書くのではない）
2. `save()` は新規作成と更新の両方に使う（IDがあれば更新、なければ新規）
3. UseCase は「流れ」だけを書く

### 理解したこと

> エンティティでフィールドデータを書き換える必要があるんですね。
> そして、エンティティでcomplete()処理を実行するんですね。

**→ 引数のデータ更新やな**

---

## 修正後の実装

### Todo Entity に追加したメソッド

```java
// 引数のデータ更新やな
public void changeTitle(TodoTitle title) {
    this.title = title;
}
```

### UpdateTodoUseCase - 合格

```java
@Service
public class UpdateTodoUseCase {
    private final TodoRepository todoRepository;

    public UpdateTodoUseCase(TodoRepository todoRepository) {
        this.todoRepository = todoRepository;
    }

    public Todo execute(Long id, String newTitle) {
        // 1. IDで既存のTodoを取得
        TodoId todoId = new TodoId(id);
        Todo todo = todoRepository.findById(todoId)
            .orElseThrow(()->new TodoNotFoundException("Todo not found: " + id));
        // 2. タイトルを変更（Entityのメソッドを呼ぶ）
        TodoTitle title = new TodoTitle(newTitle);
        todo.changeTitle(title);
        // 3. 保存して返す
        return todoRepository.save(todo);
    }
}
```

### CompleteTodoUseCase - 合格

```java
@Service
public class CompleteTodoUseCase {
    private final TodoRepository todoRepository;

    public CompleteTodoUseCase(TodoRepository todoRepository) {
        this.todoRepository = todoRepository;
    }

    public Todo execute(Long id) {
        TodoId todoId = new TodoId(id);
        Todo todo = todoRepository.findById(todoId)
            .orElseThrow(()->new TodoNotFoundException("Todo not found: " + id));
        todo.complete();  // ドメインロジックを呼ぶ
        return todoRepository.save(todo);
    }
}
```

### TodoNotFoundException

```java
package com.example.todo.domain.exception;

public class TodoNotFoundException extends RuntimeException {
    public TodoNotFoundException(String message) {
        super(message);
    }
}
```

---

## Phase 5 の学び

```
┌────────────────────────────────────────────────────┐
│ UseCase（Application層）の役割                      │
│                                                    │
│  ・処理の「流れ」を記述する                          │
│  ・取得 → 変更 → 保存 というパターン                 │
│  ・ビジネスルールは Entity に任せる                  │
│    （complete() の二重完了チェックなど）             │
└────────────────────────────────────────────────────┘
```

### 重要な気づき

| 層 | 責務 |
|---|------|
| **UseCase** | 処理の「流れ」を書く（取得→変更→保存） |
| **Entity** | 状態を変更するメソッドを持つ（`changeTitle`, `complete`） |
| **Repository** | データの永続化（新規も更新も `save()` で行う） |

---

## 作成したファイル一覧

```
src/main/java/com/example/todo/
├── domain/
│   ├── repository/
│   │   └── TodoRepository.java          ← インターフェース（課題5-1）
│   ├── entity/
│   │   └── Todo.java                    ← changeTitle()メソッド追加
│   └── exception/
│       └── TodoNotFoundException.java   ← 新規作成
└── application/
    └── usecase/
        ├── CreateTodoUseCase.java       ← 課題5-2
        ├── GetTodoUseCase.java          ← 課題5-3
        ├── ListTodosUseCase.java        ← 課題5-3
        ├── UpdateTodoUseCase.java       ← 課題5-3（詰まったポイント）
        └── CompleteTodoUseCase.java     ← 課題5-3（詰まったポイント）
```

---

## 次のフェーズ

**Phase 6: Repository（インフラ層）** - 実際にDBとやり取りする実装を作る
