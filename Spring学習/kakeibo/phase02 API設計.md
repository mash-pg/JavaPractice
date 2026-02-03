# Phase 02: API 設計（家計簿アプリ）

> **目的**: 家計簿アプリの API を「自分で」設計する。ToDo の設計を思い出しながら、新しいドメインに合わせる。

---

## 概要

| 項目 | 内容 |
|------|------|
| 設計対象 | 家計簿アプリの REST API |
| 学ぶこと | REST API 設計の再実践、複数リソースの設計、フィルタリング |
| ToDo との違い | リソースが2種類（収支・カテゴリ）、集計 API がある |

---

## 2.1 機能要件

**シニア**: まず家計簿アプリに必要な機能を整理しよう。

| # | 機能 |
|---|------|
| 1 | 収入・支出を登録できる |
| 2 | 収支の一覧を見れる |
| 3 | 収支の詳細を見れる |
| 4 | 収支を修正できる |
| 5 | 収支を削除できる |
| 6 | カテゴリ（食費、交通費など）を管理できる |
| 7 | 月ごとの集計（合計収入・合計支出・残高）を見れる |

**シニア**: この7つの機能に対して、エンドポイントを自分で設計してみてくれ。

---

## 2.2 最初の設計（1回目）

**ジュニア**: こう考えました。

```
1. /api/kakeibo/
2. /api/kakeibo/list
3. /api/kakeibo/detail/{id}
4. /api/kakeibo/edit/{id}
5. /api/kakeibo/delete/{id}
6. /api/kakeibo/manage/{id}
7. /api/kakeibo/month/{id}
```

---

### シニアのレビュー

**シニア**: 7つの機能に対して考えられてるのは良い。ただ、REST API の基本ルールを思い出してほしい。

#### 問題1: パスに動詞が入っている

ToDo のときのエンドポイントを思い出して。

| ToDo のエンドポイント | やっていたこと |
|---------------------|-------------|
| `POST /api/todos` | 作成 |
| `GET /api/todos` | 一覧 |
| `GET /api/todos/{id}` | 詳細 |
| `PUT /api/todos/{id}` | 更新 |
| `DELETE /api/todos/{id}` | 削除 |

ToDo では `/api/todos/list` や `/api/todos/delete/{id}` にはしなかった。**動詞（list, edit, delete）をパスに入れず、HTTP メソッド（GET, POST, PUT, DELETE）で操作を区別**していた。

```
❌  GET  /api/kakeibo/list        → パスに動詞「list」が入っている
✅  GET  /api/transactions        → HTTP メソッド GET が「取得」を表す

❌  POST /api/kakeibo/delete/{id} → パスに動詞「delete」が入っている
✅  DELETE /api/transactions/{id} → HTTP メソッド DELETE が「削除」を表す
```

#### 問題2: リソース名がアプリ名になっている

REST ではリソース（データの種類）を名前にする。

| 考え方 | 例 |
|--------|-----|
| ❌ アプリ名 | `/api/kakeibo/` |
| ✅ リソース名 | `/api/transactions/`（収支データ） |

#### 問題3: カテゴリは別リソース

機能6の「カテゴリ管理」は収支とは別のリソースなので、別のパスにする。

---

## 2.3 修正後の設計（2回目）

**ジュニア**: 修正しました。

| メソッド | エンドポイント | 説明 | ステータス |
|---------|---------------|------|-----------|
| `POST` | `/api/transactions` | 収支を登録 | |
| `GET` | `/api/transactions` | 収支一覧を取得 | |
| `GET` | `/api/transactions/{id}` | 収支詳細を取得 | |
| `PUT` | `/api/transactions/{id}` | 収支を更新 | |
| `DELETE` | `/api/transactions/{id}` | 収支を削除 | |
| `GET` | `/api/transactions/{id}/category` | カテゴリを管理 | |
| `GET` | `/api/transactions/{id}/month` | 月次集計 | |

---

### シニアのレビュー

**シニア**: REST の基本（HTTP メソッドで区別、リソース名は名詞）がちゃんと適用できてる。大幅に良くなった。ただ2点修正が必要。

#### 問題1: カテゴリが transactions の下にある

```
❌  GET /api/transactions/{id}/category
```

これだと「特定の収支に紐づくカテゴリを取得」という意味になる。カテゴリは収支とは**別に管理するリソース**。「食費」「交通費」「家賃」というカテゴリの一覧を作成・取得・削除する。

```
✅  POST   /api/categories       → カテゴリ作成
✅  GET    /api/categories       → カテゴリ一覧
✅  DELETE /api/categories/{id}  → カテゴリ削除
```

`/api/transactions` と `/api/categories` は**同じ階層に並ぶ兄弟**のイメージ。

#### 問題2: 月次集計に `{id}` が付いている

```
❌  GET /api/transactions/{id}/month
```

`{id}` は特定の収支1件を指す。でも月次集計は「全体の集計」だから、特定の ID に紐づかない。

```
✅  GET /api/transactions/summary?year=2026&month=1
```

集計条件（年・月）は**クエリパラメータ**で渡す。ToDo で `?sort=priority` を使ったのと同じ考え方。

---

## 2.4 ステータスコードの追加（3回目）

**シニア**: ステータスコードも付けてみて。ToDo で使った 200, 201, 204 を思い出して。

| 操作 | 何が起きた？ | コード |
|------|------------|--------|
| 作成 | 新しいものが**作られた** | 201 Created |
| 取得 | 正常に**返せた** | 200 OK |
| 更新 | 正常に**返せた** | 200 OK |
| 削除 | 消した、**返すものがない** | 204 No Content |

**ジュニア**: 修正しました。ただカテゴリの API がまだ足りなかったです。

| メソッド | エンドポイント | 説明 | ステータス |
|---------|---------------|------|-----------|
| `POST` | `/api/transactions` | 収支を登録 | 201 |
| `GET` | `/api/transactions` | 収支一覧を取得 | 200 |
| `GET` | `/api/transactions/{id}` | 収支詳細を取得 | 200 |
| `PUT` | `/api/transactions/{id}` | 収支を更新 | 200 |
| `DELETE` | `/api/transactions/{id}` | 収支を削除 | 204 |
| `POST` | `/api/categories` | カテゴリを作成 | 200 |
| `GET` | `/api/categories/{id}` | カテゴリ一覧を取得 | 200 |
| `DELETE` | `/api/categories/{id}` | カテゴリを削除 | 200 |
| `GET` | `/api/transactions/summary` | 月次集計を取得 | 200 |

---

### シニアのレビュー

**シニア**: 収支の方は完璧。カテゴリにあと2つ修正点がある。

#### 問題1: カテゴリ一覧に `{id}` が付いている

```
❌  GET /api/categories/{id}   → 特定の1件を取得する意味になる
✅  GET /api/categories        → 一覧を取得する
```

`{id}` が付くのは「1件を指定する操作」（詳細取得・削除）のとき。一覧は全部取るから `{id}` は不要。

#### 問題2: カテゴリのステータスコードが transactions と不一致

| エンドポイント | ジュニアの値 | 正しい値 | 理由 |
|--------------|------------|---------|------|
| `POST /api/categories` | 200 | **201** | 新しいものを**作った** → Created |
| `DELETE /api/categories/{id}` | 200 | **204** | 削除した、**返すものがない** → No Content |

> transactions の方では POST=201、DELETE=204 にできていた。**リソースが違っても同じルール**。

---

## 2.5 最終版（4回目） ✅

**ジュニア**: 修正しました。

**収支（Transaction）:**

| メソッド | エンドポイント | 説明 | ステータス |
|---------|---------------|------|-----------|
| `POST` | `/api/transactions` | 収支を登録 | 201 |
| `GET` | `/api/transactions` | 収支一覧を取得 | 200 |
| `GET` | `/api/transactions/{id}` | 収支詳細を取得 | 200 |
| `PUT` | `/api/transactions/{id}` | 収支を更新 | 200 |
| `DELETE` | `/api/transactions/{id}` | 収支を削除 | 204 |
| `GET` | `/api/transactions/summary` | 月次集計を取得 | 200 |

**カテゴリ（Category）:**

| メソッド | エンドポイント | 説明 | ステータス |
|---------|---------------|------|-----------|
| `POST` | `/api/categories` | カテゴリを作成 | 201 |
| `GET` | `/api/categories` | カテゴリ一覧を取得 | 200 |
| `DELETE` | `/api/categories/{id}` | カテゴリを削除 | 204 |

**シニア**: 合格。完璧。

---

## 2.6 フィルタリング（クエリパラメータ）

```
GET /api/transactions?type=EXPENSE          → 支出のみ
GET /api/transactions?category=食費          → カテゴリ指定
GET /api/transactions?from=2026-01-01&to=2026-01-31  → 日付範囲
GET /api/transactions?type=EXPENSE&category=食費     → 複合フィルタ
```

> **ToDo との違い**: ToDo は `?sort=` だけだったが、家計簿は複数条件でフィルタリングする。

---

## 2.7 集計レスポンス例

```json
{
  "year": 2026,
  "month": 1,
  "totalIncome": 250000,
  "totalExpense": 180000,
  "balance": 70000,
  "expenseByCategory": [
    { "category": "食費", "amount": 50000 },
    { "category": "交通費", "amount": 15000 },
    { "category": "家賃", "amount": 80000 }
  ]
}
```

> summary はシングルユーザー前提。認証は後の拡張（Spring Security）でやる。

---

## 2.8 ステータスコード方針

| コード | 意味 | 使う場面 |
|--------|------|---------|
| `201` | Created | 作成成功 |
| `200` | OK | 取得・更新成功 |
| `204` | No Content | 削除成功 |
| `400` | Bad Request | バリデーションエラー |
| `404` | Not Found | リソースが存在しない |
| `409` | Conflict | ビジネスルール違反（カテゴリ削除時に使用中など） |

---

## 2.9 設計ルール

- [x] Request / Response に **DTO** を使う
- [x] Entity を Controller から直接返さない

---

## 学んだこと

### 間違いから学んだ REST API のルール

| 回 | 間違い | 学んだこと |
|----|--------|-----------|
| 1回目 | パスに動詞を入れた（`/list`, `/delete`） | REST は HTTP メソッドで操作を区別する。パスは名詞（リソース名）にする |
| 1回目 | リソース名がアプリ名（`/kakeibo`） | リソース名はデータの種類（`/transactions`）にする |
| 2回目 | カテゴリを transactions の下に置いた | 別リソースは同じ階層に並べる（兄弟関係） |
| 2回目 | 集計に `{id}` を付けた | 全体の集計は特定 ID に紐づかない。クエリパラメータで条件を渡す |
| 3回目 | カテゴリ一覧に `{id}` を付けた | `{id}` は1件を指定する操作のみ。一覧には不要 |
| 3回目 | カテゴリの POST=200, DELETE=200 | リソースが違っても同じルール。POST=201, DELETE=204 |

### ToDo との比較

| 観点 | ToDo | 家計簿 |
|------|------|--------|
| リソース数 | 1（todos） | 2（transactions, categories） |
| クエリパラメータ | `?sort=` のみ | `?type=`, `?category=`, `?from=`, `?to=` |
| 集計 API | なし | `/summary` がある |
| 特殊操作 | `/complete`（状態遷移） | なし（代わりに集計がある） |

---

## チェックリスト

- [x] REST API の基本ルールを再確認（HTTP メソッドで操作を区別）
- [x] 複数リソース（transactions, categories）のエンドポイント設計
- [x] ステータスコードの使い分け（201, 200, 204）
- [x] フィルタリングの設計（クエリパラメータ）
- [x] 集計 API の設計
- [x] リソースが違っても同じルールが適用されることを理解

---

## 次のステップ

- [ ] Phase 03: レイヤ設計 へ進む

---

*このドキュメントは学習中の試行錯誤と、シニアのレビューで修正した過程を記録したものです。4回の修正を経て最終版に到達した。*
