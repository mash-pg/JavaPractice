
## 環境

| 項目         | 内容              |
| ---------- | --------------- |
| JDK        | 17（LTS）         |
| IDE        | Eclipse         |
| Build Tool | Maven           |
| Framework  | Spring Boot 3.x |

## githubにて公開

githubにてmainでpushしている
これからは、githubでブランチを切る。
```
1. git checkout main
2. git pull origin main
3. git checkout -b feature/phase○○-xxxx
4. （実装）
5. git add .
6. git commit -m "Phase○○: ○○を実装"
7. git push -u origin feature/phase○○-xxxx
8. PR 作成（GitHub）
9. シニア（Claude）がレビュー
10. 修正があれば対応してコミット
11. レビュー通過 → main にマージ
12. ローカルで main に戻って pull
    
```

## Springを導入

https://start.spring.io/

### 依存関係一覧

| ライブラリ | 用途 |
|-----------|------|
| Spring Web | REST API を作る（HTTP 通信） |
| Validation | 入力値チェック（@NotBlank 等） |
| Spring Data JPA | DB 操作を簡単にする（SQL 書かなくて済む） |
| H2 Database | 開発用インメモリ DB（設定不要ですぐ使える） |
| Lombok | getter/setter 等の定型コードを省略 |

## application.propertiesの設定

H2を導入：Web上でDBを見れるようにする・アプリをDBを連携
JPA導入：ORM導入。自動でSQL文を発行できる


gitコマンドはなるべくジュニアエンジニアが行う。
プルリクとかはclaudeが行う

## 動作確認結果

| 確認項目 | 結果 |
|---------|------|
| アプリ起動 | ✅ `Started Kakeibo2Application` がコンソールに表示 |
| http://localhost:8080 | ✅ Whitelabel Error Page が表示（正常） |
| http://localhost:8080/h2-console | ✅ H2 Console ログイン画面が表示 |

### H2 Console 接続情報

- JDBC URL: `jdbc:h2:mem:kakeibodb`
- User: `sa`
- Password: （空欄）

