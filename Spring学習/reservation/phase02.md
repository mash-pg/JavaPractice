# Phase 02: API 設計（完了）

## ステータス
✅ **完了** 2026-02-10

## ブランチ
```bash
git checkout feature/phase02-api-design
```

## 作業内容

1. API のエンドポイントを設計書にまとめる
2. リクエスト/レスポンスの形式を決める
3. エラーケースを整理する

---

## 現在の進捗

| 作業 | 状態 |
|------|------|
| ブランチ作成 | ✓ |
| API 設計書作成 | 未着手 |
| PR 作成 | 未着手 |

---

## 次回の再開方法

このファイルを渡して「Phase 2 の続きをお願い」と言ってください。

---

## 次のタスク: API 設計書を作成

`C:\dev\reservation\docs\api-design.md` を作成する。

### 考える質問（再開時に回答）

ロードマップのエンドポイント設計を見てください：

```
PUT /api/reservations/{id}/confirm   予約確定
PUT /api/reservations/{id}/cancel    予約キャンセル
PUT /api/reservations/{id}/complete  予約完了
```

**質問**: なぜこれらは `PUT /api/reservations/{id}` ではなく、`/confirm`、`/cancel`、`/complete` というパスを追加しているのでしょうか？

kakeibo2 の更新 API と比較して考えてみてください。

**ヒント**:
- kakeibo2 の更新: データ（金額、カテゴリなど）を変更する
- 予約システムの更新: 状態（PENDING → CONFIRMED など）を変更する
- 「何を」更新するかが違う

---

## エンドポイント設計（ロードマップより）

| メソッド | パス | 説明 |
|---------|------|------|
| GET | /api/resources | リソース一覧取得 |
| POST | /api/resources | リソース作成 |
| GET | /api/reservations | 予約一覧取得 |
| POST | /api/reservations | 予約作成（PENDING） |
| PUT | /api/reservations/{id}/confirm | 予約確定 |
| PUT | /api/reservations/{id}/cancel | 予約キャンセル |
| PUT | /api/reservations/{id}/complete | 予約完了 |
| GET | /api/reservations/resource/{resourceId} | リソース別予約取得 |

## ステータスコード

| 操作 | 成功 | エラー |
|------|------|--------|
| 作成 | 201 | 400（バリデーション）, 409（重複） |
| 更新 | 200 | 404（存在しない）, 409（状態遷移不可） |
| 削除 | 204 | 404 |

---

## 作成する設計書の構成（予定）

1. 概要
2. 共通仕様（ベースURL、認証、エラーレスポンス形式）
3. Resource API
4. Reservation API
5. 状態遷移図
