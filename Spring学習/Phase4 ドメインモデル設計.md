# Phase 4: ドメインモデル設計

> **目的**: ビジネスルールをコードで表現する。バグの温床になりやすい部分を domain 層に閉じ込める。

---

## Entity と Value Object の違い

| 概念 | 区別方法 | 例 |
|------|---------|-----|
| **Entity** | ID で区別する | ToDo、ユーザー |
| **Value Object** | 値で区別する | タイトル、金額、ステータス |

### 判断基準

| 質問 | 答え |
|------|------|
| 「ID で区別するか？」 | Yes → Entity |
| 「値が同じなら同じものか？」 | Yes → Value Object |

### 具体例

```
【Entity】
田中太郎（社員ID: 001）  ← 営業部の田中さん
田中太郎（社員ID: 055）  ← 経理部の田中さん（同姓同名の別人）
→ 名前が同じでも、ID が違えば別人。だから Entity。

【Value Object】
1000円 = 1000円（どの1000円かは関係ない）
"買い物に行く" = "買い物に行く"（同じ文字列は同じ値）
```

---

## 作成したクラス一覧

| 種類 | クラス | 場所 |
|------|--------|------|
| Entity | `Todo` | domain.entity |
| Value Object | `TodoId` | domain.valueobject |
| Value Object | `TodoTitle` | domain.valueobject |
| Value Object | `TodoStatus` | domain.valueobject |
| Exception | `InvalidTodoTitleException` | domain.exception |
| Exception | `AlreadyCompletedException` | domain.exception |

---

## 各クラスの実装

### TodoStatus（enum）

```java
package com.example.todo.domain.valueobject;

public enum TodoStatus {
    OPEN,
    DONE
}
```

- シンプルな enum で状態を表現
- 文字列ではなく enum を使うことで、タイポを防ぐ

---

### TodoTitle（バリデーション付き VO）

```java
package com.example.todo.domain.valueobject;

import com.example.todo.domain.exception.InvalidTodoTitleException;

public class TodoTitle {

    private static final int MAX_LENGTH = 100;
    private final String value;

    public TodoTitle(String value) {
        // バリデーション: null・空文字チェック
        if (value == null || value.isEmpty()) {
            throw new InvalidTodoTitleException("タイトルは必須です");
        }
        // バリデーション: 最大文字数チェック
        if (value.length() > MAX_LENGTH) {
            throw new InvalidTodoTitleException("タイトルは" + MAX_LENGTH + "文字以内で入力してください");
        }
        this.value = value;
    }

    public String getValue() {
        return value;
    }
}
```

#### 学んだこと: 文字列比較

```java
// NG: == は参照比較（たまたま動くこともあるが危険）
if (value == "")

// OK: isEmpty() で空文字チェック
if (value.isEmpty())

// OK: isBlank() で空白のみもチェック（"   " も true）
if (value.isBlank())
```

| メソッド | `""` | `"   "` | `"abc"` |
|---------|------|---------|---------|
| `isEmpty()` | true | false | false |
| `isBlank()` | true | true | false |

---

### TodoId（シンプルな VO）

```java
package com.example.todo.domain.valueobject;

public class TodoId {
    private final Long id;

    public TodoId(Long id) {
        if (id == null) {
            throw new IllegalArgumentException("IDは必須です");
        }
        this.id = id;
    }

    public Long getId() {
        return id;
    }
}
```

#### 学んだこと: 例外の使い分け

| 例外 | 使う場面 |
|------|---------|
| 専用例外（`InvalidTodoTitleException`） | ビジネスルール違反、ユーザーに伝えたいエラー |
| 汎用例外（`IllegalArgumentException`） | 単純な引数チェック、プログラムのバグ |

---

### Todo（Entity）

```java
package com.example.todo.domain.entity;

import com.example.todo.domain.exception.AlreadyCompletedException;
import com.example.todo.domain.valueobject.TodoId;
import com.example.todo.domain.valueobject.TodoStatus;
import com.example.todo.domain.valueobject.TodoTitle;

public class Todo {
    private final TodoId id;
    private TodoTitle title;
    private TodoStatus status;

    // コンストラクタ: 新規作成用（IDなし、ステータスはOPEN）
    public Todo(TodoTitle title) {
        this.id = null;
        this.title = title;
        this.status = TodoStatus.OPEN;
    }

    // コンストラクタ: DBから復元用（IDあり）
    public Todo(TodoId id, TodoTitle title, TodoStatus status) {
        this.id = id;
        this.title = title;
        this.status = status;
    }

    // ビジネスルール: 完了にする
    public void complete() {
        if (this.status == TodoStatus.DONE) {
            throw new AlreadyCompletedException("すでに完了しています");
        }
        this.status = TodoStatus.DONE;
    }

    // Getter
    public TodoId getId() { return id; }
    public TodoTitle getTitle() { return title; }
    public TodoStatus getStatus() { return status; }
}
```

#### ポイント: コンストラクタを2つ用意する理由

| コンストラクタ | 用途 |
|---------------|------|
| `Todo(TodoTitle title)` | 新規作成時（ID は DB が自動採番） |
| `Todo(TodoId, TodoTitle, TodoStatus)` | DB から読み込んで復元する時 |

#### ポイント: ビジネスルールを Entity に閉じ込める

```java
public void complete() {
    if (this.status == TodoStatus.DONE) {
        throw new AlreadyCompletedException("すでに完了しています");
    }
    this.status = TodoStatus.DONE;
}
```

- 「すでに完了済みなら例外」というルールを Entity が持つ
- UseCase や Controller にこのロジックを書かない
- → ルールが散らばらない、テストしやすい

---

### 例外クラス

```java
// InvalidTodoTitleException.java
package com.example.todo.domain.exception;

public class InvalidTodoTitleException extends RuntimeException {
    public InvalidTodoTitleException(String message) {
        super(message);
    }
}

// AlreadyCompletedException.java
package com.example.todo.domain.exception;

public class AlreadyCompletedException extends RuntimeException {
    public AlreadyCompletedException(String message) {
        super(message);
    }
}
```

---

## パッケージ構成（Phase 4 完了時点）

```
domain/
├── entity/
│   └── Todo.java              ← ビジネスルールの中心
├── valueobject/
│   ├── TodoId.java            ← ID（null禁止）
│   ├── TodoTitle.java         ← タイトル（空禁止、100文字以内）
│   └── TodoStatus.java        ← ステータス（OPEN/DONE）
└── exception/
    ├── InvalidTodoTitleException.java
    └── AlreadyCompletedException.java
```

---

## よくあるミス

### 1. `throw` を忘れる

```java
// NG: 例外を作るだけで投げていない
if (id == null) {
    new IllegalArgumentException("IDは必須です");
}

// OK: throw で投げる
if (id == null) {
    throw new IllegalArgumentException("IDは必須です");
}
```

### 2. 文字列を `==` で比較する

```java
// NG: 参照比較になる
if (value == "")

// OK: メソッドを使う
if (value.isEmpty())
```

---

## チェックリスト

- [x] Entity と Value Object の違いを理解した
- [x] TodoStatus を enum で作成した
- [x] TodoTitle にバリデーションを実装した
- [x] TodoId を作成した
- [x] Todo Entity を作成した
- [x] complete() にビジネスルールを実装した
- [x] 例外クラスを作成した

---

## 学んだこと

| 項目 | 内容 |
|------|------|
| Entity vs VO | ID で区別 → Entity、値で区別 → VO |
| バリデーション | VO のコンストラクタで不正値を弾く |
| ビジネスルール | Entity のメソッドに閉じ込める |
| 例外設計 | 専用例外 vs 汎用例外の使い分け |
| 文字列比較 | `==` ではなく `isEmpty()` / `isBlank()` を使う |
| `throw` | 例外は `new` だけでなく `throw` で投げる |

---

## 次のフェーズ

**Phase 5: ユースケース設計（Application 層）** へ進む
