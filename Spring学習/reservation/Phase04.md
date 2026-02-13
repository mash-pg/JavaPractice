# Phase 04 ドメインモデル設計

> **目的**: 予約システムのビジネスルールをコードで表現する

---

## 実装内容

### Value Object（値オブジェクト）

| VO | ルール | ファイル |
|----|--------|---------|
| `TimeSlot` | 開始 < 終了、null禁止 | `domain/valueobject/TimeSlot.java` |
| `ResourceId` | null禁止、0以下禁止 | `domain/valueobject/ResourceId.java` |
| `ReservationId` | null禁止、0以下禁止 | `domain/valueobject/ReservationId.java` |
| `UserId` | null禁止、0以下禁止 | `domain/valueobject/UserId.java` |
| `ResourceName` | null禁止、空白禁止 | `domain/valueobject/ResourceName.java` |
| `Email` | null禁止、空白禁止、形式チェック | `domain/valueobject/Email.java` |
| `ReservationStatus` | enum（PENDING, CONFIRMED, CANCELLED, COMPLETED） | `domain/valueobject/ReservationStatus.java` |

### Entity（エンティティ）

| Entity | フィールド | ファイル |
|--------|-----------|---------|
| `Resource` | id, name | `domain/entity/Resource.java` |
| `Reservation` | id, resourceId, userId, timeSlot, status | `domain/entity/Reservation.java` |

### Exception（例外）

| 例外 | 用途 |
|------|------|
| `InvalidTimeSlotException` | 時間帯が不正（終了 < 開始、null） |
| `InvalidResourceIdException` | ResourceIdが不正（null、0以下） |
| `InvalidReservationIdException` | ReservationIdが不正（null、0以下） |
| `InvalidUserIdException` | UserIdが不正（null、0以下） |
| `InvalidResourceNameException` | ResourceNameが不正（null、空白） |
| `InvalidEmailException` | メールアドレスが不正（null、空白、形式） |
| `InvalidStateTransitionException` | 不正な状態遷移 |

---

## 状態遷移メソッド

Reservation エンティティに状態遷移メソッドを実装。

```
PENDING → confirm() → CONFIRMED
CONFIRMED → cancel() → CANCELLED
CONFIRMED → complete() → COMPLETED
```

```java
public void confirm() {
    if(this.status != ReservationStatus.PENDING) {
        throw new InvalidStateTransitionException("ReservationStatusがPENDINGでないです");
    }
    this.status = ReservationStatus.CONFIRMED;
}
```

---

## 理解ポイント

### null チェックは最初にやる

```java
// NG: null だと isBefore() で NullPointerException
if(endTime.isBefore(startTime)) { ... }
if(startTime == null || endTime == null) { ... }

// OK: null チェックを先にやる
if(startTime == null || endTime == null) { ... }
if(endTime.isBefore(startTime)) { ... }
```

### enum は static アクセス

```java
// NG: 警告が出る
status.PENDING

// OK: クラス名で参照
ReservationStatus.PENDING
```

### isEmpty() vs isBlank()

| メソッド | `""` | `"   "` | `"会議室A"` |
|---------|------|---------|------------|
| `isEmpty()` | true | false | false |
| `isBlank()` | true | true | false |

空白のみも弾きたい場合は `isBlank()` を使う。

### Email バリデーション（ライブラリ）

```java
// 古い（deprecated）
import org.apache.commons.validator.EmailValidator;

// 新しい（routines パッケージ）
import org.apache.commons.validator.routines.EmailValidator;
```

---

## 躓いたポイント

| 問題 | 解決 |
|------|------|
| null チェック前にメソッド呼び出し | null チェックを最初に |
| コピペでメッセージ修正忘れ | コピペ後は必ずメッセージ確認 |
| `status.PENDING` で警告 | `ReservationStatus.PENDING` で static アクセス |
| `==` と `!=` の間違い | 「〜でない」なら `!=` |
| EmailValidator が deprecated | routines パッケージの方を使う |

---

## Git 操作

```
ブランチ: feature/phase04-domain
PR: #4
マージ: ✅ 完了（2026-02-12）
```

---

## 関連ドキュメント

- [[Spring Boot 予約システム API 学習ロードマップ]]
