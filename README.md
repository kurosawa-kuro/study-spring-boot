# study-spring-boot — Spring Initializr Comprehensive Guide

## 1. Project Metadata (Quick Reference)

| 項目               | 典型値                          | 役割・補足                                                               |
| ---------------- | ---------------------------- | ------------------------------------------------------------------- |
| **Project**      | Maven                        | ビルドツール。`pom.xml` が生成される                                             |
| **Language**     | Java                         | 使用言語。                                                               |
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

| カテゴリ                    | 代表 ID                                                  | 生成される starter / ライブラリ                                                               | 用途概要              |
| ----------------------- | ------------------------------------------------------ | ----------------------------------------------------------------------------------- | ----------------- |
| **Developer Tools**     | devtools, lombok, configuration-processor              | Hot Reload, アノテーション処理補助                                                             | 開発効率              |
| **Core**                | actuator, validation, aot                              | Actuator, Jakarta Validation, AOT                                                   | 運用可観測性・入力検証       |
| **Web**                 | web, webflux, graphql, websocket                       | MVC (Tomcat), WebFlux (Netty), GraphQL                                              | HTTP / GraphQL 通信 |
| **Template Engines**    | thymeleaf, mustache, freemarker                        | 各テンプレートエンジン                                                                         | サーバサイド HTML       |
| **Security**            | security, oauth2-client, oauth2-resource-server        | Spring Security                                                                     | 認証・認可             |
| **SQL**                 | data-jpa, jdbc, r2dbc, flyway, postgresql, mysql       | Driver・Migration ツール                                                                | RDB               |
| **NoSQL**               | data-mongodb, data-redis, data-cassandra, dynamodb     | Spring Data 系                                                                       | 分散 / キー値 DB       |
| **Messaging**           | `activemq`, `kafka`, `rabbitmq`, `pulsar`, `batch`     | Spring JMS (ActiveMQ Classic/Artemis), Spring Kafka, AMQP (RabbitMQ), Pulsar, Batch | 非同期・バッチ           |
| **Cloud**               | cloud-gateway, cloud-config-client, eureka-client, aws | Spring Cloud                                                                        | マイクロサービス周辺        |
| **Observability / Ops** | prometheus, wavefront, zipkin, otel                    | Micrometer, OTEL                                                                    | メトリクス・分散トレース      |
| **Testing**             | testcontainers, spring-restdocs, cucumber              | コンテナ統合テスト、API ドキュメント                                                                | テスト自動化            |

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

| 目的                                       | 推奨 ID                                                                                                                     | メモ                                                                       |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| **PoC / 個人開発**                           | web, data-jpa, h2, springdoc-openapi, security, actuator, validation, lombok, devtools                                    | 最小 REST + DB + Swagger UI + Login 認証                                     |
| **クラウド (AWS) PoC**                       | web, data-jpa, h2, springdoc-openapi, validation, security, oauth2-client, actuator, aws, micrometer-registry-cloudwatch2 | Cognito (OIDC) 認証 + CloudWatch 連携                                        |
| **クラウド (AWS) Microservice**              | web, data-jpa, h2, springdoc-openapi, validation, security, oauth2-client, actuator, aws, micrometer-registry-cloudwatch2 | Cognito 認証 + CloudWatch 連携 (本番構成想定)                                      |
| **クラウド (AWS) Microservice k8s × GitOps** | web, actuator, prometheus, cloud-kubernetes                                                                               | Liveness/Readiness, Prometheus メトリクス<br>※Grafana・Loki は Helm Chart で別途導入 |

### AWS Cognito Integration

AWS Cognito は OIDC 準拠の IdP です。Spring Security の **`oauth2-client`** スターターを追加し、
`spring.security.oauth2.client.registration.cognito.*` にクライアント ID とシークレットを設定します。API には `Authorization: Bearer <access_token>` ヘッダを送るだけで保護が機能します。

### Grafana & Loki (Observability Stack)

`prometheus` スターターで Micrometer が Prometheus エンドポイントを公開します。Kubernetes クラスタには Helm Chart:

```sh
helm repo add grafana https://grafana.github.io/helm-charts
helm install loki grafana/loki-stack
helm install grafana grafana/grafana
```

を適用し、ダッシュボードを作成すれば **メトリクス・ログの可視化** が完成します。

## 7. まとめ & 次のステップ まとめ & 次のステップ まとめ & 次のステップ まとめ & 次のステップ

1. **検索 → トグル → Selected** の 3 ステップで依存関係を決定
2. *starter* の概念を理解しておくと依存解決がシンプル
3. CLI/API を活用すれば自動生成・テンプレート化も容易
4. 必要に応じてカテゴリ表を参照し、ユースケースにマッチするスターターを追加

---

> **Next Try**: `web`, `data-jpa`, `prometheus` を選んでプロジェクト生成 → IDE インポート → `./mvnw spring-boot:run` で動作確認してみましょう。

---

## ActiveMQ (Classic & Artemis) サポートの変遷と選択指針

Spring Boot 3.0 では `jakarta.jms` への全面移行に伴い **ActiveMQ Classic**（5.x 系）のクライアントが追随しておらず Starter が一旦削除されました。しかし **5.18.0 以降**でジャカルタ対応クライアントがリリースされ、**Boot 3.2** で Starter (`spring-boot-starter-activemq`) が復活、Boot 3.3・3.4 系では **Initializr の Messaging カテゴリに `activemq` が表示** されます。

| Spring Boot バージョン | ActiveMQ Classic          | ActiveMQ Artemis                  |
| ----------------- | ------------------------- | --------------------------------- |
| 2.x               | ✅ (`javax.jms`)           | ✅ (`jakarta.jms`)                 |
| 3.0–3.1           | ❌ Starter削除               | ✅ (`spring-boot-starter-artemis`) |
| 3.2+              | ✅ `jakarta.jms` クライアントで復活 | ✅ （変わらず）                          |

**選択基準**

* **ActiveMQ Classic (5.x)**: 既存システムの移行や JMS 1.1 ベースの資産が多い場合に依然として有力。5.18+ の Jakarta 対応版を使う。
* **ActiveMQ Artemis (2.x)**: 次世代ブローカー。高スループット・非ブロッキング IO。Boot が自動で `org.apache.activemq:artemis-*/` を引き込む。
* **Kafka**: 高いパーティション並列性、大規模データストリーム。K8s／クラウドネイティブに最適。
* **RabbitMQ (AMQP)**: ルーティングや遅延キューなど多機能で軽量。シンプルな Pub/Sub や RPC に便利。
* **Pulsar**: マルチテナント、セルフバランス。大規模スケール＋ストレージ分離。
* **Batch**: Spring Batch をまとめたスターター。ETL ジョブやバッチ処理向け。

### Initializr での追加例

```bash
curl https://start.spring.io/starter.zip \
  -d dependencies=web,activemq,actuator \
  -d name=jms-demo -d javaVersion=21 -o jms-demo.zip
```

「Messaging 何を選ぶか？」で迷う場合は、\*\*トラフィック特性（低レイテンシ/高スループット）**と**メッセージングパターン（Pub/Sub vs キュー vs ストリーム）\*\*を整理すると決めやすくなります。

---

## 8. NTT 業務向け — ActiveMQ Classic Quick Start

以下は **ActiveMQ Classic 5.18+** をローカル Docker で動かし、Spring Boot (Boot 3.3) アプリから JMS メッセージを送受信する最小構成例です。

### 8.1 依存関係 (`pom.xml`)

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-activemq</artifactId>
</dependency>
<!--     Omit: spring-boot-starter-web 等他のスターター -->
```

> **Note**: Boot 3.3 で Classic を使う場合、自動で `org.apache.activemq:activemq-client:5.18.x` (Jakarta 対応版) が入ります。

### 8.2 ブローカー起動（Docker Compose）

```yaml
version: '3.8'
services:
  activemq:
    image: rmohr/activemq:5.18.4-alpine
    ports:
      - "61616:61616"   # OpenWire (JMS)
      - "8161:8161"     # Web Console
```

アクセス: [http://localhost:8161/](http://localhost:8161/) (初期 ID: admin / admin)

### 8.3 `application.yml`

```yaml
spring:
  activemq:
    broker-url: tcp://localhost:61616
    user: admin
    password: admin
  jms:
    listener:
      concurrency: 2-10   # 2 並列で開始しピーク 10 までスケール
```

### 8.4 コード例

```java
// Producer
@RequiredArgsConstructor
@Service
public class JmsProducer {
  private final JmsTemplate jmsTemplate;
  public void sendOrder(Order payload) {
    jmsTemplate.convertAndSend("order.queue", payload);
  }
}

// Consumer
@Slf4j
@Component
public class OrderListener {
  @JmsListener(destination = "order.queue")
  public void receive(Order payload) {
    log.info("<-- received {}", payload);
  }
}
```

### 8.5 テスト & 運用 Tips

| 項目               | 設定 / ツール                                       | 補足                                       |
| ---------------- | ---------------------------------------------- | ---------------------------------------- |
| Integration Test | **Testcontainers** `activemq` モジュール            | `@DynamicPropertySource` で broker URL 注入 |
| ヘルスチェック          | Actuator `health.jms`                          | `/actuator/health` が `status: UP` になるか確認 |
| モニタリング           | Web Console, Jolokia + Prometheus JMX Exporter | キューの depth / consumer 数を可視化              |

NTT プロジェクトでは **高い信頼性** と **クリティカルな SLA** が求められるため、

* **DLQ (Dead Letter Queue)** 設定 (`activemq.xml`) と
* **赤/黄/緑アラート** を Grafana などで可視化

を推奨します。Artemis への将来的な置換を見据える場合でも、Spring の抽象化 (`JmsTemplate`, `@JmsListener`) レイヤを保つことでコード変更は最小化できます。
