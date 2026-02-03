# Phase 01: 環境構築（家計簿アプリ）

> **目的**: ToDo アプリと同じ環境構築を「自力で」再現する。どこまで見ないでできるか確認する。

---

## 概要

| 項目 | 内容 |
|------|------|
| JDK | 17（LTS） |
| IDE | Eclipse |
| Build Tool | Maven |
| Framework | Spring Boot 3.x |

---

## 1.1 開発環境の確認

### JDK 17

**シニア**: まずは環境が生きてるか確認しよう。以下を実行してみて。

```powershell
java -version
echo $env:JAVA_HOME
```

**ジュニア**: どっちも Java 17 になってます。

**シニア**: OK。ToDo のときにセットアップしたものがそのまま使えるね。

---

### Eclipse のプラグイン確認

**シニア**: Eclipse のプラグインも確認しておこう。Spring Tools と Lombok が入ってるか。

**ジュニア**: Spring Tools は Help → Eclipse Marketplace → Installed タブで確認できました。Lombok は…ここには出てこないです。

**シニア**: いい気づき。Lombok は Eclipse Marketplace 経由ではなく、**専用インストーラ**（`lombok.jar`）で入れるものだから、Marketplace の Installed には出ない。

**ジュニア**: じゃあどうやって確認するんですか？

**シニア**: **Help → About Eclipse IDE** を開いてみて。ダイアログの中に `Lombok v1.18.xx` という表記があれば入っている。

| プラグイン | 確認方法 |
|-----------|---------|
| **Spring Tools** | Help → Eclipse Marketplace → Installed タブ |
| **Lombok** | Help → About Eclipse IDE → `Lombok v1.18.xx` の表記 |

> **学び**: Lombok は Marketplace ではなく専用インストーラで導入する。Maven の依存に `lombok` があるだけでは Eclipse 上でコンパイルが通らない。

---

## 1.2 Spring Initializr でプロジェクト作成

**シニア**: https://start.spring.io/ でプロジェクトを作成してくれ。ToDo のときの設定を**見ないで**やってみよう。

| 設定項目 | 値 |
|---------|-----|
| Project | Maven |
| Language | Java |
| Spring Boot | 3.x（最新安定版） |
| Group | com.example |
| Artifact | kakeibo |
| Name | kakeibo |
| Packaging | Jar |
| Java | 17 |

**Dependencies:**

| ライブラリ | 用途 | なぜ必要？ |
|-----------|------|-----------|
| Spring Web | REST API を作る | Controller, HTTP 通信 |
| Validation | 入力値チェック | @NotBlank, @Size など |
| Spring Data JPA | DB 操作 | Repository, Entity |
| H2 Database | インメモリ DB | 開発用、設定不要で使える |
| Lombok | ボイラープレート削減 | @Getter, @Setter, @Slf4j |

**ジュニア**: Initializr の設定は迷わず選べました。

**シニア**: いいね。ToDo と Dependencies も同じだから、覚えていれば迷わないはず。

---

## 1.3 Eclipse へのインポート

1. 「Generate」ボタンをクリック → ZIP ファイルがダウンロードされる
2. ZIP を展開して `C:\dev\kakeibo` に配置
3. Eclipse で「File」→「Import」→「Existing Maven Projects」
4. `C:\dev\kakeibo` を選択 → 「Finish」

**ジュニア**: インポートの手順は覚えてました。

---

## 1.4 application.properties の設定

**ジュニア**: ここは…自力では書けなかったです。プロパティ名が思い出せなくて。

**シニア**: 正直で良い。でも安心していい。プロパティ名を暗記しているエンジニアはほぼいない。

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

### 実務での調べ方

**シニア**: 実務では3つの方法でプロパティを調べる。

| 方法 | 説明 |
|------|------|
| **Spring Boot 公式リファレンス** | https://docs.spring.io/spring-boot/appendix/application-properties/index.html に全プロパティ一覧がある |
| **IDE の補完機能** | Eclipse（Spring Tools 入り）で `spring.` と打つと候補が出る。**実務で一番使う方法** |
| **公式ガイド** | 「Spring Boot H2」等で検索すると公式ガイドが見つかる |

### 覚えるべきもの vs 調べるもの

| 区分 | 内容 | 例 |
|------|------|-----|
| **覚える** | 「何を設定する必要があるか」 | DB を使うなら datasource の設定が要る |
| **調べる** | 「プロパティ名の正確なスペル」 | `spring.datasource.url` の綴り |
| **調べる** | 「設定値の書き方」 | H2 の JDBC URL 形式 |

> **ポイント**: 「DB 接続には datasource の設定が必要」と分かっていれば、あとは IDE の補完か公式ドキュメントで書ける。

---

## 1.5 動作確認

### アプリの起動

**Eclipse の場合:**
1. `KakeiboApplication.java` を右クリック
2. Run As → Spring Boot App

**コマンドラインの場合:**
```powershell
cd C:\dev\kakeibo
.\mvnw.cmd spring-boot:run
```

### 起動確認

**方法1: コンソールログ**
```
Started KakeiboApplication in x.xxx seconds
```

**方法2: ブラウザ**
```
http://localhost:8080
```
Whitelabel Error Page が出れば OK。

### H2 Console の確認

1. ブラウザで http://localhost:8080/h2-console を開く
2. JDBC URL: `jdbc:h2:mem:kakeibodb` / User Name: `sa` / Password:（空欄）
3. Connect をクリック

**ジュニア**: 全部動きました。

---

## 1.6 プロジェクト構造

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

---

## 1.7 よくあるトラブル

| 状況 | 対処 |
|------|------|
| `Port 8080 is already in use` | ToDo アプリが起動中かも。停止するか `server.port=8081` を設定 |
| Maven のダウンロードが遅い | 初回は時間がかかる。2回目以降はキャッシュされて速い |
| Lombok が効かない | Help → About Eclipse IDE で Lombok の表記を確認 → なければ `lombok.jar` で再インストール |
| `JAVA_HOME` が古い | `echo $env:JAVA_HOME` で Java 17 のパスか確認 |

---

## チェックリスト

- [x] JDK 17 がインストールされている
- [x] JAVA_HOME が Java 17 を指している
- [x] Eclipse に Spring Tools + Lombok が入っている
- [x] Spring Initializr でプロジェクト作成済み
- [x] Eclipse にインポート済み
- [x] アプリ起動確認（Whitelabel Error Page）
- [x] H2 Console にアクセスできる

---

## 自己チェック

> ToDo のときは手順書を見ながらやった。今回は **見ないで** できたか？

| チェック | 内容 |
|---------|------|
| ✅ | Spring Initializr の設定を迷わず選べた |
| ▲ | application.properties を自力で書けた → **書けなかった。調べ方を学んだ** |
| ✅ | Eclipse へのインポート手順を覚えていた |
| ✅ | 起動エラーが出ても自分で対処できた |

---

## 次のステップ

- [x] Phase 02: API 設計 へ進む

---

*このドキュメントは学習中に詰まった点と解決方法を記録したものです。*
