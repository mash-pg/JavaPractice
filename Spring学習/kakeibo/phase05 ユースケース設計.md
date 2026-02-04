# Phase 5: ユースケース設計（Application 層）

> **目的**: 家計簿の処理フローを定義する。Phase 6 で作成した Repository インターフェースを使って UseCase を実装する。

---

## シニア × ジュニアのやり取り記録

### 課題の提示（シニア）

> Repository インターフェース（Phase 6）が完成したので、UseCase を実装する。
> - Step 1: Category の UseCase（3つ）
> - Step 2: Transaction の UseCase（5つ）

---

## Step 1: Category の UseCase

### DeleteCategoryUseCase（2回で合格）

#### 1回目の提出

```java
public class DeleteCategoryUseCase {
    // @Service, @Transactional がない
    public void execute(Long categoryId) {
        categoryRepository.findById(new CategoryId(categoryId))
            .orElseThrow(() -> new CategoryNotFoundException("カテゴリがみつかりません"));

        boolean isUse = transactionRepository.existsByCategoryId(new CategoryId(categoryId));
        if(isUse) {
            throw new CategoryNotFoundException("使用中のカテゴリは削除できません");  // ← 例外が違う
        }
        categoryRepository.deleteById(new CategoryId(categoryId));
    }
}
```

#### シニアの指摘

| # | 問題 | 詳細 |
|---|------|------|
| 1 | 使用中チェックに `CategoryNotFoundException` | 「使用中」は NotFound ではなく InUse。`CategoryInUseException` を作って使う |
| 2 | `@Service` がない | Spring に Bean として認識されない |
| 3 | `@Transactional` がない | UseCase に付けるルール |

### 詰まりポイント: 例外の使い分け（再び）

Phase 4 で学んだはずの例外の使い分けを、UseCase でも間違えた。

| 状況 | 正しい例外 |
|------|-----------|
| カテゴリが**存在しない** | `CategoryNotFoundException` |
| カテゴリが**使用中** | `CategoryInUseException` |
| カテゴリ名が**ルール違反** | `InvalidCategoryException` |

> **教訓**: 例外の名前は「何が起きたか」を正確に表す。NotFound = 見つからない、InUse = 使用中、Invalid = 値が不正。

#### 2回目の提出（合格）

`@Service`, `@Transactional` 追加、`CategoryInUseException` に修正。

---

### CreateCategoryUseCase（3回で合格）

#### 1回目の提出

DeleteCategoryUseCase のロジックをそのままコピペしていた（存在チェック、使用中チェック）。

#### シニアの指摘

| # | 問題 | 詳細 |
|---|------|------|
| 1 | 削除のロジックがコピペされている | 「作成」に必要なチェックは何か考え直す |
| 2 | `TransactionRepository` は不要 | カテゴリ作成に収支リポジトリは使わない |
| 3 | 引数に `CategoryId` がある | 新規作成時、ID は DB が自動採番する |

### 詰まりポイント: UseCase ごとの責務を考えられていない

**何が起きたか：**
DeleteCategoryUseCase をコピペして、ロジックを変えずに使ってしまった。

**原因：**
各 UseCase の「責務（何をするクラスか）」を考えずにコードを書き始めた。

**学び：**
コードを書く前に「この UseCase は何をする？」を言語化する：
- Create → 「重複チェック → 保存」
- List → 「全件取得して返す」
- Delete → 「存在チェック → 使用中チェック → 削除」

#### 2回目の提出

`existsByName` で重複チェックに変更。ただし以下の問題が残った：
- `findById` での存在チェックが残っている（新規作成なのでDBにデータはない）
- 例外が `CategoryInUseException`（「既に存在」は InUse ではなく Invalid）
- 引数に `CategoryId` がまだある

#### 3回目の提出（合格）

```java
@Service
@Transactional
public class CreateCategoryUseCase {
    private final CategoryRepository categoryRepository;

    public CreateCategoryUseCase(CategoryRepository categoryRepository) {
        this.categoryRepository = categoryRepository;
    }

    public void execute(CategoryName name) {
        boolean isUse = categoryRepository.existsByName(name);
        if(isUse) {
            throw new InvalidCategoryException("すでにカテゴリが存在しております");
        }
        categoryRepository.save(new Category(null, name));
    }
}
```

**ポイント:**
- 引数は `CategoryName` だけ（ID は DB が決める）
- `new Category(null, name)` で ID は null → DB の AUTO_INCREMENT に任せる
- 重複チェックには `InvalidCategoryException` を使用

---

### ListCategoriesUseCase（2回で合格）

#### 1回目の提出

こちらも DeleteCategoryUseCase のコピペで、存在チェック・使用中チェックが入っていた。

#### シニアの指摘

一覧取得はシンプルに「全件取得して返す」だけ。不要なチェックと引数を削除。

#### 2回目の提出（合格）

```java
@Service
@Transactional
public class ListCategoriesUseCase {
    private final CategoryRepository categoryRepository;

    public ListCategoriesUseCase(CategoryRepository categoryRepository) {
        this.categoryRepository = categoryRepository;
    }

    public List<Category> execute() {
        return categoryRepository.findAll();
    }
}
```

---

## Step 2: Transaction の UseCase

### CreateTransactionUseCase（3回で合格）

#### 1回目の提出

```java
public void execute(Transaction transaction, CategoryId id) {
    boolean isUse = transactionRepository.existsByCategoryId(id);  // ← 間違ったメソッド
    if(isUse) {
        throw new InvalidCategoryException("すでにカテゴリが存在しております");
    }
    transactionRepository.save(transaction);
}
```

#### シニアの指摘

| # | 問題 | 詳細 |
|---|------|------|
| 1 | `existsByCategoryId` を使っている | これは「カテゴリを使った取引があるか」を調べるメソッド。ここでは「カテゴリ自体が存在するか」を確認したい |
| 2 | `CategoryRepository` がない | カテゴリの存在確認には `CategoryRepository.findById` を使う |
| 3 | 引数に `CategoryId` が別にある | `transaction.getCategoryId()` で取得できる |

### 詰まりポイント: existsByCategoryId の意味の誤解

**何が起きたか：**
`existsByCategoryId` を「カテゴリが存在するか確認する」メソッドだと思っていた。

**実際の意味：**
- `transactionRepository.existsByCategoryId(id)` → 「この CategoryId を持つ **Transaction** が存在するか」
- `categoryRepository.findById(id)` → 「この ID の **Category** が存在するか」

**学び：**
メソッド名をよく読む。`TransactionRepository` のメソッドは Transaction を検索する。Category を探すなら `CategoryRepository` を使う。

#### 2回目の提出

`CategoryRepository.findById` に変更。ただし `CategoryId` が引数に残っていた。

#### 3回目の提出（合格）

```java
public void execute(Transaction transaction) {
    categoryRepository.findById(transaction.getCategoryId())
        .orElseThrow(() -> new CategoryNotFoundException("カテゴリがみつかりません"));
    transactionRepository.save(transaction);
}
```

**学び:** Entity のゲッターを使えば、引数を分ける必要がない。`transaction.getCategoryId()` で十分。

---

### ListTransactionsUseCase（1回で合格）

```java
@Service
@Transactional
public class ListTransactionsUseCase {
    private final TransactionRepository transactionRepository;

    public ListTransactionsUseCase(TransactionRepository transactionRepository) {
        this.transactionRepository = transactionRepository;
    }

    public List<Transaction> execute() {
        return transactionRepository.findAll();
    }
}
```

Category の一覧と同じパターンを正しく適用できた。

---

### GetTransactionUseCase（2回で合格）

#### 1回目の提出

```java
public Optional<Transaction> execute(TransactionId id) {
    return transactionRepository.findById(id);
}
```

#### シニアの指摘

| # | 問題 | 詳細 |
|---|------|------|
| 1 | `Optional` をそのまま返している | UseCase で「見つからないときどうする？」を判断すべき。Controller に判断を委ねない |

#### 2回目の提出（合格）

```java
public Transaction execute(TransactionId id) {
    return transactionRepository.findById(id)
        .orElseThrow(() -> new TransactionNotFoundException("収支がみつかりません"));
}
```

**学び:** UseCase は「段取り係」なので、業務判断（見つからなければエラー）は UseCase で行う。Controller に Optional を返して判断させるのは責務違反。

---

### UpdateTransactionUseCase（2回で合格）

#### 1回目の提出

```java
public void execute(Transaction transaction, TransactionId id, CategoryId categoryId) {
    transactionRepository.findById(id)
        .orElseThrow(() -> new TransactionNotFoundException("収支がみつかりません"));
    categoryRepository.findById(categoryId)
        .orElseThrow(() -> new CategoryNotFoundException("選択したカテゴリが見つかりません"));
    // save が抜けている
}
```

#### シニアの指摘

| # | 問題 | 詳細 |
|---|------|------|
| 1 | `save` がない | チェックだけして保存していない |
| 2 | 引数が3つ | `transaction.getId()` と `transaction.getCategoryId()` で取得できる |

#### 2回目の提出（合格）

```java
public void execute(Transaction transaction) {
    transactionRepository.findById(transaction.getId())
        .orElseThrow(() -> new TransactionNotFoundException("収支がみつかりません"));
    categoryRepository.findById(transaction.getCategoryId())
        .orElseThrow(() -> new CategoryNotFoundException("選択したカテゴリが見つかりません"));
    transactionRepository.save(transaction);
}
```

---

### DeleteTransactionUseCase（1回で合格）

```java
@Service
@Transactional
public class DeleteTransactionUseCase {
    private final TransactionRepository transactionRepository;

    public DeleteTransactionUseCase(TransactionRepository transactionRepository) {
        this.transactionRepository = transactionRepository;
    }

    public void execute(TransactionId id) {
        transactionRepository.deleteById(id);
    }
}
```

---

## 詰まりポイントまとめ

### 1. UseCase ごとの責務を考えずにコピペしてしまう

**発生箇所:** CreateCategoryUseCase, ListCategoriesUseCase

**何が起きたか：**
DeleteCategoryUseCase を最初に書いた後、他の UseCase にそのロジックをコピペした。

**学び：**
コードを書く前に「この UseCase は何をする？」を一言で言語化する。

| UseCase | 一言 | 必要な処理 |
|---------|------|-----------|
| Create | 作る | 重複チェック → 保存 |
| List | 全件見る | 全件取得 |
| Get | 1件見る | 存在チェック → 返す |
| Update | 更新する | 存在チェック → カテゴリチェック → 保存 |
| Delete | 消す | 削除 |

### 2. Entity のゲッターを活用できていない

**発生箇所:** CreateTransactionUseCase, UpdateTransactionUseCase

**何が起きたか：**
`Transaction` の中に `categoryId` が入っているのに、引数で別途 `CategoryId` を渡していた。

**学び：**
Entity にゲッターがあるなら `entity.getXxx()` で取得できる。引数を増やす前に「Entity から取れないか？」を考える。

### 3. Repository メソッドの意味を正確に理解していない

**発生箇所:** CreateTransactionUseCase

**何が起きたか：**
`transactionRepository.existsByCategoryId()` を「カテゴリが存在するか」と誤解した。

**学び：**
- `TransactionRepository` のメソッドは **Transaction** を操作する
- `CategoryRepository` のメソッドは **Category** を操作する
- メソッド名は「何を」「どうやって」検索するかを表す

---

## 質問と学び: existsByCategoryId は TransactionRepository でいいの？

**ジュニアの質問：**
> TransactionRepository で CategoryId の存在チェックをしているけど、TransactionId のチェックの方がよくないですか？

**シニアの回答：**

| メソッド | 目的 | 必要？ |
|---------|------|--------|
| `findById(TransactionId)` | 収支をIDで取得（存在チェック兼用） | 既にある |
| `existsByCategoryId(CategoryId)` | あるカテゴリを使っている収支があるか | カテゴリ削除に必要 |
| `existsByTransactionId(TransactionId)` | 収支の存在チェック | `findById` で代用可能なので不要 |

`existsByCategoryId` は **Transaction テーブルを CategoryId で検索する** 処理なので、TransactionRepository にあるのが正しい。

---

## 完成した UseCase 一覧

```
application/usecase/
├── CreateCategoryUseCase.java     ← 重複チェック → 保存
├── ListCategoriesUseCase.java     ← 全件取得
├── DeleteCategoryUseCase.java     ← 存在チェック → 使用中チェック → 削除
├── CreateTransactionUseCase.java  ← カテゴリ存在チェック → 保存
├── ListTransactionsUseCase.java   ← 全件取得
├── GetTransactionUseCase.java     ← 存在チェック → 返す
├── UpdateTransactionUseCase.java  ← 存在チェック → カテゴリチェック → 保存
└── DeleteTransactionUseCase.java  ← 削除
```

### 全 UseCase 共通パターン

- `@Service` + `@Transactional`
- Repository はコンストラクタインジェクション（`final` フィールド）
- 見つからないときは `orElseThrow` で例外を投げる
- 業務判断は UseCase で行い、Controller に委ねない

---

## 自己チェック

| チェック | 内容 | 結果 |
|---------|------|------|
| [x] | Controller は薄く | UseCase に業務ロジックを集約できた |
| [x] | 業務判断は UseCase に集約 | 存在チェック、重複チェック、使用中チェックを UseCase で実装 |
| [x] | @Transactional は UseCase に付ける | 全 UseCase に付与 |
| [△] | コピペせず責務を考えて書けた | Category で3つ中2つがコピペからスタートだった |
| [x] | Entity のゲッターを活用できた | 指摘後に理解、引数を減らせた |

---

## 次の Phase へ

Phase 5 完了。

次は **Phase 7: DB・JPA 実装（Infrastructure）** へ進む。
- JPA Entity（TransactionJpaEntity, CategoryJpaEntity）の作成
- Repository 実装クラス（TransactionRepositoryImpl, CategoryRepositoryImpl）の作成
- Domain Entity と JPA Entity の変換メソッド

> **Phase 7 のポイント**: Domain Entity と JPA Entity を分離する。infrastructure が domain に依存する（インターフェースの実装）ことを意識する。
