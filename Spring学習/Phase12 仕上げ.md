# Phase 12: 仕上げ

> **目的**: ドキュメントを整え、学びを言語化する。プロジェクトを完成させる。

---

## 概要

| 課題 | 内容 |
|------|------|
| 課題1 | Swagger UI で API ドキュメント生成 |
| 課題2 | README 作成 |
| 課題3 | 設計判断ログの振り返り |

---

## 課題1: Swagger UI（OpenAPI）

### Swagger UI とは

**ジュニア**: API のドキュメントって手書きで作るんですか？

**シニア**: **Swagger UI** を使えば自動生成できる。コードと同期するからメンテナンスも楽。

| 項目 | 説明 |
|------|------|
| 目的 | API ドキュメントを自動生成 |
| メリット | ブラウザから API を試せる |
| URL | http://localhost:8080/swagger-ui/index.html |

---

### セットアップ

**pom.xml に依存関係を追加:**

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.8.4</version>
</dependency>
```

---

### 使い方

1. アプリを起動
2. http://localhost:8080/swagger-ui/index.html にアクセス
3. 各エンドポイントをクリック
4. 「**Try it out**」→「**Execute**」で API を実行

---

### Swagger UI のメリット

| メリット | 説明 |
|---------|------|
| **視覚的** | API の全体像が見える |
| **試しやすい** | ブラウザだけで API をテストできる |
| **共有しやすい** | フロントエンド担当に URL を渡すだけ |
| **自動更新** | コードを変えると自動で反映される |

---

## 課題2: README 作成

### README とは

**ジュニア**: README って何を書けばいいですか？

**シニア**: プロジェクトの「顔」。初めて見る人が理解できるように書く。

---

### 必須項目

| 項目 | 内容 |
|------|------|
| **概要** | 何を作ったか（1〜2文） |
| **技術スタック** | 使った技術の一覧 |
| **起動方法** | どうやって動かすか |
| **API 一覧** | エンドポイントの説明 |

---

### 作成した README

```markdown
# ToDo API

## 概要
Spring Boot で作成した ToDo 管理の REST API です。
Claude Code と一緒に DDD 軽量版の設計パターンを学習しながら作成しました。

## 技術スタック
- Java 17
- Spring Boot 3.5.10
- Spring Data JPA
- H2 Database（インメモリ）
- JUnit 5 / Mockito
- Swagger UI（springdoc-openapi）
- Lombok
- Maven

## 起動方法
### Eclipse
1. プロジェクトを右クリック
2. Run As → Spring Boot App

### コマンドライン
cd C:\dev\todo
.\mvnw.cmd spring-boot:run

## API 一覧
| メソッド | エンドポイント | 説明 |
|---------|---------------|------|
| POST | /api/todos | ToDo を作成 |
| GET | /api/todos | ToDo 一覧を取得 |
| GET | /api/todos/{id} | ToDo 詳細を取得 |
| PUT | /api/todos/{id} | ToDo を更新 |
| DELETE | /api/todos/{id} | ToDo を削除 |
| POST | /api/todos/{id}/complete | ToDo を完了にする |

## Swagger UI
http://localhost:8080/swagger-ui/index.html

## テスト実行
.\mvnw.cmd test

## H2 Console
http://localhost:8080/h2-console
```

---

## 課題3: 設計判断ログの振り返り

### なぜ振り返りが重要か

**シニア**: 「なぜこう設計したか」を言語化できると、面接で強い。

---

### 設計判断の質問と回答

#### Q1: なぜ Entity を直接返さず DTO を使った？

**回答のポイント:**
- Entity は DB の構造に依存している
- API のレスポンス形式は別で管理した方が変更に強い
- Entity に `password` などを追加しても API に漏れない

**参照**: Phase 02 API設計

---

#### Q2: なぜ Repository を Interface で切った？

**回答のポイント:**
- H2 → MySQL への切り替えが容易
- テスト時にモックに差し替えられる
- domain 層が infrastructure に依存しない（依存性逆転）

**参照**: Phase 06 Repository

---

#### Q3: なぜ Strategy パターンを使った？

**回答のポイント:**
- ソート方法を切り替え可能にするため
- 新しいソート方法を追加しても既存コードを変更しない（OCP）
- if-else の分岐より、新しいクラスを追加する方が安全

**参照**: Phase 09 パターン適用

---

#### Q4: なぜ Factory パターンを使った？

**回答のポイント:**
- 生成ルールを一箇所に集約
- `trim()` や `createdAt` の設定を毎回書かなくて済む
- UseCase をシンプルに保てる

**参照**: Phase 09 パターン適用

---

### 設計判断まとめ表

| 判断ポイント | 理由 | SOLID原則 |
|-------------|------|-----------|
| DTO 使用 | DB 構造と API 形式を分離 | 単一責任 |
| Repository Interface | DB 変更・テスト対応 | 依存性逆転 |
| Strategy パターン | 処理の切り替え | 開放閉鎖 |
| Factory パターン | 生成ルールの集約 | 単一責任 |

---

## 学習を通して難しかったこと

| 項目 | 内容 |
|------|------|
| レイヤ設計 | どのクラスをどこに置くか |
| DI（依存性注入） | @Autowired, コンストラクタインジェクション |
| モックを使ったテスト | when().thenReturn() の理解 |
| エラーログの読み方 | Caused by を探す |

---

## 次のステップ

### 再実装の題材

| 順番 | アプリ | 複雑度 | 学べるポイント |
|------|--------|--------|---------------|
| 1 | 家計簿アプリ | ★★☆ | 集計、カテゴリ分類 |
| 2 | 予約システム | ★★★ | 日時の扱い、状態遷移 |
| 3 | EC サイト | ★★★★ | 複数エンティティの関連 |

### 拡張で学ぶこと

| 拡張内容 | 学べること |
|---------|-----------|
| Spring Security | JWT、認証・認可 |
| Docker 化 | コンテナ、環境構築の自動化 |
| MySQL / PostgreSQL | 本番 DB 運用 |
| GitHub Actions | CI/CD、自動テスト |

### フロントエンド

バックエンドに慣れてから React などで画面を作成する。

---

## プロジェクト完了！

### 作成したもの

```
todo/
├── src/main/java/com/example/todo/
│   ├── presentation/     ← Controller, DTO
│   ├── application/      ← UseCase
│   ├── domain/           ← Entity, VO, Repository(I/F)
│   └── infrastructure/   ← JPA 実装
├── src/test/java/        ← テストコード
├── pom.xml
└── README.md
```

### 学んだこと一覧

| Phase | 内容 |
|-------|------|
| 01 | 環境構築（JDK, Eclipse, Spring Initializr） |
| 02 | API 設計（REST, HTTP メソッド, ステータスコード） |
| 03 | レイヤ設計（DDD 軽量版） |
| 04 | ドメインモデル（Entity, Value Object） |
| 05 | ユースケース（Application 層） |
| 06 | Repository（Interface で抽象化） |
| 07 | DB 実装（JPA, H2） |
| 08 | API 実装（Controller, 例外ハンドリング） |
| 09 | パターン適用（Factory, Strategy） |
| 10 | 品質ルール（ログ, SOLID） |
| 11 | テスト（JUnit, Mockito, WebMvcTest） |
| 12 | 仕上げ（Swagger, README, 振り返り） |

---

## チェックリスト

- [x] Swagger UI でドキュメント生成
- [x] README.md 作成
- [x] 設計判断の振り返り
- [x] 次のステップを明確化

---

*このドキュメントは学習中に詰まった点と解決方法を記録したものです。*

---

## おめでとうございます！

Spring Boot ToDo API の学習ロードマップを完走しました！

次は **別ドメインで再実装** して、手が覚えるまで繰り返しましょう。
