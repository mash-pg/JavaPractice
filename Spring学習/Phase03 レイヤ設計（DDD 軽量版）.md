# Phase 3: レイヤ設計（DDD 軽量版）

> **目的**: コードの「置き場所」を決める。責務を分離することで、変更に強いコードになる。

---

## 問題 1: パッケージの役割

以下の 4 つのパッケージを作ります。それぞれの役割を線で結んでみてください。

```
パッケージ名              役割
─────────────────────────────────────────
A. presentation     →    1. ビジネスルール（Entity, VO）
B. application      →    2. HTTP の入口（Controller, DTO）
C. domain           →    3. DB 接続などの外部技術
D. infrastructure   →    4. ユースケース（処理の流れ）
```

---

## 解答

**A→2, B→4, C→1, D→3**

---

## 各パッケージの詳細

### presentation

| 項目 | 内容 |
|------|------|
| **役割** | HTTP の入口（Controller / View / API） |
| **英語の意味** | 表現・提示 |
| **意図** | ユーザーや外部とやり取りする層。「何をどう見せるか」「どう受け取るか」に責任を持つ |

### application

| 項目 | 内容 |
|------|------|
| **役割** | ユースケース層 |
| **英語の意味** | 適用・応用 |
| **意図** | ドメインのルールを使って「処理の流れ」を組み立てる層。*何をするか* はここで決めるが、*どう計算するか* は持たない |

### domain

| 項目 | 内容 |
|------|------|
| **役割** | ビジネスルールの中心 |
| **英語の意味** | 領域 |
| **意図** | 業務そのものの知識・制約・振る舞いを表現する層。技術から独立した「一番大事な心臓部」 |

### infrastructure

| 項目 | 内容 |
|------|------|
| **役割** | DB / 外部 API / ファイル I/O など |
| **英語の意味** | 基盤 |
| **意図** | 技術的な実装を引き受ける層。「どうやって保存・通信するか」を担当 |

---

## 問題 2: domain が何にも依存しない理由

### 結論

> **domain は「最も安定させたい層」だから**

### 悪い例：domain が infrastructure に依存している

```java
// domain の Todo.java の中で...
import javax.persistence.Entity;  // ← JPA（DB技術）に依存

@Entity
public class Todo {
    ...
}
```

この場合、以下の問題が起きる：

| 変更内容 | 影響 | 問題点 |
|---------|------|--------|
| JPA → MyBatis に変更 | domain のコードを修正 | 永続化技術の変更がビジネスルールに波及。domain が技術に依存している |
| MySQL → MongoDB に変更 | domain のコードを修正 | DB の種類変更で業務ロジックが変わるのはおかしい。「業務」と「保存方法」が密結合 |
| 単体テストをしたい | DB が必要になる | テストのたびに DB が必要なのは unit test ではない。domain がインフラ層に引きずられている |

### 良い例：domain は純粋な Java だけで書く

```java
public class Todo {
    private TodoTitle title;
    private TodoStatus status;

    public void complete() {  // ← ビジネスルール
        if (this.status == TodoStatus.DONE) {
            throw new AlreadyCompletedException();
        }
        this.status = TodoStatus.DONE;
    }
}
```

DB 技術は `infrastructure` に閉じ込める。

### 比喩

| 概念 | 比喩 |
|------|------|
| domain | 会社の「経営理念・業務ルール」 |
| infrastructure | 「使っている道具（Excel、システム、DB）」 |

> 道具が変わっても、経営理念は変わらない。そういう設計にしておく。

---

## 依存の方向

```
presentation → application → domain
                    ↑
infrastructure ─────┘
```

**ポイント**: `domain` は他のレイヤに依存しない。最も安定したレイヤ。

---

## 完成したパッケージ構成

```
com.example.todo
├── TodoApplication.java          ← アプリのエントリーポイント
├── presentation/                 ← HTTP の入口
│   ├── controller/              （Controller を置く）
│   └── dto/                     （Request/Response DTO を置く）
├── application/                  ← ユースケース
│   └── usecase/                 （業務処理の流れを置く）
├── domain/                       ← ビジネスルール（最重要）
│   ├── entity/                  （Todo などのエンティティ）
│   ├── valueobject/             （TodoTitle などの値オブジェクト）
│   └── exception/               （業務例外）
└── infrastructure/               ← 外部技術との接続
    ├── jpa/                     （JPA用 Entity）
    └── repository/              （Repository 実装）
```

---

## チェックリスト

- [x] パッケージの役割を理解した
- [x] 依存の方向を理解した
- [x] domain が他に依存しない理由を理解した
- [x] 8 個のパッケージを作成した

---

## 学んだこと

- レイヤを分けることで **変更に強い** 設計になる
- **domain** は最も重要な層であり、技術に依存させない
- パッケージ名は英単語の意味から推測できる
