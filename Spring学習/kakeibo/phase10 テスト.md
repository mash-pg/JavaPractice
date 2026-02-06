# Phase 10: テスト

> **目的**: ToDo で学んだテスト技法を家計簿でも実践する。

---

## 実装したテストファイル一覧

### Domain テスト

| ファイル | 場所 | 役割 |
|---------|------|------|
| `AmountTest.java` | domain/valueobject/ | 金額 VO の境界値テスト |
| `CategoryNameTest.java` | domain/valueobject/ | カテゴリ名 VO の境界値テスト |

### UseCase テスト

| ファイル | 場所 | 役割 |
|---------|------|------|
| `DeleteCategoryUseCaseTest.java` | usecase/ | カテゴリ削除のモックテスト |

### Controller テスト

| ファイル | 場所 | 役割 |
|---------|------|------|
| `CategoryControllerTest.java` | presentation/controller/ | HTTP 層のテスト |

---

## 学んだこと

### 1. テストの種類と使い分け

| テスト種類 | 対象 | 使う技術 | 依存 |
|-----------|------|---------|------|
| Domain テスト | VO（Amount, CategoryName） | JUnit 5 のみ | なし |
| UseCase テスト | ビジネスロジック | JUnit 5 + Mockito | Repository をモック |
| Controller テスト | HTTP 層 | @WebMvcTest + MockMvc | UseCase をモック |

### 2. 境界値分析

**Amount の境界値テスト:**

| 入力値 | 期待結果 |
|--------|---------|
| null | `InvalidAmountException` |
| 0 | `InvalidAmountException` |
| 1（境界値） | 正常に生成 |
| -1 | `InvalidAmountException` |
| Integer.MAX_VALUE | 正常に生成 |

**CategoryName の境界値テスト:**

| 入力値 | 期待結果 |
|--------|---------|
| null | `InvalidCategoryException` |
| "" (空文字) | `InvalidCategoryException` |
| "a" (1文字) | 正常に生成 |
| "a" × 50 | 正常に生成 |
| "a" × 51 | `InvalidCategoryException` |

### 3. モックの基本パターン

```java
// 1. モックを作成
CategoryRepository categoryRepository = mock(CategoryRepository.class);

// 2. モックの振る舞いを定義（この呼び出しにはこの値を返せ）
when(categoryRepository.findById(any())).thenReturn(Optional.of(category));

// 3. テスト対象を実行
useCase.execute(1L);

// 4. 検証（メソッドが呼ばれたか確認）
verify(categoryRepository).deleteById(any());
```

### 4. デシジョンテーブル（カテゴリ削除）

| ケース | カテゴリ存在 | 使用中 | 期待結果 |
|--------|-------------|--------|---------|
| 1 | ✗ | - | `CategoryNotFoundException` |
| 2 | ✓ | ✓ | `CategoryInUseException` |
| 3 | ✓ | ✗ | 削除成功 |

**各ケースのモック設定:**

| ケース | findById の戻り値 | existsByCategoryId の戻り値 |
|--------|------------------|---------------------------|
| 1 | `Optional.empty()` | 設定不要 |
| 2 | `Optional.of(category)` | `true` |
| 3 | `Optional.of(category)` | `false` |

### 5. Controller テストのパターン

```java
@WebMvcTest(CategoryController.class)  // Controller だけをテスト
public class CategoryControllerTest {

    @Autowired
    private MockMvc mockMvc;  // HTTP リクエストを模擬

    @MockitoBean  // UseCase をモック化
    private DeleteCategoryUseCase deleteCategoryUseCase;

    @Test
    void カテゴリ削除で204が返る() throws Exception {
        mockMvc.perform(delete("/api/categories/1"))
               .andExpect(status().isNoContent());

        verify(deleteCategoryUseCase).execute(1L);
    }

    @Test
    void カテゴリ作成で201が返る() throws Exception {
        Category category = new Category(new CategoryId(1L), new CategoryName("食費"));
        when(createCategoryUseCase.execute("食費")).thenReturn(category);

        mockMvc.perform(post("/api/categories")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"name\": \"食費\"}"))
               .andExpect(status().isCreated());
    }
}
```

---

## 躓いたポイント

| 問題 | 原因 | 解決策 |
|------|------|--------|
| `useCase` が null | インスタンスを作成していなかった | `@BeforeEach` で `new UseCase(mock, mock)` |
| `CategoryNotFoundException` が発生 | `Optional.empty()` を返していた | 正常系では `Optional.of(category)` を返す |
| `CategoryInUseException` が発生しない | `existsByCategoryId` の設定忘れ | `when(...).thenReturn(true)` を追加 |
| `status()` が undefined | import が足りない | `import static ...MockMvcResultMatchers.status` |
| `verify(クラス名)` でエラー | クラス名ではなくインスタンスを渡す | `verify(deleteCategoryUseCase)` |

---

## UseCase テストと Controller テストの違い

| 項目 | UseCase テスト | Controller テスト |
|------|---------------|------------------|
| アノテーション | なし | `@WebMvcTest` |
| モック | `mock()` または `@Mock` | `@MockitoBean` |
| 実行方法 | `useCase.execute()` を直接呼ぶ | `mockMvc.perform()` で HTTP リクエスト |
| 検証対象 | ビジネスロジックの結果 | HTTP ステータス、レスポンス |

---

## 例外の使い分け（復習）

| 例外 | 意味 | 使う場面 | HTTP ステータス |
|-----|------|---------|---------------|
| `InvalidAmountException` | 金額が不正 | VO で値が不正なとき | 400 |
| `InvalidCategoryException` | カテゴリ名が不正 | VO で値が不正なとき | 400 |
| `TransactionNotFoundException` | 収支が見つからない | UseCase で ID 検索して見つからないとき | 404 |
| `CategoryNotFoundException` | カテゴリが見つからない | UseCase で ID 検索して見つからないとき | 404 |
| `CategoryInUseException` | カテゴリが使用中 | 使用中のカテゴリを削除しようとしたとき | 409 |

---

## 自己チェック

| チェック | 内容 |
|---------|------|
| [x] | 境界値分析でテストケースを設計できた |
| [x] | `assertThrows` で例外テストが書けた |
| [x] | `mock()` でモックを作成できた |
| [x] | `when().thenReturn()` でモックの戻り値を設定できた |
| [x] | `verify()` でメソッド呼び出しを検証できた |
| [x] | `@WebMvcTest` で Controller テストが書けた |
| [x] | `mockMvc.perform()` で HTTP リクエストを送れた |

---

## テスト実行コマンド

```powershell
# 全テスト実行
.\mvnw.cmd test

# 特定のテストクラスのみ実行
.\mvnw.cmd test -Dtest=AmountTest
.\mvnw.cmd test -Dtest=CategoryNameTest
.\mvnw.cmd test -Dtest=DeleteCategoryUseCaseTest
.\mvnw.cmd test -Dtest=CategoryControllerTest
```

---

*Phase 10 完了。次は Phase 11: 仕上げ・振り返り へ進む。*
