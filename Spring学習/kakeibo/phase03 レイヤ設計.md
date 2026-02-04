# Phase 3: レイヤ設計（DDD 軽量版）

> **目的**: ToDo で学んだレイヤ構成を「自力で」再現する。

---

## シニア × ジュニアのやり取り記録

### 課題の提示（シニア）

> 家計簿アプリの DDD 軽量版パッケージ構成を自力で作成してください。
> 以下の3つに答えてください：
> 1. パッケージ構成をツリー形式で書き出す
> 2. なぜ domain が他のレイヤに依存しないのか説明する
> 3. 実際にパッケージを作成する

---

### 1回目の回答（ジュニア）

```
src/main/java/com/example/kakeibo/
    - presentation
    - application
            - exception
    - dto
    - service
    - domain
            - exception
            - valueobject
    - service
    - infrastructure
```

#### シニアのレビュー

**良い点：**
- 4つのレイヤ（presentation, application, domain, infrastructure）を覚えていた
- domain の中に `exception` と `valueobject` を入れる判断は正しい

**指摘事項：**

| # | 問題点 | 指摘内容 |
|---|--------|----------|
| 1 | `dto` と `service` が kakeibo 直下にある | どのレイヤに属するか考えるべき |
| 2 | `service` という名前 | 家計簿では `usecase` を使う（ToDo と同じ） |
| 3 | `exception` が application にある | application 側に exception は不要。例外は domain と presentation に置く |
| 4 | domain に `entity` がない | Entity を格納するパッケージが必要 |
| 5 | domain に `repository` がない | インターフェースを置く場所が必要 |
| 6 | presentation のサブパッケージが空 | controller, dto, exception が入る |
| 7 | infrastructure のサブパッケージが空 | jpa, repository が入る |

---

### 2回目の回答（ジュニア）

```
src/main/java/com/example/kakeibo/
    - presentation/
            - controller/
            - dto/
            - exception/
    - application/
            - service/         ← まだ service になっている
    - domain/
            - exception/
            - valueobject/
            - entity/          ← 追加できた
            - repository/      ← 追加できた
    - infrastructure/          ← 中身がまだ空
```

#### シニアのレビュー

**良くなった点：**
- presentation の中に controller, dto, exception を正しく配置
- domain の中に entity, repository を追加

**残りの指摘：**

| # | 問題点 | 指摘内容 |
|---|--------|----------|
| 1 | application の中が `service` | `usecase` が正しい名前 |
| 2 | infrastructure の中身が空 | `jpa` と `repository` を入れる |

---

### 3回目の回答（ジュニア）※ ToDo を参照

```
src/main/java/com/example/kakeibo/
    - presentation/
            - controller/
            - dto/
            - exception/
    - application/
            - usecase/         ← ToDo を見て修正
    - domain/
            - exception/
            - valueobject/
            - entity/
            - repository/
    - infrastructure/
            - jpa/             ← ToDo を見て追加
            - repository/      ← ToDo を見て追加
```

#### シニアのレビュー：合格！

全レイヤ・全サブパッケージが正しい位置に配置された。

---

## 詰まったポイント

### 詰まり1: サブパッケージがレイヤの外に出てしまう

**何が起きたか：**
初回の回答で `dto`、`service` が `kakeibo/` 直下に出てしまった。

**原因：**
各サブパッケージが「どのレイヤに属するか」を意識できていなかった。

**学び：**
サブパッケージは必ずレイヤの中にネストする。まず「このクラスはどのレイヤの責務か？」を考えてから配置する。

### 詰まり2: application の中身を `service` と書いてしまう

**何が起きたか：**
2回目まで `usecase` ではなく `service` と書いていた。

**原因：**
一般的な Spring の慣習（Service クラス）と、DDD 軽量版の命名（UseCase）が混ざった。

**学び：**
この家計簿アプリ（DDD 軽量版）では `usecase` を使う。「処理の流れを組み立てるクラス」= UseCase と覚える。

**ToDo を見た：** はい（usecase という名前を確認した）

### 詰まり3: infrastructure の中身が思い出せない

**何が起きたか：**
infrastructure に何を入れるか分からなかった。

**原因：**
「外部技術との接続」が具体的に何を指すかイメージできていなかった。

**学び：**
infrastructure には「技術的な実装の詳細」を置く：
- `jpa/` → JPA Entity（DB テーブルに対応するクラス）
- `repository/` → domain の Repository インターフェースの実装クラス

**ToDo を見た：** はい（jpa と repository の配置を確認した）

---

## domain が他のレイヤに依存しない理由

### ジュニアの回答の変遷

**1回目（リッチドメインの話）：**
> リッチドメインにも権限を持たせて最終決定者としての役割を持たせます。
> 他のレイヤーによって値やインスタンスが変わると結果がおかしくなる。

→ 方向は合っているが「依存の方向」の観点が抜けている。

**2回目（DI の話）：**
> DI でコンテナにインスタンスを入れて取得する。コンストラクタインジェクションで final を使えば immutable になる。

→ DI は「手段」の話。質問は「なぜ」という「理由」の話。

**3回目（ギブアップ → シニアが解説）**

### シニアの解説

**「金額は0より大きい」** ← これはビジネスルール。

このルールは：
- DB が H2 でも MySQL でも **変わらない**
- 画面が Web でもスマホアプリでも **変わらない**
- フレームワークが Spring でも別のものでも **変わらない**

**ビジネスルールは技術的な都合で変わらない。だから技術に依存してはいけない。**

```
もし domain が infrastructure に依存していたら：
  DB を変更 → domain も変更が必要 → ビジネスルールのコードに手を入れる → バグのリスク

domain が独立していれば：
  DB を変更 → infrastructure だけ変更 → domain は一切触らない → 安全
```

**一言でまとめると：**
> domain は「ビジネスの本質」だから、技術の都合（DB、HTTP、フレームワーク）に振り回されてはいけない。だから他のレイヤに依存しない。

---

## 完成したパッケージ構成

```
src/main/java/com/example/kakeibo/
├── presentation/              ← HTTP の入口
│   ├── controller/            ← Controller クラス
│   ├── dto/                   ← Request / Response DTO
│   └── exception/             ← 例外ハンドラー（@RestControllerAdvice）
├── application/               ← ビジネスロジックの流れ
│   └── usecase/               ← UseCase クラス（処理の段取り）
├── domain/                    ← ビジネスルール（最も重要・最も安定）
│   ├── entity/                ← Entity クラス
│   ├── valueobject/           ← Value Object（Amount, CategoryName など）
│   ├── repository/            ← Repository インターフェース（ポート）
│   └── exception/             ← ドメイン例外
└── infrastructure/            ← 外部技術との接続
    ├── jpa/                   ← JPA Entity（DB テーブル対応）
    └── repository/            ← Repository 実装クラス（アダプタ）
```

### 依存の方向

```
presentation → application → domain
                               ↑
infrastructure ────────────────┘
```

- 矢印の向きは「依存する方向」
- domain はどこにも矢印を出さない = 何にも依存しない
- infrastructure は domain に依存する（インターフェースの実装だから）

---

## 各レイヤの覚え方

| レイヤ | 一言で | 役割 |
|--------|--------|------|
| presentation | 「外との窓口」 | HTTP リクエストを受けてレスポンスを返す |
| application | 「段取り係」 | UseCase で処理の流れを組み立てる |
| domain | 「ルールブック」 | ビジネスルールを守る。最も安定なレイヤ |
| infrastructure | 「技術担当」 | DB 接続など技術的な実装の詳細 |

---

## 自己チェック

| チェック | 内容 | 結果 |
|---------|------|------|
| △ | パッケージ構成を見ないで作れた | 一部 ToDo 参照（usecase, infrastructure の中身） |
| ○ | 依存の方向を説明できる | シニアの解説後に理解 |
| ○ | なぜ domain が最も安定なのか説明できる | 「ビジネスルールは技術に依存しない」 |

### ToDo を参照した箇所

| 箇所 | 何を確認したか |
|------|---------------|
| application/usecase | パッケージ名が `service` ではなく `usecase` であること |
| infrastructure/jpa | JPA Entity を infrastructure の中に置くこと |
| infrastructure/repository | Repository 実装を infrastructure の中に置くこと |

---

## 次の Phase へ

Phase 3 完了。次は **Phase 4: ドメインモデル設計** へ進む。
- Entity（Transaction, Category）の実装
- Value Object（Amount, TransactionType, CategoryName など）の実装
- ドメイン例外の実装

> **Phase 4 のポイント**: ToDo の Entity / VO 実装を参考にしつつ、家計簿固有のルール（金額は正の数、カテゴリ名は最大50文字など）を自力で表現できるか挑戦する。
