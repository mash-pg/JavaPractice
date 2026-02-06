# Spring Boot 家計簿 API 学習ロードマップ（DDD軽量版・再実装）

> このロードマップは、ToDo アプリで学んだ DDD 軽量版の設計パターンを「別ドメイン」で再実装し、手が覚えるレベルまで定着させることを目的としています。

---

## Claude への指示（次回の再開用）

> **次回このファイルを渡して「続きをお願い」と言えば、以下の指示に従って再開してくれます。**

```
あなたはシニアエンジニア（Claude）です。
私はジュニアエンジニア（学習者）です。

このロードマップの「進捗管理」セクションを確認し、
次の未完了 Phase からシニア × ジュニア形式で学習を進めてください。

【重要】作業場所について:
- 実装コード（Java）: C:\dev\kakeibo
- ドキュメント（md）: C:\Users\user\Documents\JavaPractice\Spring学習\kakeibo

※ phase○○ ドキュメント名.md などのドキュメントは必ず「ドキュメント」の場所に作成すること。
※ 実装コードの場所にドキュメントを作成しないこと。

進め方:
1. シニアが課題と要件を提示する
2. ジュニアが実装を試みる（答えを先に見せない）
3. シニアがコードレビュー（良い点・改善点を指摘）
4. 修正 → 再レビュー → 合格まで繰り返す
5. Phase が完了したら md ファイルを更新する

ルール:
- 答えをいきなり教えず、まずジュニアに考えさせる
- 間違いは具体的に指摘し、ヒントを出して自力で修正させる
- 各 Phase 完了時に、対応する phase○○.md を「ドキュメント」の場所に作成する
  （シニア・ジュニアのやり取り、詰まった箇所、学んだことを詳しく記録）
- ロードマップ（このファイル）の進捗管理とPhaseセクションも更新する
- 公式ドキュメントを読む習慣を付けさせる（下記「ドキュメントを読む習慣」セクション参照）
```

---

## 前提・ゴール

| 項目 | 内容 |
|------|------|
| **題材** | 家計簿アプリ（収支 CRUD + カテゴリ分類 + 月次集計） |
| **複雑度** | ★★☆（ToDo の次のステップとして最適） |
| **学ぶこと** | REST API / DDD（軽量）/ DI / Repository / Strategy / Factory |
| **ToDo との違い** | 複数エンティティの関連、集計処理、日付範囲フィルタリング |
| **設計原則** | SOLID / DRY / YAGNI |
| **記録** | Obsidian で設計判断ログを残す |

### ToDo アプリとの比較

| 観点 | ToDo アプリ | 家計簿アプリ |
|------|-----------|------------|
| エンティティ数 | 1（Todo） | 2（Transaction, Category） |
| 状態遷移 | OPEN → DONE | なし（代わりに集計がある） |
| 集計処理 | なし | 月次・カテゴリ別の合計 |
| フィルタリング | ソートのみ | 日付範囲、カテゴリ、収支タイプ |
| 値オブジェクト | 4個 | 5個（Amount が新登場） |
| パターン適用 | Sort の Strategy | 集計の Strategy |

---

## 学習スタイル

> **シニアエンジニア（Claude）× ジュニアエンジニア（自分）形式**

| 役割 | 担当 |
|------|------|
| **シニア（Claude）** | 課題を出す、ヒントを与える、コードレビュー、間違いを指摘 |
| **ジュニア（自分）** | 課題を解く、コードを書く、質問する |

### 進め方

1. シニアが課題と要件を提示
2. ジュニアが実装を試みる
3. シニアがコードレビュー（良い点・改善点）
4. 修正 → 再レビュー → 合格まで繰り返す
5. 動作確認して次の課題へ

> **ポイント**: ToDo で学んだことを「見ないで」書けるか挑戦する。詰まったら ToDo のコードを参考にしてOK。

### ドキュメントを読む習慣（実務で最も差がつくスキル）

> **なぜ重要か**: 実務では「ググって Qiita の記事をコピペ」では通用しない場面が必ず来る。公式ドキュメントを読んで正確な情報を取れるかどうかが、ジュニアとミドルの分かれ目。

**シニアの役割:**
- 課題を出すとき、関連する公式ドキュメントの URL を一緒に提示する
- 「この公式ドキュメントのどこに書いてある？」と質問する
- ジュニアが詰まったとき、答えを直接教えず「公式ドキュメントのこのページを読んでみて」と誘導する
- コードレビュー時に「このアノテーションの公式の説明を読んだ？」と確認する

**ジュニアの役割:**
- 新しいアノテーションや機能を使うとき、まず公式ドキュメントを開く
- 「公式で確認した」「Javadoc を読んだ」と記録に残す
- エラーが出たら、まず公式のトラブルシューティングを探す

**よく使う公式ドキュメント:**

| ドキュメント | URL | 使う場面 |
|------------|-----|---------|
| Spring Boot リファレンス | https://docs.spring.io/spring-boot/reference/ | 設定、機能の確認 |
| Spring Framework リファレンス | https://docs.spring.io/spring-framework/reference/ | DI、Web MVC、トランザクション |
| Spring Data JPA リファレンス | https://docs.spring.io/spring-data/jpa/reference/ | Repository、クエリメソッド |
| Jakarta Validation (Bean Validation) | https://jakarta.ee/specifications/bean-validation/ | @NotNull, @Size などのバリデーション |
| JUnit 5 ユーザーガイド | https://junit.org/junit5/docs/current/user-guide/ | テストの書き方 |
| Mockito ドキュメント | https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html | モックの使い方 |
| Java 17 API (Javadoc) | https://docs.oracle.com/en/java/javase/17/docs/api/ | 標準ライブラリの仕様確認 |

**Phase ごとのドキュメント読み課題:**

| Phase | 読むべきドキュメント | 確認ポイント |
|-------|-------------------|-------------|
| Phase 8 | Spring Web MVC: `@RestController`, `@RequestBody`, `@PathVariable`, `ResponseEntity` | 各アノテーションの公式説明を読んで、引数に何が指定できるか確認 |
| Phase 8 | Bean Validation: `@NotNull`, `@NotBlank`, `@Size`, `@Min` | どのアノテーションをどの型に使えるか確認 |
| Phase 8 | Spring: `@RestControllerAdvice`, `@ExceptionHandler` | 例外ハンドリングの仕組みを公式で確認 |
| Phase 9 | Spring Data JPA: Query Methods | メソッド名からクエリを自動生成する命名規則を公式で確認 |
| Phase 10 | JUnit 5: `@Test`, `assertThrows`, `@BeforeEach` | テストライフサイクルを公式で確認 |
| Phase 10 | Mockito: `mock()`, `when()`, `verify()` | モックの基本的な使い方を公式で確認 |
| Phase 10 | Spring Boot Test: `@WebMvcTest`, `MockMvc` | Controller テストの公式ガイドを確認 |
| Phase 11 | SpringDoc OpenAPI | Swagger UI の設定方法を公式で確認 |

**ドキュメントの読み方（実務テクニック）:**

```
1. 全部読もうとしない → 目次から必要な箇所を探す
2. 最初は「Getting Started」や「Quick Start」を見る
3. サンプルコードを先に見て、説明文を後から読む
4. 分からない単語があっても、まず流し読みして全体像をつかむ
5. 英語が辛くても、コード部分は読める → コードから意味を推測する
```

---

## 進捗管理

- [x] Phase 1: 環境構築 [[#Phase 1 環境構築 ✓]]
- [x] Phase 2: API 設計 [[#Phase 2 API 設計 ✓]]
- [x] Phase 3: レイヤ設計 [[#Phase 3 レイヤ設計（DDD 軽量版） ✓]]
- [x] Phase 4: ドメイン設計 [[#Phase 4 ドメインモデル設計 ✓]]
- [x] Phase 5: ユースケース  [[#Phase 5 ユースケース設計（Application 層） ✓]]
- [x] Phase 6: Repository  [[#Phase 6 Repository インターフェース設計 ✓]]
- [x] Phase 7: DB 実装 [[#Phase 7 DB・JPA 実装（Infrastructure） ✓]]
- [x] Phase 8: API 実装（基本 CRUD）[[#Phase 8 API 実装（基本 CRUD） ✓]]
- [x] Phase 9: パターン適用（集計・フィルタ）[[#Phase 9 パターン適用（集計・フィルタ） ✓]]
- [x] Phase 10: テスト [[#Phase 10 テスト ✓]]
- [ ] Phase 11: 仕上げ・振り返り [[#Phase 11 仕上げ・振り返り]]

---

## 学習フェーズ一覧

```
Phase 1: 環境構築         → 開発の土台を整える（ToDo と同じ手順を自力で）
Phase 2: API 設計         → 家計簿の API を設計する
Phase 3: レイヤ設計       → パッケージ構成を自力で作る
Phase 4: ドメイン設計     → 収支・カテゴリのビジネスルールを表現する
Phase 5: ユースケース     → 処理の流れを実装する
Phase 6: Repository      → データの出し入れを抽象化する
Phase 7: DB 実装         → 複数テーブルの保存処理を作る
Phase 8: API 実装        → HTTP の入口を作る（基本 CRUD）
Phase 9: パターン適用    → 集計・フィルタに Strategy / Factory を適用
Phase 10: テスト          → 品質を担保する
Phase 11: 仕上げ         → ドキュメント・ToDo との比較振り返り
```

---

## Phase 1: 環境構築 ✓
[[#進捗管理]]

> **目的**: ToDo と同じ手順を「自力で」できるか確認する。

### 1.1 開発環境

| 項目 | 内容 |
|------|------|
| JDK | 17（LTS） |
| IDE | Eclipse |
| Build Tool | Maven |
| Framework | Spring Boot 3.x |

#### JDK 17

ToDo のときにインストール済みのはず。確認だけしておく。

```powershell
java -version
# → openjdk version "17.0.x" ... と出れば OK

echo $env:JAVA_HOME
# → Java 17 のパスが表示されれば OK
```

#### Eclipse のセットアップ

ToDo のときにインストール済みのはず。以下が入っているか確認する。

**Spring Tools:**

| 項目 | 内容 |
|------|------|
| 確認方法 | Help → Eclipse Marketplace → Installed タブ |
| 入っていなければ | Help → Eclipse Marketplace → 「Spring Tools」で検索 → インストール → Eclipse 再起動 |

**Lombok:**

| 項目       | 内容                                                                                       |
| -------- | ---------------------------------------------------------------------------------------- |
| 確認方法     | Help → About Eclipse IDE → `Lombok v1.18.xx` の表記があるか確認                                   |
| 入っていなければ | `lombok.jar` を実行して Eclipse のインストール先を指定 → Eclipse 再起動![[Pasted image 20260203163638.png]] |

> Lombok は Eclipse Marketplace ではなく、専用インストーラ（`lombok.jar`）で導入する。Maven の依存に `lombok` があるだけではコンパイル時に動かない。

### 1.2 Spring Initializr でプロジェクト作成

**URL**: https://start.spring.io/

| 設定項目        | 値           |
| ----------- | ----------- |
| Project     | Maven       |
| Language    | Java        |
| Spring Boot | 3.x（最新安定版）  |
| Group       | com.example |
| Artifact    | kakeibo     |
| Name        | kakeibo     |
| Packaging   | Jar         |
| Java        | 17          |

**Dependencies:**

| ライブラリ | 用途 | なぜ必要？ |
|-----------|------|-----------|
| Spring Web | REST API を作る | Controller, HTTP 通信 |
| Validation | 入力値チェック | @NotBlank, @Size など |
| Spring Data JPA | DB 操作 | Repository, Entity |
| H2 Database | インメモリ DB | 開発用、設定不要で使える |
| Lombok | ボイラープレート削減 | @Getter, @Setter, @Slf4j |

### 1.3 プロジェクトの生成と Eclipse へのインポート

1. 「Generate」ボタンをクリック → ZIP ファイルがダウンロードされる
2. ZIP を展開して `C:\dev\kakeibo` に配置
3. Eclipse で「File」→「Import」→「Existing Maven Projects」
4. `C:\dev\kakeibo` を選択 → 「Finish」

### 1.4 application.properties の設定

```properties
spring.application.name=kakeibo

# H2 Database 設定
spring.datasource.url=jdbc:h2:mem:kakeibodb
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# JPA 設定
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true

# H2 Console（ブラウザでDBを見れる）
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

### 1.5 動作確認

#### アプリの起動

**Eclipse の場合:**
1. `KakeiboApplication.java` を右クリック
2. Run As → Spring Boot App

**コマンドラインの場合:**
```powershell
cd C:\dev\kakeibo
.\mvnw.cmd spring-boot:run
```

#### 起動確認

**方法1: コンソールログを確認**
```
Started KakeiboApplication in x.xxx seconds
```
このメッセージが出れば OK。

**方法2: ブラウザでアクセス**
```
http://localhost:8080
```
**Whitelabel Error Page** が表示されれば OK（まだ何も作っていないのでエラーで正常）。

#### H2 Console の確認

1. ブラウザで http://localhost:8080/h2-console を開く
2. 以下を入力:
   - JDBC URL: `jdbc:h2:mem:kakeibodb`
   - User Name: `sa`
   - Password: （空欄）
3. Connect をクリック

### 1.6 プロジェクト構造の確認

```
kakeibo/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/kakeibo/
│   │   │       └── KakeiboApplication.java  ← エントリーポイント
│   │   └── resources/
│   │       └── application.properties       ← 設定ファイル
│   └── test/
│       └── java/
│           └── com/example/kakeibo/
│               └── KakeiboApplicationTests.java  ← テスト
├── pom.xml  ← Maven の設定ファイル
└── mvnw.cmd ← Maven Wrapper
```

### 1.7 よくあるトラブル

| 状況 | 対処 |
|------|------|
| `Port 8080 is already in use` | ToDo アプリが起動中かも。停止するか `server.port=8081` を設定 |
| Maven のダウンロードが遅い | 初回は時間がかかる。2回目以降はキャッシュされて速い |
| Lombok が効かない | Eclipse Marketplace で Lombok が入っているか確認 → Eclipse 再起動 |
| `JAVA_HOME` が古い | `echo $env:JAVA_HOME` で Java 17 のパスか確認 |

### 1.8 チェックリスト

- [x] JDK 17 がインストールされている
- [x] JAVA_HOME が Java 17 を指している
- [x] Eclipse に Spring Tools + Lombok が入っている
- [x] Spring Initializr でプロジェクト作成済み
- [x] Eclipse にインポート済み
- [x] アプリ起動確認（Whitelabel Error Page）
- [x] H2 Console にアクセスできる

### 1.9 自己チェック

> ToDo のときは手順書を見ながらやった。今回は **見ないで** できたか？

| チェック | 内容                             |
| ---- | ------------------------------ |
| [ ✅] | Spring Initializr の設定を迷わず選べた   |
| [▲]  | application.properties を自力で書けた |
| [✅ ] | Eclipse へのインポート手順を覚えていた        |
| [✅ ] | 起動エラーが出ても自分で対処できた              |

### 1.10 学んだこと

#### application.properties は暗記するものではない

**実務での調べ方:**

| 方法 | 説明 |
|------|------|
| **Spring Boot 公式リファレンス** | https://docs.spring.io/spring-boot/appendix/application-properties/index.html に全プロパティ一覧がある |
| **IDE の補完機能** | Eclipse（Spring Tools 入り）で `spring.` と打つと候補が出る。実務で一番使う方法 |
| **公式ガイド** | 「Spring Boot H2」等で検索すると公式ガイドが見つかる |

**覚えるべきもの vs 調べるもの:**

| 区分 | 内容 | 例 |
|------|------|-----|
| **覚える** | 「何を設定する必要があるか」 | DB を使うなら datasource の設定が要る |
| **調べる** | 「プロパティ名の正確なスペル」 | `spring.datasource.url` の綴り |
| **調べる** | 「設定値の書き方」 | H2 の JDBC URL 形式 |

> **ポイント**: プロパティ名を暗記しているエンジニアはほぼいない。「何を設定する必要があるか」が分かっていれば、あとは IDE の補完か公式ドキュメントで書ける。

---

## Phase 2: API 設計 ✓
[[#進捗管理]]

> **目的**: 家計簿アプリの API を「自分で」設計する。ToDo の設計を参考にしつつ、新しいドメインに合わせる。

### 2.1 エンドポイント一覧

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

| メソッド     | エンドポイント                | 説明        | ステータス |
| -------- | ---------------------- | --------- | ----- |
| `POST`   | `/api/categories`      | カテゴリを作成   | 201   |
| `GET`    | `/api/categories`      | カテゴリ一覧を取得 | 200   |
| `DELETE` | `/api/categories/{id}` | カテゴリを削除   | 204   |

### 2.2 フィルタリング（クエリパラメータ）

```
GET /api/transactions?type=EXPENSE          → 支出のみ
GET /api/transactions?category=食費          → カテゴリ指定
GET /api/transactions?from=2026-01-01&to=2026-01-31  → 日付範囲
GET /api/transactions?type=EXPENSE&category=食費     → 複合フィルタ
```

> **ToDo との違い**: ToDo は `?sort=` だけだったが、家計簿は複数条件でフィルタリングする。

### 2.3 集計レスポンス例

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

### 2.4 ステータスコード方針

| コード | 意味 | 使う場面 |
|--------|------|---------|
| `201` | Created | 作成成功 |
| `200` | OK | 取得・更新成功 |
| `204` | No Content | 削除成功 |
| `400` | Bad Request | バリデーションエラー |
| `404` | Not Found | リソースが存在しない |
| `409` | Conflict | ビジネスルール違反（カテゴリ削除時に使用中など） |

### 2.5 設計ルール

- [x] Request / Response に **DTO** を使う
- [x] Entity を Controller から直接返さない

### 2.6 学んだこと（4回の修正で到達）

| 回 | 間違い | 学んだこと |
|----|--------|-----------|
| 1回目 | パスに動詞を入れた（`/list`, `/delete`） | REST は HTTP メソッドで操作を区別する |
| 1回目 | リソース名がアプリ名（`/kakeibo`） | リソース名はデータの種類にする |
| 2回目 | カテゴリを transactions の下に置いた | 別リソースは同じ階層に並べる |
| 2回目 | 集計に `{id}` を付けた | 全体の集計は特定 ID に紐づかない |
| 3回目 | カテゴリ一覧に `{id}` を付けた | 一覧取得に `{id}` は不要 |
| 3回目 | カテゴリの POST=200, DELETE=200 | リソースが違っても同じルール |

> **詳細**: [[Spring学習/kakeibo/phase02 API設計]] を参照

---

## Phase 3: レイヤ設計（DDD 軽量版） ✓
[[#進捗管理]]

> **目的**: ToDo で学んだレイヤ構成を「自力で」再現する。

### 3.1 課題

**以下のパッケージ構成を自分で作成してください。**

```
src/main/java/com/example/kakeibo/
├── presentation/          ← HTTP の入口
│   ├── controller/
│   ├── dto/
│   └── exception/
├── application/           ← ビジネスロジックの流れ
│   └── usecase/
├── domain/                ← ビジネスルール（最も重要）
│   ├── entity/
│   ├── valueobject/
│   ├── repository/
│   └── exception/
└── infrastructure/        ← 外部技術との接続
    ├── jpa/
    └── repository/
```

### 3.2 確認ポイント

- [x] ToDo と同じ依存方向になっているか
- [x] domain が他のレイヤに依存していないか

```
presentation → application → domain
                    ↑
infrastructure ─────┘
```

### 3.3 自己チェック

| チェック | 内容 |
|---------|------|
| [△] | パッケージ構成を見ないで作れた（一部 ToDo 参照） |
| [x] | 依存の方向を説明できる |
| [x] | なぜ domain が最も安定なのか説明できる |

> **詳細**: [[Spring学習/kakeibo/phase03 レイヤ設計]] を参照

---

## Phase 4: ドメインモデル設計 ✓
[[#進捗管理]]

> **目的**: 家計簿ドメインのビジネスルールをコードで表現する。

### 4.1 Entity

| Entity | 説明 |
|--------|------|
| `Transaction` | 収支のメインエンティティ |
| `Category` | カテゴリ（食費、交通費、家賃など） |

### 4.2 Value Object（値オブジェクト）

| VO | ルール |
|----|--------|
| `TransactionId` | null 禁止 |
| `Amount` | null 禁止、**0 より大きい**（金額は正の数） |
| `TransactionType` | INCOME（収入）/ EXPENSE（支出） |
| `CategoryId` | null 禁止 |
| `CategoryName` | 空禁止、最大50文字、重複禁止 |

> **ToDo との違い**: `Amount` は数値の VO。文字列の `TodoTitle` とは別のバリデーションが必要。

### 4.3 Exception

| 例外 | 用途 |
|------|------|
| `InvalidAmountException` | 金額が 0 以下のとき |
| `InvalidCategoryNameException` | カテゴリ名が不正なとき |
| `TransactionNotFoundException` | 収支が見つからないとき |
| `CategoryNotFoundException` | カテゴリが見つからないとき |
| `CategoryInUseException` | 使用中のカテゴリを削除しようとしたとき |

### 4.4 ドメインルール

- [ ] 金額は必ず正の数（Amount VO で保証）
- [ ] カテゴリ名は重複禁止（UseCase で検証）
- [ ] 使用中のカテゴリは削除できない（UseCase で検証）

```java
// 例: Amount VO のバリデーション
public class Amount {
    private final int value;

    public Amount(int value) {
        if (value <= 0) {
            throw new InvalidAmountException("金額は0より大きい必要があります");
        }
        this.value = value;
    }
}
```

### 4.5 Entity のフィールド

**Transaction:**

| フィールド | 型 | 説明 |
|-----------|-----|------|
| id | `TransactionId` | 収支ID |
| type | `TransactionType` | INCOME / EXPENSE |
| amount | `Amount` | 金額 |
| categoryId | `CategoryId` | カテゴリID |
| description | `String` | メモ（任意） |
| transactionDate | `LocalDate` | 取引日 |
| createdAt | `LocalDateTime` | 作成日時 |

**Category:**

| フィールド | 型 | 説明 |
|-----------|-----|------|
| id | `CategoryId` | カテゴリID |
| name | `CategoryName` | カテゴリ名 |

### 4.6 自己チェック

| チェック | 内容 |
|---------|------|
| [x] | VO のコンストラクタでバリデーションを書けた |
| [x] | Entity のフィールドに VO を使えた |
| [x] | ドメイン例外を自分で設計できた |

> **詳細**: [[Spring学習/kakeibo/phase04 ドメインモデル設計]] を参照

---

## Phase 5: ユースケース設計（Application 層） ✓
[[#進捗管理]]

> **目的**: 家計簿の処理フローを定義する。

### 5.1 ユースケース一覧

**Transaction:**

| クラス名 | 責務 |
|---------|------|
| `CreateTransactionUseCase` | 収支を登録する |
| `ListTransactionsUseCase` | 収支一覧を取得する（フィルタリング対応） |
| `GetTransactionUseCase` | 収支詳細を取得する |
| `UpdateTransactionUseCase` | 収支を更新する |
| `DeleteTransactionUseCase` | 収支を削除する |

**Category:**

| クラス名 | 責務 |
|---------|------|
| `CreateCategoryUseCase` | カテゴリを作成する |
| `ListCategoriesUseCase` | カテゴリ一覧を取得する |
| `DeleteCategoryUseCase` | カテゴリを削除する（使用中チェック付き） |

**集計:**

| クラス名 | 責務 |
|---------|------|
| `GetMonthlySummaryUseCase` | 月次集計を取得する |

### 5.2 設計原則（ToDo と同じ）

- [x] **Controller は薄く**: HTTP の受け取りと返却だけ
- [x] **業務判断は UseCase に集約**
- [x] **@Transactional は UseCase に付ける**

> **詳細**: [[Spring学習/kakeibo/phase05 ユースケース設計]] を参照
>
> ※ Repository インターフェースの設計は Phase 6 を参照。

### 5.3 カテゴリ削除の業務判断

```java
// DeleteCategoryUseCase の中で判断する
public void execute(Long categoryId) {
    // 1. カテゴリ存在チェック
    Category category = categoryRepository.findById(new CategoryId(categoryId))
        .orElseThrow(() -> new CategoryNotFoundException("カテゴリが見つかりません"));

    // 2. 使用中チェック（この収支がこのカテゴリを使っているか）
    boolean inUse = transactionRepository.existsByCategoryId(new CategoryId(categoryId));
    if (inUse) {
        throw new CategoryInUseException("使用中のカテゴリは削除できません");
    }

    // 3. 削除
    categoryRepository.deleteById(new CategoryId(categoryId));
}
```

> **ToDo との比較**: ToDo の `complete()` はエンティティ内の判断だった。カテゴリ削除は **他のエンティティとの関連** を見る必要があるため UseCase で判断する。

---

## Phase 6: Repository インターフェース設計 ✓
[[#進捗管理]]

> **目的**: データアクセスを抽象化する Repository インターフェースを domain 層に設計する。

### 6.1 Repository Interface

**TransactionRepository:**

```java
public interface TransactionRepository {
    Transaction save(Transaction transaction);
    Optional<Transaction> findById(TransactionId id);
    List<Transaction> findAll();
    void deleteById(TransactionId id);
    boolean existsByCategoryId(CategoryId categoryId);
}
```

> `findByConditions` は Phase 9 で追加予定（YAGNI）

**CategoryRepository:**

```java
public interface CategoryRepository {
    Category save(Category category);
    Optional<Category> findById(CategoryId id);
    List<Category> findAll();
    void deleteById(CategoryId id);
    boolean existsByName(CategoryName name);
}
```

### 6.2 ToDo との違い

| 観点 | ToDo | 家計簿 |
|------|------|--------|
| Repository 数 | 1 | 2 |
| 検索メソッド | `findAll()` のみ | `findByConditions()` を Phase 9 で追加予定 |
| 存在チェック | なし | `existsByCategoryId()`, `existsByName()` |

### 6.3 設計原則

- [x] Interface は `domain` パッケージに置く（DIP）
- [x] JPA / SQL などの DB 技術を domain に漏らさない
- [x] 引数・戻り値は VO / Domain Entity を使う（型安全）
- [x] 各 Repository に対して Impl を作る（Phase 7 で実装）

### 6.4 深堀りトピック

| トピック | 学んだこと |
|---------|-----------|
| DIP（依存性逆転） | infrastructure が domain に依存する（逆転） |
| Repository vs DAO | Repository はドメインの視点、DAO は DB の視点 |
| VO を引数に使う理由 | Long 同士の取り違えをコンパイル時に防げる |
| Optional を返す理由 | 「見つからない場合」の判断を UseCase に委ねる |
| existsByCategoryId の配置 | Transaction テーブルを検索するので TransactionRepository |
| テスタビリティ | インターフェースだからモックに差し替えてテストできる |

> **詳細**: [[Spring学習/kakeibo/phase06 Repositoryインターフェース設計]] を参照

---

## Phase 7: DB・JPA 実装（Infrastructure） ✓
[[#進捗管理]]

> **目的**: 2つのテーブルを持つ DB 実装を作る。

### 7.1 テーブル設計

**テーブル: `categories`**

| カラム | 型 | 説明 |
|--------|-----|------|
| id | BIGINT | 主キー（AUTO_INCREMENT） |
| name | VARCHAR(50) | カテゴリ名（NOT NULL, UNIQUE） |

**テーブル: `transactions`**

| カラム              | 型           | 説明                  |
| ---------------- | ----------- | ------------------- |
| id               | BIGINT      | 主キー（AUTO_INCREMENT） |
| type             | VARCHAR(20) | INCOME / EXPENSE    |
| amount           | INT         | 金額（NOT NULL）        |
| category_id      | BIGINT      | FK → categories.id  |
| description      | TEXT        | メモ（NULL OK）         |
| transaction_date | DATE        | 取引日（NOT NULL）       |
| created_at       | TIMESTAMP   | 作成日時                |

### 7.2 実装チェックリスト

**JPA Entity:**
- [x] `TransactionJpaEntity` を作成
- [x] `CategoryJpaEntity` を作成
- [x] Domain Entity と分離する

**Repository 実装:**
- [x] `TransactionRepositoryImpl` が `TransactionRepository` を実装
- [x] `CategoryRepositoryImpl` が `CategoryRepository` を実装
- [x] 変換メソッド（`toJpaEntity`, `toDomainEntity`）を Impl 内に書く

**フィルタリングの実装:**
- [ ] `findByConditions()` を JPA の動的クエリで実装（Phase 9 で対応予定）

> **詳細**: [[Spring学習/kakeibo/phase07 DB・JPA実装]] を参照

### 7.3 YAGNI 判断

| 項目 | 判断 | 理由 |
|------|------|------|
| `Amount` | VO で作成 | 「0 より大きい」というルールがある |
| `TransactionType` | enum で作成 | 値が固定（2値） |
| `description` | String で作成 | ビジネスルールなし |
| `transactionDate` | LocalDate で作成 | 型で保証される |

### 7.4 application.properties

```properties
spring.application.name=kakeibo

# H2 Database 設定
spring.datasource.url=jdbc:h2:mem:kakeibodb
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# JPA 設定
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true

# H2 Console
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

---

## Phase 8: API 実装（基本 CRUD） ✓
[[#進捗管理]]

> **目的**: HTTP リクエストを受け取り、レスポンスを返す入口を作る。

### 8.1 実装チェックリスト

**DTO:**
- [x] `CreateTransactionRequest` - 収支登録リクエスト
- [x] `UpdateTransactionRequest` - 収支更新リクエスト
- [x] `TransactionResponse` - 収支レスポンス
- [x] `CreateCategoryRequest` - カテゴリ作成リクエスト
- [x] `CategoryResponse` - カテゴリレスポンス
- [x] `ErrorResponse` - エラーレスポンス

**Controller:**
- [x] `CategoryController`（`/api/categories`）
- [x] `TransactionController`（`/api/transactions`）

**例外ハンドリング:**
- [x] `GlobalExceptionHandler` - 例外 → HTTP ステータス変換

**UseCase 修正:**
- [x] `CreateCategoryUseCase` - 引数を String に変更
- [x] `CreateTransactionUseCase` - 引数をプリミティブ型に変更
- [x] `UpdateTransactionUseCase` - 引数をプリミティブ型に変更

### 8.2 動作確認結果

| API | メソッド | 結果 |
|-----|---------|------|
| カテゴリ作成 | POST /api/categories | ✅ 201 |
| カテゴリ一覧 | GET /api/categories | ✅ 200 |
| 収支登録 | POST /api/transactions | ✅ 201 |
| 収支一覧 | GET /api/transactions | ✅ 200 |
| 収支詳細 | GET /api/transactions/{id} | ✅ 200 |
| 収支更新 | PUT /api/transactions/{id} | ✅ 200 |
| 収支削除 | DELETE /api/transactions/{id} | ✅ 204 |
| 存在しないリソース | GET /api/transactions/999 | ✅ 404 |
| バリデーションエラー | POST (amount=0) | ✅ 400 |

### 8.3 躓いたポイント・学んだこと

| トピック | 学んだこと |
|---------|-----------|
| DTO と VO の使い分け | DTO は Presentation 層でプリミティブ型、VO は Domain 層 |
| UseCase の引数 | プリミティブ型で受け取り、UseCase 内で VO に変換 |
| レイヤの依存方向 | Application → Presentation は NG（依存方向が逆） |
| id の渡し方 | パスパラメータから取得、Request Body には含めない |
| 例外ハンドリング | 自作例外は `getMessage()` のみ、Spring 例外は `getBindingResult()` も使える |
| 調べ方 | 分からないメソッドは Javadoc を見る |

> **詳細**: [[phase08_API実装]] を参照（シニア × ジュニアのやり取り、詰まった箇所を詳しく記録）

---

## Phase 9: パターン適用（集計・フィルタ） ✓
[[#進捗管理]]

> **目的**: Strategy / Factory パターンを家計簿ドメインに適用する。

### 9.0 公式ドキュメントを読む（実装前に必ず）

- [x] **Collectors の Javadoc** を確認する
  - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/Collectors.html
  - 確認: `groupingBy`, `summingInt` の使い方（集計で使う）

### 9.1 Factory パターン

**TransactionFactory:**
- [x] transactionDate 未指定時に `LocalDate.now()` を設定
- [x] createdAt の自動付与

> **ToDo との比較**: TodoFactory と同じ役割。「生成ルールを一箇所に集約」する。

### 9.2 Strategy パターン（集計戦略）

> **ToDo との違い**: ToDo ではソート戦略だった。家計簿では **集計戦略** に適用する。

**SummaryStrategy（集計戦略）:**

| 実装 | 説明 |
|------|------|
| `MonthlySummaryStrategy` | 月別の収入・支出・残高を集計 |
| `CategorySummaryStrategy` | カテゴリ別の支出を集計 |

**型安全な Result クラス:**

| クラス | 役割 |
|--------|------|
| `SummaryResult` | マーカーインターフェース |
| `MonthlySummaryResult` | 月次集計の結果 |
| `CategorySummaryResult` | カテゴリ別集計の結果 |

### 9.3 集計 API

```
GET /api/transactions/summary?year=2026&month=1              → 月次サマリー
GET /api/transactions/summary?year=2026&month=1&type=category → カテゴリ別サマリー
```

### 9.4 集計レスポンス DTO

```java
// 月次サマリー
public class MonthlySummaryResponse {
    private int year;
    private int month;
    private int totalIncome;
    private int totalExpense;
    private int balance;
}

// カテゴリ別サマリー
public class CategorySummaryResponse {
    private Long categoryId;
    private int totalAmount;
}
```

### 9.5 自己チェック

| チェック | 内容 |
|---------|------|
| [x] | Factory を自力で設計・実装できた |
| [x] | Strategy パターンを別のユースケース（集計）に適用できた |
| [x] | マーカーインターフェースで型安全にできた |
| [x] | Stream API（groupingBy, summingInt）を使えた |

### 9.6 躓いたポイント・学んだこと

| トピック | 学んだこと |
|---------|-----------|
| Factory vs UseCase | Factory は static、UseCase は instance（DI が必要） |
| マーカーインターフェース | `SummaryResult` で型安全に。`@SuppressWarnings` 不要に |
| instanceof | 型安全なキャストに使う |
| JSON キー名 | getter から生成される（getCategoryId → categoryId） |
| Stream API | `groupingBy` + `summingInt` でカテゴリ別集計 |

> **詳細**: [[Spring学習/kakeibo/phase09 パターン適用]] を参照

---

## Phase 10: テスト ✓
[[#進捗管理]]

> **目的**: ToDo で学んだテスト技法を家計簿でも実践する。

### 10.1 実装したテスト

**Domain テスト:**
- [x] `AmountTest` - 境界値テスト（null, 0, 1, -1, Integer.MAX_VALUE）
- [x] `CategoryNameTest` - 境界値テスト（null, 空文字, 1文字, 50文字, 51文字）

**UseCase テスト:**
- [x] `DeleteCategoryUseCaseTest` - モックを使ったテスト（3ケース）
  - カテゴリが存在しない → `CategoryNotFoundException`
  - カテゴリが使用中 → `CategoryInUseException`
  - 正常に削除 → `deleteById` が呼ばれる

**Controller テスト:**
- [x] `CategoryControllerTest` - HTTP 層のテスト
  - DELETE /api/categories/{id} → 204
  - POST /api/categories → 201

### 10.2 学んだこと

| テスト種類 | 対象 | 使う技術 |
|-----------|------|---------|
| Domain テスト | VO | JUnit 5（assertThrows, assertEquals） |
| UseCase テスト | ビジネスロジック | Mockito（mock, when, verify） |
| Controller テスト | HTTP 層 | @WebMvcTest, MockMvc, @MockitoBean |

### 10.3 モックの基本パターン

```java
// 1. モックを作成
CategoryRepository categoryRepository = mock(CategoryRepository.class);

// 2. モックの振る舞いを定義
when(categoryRepository.findById(any())).thenReturn(Optional.of(category));

// 3. テスト対象を実行
useCase.execute(1L);

// 4. 検証
verify(categoryRepository).deleteById(any());
```

### 10.4 躓いたポイント

| 問題 | 原因 | 解決策 |
|------|------|--------|
| `useCase` が null | インスタンスを作成していなかった | `@BeforeEach` で new |
| 正常系で例外発生 | `Optional.empty()` を返していた | `Optional.of(category)` を返す |
| 使用中テストが失敗 | `existsByCategoryId` の設定忘れ | `when(...).thenReturn(true)` |

### 10.5 自己チェック

| チェック | 内容 |
|---------|------|
| [x] | 境界値分析でテストケースを設計できた |
| [x] | モックを使って UseCase テストが書けた |
| [x] | `@WebMvcTest` で Controller テストが書けた |

> **詳細**: [[Spring学習/kakeibo/phase10 テスト]] を参照

---

## Phase 11: 仕上げ・振り返り
[[#進捗管理]]

> **目的**: ドキュメントを整え、ToDo との比較で学びを言語化する。

### 11.0 公式ドキュメントを読む（実装前に必ず）

- [ ] **SpringDoc OpenAPI** の公式を読む
  - https://springdoc.org/
  - 確認: pom.xml に何を追加すれば Swagger UI が使えるか？
- [ ] **OpenAPI 仕様** の概要を読む
  - https://swagger.io/specification/
  - 確認: API ドキュメントの自動生成がどういう仕組みか把握する

### 11.1 Swagger UI

- [ ] OpenAPI / Swagger で API ドキュメント自動生成
- [ ] http://localhost:8080/swagger-ui/index.html

### 11.2 README

- [ ] 起動方法
- [ ] テスト実行方法
- [ ] API 一覧
- [ ] Swagger UI / H2 Console の URL

### 11.3 ToDo との比較振り返り（最重要）

> この振り返りが「再実装」の最大の価値。

| 観点 | 質問 |
|------|------|
| **設計判断** | ToDo と同じ判断をした箇所はどこか？違う判断をした箇所はどこか？ |
| **パターン適用** | Strategy を「ソート」→「集計」に変えたとき、何が同じで何が違ったか？ |
| **VO の判断** | 新しい VO（Amount）を作る判断は正しかったか？ |
| **複数エンティティ** | Category を別エンティティにした設計判断は妥当か？ |
| **難しかった点** | フィルタリングの実装で苦労した点は？ |
| **手が覚えた点** | 見ないで書けるようになったコードはどれか？ |

### 11.4 設計判断ログ

| 判断ポイント | 理由 | ToDo との比較 |
|-------------|------|--------------|
| **DTO 使用** | DB 構造と API 形式を分離 | 同じ |
| **Repository Interface** | DB 変更・テスト時のモック対応 | 同じ |
| **Strategy パターン** | 集計処理を切り替え可能に | ToDo はソート、家計簿は集計 |
| **Factory パターン** | 生成ルールを一箇所に集約 | 同じ |
| **Category を別エンティティに** | 再利用・一覧表示のため | ToDo にはない新しい判断 |

### 11.5 次のステップへ

この家計簿アプリが完成したら：

```
✅ ToDo アプリ（1個目）
✅ 家計簿アプリ（2個目） ← 今ここ
→ 予約システム（3個目）← 次はこれ
→ 拡張（Docker、認証など）
→ DDD 深掘り
```

---

## 困ったときは

| 状況 | 対処 |
|------|------|
| ToDo と同じ構造が思い出せない | ToDo のコードを見てOK。ただし「見た」と記録する |
| 新しいドメインで設計に迷う | YAGNI を思い出す。今必要なものだけ作る |
| フィルタリングの実装が分からない | Phase 9 で段階的にやるので Phase 8 では基本 CRUD だけでOK |
| テストが書けない | まず Domain のテストだけ書く |
| アノテーションの意味が分からない | **まず公式ドキュメントを開く**。Qiita やブログより先に公式を見る習慣をつける |
| エラーメッセージが分からない | エラーメッセージをそのまま公式ドキュメントやGitHubのIssueで検索する |
| 公式が英語で読めない | コード例だけ先に見る。コードは万国共通。説明文は後から読む |

---

## 関連ドキュメント

- [[Spring学習/kakeibo/phase01 環境構築]]
- [[Spring学習/kakeibo/phase02 API設計]]
- [[Spring学習/kakeibo/phase03 レイヤ設計]]
- [[Spring学習/kakeibo/phase04 ドメインモデル設計]]
- [[Spring学習/kakeibo/phase05 ユースケース設計]]
- [[Spring学習/kakeibo/phase06 Repositoryインターフェース設計]]
- [[Spring学習/kakeibo/phase07 DB・JPA実装]]
- [[Spring学習/kakeibo/Phase08 API実装]]
- [[Spring学習/kakeibo/phase09 パターン適用]]

---

*このロードマップは実装しながら更新していく。ToDo との比較を常に意識すること。*
