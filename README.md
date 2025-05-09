# study-spring-boot — Spring Initializr Comprehensive Guide

## 1. Project Metadata (Quick Reference)

| 項目               | 典型値                          | 役割・補足                                                               |
| ---------------- | ---------------------------- | ------------------------------------------------------------------- |
| **Project**      | Maven                        | ビルドツール。`pom.xml` が生成される                                             |
| **Language**     | Java                         | Kotlin × Spring Boot も公式サポートが厚い                                     |
| **Spring Boot**  | 3.3.x (最新安定版)                | ここで指定した Boot バージョンが **BOM** に反映され、依存ライブラリのバージョンを統一                  |
| **Group**        | com.example                  | Java の `groupId`。パッケージプレフィクスにも利用                                    |
| **Artifact**     | demo                         | jar/war ファイル名・`artifactId`                                          |
| **Name**         | demo                         | アプリ名（`DemoApplication` 等）                                           |
| **Description**  | Demo project for Spring Boot | README や POM コメントに反映                                                |
| **Package name** | com.example.demo             | 既定は `Group` + `Artifact`                                            |
| **Packaging**    | Jar / War                    | *Jar*: 組み込み Tomcat/Netty で `java -jar` 実行<br>*War*: 外部サーブレットコンテナに配備 |
| **Java**         | 17 (LTS)                     | Initializr UI では 17 / 21 が選択肢 (8/11 は非表示) citeturn0search0       |

---

## 2. Dependencies パネル ― UI の使い方

| UI 要素                     | 機能                                              | 補足                                             |
| ------------------------- | ----------------------------------------------- | ---------------------------------------------- |
| **Search bar**            | 依存関係をキーワード検索 (部分一致)。スペース区切りで AND 検索             | `/` プレフィクスで *カテゴリ名\:filter* も可                 |
| **Category グループ**         | Developer Tools / Web / Security … などテーマ別に折りたたみ | 各依存関係は必ず 1 カテゴリに属する citeturn0search1        |
| **Dependency チップ**        | 緑＝未選択、青＝選択済み。クリックでトグル                           | 選択すると上部 *Selected* パネルに追加                      |
| **Selected Dependencies** | 現在選択済みの依存関係を一覧 & 削除                             | コード生成時は `dependencies=<id1>,<id2>` として API に渡る |

> **TIP** : 依存関係 ID はそのまま *starter* 名 (`spring-boot-starter-<id>`) に紐付くことが多い。

---

## 3. 依存関係カテゴリ & 代表スターター

| カテゴリ                    | 代表 ID                                                  | 生成される starter / ライブラリ                  | 用途概要              |
| ----------------------- | ------------------------------------------------------ | -------------------------------------- | ----------------- |
| **Developer Tools**     | devtools, lombok, configuration-processor              | Hot Reload, アノテーション処理補助                | 開発効率              |
| **Core**                | actuator, validation, aot                              | Actuator, Jakarta Validation, AOT      | 運用可観測性・入力検証       |
| **Web**                 | web, webflux, graphql, websocket                       | MVC (Tomcat), WebFlux (Netty), GraphQL | HTTP / GraphQL 通信 |
| **Template Engines**    | thymeleaf, mustache, freemarker                        | 各テンプレートエンジン                            | サーバサイド HTML       |
| **Security**            | security, oauth2-client, oauth2-resource-server        | Spring Security                        | 認証・認可             |
| **SQL**                 | data-jpa, jdbc, r2dbc, flyway, postgresql, mysql       | Driver・Migration ツール                   | RDB               |
| **NoSQL**               | data-mongodb, data-redis, data-cassandra, dynamodb     | Spring Data 系                          | 分散 / キー値 DB       |
| **Messaging**           | kafka, rabbitmq, pulsar, batch                         | Kafka, AMQP, Pulsar, Batch             | 非同期処理・バッチ         |
| **Cloud**               | cloud-gateway, cloud-config-client, eureka-client, aws | Spring Cloud                           | マイクロサービス周辺        |
| **Observability / Ops** | prometheus, wavefront, zipkin, otel                    | Micrometer, OTEL                       | メトリクス・分散トレース      |
| **Testing**             | testcontainers, spring-restdocs, cucumber              | コンテナ統合テスト、API ドキュメント                   | テスト自動化            |

(メタデータ参照: Spring Initializr Reference Guide) citeturn0search1

---

## 4. *starter* の仕組み

* 「ID = `spring-boot-starter-<id>`」 が Maven/Gradle へ自動で追加
* スターター内部で **必要なサーブレットコンテナ・JSON パーサ・ログ** などを transitively 解決
* **BOM** により互換性のあるバージョンが一括インポートされる citeturn0search3turn0search5

---

## 5. CLI / REST API でテンプレート生成

```bash
# 依存関係 ID 一覧を JSON 取得
curl -H 'Accept: application/json' https://start.spring.io \
  | jq '.dependencies.values[].values[].id'
```

```bash
# プロジェクト ZIP を生成
curl https://start.spring.io/starter.zip \
  -d dependencies=web,data-jpa,h2,springdoc-openapi,actuator,validation,lombok \
  -d name=demo-api -d type=maven-project -d javaVersion=21 \
  -o demo-api.zip
```

> Initializr の REST API は IDE や CI/CD パイプラインでも利用可能。

---

## 6. ユースケース別スターターセット

| 目的                 | 推奨 ID                                                                                            | メモ                             |
| ------------------ | ------------------------------------------------------------------------------------------------ | ------------------------------ |
| **PoC / 個人開発**     | web, data-jpa, h2, springdoc-openapi, actuator, validation, lombok, devtools                     | 最小 REST + DB + Swagger UI      |
| **クラウド (AWS) PoC** | web, data-jpa, h2, springdoc-openapi, validation, actuator, aws, micrometer-registry-cloudwatch2 | CloudWatch 連携                  |
| **k8s × GitOps**   | web, actuator, prometheus, cloud-kubernetes                                                      | Liveness/Prometheus エンドポイント即有効 |
| **ブロックチェーン API**   | webflux, validation, springdoc-openapi                                                           | 高スループット REST + Swagger         |

---

## 7. まとめ & 次のステップ

1. **検索 → トグル → Selected** の 3 ステップで依存関係を決定
2. *starter* の概念を理解しておくと依存解決がシンプル
3. CLI/API を活用すれば自動生成・テンプレート化も容易
4. 必要に応じてカテゴリ表を参照し、ユースケースにマッチするスターターを追加

---

> **Next Try**: `web`, `data-jpa`, `prometheus` を選んでプロジェクト生成 → IDE インポート → `./mvnw spring-boot:run` で動作確認してみましょう。
