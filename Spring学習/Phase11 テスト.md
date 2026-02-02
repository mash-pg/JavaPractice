# Phase 11: テスト

> **目的**: テスターとしての考え方と、開発者としてのテストコード実装を学ぶ。

---

## 概要

| パート | 内容 | 対象 |
|--------|------|------|
| **Part A** | テストの考え方・設計技法 | テスター向け |
| **Part B** | JUnit でテストコード実装 | 開発者向け |

---

# Part A: テストの考え方（テスター向け）

> **目的**: 「何をテストするか」「どうテストするか」を設計できるようになる。

---

## 11.1 テストの種類

### 質問形式で学ぶ

**ジュニア**: テストって色々種類があるって聞いたんですけど、何が違うんですか？

**シニア**: 大きく分けて4種類ある。テストする「範囲」が違うんだ。

| 種類 | 範囲 | 例 | 誰がやる？ |
|------|------|-----|-----------|
| **単体テスト** | クラス・メソッド単位 | `TodoTitle` のバリデーション | 開発者 |
| **結合テスト** | 複数コンポーネント | UseCase + Repository | 開発者 |
| **E2E テスト** | システム全体 | ブラウザから API を叩く | テスター / QA |
| **回帰テスト** | 変更後の既存機能 | 修正後に他が壊れてないか | テスター / 自動化 |

**ジュニア**: 全部やらないといけないんですか？

**シニア**: 理想はそうだけど、**テストピラミッド**という考え方がある。

```
        /\
       /  \      E2E テスト（少ない）
      /────\
     /      \    結合テスト（中くらい）
    /────────\
   /          \  単体テスト（たくさん）
  /────────────\
```

**ポイント**:
- 単体テストは高速・安定・大量に書く
- E2E テストは遅い・不安定・重要なシナリオだけ

---

### テストの種類：詳細

#### 単体テスト（Unit Test）

```
目的: 1つのクラス・メソッドが正しく動くか確認
範囲: 他のクラスには依存しない（モックを使う）
速度: 超高速（ミリ秒）
```

**例**: `TodoTitle` に空文字を渡したら例外が出るか

```java
@Test
void 空文字は例外() {
    assertThrows(InvalidTodoTitleException.class,
        () -> new TodoTitle(""));
}
```

#### 結合テスト（Integration Test）

```
目的: 複数のコンポーネントが連携して動くか確認
範囲: UseCase + Repository + DB など
速度: 中速（秒単位）
```

**例**: ToDo を作成して DB に保存されるか

#### E2E テスト（End-to-End Test）

```
目的: ユーザー視点でシステム全体が動くか確認
範囲: ブラウザ → API → DB → レスポンス
速度: 遅い（数秒〜数分）
```

**例**: ブラウザで ToDo を作成 → 一覧に表示される

#### 回帰テスト（Regression Test）

```
目的: 変更によって既存機能が壊れていないか確認
タイミング: コード修正後、リリース前
```

**例**: 「完了機能を修正したら、作成機能が動かなくなった」を防ぐ

---

## 11.2 テスト設計技法

### 質問形式で学ぶ

**ジュニア**: テストケースってどうやって考えればいいですか？全パターンをテストするのは無理ですよね。

**シニア**: そう、全パターンは無理。だから**効率よくバグを見つける技法**がある。代表的なのは3つ。

| 技法 | 考え方 |
|------|--------|
| **同値分割法** | 同じ結果になるグループから1つ選ぶ |
| **境界値分析** | 境界の値を重点的にテスト |
| **デシジョンテーブル** | 条件の組み合わせを表にする |

---

### 同値分割法（Equivalence Partitioning）

**シニア**: 例えば、`TodoTitle` は「1文字以上100文字以下」というルールがある。全部の文字数をテストする必要はある？

**ジュニア**: 1文字、2文字、3文字... 100通りテストするのは大変ですね。

**シニア**: そう。だから「同じ結果になるグループ」に分ける。

```
【TodoTitle のルール】
- 1〜100文字: 有効
- 0文字（空）: 無効
- 101文字以上: 無効

【同値分割】
┌─────────┬─────────────────┬───────────┐
│  無効   │      有効       │   無効    │
│ 0文字   │   1〜100文字    │ 101文字〜 │
└─────────┴─────────────────┴───────────┘

【テストケース】
- 0文字 → 1つ代表を選ぶ（空文字）
- 1〜100文字 → 1つ代表を選ぶ（例: 50文字）
- 101文字以上 → 1つ代表を選ぶ（例: 101文字）
```

**ポイント**: 各グループから1つ選べば、そのグループ全体をカバーしたとみなせる。

---

### 境界値分析（Boundary Value Analysis）

**シニア**: でも、バグが起きやすいのはどこだと思う？

**ジュニア**: うーん... 真ん中あたり？

**シニア**: 実は**境界**なんだ。`<=` と `<` を間違えたり、`100` と `101` の判定を間違えたり。

```
【境界値】
      0    1                   100   101
      │    │                    │     │
──────┴────┴────────────────────┴─────┴──────
   無効  境界        有効      境界  無効

【テストすべき値】
- 0文字（境界の外）
- 1文字（境界の内側）← 最小有効値
- 100文字（境界の内側）← 最大有効値
- 101文字（境界の外）
```

**ポイント**: 境界とその前後をテストする。

---

### 境界値分析の実践

**ジュニア**: ToDo アプリで境界値分析をやるとどうなりますか？

**シニア**: `TodoTitle` で考えてみよう。

| テストケース | 入力 | 期待結果 |
|-------------|------|---------|
| 空文字 | `""` | 例外 `InvalidTodoTitleException` |
| 1文字（最小） | `"a"` | 正常に作成 |
| 100文字（最大） | `"a" × 100` | 正常に作成 |
| 101文字（超過） | `"a" × 101` | 例外 `InvalidTodoTitleException` |

**ジュニア**: 50文字とかはテストしなくていいんですか？

**シニア**: 同値分割の考え方では、1〜100文字は同じグループ。1文字と100文字をテストすれば、50文字も同じ結果になるはず。バグが起きやすいのは境界だから、境界を重点的にテストする。

---

### デシジョンテーブル（Decision Table）

**シニア**: 条件が複数あるときは、**デシジョンテーブル**を使う。

**ジュニア**: どういうときに使いますか？

**シニア**: 例えば「ToDo を完了できる条件」を考えてみよう。

```
【条件】
- ステータスが OPEN である
- ログインしている（将来の機能として）

【デシジョンテーブル】
| # | OPEN? | ログイン? | 結果 |
|---|-------|----------|------|
| 1 | Yes   | Yes      | 完了できる |
| 2 | Yes   | No       | エラー（未認証） |
| 3 | No    | Yes      | エラー（既に完了） |
| 4 | No    | No       | エラー（未認証） |
```

**ポイント**: 条件の組み合わせを漏れなく洗い出せる。

---

## 11.3 テストケース作成

### テストケースの書き方

**ジュニア**: テストケースってどう書けばいいですか？

**シニア**: 最低限、以下の3つを書く。

| 項目 | 内容 |
|------|------|
| **前提条件** | テスト前の状態 |
| **操作** | 何をするか |
| **期待結果** | どうなるべきか |

---

### 例: ToDo 完了機能のテストケース

```
【テストケース ID】TC-COMPLETE-001
【テスト項目】OPEN の ToDo を完了にする（正常系）
【前提条件】
- ステータスが OPEN の ToDo が存在する（ID: 1）
【操作】
- POST /api/todos/1/complete を実行
【期待結果】
- ステータスコード: 200
- レスポンスの status が "DONE" になっている
```

```
【テストケース ID】TC-COMPLETE-002
【テスト項目】既に完了した ToDo を再度完了にする（異常系）
【前提条件】
- ステータスが DONE の ToDo が存在する（ID: 2）
【操作】
- POST /api/todos/2/complete を実行
【期待結果】
- ステータスコード: 409 Conflict
- エラーメッセージ: "すでに完了しています"
```

```
【テストケース ID】TC-COMPLETE-003
【テスト項目】存在しない ToDo を完了にする（異常系）
【前提条件】
- ID: 999 の ToDo は存在しない
【操作】
- POST /api/todos/999/complete を実行
【期待結果】
- ステータスコード: 404 Not Found
- エラーメッセージ: "Todo not found: 999"
```

---

### 正常系と異常系

**ジュニア**: 正常系と異常系って何ですか？

**シニア**:

| 種類 | 説明 | 例 |
|------|------|-----|
| **正常系** | 期待通りの入力で期待通りの結果 | 有効なタイトルで ToDo 作成 |
| **異常系** | エラーになるべき入力でエラーになる | 空タイトルで例外発生 |

**ポイント**: 異常系のテストが重要。正常系だけテストしても、エラー処理のバグは見つからない。

---

## 11.4 バグ報告・調査

### バグ報告の書き方

**ジュニア**: バグを見つけたらどう報告すればいいですか？

**シニア**: **再現できる情報**を書く。これが不十分だと開発者が困る。

```
【悪い例】
「完了ボタンを押したらエラーになりました」

→ 問題点:
- いつ？どの ToDo で？
- 何のエラー？
- 再現手順がない
```

```
【良い例】
【タイトル】既に完了した ToDo を再度完了すると 500 エラーになる

【環境】
- OS: Windows 11
- ブラウザ: Chrome 120
- API バージョン: 1.0.0

【再現手順】
1. POST /api/todos で ToDo を作成（ID: 1）
2. POST /api/todos/1/complete で完了にする → 成功
3. 再度 POST /api/todos/1/complete を実行

【期待結果】
- 409 Conflict が返る
- メッセージ: "すでに完了しています"

【実際の結果】
- 500 Internal Server Error が返る

【発生日時】2026-02-02 11:00

【ログ】
ERROR GlobalExceptionHandler : NullPointerException at ...
```

---

### 調査の進め方

**ジュニア**: 「このエラー調査して」って言われたらどうすればいいですか？

**シニア**: 以下の手順で進める。

```
【調査の流れ】

1. 再現確認
   └→ 自分の環境で再現できるか試す

2. ログ確認
   └→ ERROR ログを探す
   └→ 「いつ」「どのリクエストで」「何が起きたか」を特定

3. 原因の切り分け
   └→ どのレイヤで問題が起きているか
      ├── Controller? → リクエストの受け取りミス
      ├── UseCase? → ビジネスロジックのバグ
      ├── Repository? → DB 操作のミス
      └── Domain? → バリデーションの漏れ

4. 報告
   └→ 原因と影響範囲をまとめる
```

---

### ログの読み方

**ジュニア**: ログってどう読めばいいですか？

**シニア**: まず ERROR を探す。次に、その前後のログを読む。

```
【ログの例】

2026-02-02 11:00:00 INFO  CreateTodoUseCase : Todo created: 1, title: 買い物
2026-02-02 11:00:05 INFO  CompleteTodoUseCase : Todo completed: 1
2026-02-02 11:00:10 ERROR GlobalExceptionHandler : Exception occurred: すでに完了しています
                    ↑ ここが問題

【読み方】
1. 11:00:00 - ToDo ID:1 が作成された
2. 11:00:05 - ToDo ID:1 が完了した
3. 11:00:10 - 再度完了しようとしてエラー

→ 原因: 既に完了した ToDo を再度完了しようとした
→ これは正常な動作（AlreadyCompletedException）
```

**ポイント**: ログは時系列で読む。何が起きて、何が原因でエラーになったかを追う。

---

# Part B: テストコード実装（開発者向け）

> **目的**: JUnit でテストコードを書けるようになる。

---

## 11.5 Domain Unit Test

### 課題1: TodoTitle のテスト

**シニア**: まず、Part A で学んだ境界値分析を使って、`TodoTitle` のテストを書いてみよう。

**要件（復習）:**
	- 空文字は不可 → `InvalidTodoTitleException`
	- 100文字を超えると不可 → `InvalidTodoTitleException`
	- 1〜100文字は有効

**テストクラスを作成:**

```
src/test/java/com/example/todo/domain/valueobject/TodoTitleTest.java
```

---

### ヒント: JUnit 5 の基本

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import static org.junit.jupiter.api.Assertions.*;

class TodoTitleTest {

    @Test
    @DisplayName("1文字のタイトルは正常に作成できる")
    void 最小文字数で作成できる() {
        // Arrange（準備）
        String input = "a";

        // Act（実行）
        TodoTitle title = new TodoTitle(input);

        // Assert（検証）
        assertEquals("a", title.getValue());
    }

    @Test
    @DisplayName("空文字は例外が発生する")
    void 空文字は例外() {
        assertThrows(InvalidTodoTitleException.class,
            () -> new TodoTitle(""));
    }
}
```

---

### 課題1 のテストケース

境界値分析に基づいて、以下のテストを書いてください：

| テストメソッド名      | 入力          | 期待結果                        |
| ------------- | ----------- | --------------------------- |
| `空文字は例外`      | `""`        | `InvalidTodoTitleException` |
| `1文字で作成できる`   | `"a"`       | 正常に作成                       |
| `100文字で作成できる` | `"a" × 100` | 正常に作成                       |
| `101文字は例外`    | `"a" × 101` | `InvalidTodoTitleException` |

---

### 詰まったところ（課題1）

#### Q: `The import org.junit cannot be resolved` が出る

**ジュニア**: テストファイルを作ったら `The import org.junit cannot be resolved` というエラーが出ました。

**シニア**: いくつか原因が考えられる。順番に確認しよう。

---

##### 原因1: JAVA_HOME のバージョンが違う

**シニア**: まず Java のバージョンを確認。

```powershell
# 確認コマンド
echo $env:JAVA_HOME
```

**ジュニア**: `C:\Program Files\Eclipse Adoptium\jdk-11.0.24.8-hotspot\` でした。

**シニア**: pom.xml は Java 17 を要求しているのに、JAVA_HOME が Java 11 を指している。これが原因。

**解決方法:**

```powershell
# 一時的に JAVA_HOME を変更
$env:JAVA_HOME = "C:\Program Files\Eclipse Adoptium\jdk-17.0.17.10-hotspot"

# ビルド実行
.\mvnw.cmd clean compile test-compile
```

> **ポイント**: システム環境変数で恒久的に変更するのがベスト。

---

##### 原因2: テストファイルの配置場所が間違っている

**ジュニア**: ビルドしたら別のエラーが出ました。

```
[ERROR] /C:/dev/todo/src/main/java/.../TodoTitleTest.java
```

**シニア**: `src/main/java` に置いてしまっている。テストファイルは `src/test/java` に置く必要がある。

```
❌ 間違い: src/main/java/com/example/todo/domain/valueobject/TodoTitleTest.java
✅ 正しい: src/test/java/com/example/todo/domain/valueobject/TodoTitleTest.java
```

**理由:**
- `src/main/java` → 本番コード
- `src/test/java` → テストコード（`spring-boot-starter-test` の依存が有効になる）

---

##### 原因3: @DisplayName の構文エラー

**ジュニア**: こんなエラーも出ました。「型の開始が不正です」

**シニア**: コードを見せて。

```java
// 間違い
@DisplayName();  // ← セミコロン不要、引数が必要

// 正しい
@DisplayName("1文字のタイトルは正常に作成できる")
```

**ポイント:**
- `@DisplayName` は引数に文字列を渡す
- アノテーションの後にセミコロンは不要

---

##### 原因4: Eclipse でインポートエラーが消えない

**ジュニア**: コマンドラインではビルドできたのに、Eclipse ではまだエラーが出ています。

**シニア**: Eclipse のキャッシュが古い状態のままになっている。以下を試して。

**解決方法:**

1. プロジェクトを右クリック → **Maven** → **Update Project...**
2. **Force Update of Snapshots/Releases** にチェック
3. **OK** をクリック

または

1. メニューの **Project** → **Clean...**
2. todo プロジェクトを選択
3. **Clean** をクリック

> **ポイント**: Maven の依存関係を変更したら、Eclipse に反映するためにこの操作が必要になることがある。

---

### 学んだこと（課題1）

| 項目 | 内容 |
|------|------|
| テストファイルの場所 | `src/test/java` に配置する |
| JAVA_HOME | pom.xml の Java バージョンと一致させる |
| @DisplayName | 引数に文字列を渡す、セミコロン不要 |
| Eclipse の更新 | Maven → Update Project でリフレッシュ |

---

## 11.6 Application Unit Test

### 課題2: CreateTodoUseCase のテスト

**シニア**: 次は UseCase のテスト。UseCase は Repository に依存しているから、**モック**を使う。

**ジュニア**: モックって何ですか？

**シニア**: 「偽物のオブジェクト」のこと。本物の Repository を使うと DB が必要になる。モックを使えば DB なしでテストできる。

```java
// 本物の Repository
// → DB に接続する
// → テストが遅い、環境依存

// モックの Repository
// → 「save() が呼ばれたら、こう返す」を定義
// → DB 不要、高速
```

---

### ヒント: Mockito の基本

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

@ExtendWith(MockitoExtension.class)
class CreateTodoUseCaseTest {

    @Mock
    private TodoRepository todoRepository;

    @Mock
    private TodoFactory todoFactory;

    @InjectMocks
    private CreateTodoUseCase useCase;

    @Test
    @DisplayName("ToDo を作成できる")
    void 正常に作成できる() {
        // Arrange（準備）
        String title = "テスト";
        Todo todo = new Todo(new TodoTitle(title));
        Todo savedTodo = new Todo(new TodoTitle(title));

        // モックの振る舞いを定義
        when(todoFactory.create(title)).thenReturn(todo);
        when(todoRepository.save(todo)).thenReturn(savedTodo);

        // Act（実行）
        Todo result = useCase.execute(title);

        // Assert（検証）
        assertNotNull(result);
        verify(todoRepository, times(1)).save(todo);  // save が1回呼ばれたか確認
    }
}
```

---

### Mockito のアノテーション

| アノテーション | 意味 |
|---------------|------|
| `@Mock` | モックオブジェクトを作成 |
| `@InjectMocks` | モックを注入したテスト対象を作成 |
| `@ExtendWith(MockitoExtension.class)` | Mockito を有効化 |

### Mockito のメソッド

| メソッド | 意味 |
|---------|------|
| `when(...).thenReturn(...)` | 「〇〇が呼ばれたら△△を返す」 |
| `verify(..., times(n))` | 「〇〇がn回呼ばれたか確認」 |

---

### 詰まったところ（課題2）

#### Q: NullPointerException が発生する

**ジュニア**: テストを実行したら `NullPointerException: Cannot invoke "TodoId.getValue()" because the return value of "Todo.getId()" is null` というエラーが出ました。

**シニア**: エラーメッセージを読んでみよう。`Todo.getId()` が null を返していると言っている。

**ジュニア**: でも Todo は作成しましたよ？

**シニア**: どうやって作った？

```java
// テストコード
Todo todo = new Todo(new TodoTitle(title));
Todo savedTodo = new Todo(new TodoTitle(title));
```

**シニア**: これが問題。`Todo` のコンストラクタを見てみよう。

```java
// Todo.java
public Todo(TodoTitle title) {
    this.id = null;  // ← ID は null
    ...
}
```

**ジュニア**: あ、ID が null のままですね。

**シニア**: そう。`CreateTodoUseCase` では保存後にログを出力している：

```java
log.info("Todo created : {}", savedTodo.getId().getValue());
//                            ↑ getId() が null なので NullPointerException
```

**解決方法**: 保存後の Todo には ID を設定する。

```java
// Factory が返す Todo（保存前 = ID なし）
Todo todo = new Todo(new TodoTitle(title));

// Repository が返す Todo（保存後 = ID あり）
Todo savedTodo = new Todo(
    new TodoId(1L),           // ← ID を設定！
    new TodoTitle(title),
    TodoStatus.OPEN,
    TodoPriority.LOW,
    null,                      // description
    null,                      // dueDate
    LocalDateTime.now(),       // createdAt
    null                       // updatedAt
);
```

---

#### 学んだこと（課題2）

| 項目 | 内容 |
|------|------|
| 保存前 vs 保存後 | 保存前の Todo は ID なし、保存後は ID あり |
| モックの戻り値 | 本番と同じ状態のオブジェクトを返す必要がある |
| エラーメッセージ | `null` が出たら、どこで null になっているか追跡する |

---

## 11.7 Controller Test

### 課題3: TodoController のテスト

**シニア**: 最後は Controller のテスト。`@WebMvcTest` を使うと、HTTP リクエスト/レスポンスをテストできる。

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.mockito.Mockito.*;

@WebMvcTest(TodoController.class)
class TodoControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private CreateTodoUseCase createTodoUseCase;

    // 他の UseCase も @MockBean で定義

    @Test
    @DisplayName("POST /api/todos - 正常に作成できる")
    void 作成成功() throws Exception {
        // Arrange
        Todo todo = new Todo(new TodoTitle("テスト"));
        when(createTodoUseCase.execute("テスト")).thenReturn(todo);

        // Act & Assert
        mockMvc.perform(post("/api/todos")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"title\": \"テスト\"}"))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.title").value("テスト"));
    }

    @Test
    @DisplayName("POST /api/todos - 空タイトルは 400 エラー")
    void 空タイトルは400() throws Exception {
        mockMvc.perform(post("/api/todos")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"title\": \"\"}"))
            .andExpect(status().isBadRequest());
    }
}
```

---

### @WebMvcTest のポイント

| 項目 | 説明 |
|------|------|
| `@WebMvcTest(TodoController.class)` | Controller だけをテスト対象にする |
| `@MockBean` | UseCase をモックに置き換える |
| `MockMvc` | HTTP リクエストをシミュレート |

### MockMvc のメソッド

| メソッド | 意味 |
|---------|------|
| `perform(post(...))` | POST リクエストを実行 |
| `andExpect(status().isCreated())` | ステータスコードを検証 |
| `andExpect(jsonPath("$.title").value(...))` | JSON のフィールドを検証 |

---

### 詰まったところ（課題3）

#### Q: Failed to load ApplicationContext というエラーが出る

**ジュニア**: テストを実行したら `Failed to load ApplicationContext` という長いエラーが出ました。どこを見ればいいですか？

**シニア**: 長いエラーログの読み方を教えよう。**一番下の `Caused by` を探す**のがコツ。

```
java.lang.IllegalStateException: Failed to load ApplicationContext
    at ...（長いスタックトレース）
    at ...

Caused by: UnsatisfiedDependencyException: Error creating bean 'todoController'
    at ...

Caused by: NoSuchBeanDefinitionException: ListTodosUseCase ← ★ここが本当の原因
    at ...
```

**ジュニア**: `NoSuchBeanDefinitionException: ListTodosUseCase` ってどういう意味ですか？

**シニア**: キーワードを分解してみよう。

| キーワード | 意味 |
|-----------|------|
| `NoSuchBeanDefinitionException` | Bean（部品）が見つからない |
| `No qualifying bean of type` | この型の Bean がない |
| `ListTodosUseCase` | 見つからないクラス名 |

**ジュニア**: つまり `ListTodosUseCase` がないってことですか？

**シニア**: そう。`TodoController` は6つの UseCase に依存しているけど、テストでは `CreateTodoUseCase` しかモック化していなかった。

```java
// 足りなかった
@MockitoBean
private CreateTodoUseCase createTodoUseCase;

// 必要だったもの
@MockitoBean
private ListTodosUseCase listTodosUseCase;

@MockitoBean
private GetTodoUseCase getTodoUseCase;

// ... 他の UseCase も全部必要
```

---

#### エラーログの読み方（重要）

**ジュニア**: エラーログって長くて読む気がしないんですけど...

**シニア**: 全部読む必要はない。**下から読む**のがコツ。

```
【エラーログの構造】

java.lang.IllegalStateException: Failed to load ApplicationContext
    at ...（ここは読まなくていい）
    at ...

Caused by: UnsatisfiedDependencyException: ...
    at ...

Caused by: NoSuchBeanDefinitionException: ListTodosUseCase
           ↑ ★一番下の Caused by が本当の原因
```

**読み方のステップ:**

1. **一番下の `Caused by` を探す**
2. **例外クラス名を読む**（`NoSuchBeanDefinitionException` など）
3. **メッセージを読む**（`ListTodosUseCase` がないなど）
4. **原因を推測して修正**

---

#### よくあるエラーパターン

| エラー | 意味 | 解決策 |
|--------|------|--------|
| `NoSuchBeanDefinitionException` | Bean が見つからない | `@MockitoBean` を追加 |
| `NullPointerException` | null を参照した | 値が設定されているか確認 |
| `UnsatisfiedDependencyException` | 依存関係が解決できない | 必要な Bean を追加 |
| `MethodArgumentNotValidException` | バリデーションエラー | 入力値を確認 |

---

#### Q: @MockBean が deprecated と警告される

**ジュニア**: `@MockBean` を使ったら「deprecated」って警告が出ました。

**シニア**: Spring Boot 3.4.0 から `@MockBean` は非推奨になった。代わりに `@MockitoBean` を使う。

```java
// 古い書き方（非推奨）
import org.springframework.boot.test.mock.mockito.MockBean;
@MockBean
private CreateTodoUseCase createTodoUseCase;

// 新しい書き方（推奨）
import org.springframework.test.context.bean.override.mockito.MockitoBean;
@MockitoBean
private CreateTodoUseCase createTodoUseCase;
```

---

#### 学んだこと（課題3）

| 項目 | 内容 |
|------|------|
| エラーログの読み方 | 一番下の `Caused by` を探す |
| 依存関係 | Controller が依存する全ての UseCase をモック化 |
| @MockitoBean | @MockBean の代わりに使う（Spring Boot 3.4+） |

---

## まとめ

### Part A: テストの考え方

| トピック | 学んだこと |
|---------|-----------|
| テストの種類 | 単体 < 結合 < E2E（ピラミッド） |
| 同値分割法 | 同じ結果のグループから1つ選ぶ |
| 境界値分析 | 境界の前後を重点的にテスト |
| テストケース | 前提条件・操作・期待結果を書く |
| バグ報告 | 再現手順を明確に |
| 調査 | ログを時系列で追う |

### Part B: テストコード実装

| 対象 | 使うもの |
|------|---------|
| Domain（VO, Entity） | JUnit 5 |
| UseCase | JUnit 5 + Mockito |
| Controller | @WebMvcTest + MockMvc |

---

## 課題一覧

| 課題 | 内容 | 状態 |
|------|------|------|
| 課題1 | TodoTitle のテスト（境界値分析） | ✅ 完了 |
| 課題2 | CreateTodoUseCase のテスト（モック） | ✅ 完了 |
| 課題3 | TodoController のテスト（API） | ✅ 完了 |

---

## 作成したテストファイル

```
src/test/java/com/example/todo/
├── domain/
│   └── valueobject/
│       ├── TodoTitleTest.java          ← 課題1
│       └── CreateTodoUseCaseTest.java  ← 課題2
└── presentation/
    └── controller/
        └── TodoControllerTest.java     ← 課題3
```

---

## Phase 11 で学んだこと（総まとめ）

### Part A: テストの考え方

| トピック | 内容 |
|---------|------|
| テストの種類 | 単体・結合・E2E・回帰テスト |
| テストピラミッド | 単体テストを多く、E2E は少なく |
| 同値分割法 | 同じ結果のグループから代表を選ぶ |
| 境界値分析 | 境界の前後を重点的にテスト |
| テストケース作成 | 前提条件・操作・期待結果を書く |
| バグ報告 | 再現手順を明確に書く |
| エラーログの読み方 | 一番下の `Caused by` を探す |

### Part B: テストコード実装

| 対象 | 使うもの |
|------|---------|
| Domain（VO, Entity） | JUnit 5 + `assertEquals`, `assertThrows` |
| UseCase | JUnit 5 + Mockito（`@Mock`, `@InjectMocks`） |
| Controller | `@WebMvcTest` + `MockMvc` + `@MockitoBean` |

### 詰まりやすいポイント

| 問題 | 原因 | 解決策 |
|------|------|--------|
| `import org.junit cannot be resolved` | JAVA_HOME が違う / テストが src/main にある | JAVA_HOME を確認 / src/test に移動 |
| `NullPointerException: getId() is null` | 保存後の Todo に ID がない | ID 付きの Todo を作成 |
| `NoSuchBeanDefinitionException` | @MockitoBean が足りない | 全ての依存を定義 |
| `@MockBean` is deprecated | Spring Boot 3.4+ で非推奨 | `@MockitoBean` を使う |

---

## 次のステップ

- [ ] Phase 12: 仕上げ へ進む

---

*このドキュメントは学習中に詰まった点と解決方法を記録したものです。*
