# Phase 11: 仕上げ

## 概要

プロジェクトの仕上げとして、API ドキュメント自動生成と README を作成する。

---

## 作業内容

| 項目 | 内容 |
|------|------|
| Swagger UI | API ドキュメントを自動生成 |
| README | プロジェクトの説明を作成 |

---

## Swagger UI（OpenAPI）

### Swagger とは？

REST API のドキュメントを **自動生成** してくれるツール。
コードにアノテーションを追加するだけで、API 仕様書ができる。

### 導入方法

#### 1. 依存関係を追加（pom.xml）

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.8.4</version>
</dependency>
```

#### 2. アプリを起動

```bash
./mvnw spring-boot:run
```

#### 3. ブラウザでアクセス

```
http://localhost:8080/swagger-ui.html
```

### Swagger UI でできること

| 機能 | 説明 |
|------|------|
| API 一覧表示 | 全エンドポイントを一覧で確認 |
| リクエスト/レスポンス確認 | 各 API の入出力形式を確認 |
| 実際に API を実行 | ブラウザ上で API をテスト実行 |

### 公式ドキュメントの探し方

1. 検索: `springdoc-openapi` または `Spring Boot Swagger`
2. 公式サイト: https://springdoc.org/
3. 見るべきポイント:
   - 依存関係（何を pom.xml に追加するか）
   - 設定（application.properties に何を書くか）
   - アクセス URL

---

## README

### README とは？

プロジェクトの **説明書**。GitHub でリポジトリを開いたときに最初に表示される。

### 書くべき内容

| 項目 | 内容 |
|------|------|
| プロジェクト名 | 何のプロジェクトか |
| 概要 | 何をするアプリか |
| 技術スタック | 使った技術 |
| 起動方法 | どうやって動かすか |
| API エンドポイント | どんな API があるか |

### Markdown テーブルの書き方

```markdown
| 列1 | 列2 | 列3 |
|-----|-----|-----|
| A   | B   | C   |
| D   | E   | F   |
```

ポイント:
- `|` で区切る
- 2行目は `|-----|` でヘッダーと本体を分ける

---

## 全体の振り返り

### Phase 1〜11 で学んだこと

| Phase | 学んだこと |
|-------|-----------|
| 1 | 環境構築、依存関係の意味 |
| 2 | REST API 設計、HTTP ステータスコード |
| 3 | レイヤードアーキテクチャ、依存の方向 |
| 4 | Entity, Value Object, ドメイン例外 |
| 5 | UseCase、@Transactional |
| 6 | Repository Interface、DIP（依存性逆転） |
| 7 | JPA Entity、Domain Entity との分離 |
| 8 | DTO、Controller、例外ハンドリング |
| 9 | Factory パターン、Strategy パターン |
| 10 | テスト（JUnit, Mockito, MockMvc） |
| 11 | Swagger UI、README |

### 説明できるようになったこと

- なぜレイヤを分けるのか？
- なぜ Value Object を使うのか？
- なぜ DTO を使うのか？
- なぜ Repository Interface を domain に置くのか？
- Factory パターン、Strategy パターンはいつ使うのか？

### 身についた Git フロー

```
main
  │
  ├── feature/phase01-setup
  ├── feature/phase02-api-design
  ├── ...
  └── feature/phase11-finish
```

1. feature ブランチを切る
2. 実装する
3. コミット & プッシュ
4. PR 作成
5. レビュー & 修正
6. マージ

---

## kakeibo2 完了！

おめでとう！全 11 Phase を完走しました。

次のステップ:
- 予約システムなど別ドメインで再実装
- Spring Security（認証・認可）
- Docker 化
- MySQL/PostgreSQL への移行
