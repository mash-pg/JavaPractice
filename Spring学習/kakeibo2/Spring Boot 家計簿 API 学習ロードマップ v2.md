# Spring Boot 家計簿 API 学習ロードマップ v2（理解重視 + Git 実務フロー）

> このロードマップは、kakeibo で学んだ内容を **理解を深めながら** 再実装し、**Git の実務フロー（feature ブランチ + PR レビュー）** も同時に習得することを目的としています。

---

## Claude への指示（次回の再開用）

> **次回このファイルを渡して「続きをお願い」と言えば、以下の指示に従って再開してくれます。**

```
あなたはシニアエンジニア（Claude）です。
私はジュニアエンジニア（学習者）です。

このロードマップの「進捗管理」セクションを確認し、
次の未完了 Phase からシニア × ジュニア形式で学習を進めてください。

【重要】作業場所について:
- 実装コード（Java）: C:\dev\kakeibo2
- ドキュメント（md）: C:\Users\user\Documents\JavaPractice\Spring学習\kakeibo2

【役割分担】（実務に近い形）
■ ジュニア（学習者 = ユーザー）の担当:
  - コードを実装する
  - git add & git commit & git push する
  - PR を作成する（下記の「PR 作成方法」参照）
  - レビュー指摘を修正してプッシュする
  - 「なぜそうするか」を説明する（理解確認）
  - Approve 後にマージする

■ シニア（Claude）の担当:
  - 課題と「なぜそうするか」を説明する
  - コードをレビューして指摘する
  - 修正確認後に Approve（承認）する
  - Phase 完了時に md ファイルを更新する

【ジュニア向け PR 作成方法】
■ 方法1: GitHub CLI（gh）を使う場合
  ```powershell
  # feature ブランチにいる状態で
  gh pr create --title "Phase○○: タイトル" --body "変更内容の説明"
  ```

■ 方法2: ブラウザから作成する場合
  1. GitHub のリポジトリページを開く
  2. 「Compare & pull request」ボタンをクリック
  3. タイトルと説明を入力
  4. 「Create pull request」をクリック

■ マージ方法（Approve 後）
  ```powershell
  # PR 番号を指定してマージ
  gh pr merge PR番号 --merge --delete-branch
  ```
  または GitHub のブラウザで「Merge pull request」ボタンをクリック

【Git フロー】
- 各 Phase は feature ブランチで作業する
- Phase 完了時に PR を作成し、シニア（Claude）がレビューする
- レビュー通過後に main へマージする

進め方:
1. ジュニア: feature ブランチを切る（例: feature/phase05-usecase）
2. シニア: 課題と「なぜそうするか」を説明する
3. ジュニア: 実装する（理解を確認しながら）
4. ジュニア: git add → git commit → git push する
5. ジュニア: PR を作成する（「レビューお願いします」）
6. シニア: コードレビュー（良い点・改善点を指摘）
7. ジュニア: 指摘を修正して push する
8. シニア: 再レビュー → Approve（承認）
9. ジュニア: main にマージする
10. ジュニア: ローカルで main に戻って pull する
11. シニア: Phase 完了時に md ファイルを更新する

ルール:
- 「なぜそうするか」を必ず説明する（理解重視）
- コードを書く前に「何をするか」を言語化させる
- 間違いは具体的に指摘し、自力で修正させる
- 各 Phase 完了時に、対応する phase○○.md を作成する
- Git コミットメッセージは日本語で分かりやすく
```

---

## 前提・ゴール

| 項目 | 内容 |
|------|------|
| **目的** | kakeibo で学んだことを「理解」レベルに引き上げる |
| **Git 学習** | feature ブランチ、PR、コードレビューの実務フロー |
| **kakeibo との違い** | 「見ながら写す」→「なぜそうするか説明できる」 |

---

## Git 戦略

### ブランチ構成

```
main（本番相当）
  │
  ├── feature/phase01-setup        ← Phase 1 の作業ブランチ
  ├── feature/phase02-api-design   ← Phase 2 の作業ブランチ
  ├── feature/phase03-layer        ← Phase 3 の作業ブランチ
  └── ...
```

### 各 Phase の Git フロー

```powershell
# === 1. ブランチ作成 ===
git checkout main
git pull origin main
git checkout -b feature/phase○○-xxxx

# === 2. 実装 → コミット → プッシュ ===
# （コードを実装する）
git status                              # 変更確認
git add .                               # ステージング
git commit -m "Phase○○: ○○を実装"       # コミット
git push -u origin feature/phase○○-xxxx # 初回プッシュ

# === 3. PR 作成 ===
gh pr create --title "Phase○○: タイトル" --body "説明"

# === 4. レビュー指摘があった場合 ===
# （コードを修正する）
git add .
git commit -m "レビュー指摘修正: ○○を改善"
git push                                # 2回目以降は push だけでOK

# === 5. Approve 後にマージ ===
gh pr merge PR番号 --merge --delete-branch

# === 6. main に戻る ===
git checkout main
git pull origin main
```

### ポイント

| 操作 | コマンド | 備考 |
|------|---------|------|
| 初回プッシュ | `git push -u origin ブランチ名` | `-u` でリモート追跡設定 |
| 2回目以降 | `git push` | `-u` 不要 |
| 変更を持ったままブランチ移動 | `git checkout -b 新ブランチ` | コミット前なら可能 |

### コミットメッセージ規約

```
Phase01: 環境構築を完了
Phase02: API設計を追加
Phase03: パッケージ構成を作成
Phase04: ドメインモデルを実装
...
```

### Claude による PR 作成・マージ方法

> **重要**: Claude の Bash 環境では `gh` コマンドにパスが通っていないため、以下の方法で実行する。

**PR 作成:**
```bash
powershell.exe -Command "& 'C:\Program Files\GitHub CLI\gh.exe' pr create --title 'Phase○○: タイトル' --body '説明文' --repo mash-pg/kakeibo2"
```

**PR マージ:**
```bash
powershell.exe -Command "& 'C:\Program Files\GitHub CLI\gh.exe' pr merge PR番号 --repo mash-pg/kakeibo2 --merge --delete-branch"
```

**PR 確認:**
```bash
powershell.exe -Command "& 'C:\Program Files\GitHub CLI\gh.exe' pr view PR番号 --repo mash-pg/kakeibo2"
```

**前提条件:**
- ユーザーが `gh auth login` で GitHub 認証済みであること
- GitHub CLI がインストールされていること（`C:\Program Files\GitHub CLI\gh.exe`）

---

## 進捗管理

- [x] Phase 1: 環境構築 [[#Phase 1 環境構築]] ✓
- [x] Phase 2: API 設計 [[#Phase 2 API 設計]] ✓
- [x] Phase 3: レイヤ設計 [[#Phase 3 レイヤ設計]] ✓
- [x] Phase 4: ドメインモデル設計 [[#Phase 4 ドメインモデル設計]] ✓
- [x] Phase 5: ユースケース設計 [[#Phase 5 ユースケース設計]] ✓
- [x] Phase 6: Repository インターフェース [[#Phase 6 Repository インターフェース]] ✓
- [x] Phase 7: DB・JPA 実装 [[#Phase 7 DB・JPA 実装]] ✓
- [x] Phase 8: API 実装 [[#Phase 8 API 実装]] ✓
- [ ] Phase 9: パターン適用 [[#Phase 9 パターン適用]]
- [ ] Phase 10: テスト [[#Phase 10 テスト]]
- [ ] Phase 11: 仕上げ [[#Phase 11 仕上げ]]

---

## 理解度チェックポイント

各 Phase で「なぜそうするか」を説明できるようになる。

| Phase | 説明できるようになること |
|-------|------------------------|
| 1 | Spring Boot の依存関係が何をしているか |
| 2 | REST API 設計のルール（なぜ動詞を使わないか等） |
| 3 | レイヤ分離の理由、依存の方向 |
| 4 | VO を使う理由、Entity との違い |
| 5 | UseCase の責務、なぜ Controller に書かないか |
| 6 | Repository Interface を domain に置く理由（DIP） |
| 7 | JPA Entity と Domain Entity を分ける理由 |
| 8 | DTO を使う理由、例外ハンドリングの仕組み |
| 9 | Strategy / Factory パターンを使う場面 |
| 10 | テストの種類と使い分け |
| 11 | 全体の設計判断を説明できる |

---

## Phase 1: 環境構築

> **ブランチ**: `feature/phase01-setup`

### 理解ポイント

| 質問 | 答えられるようになる |
|------|---------------------|
| Spring Web は何をするライブラリ？ | REST API（HTTP通信）を作るため |
| Spring Data JPA は何をする？ | DB操作を簡単にする（SQL を書かなくていい） |
| H2 Database はなぜ使う？ | 開発用。設定不要ですぐ使える |
| Lombok はなぜ使う？ | getter/setter 等の定型コードを省略 |
| Validation は何をする？ | 入力値チェック（@NotBlank 等） |

### 作業内容

1. Spring Initializr でプロジェクト作成
2. Eclipse にインポート
3. application.properties 設定
4. 起動確認

### Git 操作

```powershell
cd C:\dev\kakeibo2
git init
git add .
git commit -m "Phase01: プロジェクト初期化"
# GitHub にリポジトリ作成後
git remote add origin https://github.com/xxxxx/kakeibo2.git
git push -u origin main
```

---

## Phase 2: API 設計

> **ブランチ**: `feature/phase02-api-design`

### 理解ポイント

| 質問 | 答えられるようになる |
|------|---------------------|
| なぜ `/api/transactions/list` ではなく `/api/transactions` ？ | HTTP メソッドで操作を区別するから |
| なぜ POST は 201 を返す？ | 「リソースが作成された」ことを示すため |
| なぜ DELETE は 204 を返す？ | 「成功したが返すボディがない」ため |
| 409 Conflict はいつ使う？ | ビジネスルール違反（使用中カテゴリの削除等） |

### 作業内容

1. エンドポイント一覧を設計
2. ステータスコード方針を決める
3. ドキュメントに記録

---

## Phase 3: レイヤ設計

> **ブランチ**: `feature/phase03-layer`

### 理解ポイント

| 質問 | 答えられるようになる |
|------|---------------------|
| なぜレイヤを分ける？ | 変更の影響範囲を限定するため |
| なぜ domain が中心？ | ビジネスルールは最も変わりにくいから |
| presentation → application → domain の矢印は何を意味する？ | 依存の方向。上位が下位に依存する |
| infrastructure はなぜ domain に依存する？ | DIP（依存性逆転）。詳細が抽象に依存する |

### 作業内容

1. パッケージ構成を作成
2. 空のパッケージを用意

---

## Phase 4: ドメインモデル設計

> **ブランチ**: `feature/phase04-domain`

### 理解ポイント

| 質問 | 答えられるようになる |
|------|---------------------|
| なぜ Amount を int ではなく VO にする？ | 「0より大きい」というルールを型で保証するため |
| VO と Entity の違いは？ | VO は値で比較、Entity は ID で比較 |
| なぜコンストラクタでバリデーション？ | 不正な状態のオブジェクトを作らせないため |
| ドメイン例外を使う理由は？ | ビジネスルール違反を明確に表現するため |

### 作業内容

1. Value Object を実装（Amount, CategoryName, etc.）
2. Entity を実装（Transaction, Category）
3. ドメイン例外を実装

---

## Phase 5: ユースケース設計

> **ブランチ**: `feature/phase05-usecase`

### 理解ポイント

| 質問 | 答えられるようになる |
|------|---------------------|
| なぜ UseCase を作る？Controller に書かないの？ | 業務ロジックを再利用可能にするため |
| @Transactional をなぜ UseCase に付ける？ | 業務処理の単位でトランザクションを管理するため |
| UseCase は何を受け取って何を返す？ | プリミティブ型を受け取り、ドメインオブジェクトを返す |

### 作業内容

1. Transaction 用 UseCase を実装
2. Category 用 UseCase を実装

---

## Phase 6: Repository インターフェース

> **ブランチ**: `feature/phase06-repository`

### 理解ポイント

| 質問 | 答えられるようになる |
|------|---------------------|
| なぜ Interface を domain に置く？ | DIP。domain が infrastructure に依存しないため |
| なぜ VO を引数にする？（Long ではなく TransactionId） | 型安全。ID の取り違えをコンパイル時に防ぐ |
| Optional を返す理由は？ | 「見つからない」ケースの判断を呼び出し側に委ねる |

### 作業内容

1. TransactionRepository インターフェース作成
2. CategoryRepository インターフェース作成

---

## Phase 7: DB・JPA 実装

> **ブランチ**: `feature/phase07-jpa`

### 理解ポイント

| 質問 | 答えられるようになる |
|------|---------------------|
| なぜ JPA Entity と Domain Entity を分ける？ | DB の都合（カラム名等）を domain に漏らさないため |
| @Entity, @Id, @GeneratedValue は何をする？ | JPA にテーブル/主キー/自動採番を伝える |
| toDomainEntity / toJpaEntity は何をしている？ | 2つの Entity 間の変換 |

### 作業内容

1. JPA Entity を実装
2. JpaRepository を実装
3. Repository 実装クラスを作成

---

## Phase 8: API 実装

> **ブランチ**: `feature/phase08-api`

### 理解ポイント

| 質問 | 答えられるようになる |
|------|---------------------|
| なぜ DTO を使う？Entity をそのまま返さないの？ | API の形式と DB の形式を分離するため |
| @RestControllerAdvice は何をする？ | 全 Controller の例外を一箇所でハンドリング |
| ResponseEntity を使う理由は？ | ステータスコードを明示的に制御するため |

### 作業内容

1. DTO を実装
2. Controller を実装
3. GlobalExceptionHandler を実装

---

## Phase 9: パターン適用

> **ブランチ**: `feature/phase09-pattern`

### 理解ポイント

| 質問 | 答えられるようになる |
|------|---------------------|
| Factory パターンはいつ使う？ | 生成ルールが複雑な時、生成ロジックを一箇所に集約 |
| Strategy パターンはいつ使う？ | 処理を切り替えたい時（if 文の代わり） |
| Strategy を使うメリットは？ | 新しい処理を追加しても既存コードを変更しない（OCP） |

### 作業内容

1. TransactionFactory を実装
2. SummaryStrategy を実装
3. 集計 API を実装

---

## Phase 10: テスト

> **ブランチ**: `feature/phase10-test`

### 理解ポイント

| 質問 | 答えられるようになる |
|------|---------------------|
| Domain テストと UseCase テストの違いは？ | Domain は単体、UseCase はモックを使う |
| なぜモックを使う？ | DB なしでロジックだけをテストするため |
| @WebMvcTest は何をする？ | Controller だけを起動してテスト |
| 境界値テストとは？ | 境界（0, 1, 最大値等）を重点的にテスト |

### 作業内容

1. Domain テスト（VO のバリデーション）
2. UseCase テスト（モック使用）
3. Controller テスト（MockMvc 使用）

---

## Phase 11: 仕上げ

> **ブランチ**: `feature/phase11-finish`

### 作業内容

1. Swagger UI 設定
2. README 作成
3. 全体の振り返り

### 最終確認

| チェック | 内容 |
|---------|------|
| [ ] | 各 Phase の「理解ポイント」を全て説明できる |
| [ ] | Git フロー（branch → PR → review → merge）を体験した |
| [ ] | kakeibo と比べて「なぜそうするか」が言えるようになった |

---

## 関連ドキュメント

- [[Phase 01 環境構築]]
- [[Phase 02 API設計]]
- [[Phase 03 レイヤー]]
- [[Phase 04 ドメインモデル設計]]
- [[Phase 05 ユースケース設計]]
- [[Phase 06 Repository インターフェース]]
- [[Phase 07 DB・JPA 実装]]
- [[Phase 08 API実装]]

---

*このロードマップは実装しながら更新していく。「理解」と「Git 実務フロー」を意識すること。*
