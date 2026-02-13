# Phase 5 & 6: ユースケース設計 + Repository インターフェース

> **完了日**: 2026-02-13
> **ブランチ**: `feature/phase05-usecase`

---

## 学んだこと

### @Service と @Transactional の役割

| アノテーション | 役割 |
|---------------|------|
| `@Service` | DI コンテナに Bean として登録する |
| `@Transactional` | トランザクション管理（全部成功 or 全部ロールバック） |

### インターフェースに依存する理由（疎結合）

```
UseCase → ReservationRepository（インターフェース）
              ↑
    ┌─────────┴─────────┐
    │                   │
JpaRepository      MyBatisRepository
  (Phase 7)           (Phase 9)
```

**メリット**: 実装（JPA → MyBatis）を切り替えても **UseCase のコードは変更不要**

---

## 作成したファイル

### Repository インターフェース（Phase 6）

| ファイル | メソッド |
|---------|---------|
| `domain/repository/ReservationRepository.java` | save, findById, findAll, findByResourceId |
| `domain/repository/ResourceRepository.java` | save, findById, findAll |

### UseCase（Phase 5）

| ファイル | メソッド |
|---------|---------|
| `application/ResourceUseCase.java` | create, findAll, findById |
| `application/ReservationUseCase.java` | create, confirm, cancel, complete, findAll, findByResourceId |

### 例外クラス

| ファイル | 用途 |
|---------|------|
| `domain/exception/ResourceNotFoundException.java` | リソースが見つからない |
| `domain/exception/ReservationNotFoundException.java` | 予約が見つからない |

---

## 躓いたポイントと解決

### 1. Optional の orElseThrow() の使い方

**間違い**: Optional をそのまま返していた
```java
// NG: 呼び出し側が毎回 get() や orElseThrow() をしないといけない
public Optional<Resource> findById(ResourceId id) {
    return repository.findById(id);
}
```

**正解**: orElseThrow() で値を取り出し、なければ例外を投げる
```java
// OK: 見つからなければ例外、見つかれば Resource を返す
public Resource findById(ResourceId id) {
    return repository.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("リソースが見つかりません"));
}
```

### 2. orElseThrow() の結果を変数に受け取る

**間違い**: 取得した結果を捨てて、新しいオブジェクトを作っていた
```java
// NG: orElseThrow() の結果を使っていない！
public void confirm(ReservationId id) {
    repository.findById(id).orElseThrow(...);  // 結果を捨てている
    Reservation reservation = new Reservation(id, null, null, null, null);  // 新しく作っている
    reservation.confirm();
}
```

**正解**: 取得した予約に対して操作する
```java
// OK: DBから取得した予約を操作している
public Reservation confirm(ReservationId id) {
    Reservation reservation = repository.findById(id)
        .orElseThrow(() -> new ReservationNotFoundException("予約が見つかりません"));
    reservation.confirm();
    return repository.save(reservation);
}
```

### 3. 戻り値の型による処理の違い

| 戻り値 | 例外を投げる？ | 理由 |
|--------|---------------|------|
| `Optional<T>` → `T` | 見つからなければ投げる | 1件取得は「ない」がエラー |
| `List<T>` | 投げない | 0件も正常な結果 |

```java
// Optional の場合: 見つからなければ例外
public Resource findById(ResourceId id) {
    return repository.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("見つかりません"));
}

// List の場合: そのまま返す（0件も正常）
public List<Reservation> findByResourceId(ResourceId resourceId) {
    return repository.findByResourceId(resourceId);
}
```

### 4. 実装クラスがないと Bean が見つからないエラー

**エラー内容**:
```
Parameter 0 of constructor in ReservationUseCase required a bean of type
'ReservationRepository' that could not be found.
```

**原因**: インターフェースだけ作っても、実装クラスがないと Spring は Bean を作れない

**解決**: Phase 7 で JPA 実装クラスを作成すれば解消

```
UseCase → Repository（インターフェース）
              ↑
         実装クラスが必要！（@Repository をつける）
```

---

## 状態遷移メソッドのパターン（重要）

confirm, cancel, complete は全て同じパターン：

```java
public Reservation confirm(ReservationId id) {
    // 1. DBから取得（なければ例外）
    Reservation reservation = repository.findById(id)
        .orElseThrow(() -> new ReservationNotFoundException("予約が見つかりません"));

    // 2. Entity のメソッドで状態遷移
    reservation.confirm();

    // 3. 保存して返す
    return repository.save(reservation);
}
```

---

## 次のフェーズ

**Phase 7: DB・JPA 実装（H2）** で Repository の実装クラスを作成する。
これで「Bean が見つからない」エラーが解消される。
