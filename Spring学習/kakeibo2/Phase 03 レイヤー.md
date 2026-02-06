# Phase 03: レイヤ設計

> **目的**: パッケージ構成を作り、各レイヤの役割を理解する

---

## なぜレイヤを分けるか？

### 理由1: 変更の影響を小さくするため

```
例: DB を H2 から MySQL に変える

レイヤ分けてない場合:
→ Controller も UseCase も全部書き直し

レイヤ分けてる場合:
→ Infrastructure 層だけ修正すればOK
```

### 理由2: テストしやすくするため

```
レイヤ分けてない場合:
→ DB がないとテストできない

レイヤ分けてる場合:
→ UseCase だけテストできる（DB はモックで代用）
```

### 理由3: 役割が明確になるため

```
レイヤ分けてない場合:
→ 1つのクラスに1000行...どこに何があるか分からない

レイヤ分けてる場合:
→ 「HTTP の処理は Controller」「業務ロジックは UseCase」と分かる
```

---

## 4つのレイヤと役割

| レイヤ | 役割 | 担当すること |
|--------|------|-------------|
| Presentation | HTTP の入口 | リクエストを受け取り、レスポンス（JSON）を返す |
| Application | 業務処理の流れ | 「何をどの順番でやるか」を決める |
| Domain | ビジネスルール | 「金額は0より大きい」等のルールを守る |
| Infrastructure | 外部との接続 | DB に保存・取得する |

---

## 各レイヤのサブパッケージ

### Presentation層

| パッケージ | 何を置く | 具体例 |
|-----------|---------|--------|
| controller | HTTP リクエストを受ける | TransactionController, CategoryController |
| dto | リクエスト/レスポンスの形 | CreateTransactionRequest, TransactionResponse |
| exception | エラー処理 | GlobalExceptionHandler（全例外をキャッチ） |

**役割:** 外の世界（HTTP）との窓口

---

### Application層

| パッケージ | 何を置く | 具体例 |
|-----------|---------|--------|
| usecase | 業務処理の流れ | CreateTransactionUseCase, DeleteCategoryUseCase |

**役割:** 「何を」「どの順番で」やるか決める

---

### Domain層

| パッケージ | 何を置く | 具体例 |
|-----------|---------|--------|
| entity | ビジネスの主役 | Transaction, Category |
| valueobject | 値のルールを持つ | Amount（0より大きい）, CategoryName（50文字以内） |
| repository | データ操作の **インターフェース** | TransactionRepository, CategoryRepository |
| exception | ビジネスルール違反 | InvalidAmountException, CategoryNotFoundException |

**役割:** ビジネスルールを守る（一番大事！）

---

### Infrastructure層

| パッケージ | 何を置く | 具体例 |
|-----------|---------|--------|
| jpa | DB用のEntity | TransactionJpaEntity, CategoryJpaEntity |
| repository | Repository の **実装** | TransactionRepositoryImpl, CategoryRepositoryImpl |

**役割:** 実際に DB とやり取りする

---

## 依存の方向

```
【変更が多い】              【変更が少ない】
      ↓                          ↓
Presentation → Application → Domain ← Infrastructure
   (HTTP)        (流れ)      (ルール)     (DB)
```

**ポイント:** 外側ほど変わりやすい。内側（Domain）は安定。だから Domain を中心に守る。

---

## DIP（依存性逆転の原則）

### なぜ Domain に Repository インターフェースがある？

```
Domain層:         TransactionRepository（インターフェース）
                          ↑
                          │ 実装する
                          │
Infrastructure層:   TransactionRepositoryImpl（実装クラス）
```

**普通の依存:**
```
Domain → Infrastructure（Domain が DB を知ってる）❌
```

**依存性逆転:**
```
Domain ← Infrastructure（Infrastructure が Domain に合わせる）✅
```

**メリット:**
- Domain は「何が必要か」だけ知ってればいい
- 「どうやって保存するか（MySQL? H2?）」は知らなくていい
- DB を変えても Domain は変わらない

---

## なぜ JPA Entity と Domain Entity を分ける？

| Entity | 目的 | 置き場所 |
|--------|------|---------|
| Domain Entity | ビジネスルールを表現 | domain/entity |
| JPA Entity | DB のテーブル構造を表現 | infrastructure/jpa |

**理由:** DB の都合（カラム名、型）を Domain に持ち込まないため

---

## パッケージ構成（完成形）

```
com.example.kakeibo2
├── presentation
│   ├── controller
│   ├── dto
│   └── exception
├── application
│   └── usecase
├── domain
│   ├── entity
│   ├── valueobject
│   ├── repository
│   └── exception
└── infrastructure
    ├── jpa
    └── repository
```

---

## 学んだこと

- レイヤを分けると変更の影響が小さくなる
- Domain 層が一番大事（ビジネスルールを守る）
- DIP: インターフェースを Domain に置き、実装を Infrastructure に置く
- JPA Entity と Domain Entity を分けて、DB の都合を Domain に持ち込まない
