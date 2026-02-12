# Spring Boot 予約システム API 学習ロードマップ（実践重視 + エラー調査力）

> このロードマップは、kakeibo2 で学んだ設計パターンを **より複雑なドメイン** で再実装し、
> **Stream API と拡張 for 文の両方**、**エラー調査力** を身につけることを目的としています。

---

## Claude への指示（次回の再開用）

> **次回このファイルを渡して「続きをお願い」と言えば、以下の指示に従って再開してくれます。**

```
あなたはシニアエンジニア（Claude）です。
私はジュニアエンジニア（学習者）です。

このロードマップの「進捗管理」セクションを確認し、
次の未完了 Phase からシニア × ジュニア形式で学習を進めてください。

【重要】作業場所について:
- 実装コード（Java）: C:\dev\reservation
- ドキュメント（md）: C:\Users\user\Documents\JavaPractice\Spring学習\reservation

【役割分担】（実務に近い形）
■ ジュニア（学習者 = ユーザー）の担当:
  - コードを実装する
  - git add & git commit & git push する
  - PR を作成する
  - レビュー指摘を修正してプッシュする
  - 「なぜそうするか」を説明する（理解確認）
  - Approve 後にマージする

■ シニア（Claude）の担当:
  - 課題と「なぜそうするか」を説明する
  - コードをレビューして指摘する
  - 修正確認後に Approve（承認）する
  - Phase 完了時に md ファイルを更新する
  - わざとエラーを発生させる課題を出す（Phase 10）

【学習方針】
■ Stream API と拡張 for 文の両方を練習:
  - 同じ処理を両方の書き方で実装する
  - どちらが読みやすいか、デバッグしやすいかを体感する

■ エラー調査力を鍛える:
  - わざとエラーを発生させる
  - スタックトレースの読み方を学ぶ
  - デバッグの手順を身につける

【Git フロー】
- 各 Phase は feature ブランチで作業する
- Phase 完了時に PR を作成し、シニア（Claude）がレビューする
- レビュー通過後に main へマージする

ルール:
- 「なぜそうするか」を必ず説明する（理解重視）
- 間違いは具体的に指摘し、自力で修正させる
- 各 Phase 完了時に、対応する phase○○.md を作成する
```

---

## 前提・ゴール

| 項目 | 内容 |
|------|------|
| **目的** | kakeibo2 の知識を複雑なドメインで応用する |
| **複雑度** | ★★★（日時の扱い、状態遷移が複雑） |
| **新しく学ぶこと** | 状態遷移、日時バリデーション、エラー調査 |

---

## kakeibo2 との違い

| 観点 | kakeibo2 | 予約システム |
|------|----------|-------------|
| 複雑度 | ★★☆ | ★★★ |
| 状態管理 | なし | 予約状態（PENDING→CONFIRMED→COMPLETED） |
| 日時 | 単純な日付 | 日時 + 時間帯 + 重複チェック |
| エンティティ数 | 2（Transaction, Category） | 3以上（Reservation, Resource, TimeSlot） |
| 学習の重点 | 設計パターンの理解 | 実践力 + エラー調査力 |

---

## 予約システムの概要

### ドメイン

**会議室予約システム** を想定

| エンティティ | 説明 |
|-------------|------|
| Resource（リソース） | 予約対象（会議室 A, B, C など） |
| Reservation（予約） | 誰が、どの部屋を、いつ予約したか |
| TimeSlot（時間帯） | 開始時刻〜終了時刻 |

### 予約状態（ステート）

```
PENDING（仮予約）
    ↓ confirm()
CONFIRMED（確定）
    ↓ complete() or cancel()
COMPLETED（完了） / CANCELLED（キャンセル）
```

### ビジネスルール

- 過去の日時は予約不可
- 同じリソース・同じ時間帯の重複予約は不可
- キャンセルは予約日の前日まで
- 確定済みの予約のみ完了にできる

---

## 学習方針

### Stream API と拡張 for 文の両方を練習

```java
// Stream API（モダン）
List<Reservation> confirmed = reservations.stream()
    .filter(r -> r.getStatus() == ReservationStatus.CONFIRMED)
    .collect(Collectors.toList());

// 拡張 for 文（クラシック）
List<Reservation> confirmed = new ArrayList<>();
for (Reservation r : reservations) {
    if (r.getStatus() == ReservationStatus.CONFIRMED) {
        confirmed.add(r);
    }
}
```

**両方書けるようにする！**

### エラー調査力を鍛える

Phase 10 で意図的にエラーを発生させて調査する練習をする。

| エラーの種類 | 学ぶこと |
|------------|---------|
| NullPointerException | null チェックの重要性 |
| LazyInitializationException | JPA の遅延ロード問題 |
| DataIntegrityViolationException | 一意制約違反 |
| OptimisticLockingFailureException | 楽観的ロック |

---

## 進捗管理

- [x] Phase 1: 環境構築 [[#Phase 1 環境構築]] ✓ 2026-02-09
- [x] Phase 2: API 設計 [[#Phase 2 API 設計]] ✓ 2026-02-10
- [x] Phase 3: レイヤ設計 [[#Phase 3 レイヤ設計]] ✓ 2026-02-11
- [ ] Phase 4: ドメインモデル設計 [[#Phase 4 ドメインモデル設計]] 🔄 進行中
- [ ] Phase 5: ユースケース設計 [[#Phase 5 ユースケース設計]]
- [ ] Phase 6: Repository インターフェース [[#Phase 6 Repository インターフェース]]
- [ ] Phase 7: DB・JPA 実装（H2） [[#Phase 7 DB・JPA 実装]]
- [ ] Phase 8: API 実装 [[#Phase 8 API 実装]]
- [ ] Phase 9: MyBatis 実装（SQL練習） [[#Phase 9 MyBatis 実装]]
- [ ] Phase 10: PostgreSQL 切り替え [[#Phase 10 PostgreSQL 切り替え]]
- [ ] Phase 11: 認証・認可（Spring Security + JWT） [[#Phase 11 認証・認可]]
- [ ] Phase 12: パターン適用（State パターン） [[#Phase 12 パターン適用]]
- [ ] Phase 13: エラー調査・デバッグ練習 [[#Phase 13 エラー調査・デバッグ練習]]
- [ ] Phase 14: テスト [[#Phase 14 テスト]]
- [ ] Phase 15: 仕上げ [[#Phase 15 仕上げ]]

---

## Phase 1: 環境構築

> **ブランチ**: `feature/phase01-setup`

### 作業内容

1. Spring Initializr でプロジェクト作成
2. GitHub リポジトリ作成（reservation）
3. Eclipse にインポート
4. 起動確認

### 依存関係

- Spring Web
- Spring Data JPA
- H2 Database
- Lombok
- Validation

---

## Phase 2: API 設計

> **ブランチ**: `feature/phase02-api-design`

### エンドポイント設計

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

### ステータスコード

| 操作  | 成功  | エラー                     |
| --- | --- | ----------------------- |
| 作成  | 201 | 400（バリデーション）, 409（重複）   |
| 更新  | 200 | 404（存在しない）, 409（状態遷移不可） |
| 削除  | 204 | 404                     |
|     |     |                         |

---

## Phase 3: レイヤ設計

> **ブランチ**: `feature/phase03-layer`

kakeibo2 と同じ4層構造。

```
presentation  → Controller, DTO
application   → UseCase（@Transactional）
domain        → Entity, VO, Repository Interface
infrastructure → JPA Entity, Repository 実装
```

---

## Phase 4: ドメインモデル設計

> **ブランチ**: `feature/phase04-domain`

### 理解ポイント

| 質問 | 答えられるようになる |
|------|---------------------|
| なぜ ReservationStatus を enum にする？ | 状態遷移を型で制限するため |
| TimeSlot を VO にする理由は？ | 開始 < 終了というルールを型で保証 |
| 重複チェックはどこでやる？ | Domain Service または UseCase |

### 作成するクラス

**Value Object:**
- ResourceId
- ResourceName
- ReservationId
- UserId
- Email
- TimeSlot（startTime, endTime）
- ReservationStatus（enum: PENDING, CONFIRMED, CANCELLED, COMPLETED）

**Entity:**
- Resource（会議室などのリソース）
- Reservation（予約）
- User（ユーザー）※Phase 9 で詳細実装

**Domain Exception:**
- InvalidTimeSlotException（終了が開始より前）
- PastDateReservationException（過去日時の予約）
- DuplicateReservationException（重複予約）
- InvalidStateTransitionException（不正な状態遷移）

---

## Phase 5: ユースケース設計

> **ブランチ**: `feature/phase05-usecase`

### 作成するユースケース

- ResourceUseCase
- ReservationUseCase
  - create（予約作成）
  - confirm（確定）
  - cancel（キャンセル）
  - complete（完了）
  - findByResourceId（リソース別取得）

---

## Phase 6: Repository インターフェース

> **ブランチ**: `feature/phase06-repository`

### 追加クエリ

```java
// 重複チェック用
List<Reservation> findByResourceIdAndTimeSlotOverlapping(
    ResourceId resourceId,
    LocalDateTime start,
    LocalDateTime end
);
```

---

## Phase 7: DB・JPA 実装（H2）

> **ブランチ**: `feature/phase07-jpa`

### 理解ポイント

- JPA での日時の扱い（LocalDateTime）
- カスタムクエリ（@Query）
- 時間帯の重複判定 SQL

### 使用DB

- **H2 Database**（開発用・インメモリ）

---

## Phase 8: API 実装

> **ブランチ**: `feature/phase08-api`

### 理解ポイント

- 状態遷移 API（confirm, cancel, complete）
- エラーレスポンスの設計
- DTO の設計

### 作成するファイル

**DTO:**
- ReservationRequest
- ReservationResponse
- ResourceRequest
- ResourceResponse

**Controller:**
- ReservationController
- ResourceController

**例外ハンドリング:**
- GlobalExceptionHandler

---

## Phase 9: MyBatis 実装（SQL練習）

> **ブランチ**: `feature/phase09-mybatis`

### 理解ポイント

| 質問 | 答えられるようになる |
|------|---------------------|
| JPA と MyBatis の違いは？ | JPA=オブジェクト中心、MyBatis=SQL中心 |
| なぜ MyBatis を学ぶ？ | 日本の現場（特にSIer）で需要が高い |
| XML で SQL を書くメリットは？ | SQL を直接書けるので細かい制御が可能 |

### 作成するファイル

```
src/main/resources/mapper/
  ├── ReservationMapper.xml
  └── ResourceMapper.xml
```

### MyBatis の基本構造

**Mapper インターフェース:**
```java
@Mapper
public interface ReservationMapper {
    List<Reservation> findAll();
    Reservation findById(Long id);
    void insert(Reservation reservation);
    void update(Reservation reservation);
    void delete(Long id);
}
```

**XML マッパー:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.reservation.mapper.ReservationMapper">

  <select id="findAll" resultType="Reservation">
    SELECT id, resource_id, user_id, start_time, end_time, status
    FROM reservations
  </select>

  <select id="findById" resultType="Reservation">
    SELECT * FROM reservations WHERE id = #{id}
  </select>

  <insert id="insert">
    INSERT INTO reservations (resource_id, user_id, start_time, end_time, status)
    VALUES (#{resourceId}, #{userId}, #{startTime}, #{endTime}, #{status})
  </insert>

</mapper>
```

### 学ぶ SQL

| SQL | 用途 |
|-----|------|
| SELECT + JOIN | リソースと予約を結合 |
| WHERE + AND/OR | 複数条件での絞り込み |
| BETWEEN | 日時範囲の検索 |
| GROUP BY | 集計 |
| サブクエリ | 重複チェック |

### JPA と MyBatis の使い分け

| 観点 | JPA | MyBatis |
|------|-----|---------|
| 学習コスト | 高い | 低い（SQLが書ければOK） |
| 細かいSQL制御 | 難しい | 簡単 |
| 複雑なクエリ | @Query + JPQL | XML で自由に書ける |
| 日本の現場 | 新規プロジェクト | 既存システム・SIer |

---

## Phase 10: PostgreSQL 切り替え

> **ブランチ**: `feature/phase10-postgresql`

### 理解ポイント

| 質問 | 答えられるようになる |
|------|---------------------|
| なぜ H2 から切り替える？ | H2 は開発用、本番では PostgreSQL/MySQL |
| Docker を使う理由は？ | 環境構築が簡単、チームで共有しやすい |
| application.properties の切り替え方は？ | プロファイル（dev/prod）で分ける |

### Docker で PostgreSQL 起動

```yaml
# docker-compose.yml
version: '3.8'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: reservation
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
```

```bash
docker-compose up -d
```

### application.properties（プロファイル分け）

**application-dev.properties（開発用・H2）:**
```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
```

**application-prod.properties（本番用・PostgreSQL）:**
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/reservation
spring.datasource.username=user
spring.datasource.password=password
spring.datasource.driver-class-name=org.postgresql.Driver
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
```

### 起動時のプロファイル指定

```bash
# 開発（H2）
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev

# 本番（PostgreSQL）
./mvnw spring-boot:run -Dspring-boot.run.profiles=prod
```

### 学ぶこと

- Docker の基本操作
- docker-compose の使い方
- Spring Boot プロファイル機能
- PostgreSQL と H2 の SQL 方言の違い

---

## Phase 11: 認証・認可（Spring Security + JWT）

> **ブランチ**: `feature/phase11-auth`

### 理解ポイント

| 質問 | 答えられるようになる |
|------|---------------------|
| 認証と認可の違いは？ | 認証=誰か確認、認可=何ができるか確認 |
| JWT とは？ | トークンベースの認証方式 |
| なぜセッションではなく JWT？ | ステートレス、スケーラブル |

### 作成するクラス

**Entity:**
- User（ユーザー）
- Role（権限: ADMIN, USER）

**Security:**
- JwtTokenProvider（トークン生成・検証）
- JwtAuthenticationFilter（リクエストごとのトークン検証）
- SecurityConfig（Spring Security 設定）

**DTO:**
- LoginRequest（ログイン）
- SignupRequest（ユーザー登録）
- AuthResponse（トークン返却）

### エンドポイント

| メソッド | パス | 説明 | 認証 |
|---------|------|------|------|
| POST | /api/auth/signup | ユーザー登録 | 不要 |
| POST | /api/auth/login | ログイン | 不要 |
| GET | /api/reservations | 予約一覧 | 必要 |
| POST | /api/reservations | 予約作成 | 必要 |

### 学ぶこと

- Spring Security の基本設定
- JWT トークンの生成と検証
- PasswordEncoder（パスワードハッシュ化）
- @PreAuthorize（メソッドレベルの認可）
- SecurityContext からユーザー情報取得

---

## Phase 12: パターン適用（State パターン）

> **ブランチ**: `feature/phase12-pattern`

### 理解ポイント

| 質問 | 答えられるようになる |
|------|---------------------|
| State パターンはいつ使う？ | 状態によって振る舞いが変わる時 |
| if-else より State パターンが良い理由は？ | 状態追加時に既存コードを変更しない |

### 作成するクラス

- ReservationState（インターフェース）
- PendingState
- ConfirmedState
- CancelledState
- CompletedState

---

## Phase 13: エラー調査・デバッグ練習

> **ブランチ**: `feature/phase13-debug`

### 意図的にエラーを発生させて調査する

| エラー | 発生させる方法 | 学ぶこと |
|--------|---------------|---------|
| NullPointerException | Optional を無視 | スタックトレースの読み方 |
| LazyInitializationException | トランザクション外でアクセス | JPA の仕組み |
| ConstraintViolationException | 一意制約違反 | DB エラーのハンドリング |

### デバッグ手順

1. エラーメッセージを読む
2. スタックトレースから発生箇所を特定
3. 原因を仮説立て
4. ログを追加して検証
5. 修正 → テスト

---

## Phase 14: テスト

> **ブランチ**: `feature/phase14-test`

### テスト内容

- Domain テスト（TimeSlot, ReservationStatus）
- UseCase テスト（状態遷移、重複チェック）
- Controller テスト
- Security テスト（認証が必要な API）
- MyBatis テスト（SQL の動作確認）

---

## Phase 15: 仕上げ

> **ブランチ**: `feature/phase15-finish`

### 作業内容

1. Swagger UI 設定
2. README 作成
3. 全体の振り返り

---

## 関連ドキュメント

（Phase 完了時に追加していく）

---

*このロードマップは実装しながら更新していく。「Stream と for 文の両方」「エラー調査力」を意識すること。*
