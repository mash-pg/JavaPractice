# Phase 01: 環境構築

## 完了日
2026-02-09

## 作業内容

1. Spring Initializr でプロジェクト作成
2. GitHub リポジトリ作成（mash-pg/reservation）
3. Eclipse にインポート
4. H2 Database 設定
5. 起動確認

## 依存関係

- Spring Web
- Spring Data JPA
- H2 Database
- Lombok
- Validation

## 設定ファイル

### application.properties

```properties
spring.application.name=reservation

# H2 Database 設定
spring.datasource.url=jdbc:h2:mem:reservationdb
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

## Git コマンド

### ブランチ作成 & 切り替え
```bash
git checkout -b feature/phase01-setup
```

### コミット & プッシュ
```bash
git add .
git commit -m "Phase01: 環境構築"
git push -u origin feature/phase01-setup
```

### PR 作成
```bash
gh pr create --title "Phase01: 環境構築" --body "Spring Initializr でプロジェクト作成、H2設定、起動確認"
```

### PR マージ
```bash
gh pr merge 1 --merge --delete-branch
```

### ローカルを最新に
```bash
git checkout main
git pull
```

## 学んだこと

- コピペ時はプロジェクト固有の値（DB名など）を変更し忘れないこと
- Git フロー: feature ブランチ → PR → レビュー → マージ

## PR

- https://github.com/mash-pg/reservation/pull/1
