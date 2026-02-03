# Phase 01: 環境構築

> **目的**: 開発を始められる状態を作る。Spring Boot プロジェクトを立ち上げて動作確認する。

---

## 概要

| 項目 | 内容 |
|------|------|
| JDK | 17（LTS） |
| IDE | Eclipse |
| Build Tool | Maven |
| Framework | Spring Boot 3.x |

---

## 1.1 開発環境のセットアップ

### JDK 17 のインストール

**ジュニア**: Java をインストールしたいんですけど、どれを選べばいいですか？

**シニア**: **JDK 17** を選ぼう。理由は2つある。

| 理由 | 説明 |
|------|------|
| **LTS** | Long Term Support（長期サポート版）で安定している |
| **Spring Boot 3.x** | Spring Boot 3.x は Java 17 以上が必須 |

**おすすめのディストリビューション:**

| 名前 | 特徴 |
|------|------|
| **Eclipse Temurin** | 無料、商用利用 OK、広く使われている |
| Amazon Corretto | AWS が提供、無料 |
| Oracle JDK | 商用利用は有料ライセンスが必要な場合あり |

**ダウンロード先**: https://adoptium.net/

---

### インストール確認

```powershell
java -version
```

**期待する出力:**
```
openjdk version "17.0.x" ...
```

---

### JAVA_HOME の設定

**ジュニア**: `JAVA_HOME` って何ですか？

**シニア**: Java のインストール場所を指す環境変数。Maven や IDE が Java を見つけるために使う。

**設定方法（Windows）:**

1. 「環境変数」で検索 → 「システム環境変数の編集」
2. 「環境変数」ボタンをクリック
3. 「システム環境変数」の「新規」をクリック
4. 以下を入力:
   - 変数名: `JAVA_HOME`
   - 変数値: `C:\Program Files\Eclipse Adoptium\jdk-17.x.x-hotspot`
5. 「OK」で閉じる

**確認:**
```powershell
echo $env:JAVA_HOME
```

---

### 詰まったところ

#### Q: 複数の Java がインストールされている

**ジュニア**: `java -version` は 17 なのに、Maven が古い Java を使っています。

**シニア**: `JAVA_HOME` が古い Java を指している可能性がある。確認してみよう。

```powershell
echo $env:JAVA_HOME
```

**出力例:**
```
C:\Program Files\Eclipse Adoptium\jdk-11.0.24.8-hotspot\
```

**シニア**: Java 11 を指しているね。これを Java 17 に変更する必要がある。

**解決方法（一時的）:**
```powershell
$env:JAVA_HOME = "C:\Program Files\Eclipse Adoptium\jdk-17.0.17.10-hotspot"
```

**解決方法（恒久的）:**
システム環境変数で `JAVA_HOME` を Java 17 のパスに変更する。

---

## 1.2 IDE（Eclipse）のセットアップ

### Eclipse のインストール

**ダウンロード先**: https://www.eclipse.org/downloads/

**選ぶパッケージ**: **Eclipse IDE for Java Developers** または **Eclipse IDE for Enterprise Java and Web Developers**

---

### プラグインのインストール

**ジュニア**: Eclipse に何かプラグインは必要ですか？

**シニア**: 以下の2つがあると便利。

| プラグイン | 用途 |
|-----------|------|
| **Spring Tools** | Spring Boot の開発支援 |
| **Lombok** | Lombok アノテーションの認識 |

**インストール方法:**

1. Help → Eclipse Marketplace...
2. 「Spring Tools」で検索 → インストール
3. 「Lombok」で検索 → インストール
4. Eclipse を再起動

---

## 1.3 Spring Initializr でプロジェクト作成

### Spring Initializr とは

**ジュニア**: プロジェクトって手動で作るんですか？

**シニア**: **Spring Initializr** という公式ツールを使う。必要な設定を選ぶだけで、プロジェクトの雛形を自動生成してくれる。

**URL**: https://start.spring.io/

---

### プロジェクト設定

| 設定項目 | 値 |
|---------|-----|
| Project | Maven |
| Language | Java |
| Spring Boot | 3.5.10（または最新の安定版） |
| Group | com.example |
| Artifact | todo |
| Name | todo |
| Packaging | Jar |
| Java | 17 |

---

### Dependencies（依存ライブラリ）

**ジュニア**: どのライブラリを追加すればいいですか？

**シニア**: 以下の5つを追加しよう。

| ライブラリ | 用途 | なぜ必要？ |
|-----------|------|-----------|
| **Spring Web** | REST API を作る | Controller, HTTP 通信 |
| **Validation** | 入力値チェック | @NotBlank, @Size など |
| **Spring Data JPA** | DB 操作 | Repository, Entity |
| **H2 Database** | インメモリ DB | 開発用、設定不要で使える |
| **Lombok** | ボイラープレート削減 | @Getter, @Setter, @Slf4j |

---

### プロジェクトの生成とインポート

1. 「Generate」ボタンをクリック → ZIP ファイルがダウンロードされる
2. ZIP を展開して `C:\dev\todo` に配置
3. Eclipse で「File」→「Import」→「Existing Maven Projects」
4. `C:\dev\todo` を選択 → 「Finish」

---

## 1.4 動作確認

### アプリの起動

**Eclipse の場合:**
1. `TodoApplication.java` を右クリック
2. Run As → Spring Boot App

**コマンドラインの場合:**
```powershell
cd C:\dev\todo
.\mvnw.cmd spring-boot:run
```

---

### 起動確認

**ジュニア**: 起動したかどうか、どうやって確認しますか？

**シニア**: 2つの方法がある。

**方法1: コンソールログを確認**
```
Started TodoApplication in x.xxx seconds
```
このメッセージが出れば OK。

**方法2: ブラウザでアクセス**
```
http://localhost:8080
```
**Whitelabel Error Page** が表示されれば OK。

**ジュニア**: エラーページなのに OK なんですか？

**シニア**: そう。まだ何も作っていないから「表示するものがない」というエラー。**アプリ自体は正常に動いている**という証拠。

---

### 詰まったところ

#### Q: ポート 8080 が使用中

**ジュニア**: `Port 8080 is already in use` というエラーが出ました。

**シニア**: 別のアプリがポート 8080 を使っている。2つの解決方法がある。

**方法1: 使用中のプロセスを終了**
```powershell
# ポート 8080 を使っているプロセスを確認
netstat -ano | findstr :8080

# プロセスを終了（PID は上のコマンドで確認）
taskkill /PID <PID> /F
```

**方法2: 別のポートを使う**

`application.properties` に以下を追加:
```properties
server.port=8081
```

---

#### Q: Maven のダウンロードが遅い

**ジュニア**: 初回起動がすごく遅いです。

**シニア**: 初回は Maven が依存ライブラリをダウンロードするから時間がかかる。2回目以降はキャッシュされるので速くなる。

気長に待とう。

---

## 1.5 プロジェクト構造の確認

```
todo/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/todo/
│   │   │       └── TodoApplication.java  ← エントリーポイント
│   │   └── resources/
│   │       └── application.properties    ← 設定ファイル
│   └── test/
│       └── java/
│           └── com/example/todo/
│               └── TodoApplicationTests.java  ← テスト
├── pom.xml  ← Maven の設定ファイル
└── mvnw.cmd ← Maven Wrapper（Maven がなくても使える）
```

---

## まとめ

| 項目 | 内容 |
|------|------|
| JDK | 17（Temurin 推奨） |
| JAVA_HOME | Java 17 のパスを設定 |
| IDE | Eclipse + Spring Tools + Lombok |
| プロジェクト作成 | Spring Initializr |
| 依存ライブラリ | Web, Validation, JPA, H2, Lombok |
| 動作確認 | localhost:8080 で Whitelabel Error Page |

---

## チェックリスト

- [x] JDK 17 をインストール
- [x] JAVA_HOME を設定
- [x] Eclipse をセットアップ
- [x] Spring Initializr でプロジェクト作成
- [x] アプリ起動確認（Whitelabel Error Page）

---

## 次のステップ

- [ ] Phase 02: API 設計 へ進む

---

*このドキュメントは学習中に詰まった点と解決方法を記録したものです。*
