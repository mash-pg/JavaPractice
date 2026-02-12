# Phase 10: テスト

## 概要

アプリケーションの品質を担保するため、各レイヤーのテストを実装する。

---

## テストの種類

| テストの種類 | 対象 | 特徴 |
|------------|------|------|
| Domain テスト | VO, Entity | 単体テスト。外部依存なし |
| UseCase テスト | UseCase | モックを使って Repository を偽装 |
| Controller テスト | Controller | MockMvc で HTTP リクエストを模擬 |

---

## なぜテストを書くのか？

| 理由 | 説明 |
|------|------|
| **回帰テスト** | コード変更後、既存機能が壊れていないか確認できる |
| **ドキュメント代わり** | テストを見れば「何ができるか」が分かる |
| **リファクタリングの安心感** | テストがあれば安心してコードを改善できる |
| **バグの早期発見** | 手動確認より早く問題を見つけられる |

---

## Step 1: Domain テスト（VO のテスト）

### テスト対象

`Amount` クラス（0以下の値を許可しない）

```java
public class Amount {
    private final int value;

    public Amount(int value) {
        if (value <= 0) {
            throw new InvalidAmountException("...");
        }
        this.value = value;
    }
}
```

### テストすべきケース

| ケース | 入力値 | 期待する結果 |
|--------|--------|-------------|
| 正常系 | 1 | オブジェクト生成成功 |
| 境界値（異常） | 0 | 例外がスローされる |
| 異常系 | -1 | 例外がスローされる |

### テストコード

```java
package com.example.kakeibo2.domain.valueobject;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

import com.example.kakeibo2.domain.exception.InvalidAmountException;

class AmountTest {

    @Test
    void 正常_1以上の値で作成できる() {
        // 実行
        Amount amount = new Amount(1);

        // 検証
        assertEquals(1, amount.getValue());
    }

    @Test
    void 異常_0で作成すると例外() {
        // 実行 & 検証
        assertThrows(InvalidAmountException.class, () -> {
            new Amount(0);
        });
    }

    @Test
    void 異常_負の値で作成すると例外() {
        // 実行 & 検証
        assertThrows(InvalidAmountException.class, () -> {
            new Amount(-1);
        });
    }
}
```

### JUnit アサーション

| メソッド | 用途 |
|---------|------|
| `assertEquals(期待値, 実際の値)` | 値が等しいことを検証 |
| `assertThrows(例外クラス, () -> {...})` | 例外がスローされることを検証 |
| `assertTrue(条件)` | 条件が true であることを検証 |
| `assertFalse(条件)` | 条件が false であることを検証 |
| `assertNotNull(値)` | null でないことを検証 |

---

## Step 2: UseCase テスト（モック使用）

### なぜモックを使うのか？

UseCase は Repository に依存している。本物の Repository を使うと DB が必要になる。

```
UseCase → Repository → DB
```

**モック** を使うと、DB なしで UseCase のロジックだけをテストできる。

```
UseCase → MockRepository（偽物）
```

### Mockito の基本

```java
@ExtendWith(MockitoExtension.class)
class TransactionUseCaseTest {

    @Mock
    private TransactionRepository transactionRepository;  // 偽物

    @Mock
    private CategoryRepository categoryRepository;

    @Mock
    private TransactionFactory transactionFactory;

    @Mock
    private SummaryStrategyProvider summaryStrategyProvider;

    @InjectMocks
    private TransactionUseCase useCase;  // テスト対象

    @Test
    void create_正常に作成できる() {
        // モックの振る舞いを定義
        when(categoryRepository.findById(any()))
            .thenReturn(Optional.of(new Category(...)));
        when(transactionFactory.createNew(...))
            .thenReturn(new Transaction(...));
        when(transactionRepository.save(any()))
            .thenReturn(new Transaction(...));

        // 実行
        Transaction result = useCase.create(...);

        // 検証
        assertNotNull(result);
        verify(transactionRepository).save(any());  // save が呼ばれたか確認
    }
}
```

### Mockito アノテーション

| アノテーション | 役割 |
|---------------|------|
| `@Mock` | 偽物（モック）を作成 |
| `@InjectMocks` | モックを注入してテスト対象を作成 |
| `@ExtendWith(MockitoExtension.class)` | Mockito を有効化 |

### Mockito メソッド

| メソッド | 用途 |
|---------|------|
| `when(...).thenReturn(...)` | メソッドが呼ばれたら○○を返す |
| `when(...).thenThrow(...)` | メソッドが呼ばれたら例外を投げる |
| `verify(mock).method(...)` | メソッドが呼ばれたことを検証 |
| `any()` | 任意の引数にマッチ |

---

## Step 3: Controller テスト（MockMvc 使用）

### MockMvc とは？

実際にサーバーを起動せずに、HTTP リクエストを模擬してテストできる。

### テストコード

```java
@WebMvcTest(TransactionController.class)
class TransactionControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private TransactionUseCase useCase;

    @Test
    void findAll_正常にリストを取得() throws Exception {
        // モックの振る舞いを定義
        when(useCase.findAll()).thenReturn(List.of());

        // 実行 & 検証
        mockMvc.perform(get("/api/transactions"))
            .andExpect(status().isOk())
            .andExpect(content().contentType(MediaType.APPLICATION_JSON));
    }

    @Test
    void create_正常に作成できる() throws Exception {
        // モックの振る舞いを定義
        when(useCase.create(any(), anyInt(), anyLong(), any(), any()))
            .thenReturn(new Transaction(...));

        // 実行 & 検証
        mockMvc.perform(post("/api/transactions")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"type\":\"EXPENSE\",\"amount\":1000,...}"))
            .andExpect(status().isCreated());
    }
}
```

### MockMvc アノテーション

| アノテーション | 役割 |
|---------------|------|
| `@WebMvcTest(Controller.class)` | 指定した Controller だけ起動 |
| `@MockBean` | Spring コンテキストにモックを登録 |

### MockMvc メソッド

| メソッド | 用途 |
|---------|------|
| `perform(get("/path"))` | GET リクエストを実行 |
| `perform(post("/path"))` | POST リクエストを実行 |
| `.andExpect(status().isOk())` | ステータス 200 を期待 |
| `.andExpect(status().isCreated())` | ステータス 201 を期待 |
| `.andExpect(content().json(...))` | JSON レスポンスを検証 |

---

## 境界値テストとは？

バグは「境界」で起きやすい。境界値を重点的にテストする手法。

### 例：Amount（0より大きい値のみ許可）

| 値 | 境界 | テストすべき理由 |
|----|------|-----------------|
| -1 | 境界外 | 明らかに異常 |
| 0 | 境界上 | ここでバグが起きやすい |
| 1 | 境界上 | ギリギリ正常 |
| 100 | 境界内 | 普通の正常値 |

**「0 と 1」を必ずテストする！**

---

## テストの命名規則

テスト名で「何をテストしているか」が分かるようにする。

```java
// 良い例
void 正常_1以上の値で作成できる()
void 異常_0で作成すると例外()
void create_カテゴリが存在しない場合は例外()

// 悪い例
void test1()
void testAmount()
```

---

## テスト実行方法

```powershell
# 全テスト実行
./mvnw test

# 特定のテストクラスを実行
./mvnw test -Dtest=AmountTest

# 特定のテストメソッドを実行
./mvnw test -Dtest=AmountTest#正常_1以上の値で作成できる
```

---

## まとめ

| レイヤー | テスト方法 | 使う技術 |
|---------|-----------|---------|
| Domain | 単体テスト | JUnit のみ |
| UseCase | モックテスト | JUnit + Mockito |
| Controller | HTTP テスト | JUnit + MockMvc |

**テストは「未来の自分への保険」**
