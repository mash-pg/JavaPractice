# Phase 9: パターン適用

> **目的**: 設計パターンを実践し、拡張性・保守性の高いコードを書けるようになる。

---

## 概要

| パターン | 目的 | 作成したクラス |
|---------|------|---------------|
| **Factory** | 生成ルールを一箇所に集約 | `TodoFactory` |
| **Strategy** | 処理を切り替え可能に | `SortStrategy` + 3つの実装 |
| **Strategy Factory** | どの Strategy を使うか選ぶ | `SortStrategyFactory` |

---

## 課題1: Factory パターン

### 背景

`CreateTodoUseCase` に Todo の生成ロジックが直接書かれていた：

```java
// 修正前：UseCase に生成ロジックがある
public Todo execute(String title) {
    TodoTitle todoTitle = new TodoTitle(title);
    Todo todo = new Todo(todoTitle);  // ← 生成ロジックが UseCase に散らばる
    Todo savedTodo = todoRepository.save(todo);
    return savedTodo;
}
```

**問題点:**
- Todo の生成ロジックが UseCase に散らばる可能性がある
- 将来「description も指定したい」「priority も指定したい」となったとき、一貫性が保てなくなる

### 要件

`TodoFactory` クラスを作成：
1. パッケージ: `com.example.todo.domain.factory`
2. メソッド: `create(String title)` → `Todo` を返す
3. Factory が行うこと:
   - title の前後空白を除去（trim）
   - `createdAt` に現在日時を自動設定

---

### 最初の実装（間違い）

```java
@Component
public class TodoFactory {

    private final TodoRepository todoRepository;  // ❌ 問題1: Factory に Repository を注入

    public TodoFactory(TodoRepository todorepository){
        this.todoRepository = todorepository;
    }

    public Todo create(String title) {
        String titletrim = title.trim();
        TodoTitle todoTitle = new TodoTitle(titletrim);

        Todo todo = new Todo(todoTitle);
        todo.changeTitle(todoTitle);  // ❌ 問題2: 重複（コンストラクタで既に設定済み）

        LocalDateTime createAt = LocalDateTime.now();
        todo.changecreatAt(createAt);  // ❌ 問題3: 命名規則違反（changeCreatedAt が正しい）

        return todoRepository.save(todo);  // ❌ 問題4: Factory が保存してはダメ
    }
}
```

### 間違いと修正

| 問題 | 間違い | 正しい考え方 |
|------|--------|-------------|
| **責務の分離** | Factory で `save()` を呼んでいた | Factory は「作る」だけ。「保存する」のは UseCase の仕事 |
| **重複コード** | コンストラクタで設定済みの title を再設定 | 不要なコードは削除 |
| **命名規則** | `changecreatAt` | Java はキャメルケース → `changeCreatedAt` |
| **不要な依存** | Factory に Repository を注入 | Factory は Repository を知らなくて良い |

---

### 正しい実装

**TodoFactory.java:**

```java
package com.example.todo.domain.factory;

import java.time.LocalDateTime;
import org.springframework.stereotype.Component;
import com.example.todo.domain.entity.Todo;
import com.example.todo.domain.valueobject.TodoTitle;

@Component
public class TodoFactory {

    public Todo create(String title) {
        // title の前後空白を除去（trim）してから TodoTitle に渡す
        String titletrim = title.trim();
        TodoTitle todoTitle = new TodoTitle(titletrim);

        // コンストラクタで title 設定
        Todo todo = new Todo(todoTitle);

        // createdAt に現在日時を自動設定
        LocalDateTime createAt = LocalDateTime.now();
        todo.changeCreatedAt(createAt);

        return todo;  // ← 保存せず、作った Todo を返すだけ
    }
}
```

**CreateTodoUseCase.java（修正後）:**

```java
@Service
@Transactional
public class CreateTodoUseCase {
    private final TodoRepository todoRepository;
    private final TodoFactory todoFactory;  // ← Factory を DI

    public CreateTodoUseCase(TodoRepository todoRepository, TodoFactory todoFactory){
        this.todoRepository = todoRepository;
        this.todoFactory = todoFactory;
    }

    public Todo execute(String title) {
        Todo todo = todoFactory.create(title);  // ← Factory で生成
        return todoRepository.save(todo);       // ← UseCase で保存
    }
}
```

---

### 学んだこと

#### 役割分担の重要性

| クラス | 責務 |
|--------|------|
| **Factory** | オブジェクトを「作る」 |
| **UseCase** | 作ったオブジェクトを「保存する」 |

#### @Component を書かないとどうなる？

```java
// @Component がないと...
public class TodoFactory { ... }
```

Spring が「このクラスは DI 対象じゃない」と判断し、起動時にエラー：

```
No qualifying bean of type 'TodoFactory' available
```

---

## 課題2: Strategy パターン

### 背景

`ListTodosUseCase` は全件取得するだけだった：

```java
public List<Todo> execute(){
    return todoRepository.findAll();  // ← ソート順を選べない
}
```

**要件**: ソート順を選べるようにしたい
- 作成日順（デフォルト）
- 優先度順（HIGH → MEDIUM → LOW）
- 期限順（早い順、null は最後）

---

### Strategy パターンとは

「処理のやり方」を切り替え可能にするパターン。

```
┌─────────────┐
│ UseCase     │ ←── どの戦略を使うか選ぶ
└──────┬──────┘
       │
       ▼
┌─────────────────┐
│ SortStrategy    │ ←── インターフェース
│  + sort(list)   │
└────────┬────────┘
         │
    ┌────┴────┬──────────┐
    ▼         ▼          ▼
 CreatedAt  Priority   DueDate
 Strategy   Strategy   Strategy
```

---

### Step 1: インターフェース作成

**SortStrategy.java:**

```java
package com.example.todo.domain.strategy;

import java.util.List;
import com.example.todo.domain.entity.Todo;

public interface SortStrategy {
    List<Todo> sort(List<Todo> todos);
}
```

---

### Step 2: 実装クラス作成

#### 詰まったところ: Java のソート処理

**最初わからなかったこと:**
- `Comparator.comparing()` の使い方
- リストのコピーを作る理由

**学んだこと:**

```java
// Comparator.comparing() の基本
todos.sort(Comparator.comparing(todo -> todo.getCreatedAt()));

// メソッド参照を使うとより簡潔
todos.sort(Comparator.comparing(Todo::getCreatedAt));

// 降順にしたい場合
todos.sort(Comparator.comparing(Todo::getCreatedAt).reversed());

// null がある場合（dueDate は null の可能性あり）
todos.sort(Comparator.comparing(
    Todo::getDueDate,
    Comparator.nullsLast(Comparator.naturalOrder())
));
```

**なぜコピーを作る？**

```java
// 元のリストを変更してしまうと、呼び出し元に影響が出る
List<Todo> sorted = new ArrayList<>(todos);  // コピーを作る
sorted.sort(...);  // コピーをソート
return sorted;
```

---

#### 最初の実装（バグあり）

```java
@Override
public List<Todo> sort(List<Todo> todos) {
    List<Todo> sorted = new ArrayList<>(todos);  // コピーを作った
    todos.sort(Comparator.comparing(todo->todo.getCreatedAt()));  // ❌ 元のリストをソートしている！
    return sorted;  // ❌ ソートされてないコピーを返している
}
```

**バグ**: せっかくコピーを作ったのに、元の `todos` をソートしていた。

---

#### 正しい実装

**CreatedAtSortStrategy.java:**

```java
package com.example.todo.domain.strategy;

import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;
import org.springframework.stereotype.Component;
import com.example.todo.domain.entity.Todo;

@Component
public class CreatedAtSortStrategy implements SortStrategy {
    @Override
    public List<Todo> sort(List<Todo> todos) {
        List<Todo> sorted = new ArrayList<>(todos);
        sorted.sort(Comparator.comparing(todo -> todo.getCreatedAt()));  // ← コピーをソート
        return sorted;
    }
}
```

**PrioritySortStrategy.java:**

```java
@Component
public class PrioritySortStrategy implements SortStrategy {
    @Override
    public List<Todo> sort(List<Todo> todos) {
        List<Todo> sorted = new ArrayList<>(todos);
        // enum の ordinal() で比較（HIGH=0, MEDIUM=1, LOW=2）
        sorted.sort(Comparator.comparing(todo -> todo.getPriority().ordinal()));
        return sorted;
    }
}
```

**DueDateSortStrategy.java:**

```java
@Component
public class DueDateSortStrategy implements SortStrategy {
    @Override
    public List<Todo> sort(List<Todo> todos) {
        List<Todo> sorted = new ArrayList<>(todos);
        // nullsLast: null は最後に配置
        sorted.sort(Comparator.comparing(
            Todo::getDueDate,
            Comparator.nullsLast(Comparator.naturalOrder())
        ));
        return sorted;
    }
}
```

---

### Step 3: Strategy Factory 作成

**問題**: `sortType` が `"priority"` なら `PrioritySortStrategy` を使いたい。どうやって選ぶ？

**解決**: Strategy を選ぶ Factory を作る。

**SortStrategyFactory.java:**

```java
package com.example.todo.domain.strategy;

import org.springframework.stereotype.Component;

@Component
public class SortStrategyFactory {

    private final CreatedAtSortStrategy createdAtSort;
    private final PrioritySortStrategy prioritySort;
    private final DueDateSortStrategy dueDateSort;

    public SortStrategyFactory(
            CreatedAtSortStrategy createdAtSort,
            PrioritySortStrategy prioritySort,
            DueDateSortStrategy dueDateSort) {
        this.createdAtSort = createdAtSort;
        this.prioritySort = prioritySort;
        this.dueDateSort = dueDateSort;
    }

    public SortStrategy create(String sortType) {
        return switch (sortType) {
            case "priority" -> prioritySort;
            case "dueDate" -> dueDateSort;
            default -> createdAtSort;  // デフォルトは作成日順
        };
    }
}
```

---

### Step 4: UseCase の修正

#### 詰まったところ

**最初の間違い:**

```java
public List<Todo> execute(){
    return sortStrategy.create(null);  // ❌ SortStrategy を返している（Todo リストじゃない）
}
```

**理解すべき流れ:**

```
1. DBから全件取得      → todoRepository.findAll()
2. Strategyを選ぶ      → sortStrategyFactory.create(sortType)
3. ソートを実行        → strategy.sort(todos)
4. 結果を返す          → return sortedTodos
```

**正しい実装:**

```java
@Service
public class ListTodosUseCase {

    private final TodoRepository todoRepository;
    private final SortStrategyFactory sortStrategyFactory;

    public ListTodosUseCase(TodoRepository todoRepository, SortStrategyFactory sortStrategyFactory) {
        this.todoRepository = todoRepository;
        this.sortStrategyFactory = sortStrategyFactory;
    }

    public List<Todo> execute(String sortType){
        // 1. DBから全件取得
        List<Todo> todos = todoRepository.findAll();

        // 2. sortType に応じた Strategy を取得
        SortStrategy strategy = sortStrategyFactory.create(sortType);

        // 3. Strategy でソートして返す
        return strategy.sort(todos);
    }
}
```

---

### Step 5: Controller の修正

#### 詰まったところ: クエリパラメータの受け取り方

**クエリパラメータとは？**

```
GET /api/todos              → ソートなし（デフォルト）
GET /api/todos?sort=priority   → 優先度順
GET /api/todos?sort=dueDate    → 期限順
```

`?sort=xxx` の部分がクエリパラメータ。

**解決: @RequestParam を使う**

```java
@GetMapping
public ResponseEntity<List<TodoResponse>> list(
        @RequestParam(defaultValue = "createdAt") String sort  // ← クエリパラメータを受け取る
) {
    List<Todo> todos = listtodousecase.execute(sort);
    List<TodoResponse> responses = todos.stream()
            .map(TodoResponse::from)
            .collect(Collectors.toList());
    return ResponseEntity.ok(responses);
}
```

**@RequestParam のポイント:**

| 設定 | 意味 |
|------|------|
| `@RequestParam` | クエリパラメータを受け取る |
| `defaultValue = "createdAt"` | パラメータがない場合のデフォルト値 |

---

## 動作確認

### Factory パターン

```powershell
# 前後の空白が trim されることを確認
Invoke-RestMethod -Uri "http://localhost:8080/api/todos" -Method POST -Body '{"title":"  hello world  "}' -ContentType "application/json"

# 結果: title が "hello world" になる（空白除去）
# 結果: createdAt が自動設定される
```

### Strategy パターン

```powershell
# テストデータの priority を H2 Console で変更
# UPDATE TODOS SET PRIORITY = 'HIGH' WHERE ID = 3;
# UPDATE TODOS SET PRIORITY = 'MEDIUM' WHERE ID = 5;

# 優先度順（HIGH → MEDIUM → LOW）
Invoke-RestMethod -Uri "http://localhost:8080/api/todos?sort=priority" -Method GET

# 期限順（早い日付順、null は最後）
Invoke-RestMethod -Uri "http://localhost:8080/api/todos?sort=dueDate" -Method GET
```

---

## 作成したファイル一覧

```
src/main/java/com/example/todo/
├── domain/
│   ├── factory/
│   │   └── TodoFactory.java          ← NEW
│   └── strategy/
│       ├── SortStrategy.java          ← NEW（インターフェース）
│       ├── CreatedAtSortStrategy.java ← NEW
│       ├── PrioritySortStrategy.java  ← NEW
│       ├── DueDateSortStrategy.java   ← NEW
│       └── SortStrategyFactory.java   ← NEW
├── application/
│   └── usecase/
│       ├── CreateTodoUseCase.java     ← 修正（Factory を使用）
│       └── ListTodosUseCase.java      ← 修正（Strategy を使用）
└── presentation/
    └── controller/
        └── TodoController.java        ← 修正（@RequestParam 追加）
```

---

## まとめ

### Factory パターン

| 項目 | 内容 |
|------|------|
| **目的** | 生成ルールを一箇所に集約 |
| **メリット** | 生成ロジックの変更が1箇所で済む |
| **注意点** | Factory は「作る」だけ。「保存」は UseCase の責務 |

### Strategy パターン

| 項目 | 内容 |
|------|------|
| **目的** | 処理のやり方を切り替え可能にする |
| **メリット** | 新しいソート方法を追加しても既存コードを変更しない（OCP） |
| **構成** | インターフェース + 複数の実装 + 選択する Factory |

### 覚えるべき Java の書き方

```java
// リストのソート
list.sort(Comparator.comparing(Todo::getCreatedAt));

// 降順
list.sort(Comparator.comparing(Todo::getCreatedAt).reversed());

// null 対応
list.sort(Comparator.comparing(Todo::getDueDate, Comparator.nullsLast(Comparator.naturalOrder())));

// リストのコピー
List<Todo> copy = new ArrayList<>(original);
```

---

## 次のステップ

- [ ] Phase 10: 品質ルール へ進む

---

*このドキュメントは学習中に詰まった点と解決方法を記録したものです。*
