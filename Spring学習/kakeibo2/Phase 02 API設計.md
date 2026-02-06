# Phase 02: API設計

> **目的**: 家計簿アプリの REST API を設計する

---

## エンドポイントとは？

APIの「入口」となるURLのこと。クライアントからリクエストを受け取る場所。

```
クライアント（ブラウザ、アプリ）
      ↓
      ↓  HTTP リクエスト
      ↓
【エンドポイント】 ← ここが入口
      ↓
   サーバー（Spring Boot）
      ↓
   データベース
```

---

## REST API のルール

### なぜ URL に動詞を入れない？

```
❌ /api/transactions/create
❌ /api/transactions/delete
⭕ /api/transactions
```

**理由: HTTP メソッドが「動詞」の役割をするから**

| HTTP メソッド | 意味 |
|--------------|------|
| POST | 作る |
| GET | 取る |
| PUT | 更新する |
| DELETE | 削除する |

### URL とメソッドの組み合わせ

```
POST   /api/transactions      →「収支を」「作る」
GET    /api/transactions      →「収支を」「取る」
PUT    /api/transactions/1    →「収支1を」「更新する」
DELETE /api/transactions/1    →「収支1を」「削除する」
```

---

## REST とは

**URL は「リソース（データ）」を表し、操作は HTTP メソッドで区別する設計スタイル。**

| 用語 | 意味 |
|------|------|
| リソース | 操作したいデータのこと |
| transactions | 収支データ |
| categories | カテゴリデータ |

---

## API 設計一覧

### 収支（Transaction）

| 機能 | 説明 | HTTPメソッド | API | ステータスコード |
|------|------|-------------|-----|----------------|
| 収支を登録 | 今日1000円使った、等を記録 | POST | /api/transactions | 201 Created |
| 収支一覧を取得 | 登録した収支を見る | GET | /api/transactions | 200 OK |
| 収支詳細を取得 | 特定の1件を見る | GET | /api/transactions/{id} | 200 OK |
| 収支を更新 | 金額を間違えたので直す | PUT | /api/transactions/{id} | 200 OK |
| 収支を削除 | 間違って登録したので消す | DELETE | /api/transactions/{id} | 204 No Content |

### カテゴリ（Category）

| 機能 | 説明 | HTTPメソッド | API | ステータスコード |
|------|------|-------------|-----|----------------|
| カテゴリを作成 | 「食費」「交通費」等を作る | POST | /api/categories | 201 Created |
| カテゴリ一覧を取得 | カテゴリを見る | GET | /api/categories | 200 OK |
| カテゴリを削除 | いらないカテゴリを消す | DELETE | /api/categories/{id} | 204 No Content |

---

## ステータスコード一覧

| コード | 意味 | 使う場面 |
|--------|------|---------|
| 200 OK | 成功 | 取得・更新成功 |
| 201 Created | 作成成功 | POST で新規作成 |
| 204 No Content | 成功（返すデータなし） | DELETE 成功 |
| 400 Bad Request | リクエストが不正 | バリデーションエラー |
| 404 Not Found | 見つからない | 指定した ID が存在しない |

---

## 学んだこと

- エンドポイント = API の入口となる URL
- REST では URL は「リソース（名詞）」、操作は「HTTP メソッド（動詞）」で表す
- 同じ URL でも HTTP メソッドで操作が変わる
