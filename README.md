# study-spring-boot — Spring Initializr Comprehensive Guide



## 3. 依存関係カテゴリ & 代表スターター

**整理依頼の確認**

ユーザーは「この表をリファクタリング依頼」と言っていますが、既に提供されたテーブルをリファクタリングして、ロシアと中国の弱みの列を分けたことがありました。今、新しいテーブル（依存関係/ライブラリ）を別の形式に整理したい可能性があります。依頼内容が「リファクタリング」とは、大まかにカラムを分け、再構成することかも。例えば、開発、コア、Web層ごとに並べ替え、「主なスターター」「使用例」「主要アーティファクト」「Spring Starter ID」などに整理する提案をした方がいいかもしれません。


思考時間: 6 秒


### 使い どころが一目で分かるよう **カラムを整理し直した“リファクタリング版”**

— 「① Spring Boot Starter ID」「② 追加ライブラリ（任意）」「③ 主な用途」 の３列構成にしました。

思考時間: 数秒


失礼しました。**ActiveMQ （Artemis／Classic）を必須として明示**した最新版に差し替えます。
――Messaging カテゴリを **“ActiveMQ 専用”** 行として独立させ、それ以外のメッセージブローカは別枠に移動しました。

| カテゴリ                    | ① Starter artifactId                                                                                        | ② 追加ライブラリ例                                                                        | ③ 主な用途                    |
| ----------------------- | ----------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- | ------------------------- |
| **Developer Tools**     | `spring-boot-devtools`                                                                                      | `lombok`, `spring-boot-configuration-processor`                                   | ホットリロード／アノテ補助             |
| **Core／Runtime**        | `spring-boot-starter-actuator`                                                                              | `jakarta.validation-api`, `spring-boot-starter-aot`                               | 可観測性／入力バリデーション／AOT        |
| **Web (MVC)**           | `spring-boot-starter-web`                                                                                   | —                                                                                 | REST HTTP (Tomcat)        |
| **WebFlux**             | `spring-boot-starter-webflux`                                                                               | —                                                                                 | ノンブロッキング HTTP             |
| **GraphQL**             | `spring-boot-starter-graphql`                                                                               | —                                                                                 | GraphQL API               |
| **WebSocket**           | `spring-boot-starter-websocket`                                                                             | —                                                                                 | 双方向 WebSocket             |
| **Security**            | `spring-boot-starter-security`                                                                              | `spring-boot-starter-oauth2-client`, `spring-boot-starter-oauth2-resource-server` | 認証・認可                     |
| **SQL (RDB)**           | `spring-boot-starter-data-jpa`, `spring-boot-starter-jdbc`                                                  | `flyway-core`, `postgresql`, `mysql`                                              | ORM／JDBC／マイグレーション         |
| **Reactive SQL**        | `spring-boot-starter-data-r2dbc`                                                                            | `r2dbc-postgresql`, `r2dbc-mysql`                                                 | リアクティブ RDB                |
| **NoSQL**               | `spring-boot-starter-data-redis`, `spring-boot-starter-data-mongodb`                                        | `spring-boot-starter-data-cassandra`, `dynamodb-enhanced`                         | 分散／キー値ストア                 |
| **ActiveMQ (必須)**       | `spring-boot-starter-activemq`<br>※Artemis に切替可                                                             | `org.apache.activemq:artemis-jms-client` など                                       | JMS／非同期メッセージング            |
| **その他 Messaging**       | `spring-boot-starter-amqp`, `spring-kafka`, `spring-boot-starter-pulsar`, `spring-boot-starter-batch`       | —                                                                                 | RabbitMQ／Kafka／Pulsar／バッチ |
| **Cloud & MS**          | `spring-cloud-starter-gateway`, `spring-cloud-starter-config`, `spring-cloud-starter-netflix-eureka-client` | `spring-cloud-starter-aws`                                                        | API Gateway／設定集中／サービス発見   |
| **Observability / Ops** | `micrometer-registry-prometheus`, `micrometer-tracing-bridge-otel`                                          | `zipkin-reporter-brave`, `wavefront-spring-boot-starter`                          | メトリクス／分散トレース              |
| **Testing**             | `spring-boot-starter-test`                                                                                  | `testcontainers-bom`, `spring-restdocs-mockmvc`, `cucumber-java`                  | 単体・統合テスト／API ドキュメント       |

これで **ActiveMQ** が確実に含まれています。
ほかに必須ライブラリや分類があればお知らせください。



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

### Spring Boot ▶ Microservices ▶ EKS までの学習フェーズ（リファクタリング版）

|#|フェーズ名|ゴール & 技術範囲|完了チェックポイント|
|---|---|---|---|
|**0**|**PoC-Zero (メモリ専用)**|_Spring Boot 3_／`spring-boot-starter-web` のみDBなし・`@RestController` がメモリ変数を返すだけ|`GET /ping` が 200 OK|
|**1**|**PoC-H2 シングル**|H2 (in-memory) 追加JPA + `spring-boot-starter-data-jpa`flyway/liquibase で schema version 管理|`POST /todos`→H2 に行が増える|
|**2**|**Core Secure PoC**|フェーズ 1 + **Spring Security** (JWT) + **SpringDoc Swagger UI** + **Junit5 / Testcontainers(LocalStack)**|1) Swagger で API 試せる2) Integration Test green|
|**3**|**Local Microservice**|_Spring Cloud_ (Gateway & Config) + OpenFeignサービス分割 (auth-svc / order-svc など) を **Docker Compose** で起動|Gateway 経由で 2 サービスが通信|
|**4**|**Cloud PoC (非 k8s)**|**AWS ECS/Fargate** 1 タスク構成RDS (PostgreSQL) or DynamoDB へ接続CloudWatch Logs & X-Ray|GitHub Actions → ECR → ECS Blue/Green|
|**5**|**Kubernetes on EC2 (Kind/EKS-Small)**|EC2 Linux 上に **kind** で k8s 基礎Spring Boot イメージを `kubectl apply`→IngressHelm Chart 化|`kubectl port-forward` なしで公開 URL|
|**6**|**EKS Microservice & GitOps**|**EKS** クラスタに Helm → Argo CDIstio Ingress (optional)・Prometheus/Loki/GrafanaSecret Store = AWS SSM / IRSA|Argo CD ⬌ Git が自動同期／Grafana ダッシュボードが表示|

#### 学習フローのポイント

1. **0→1→2** で “REST＋DB＋認証＋テスト” の _単体アプリ_ を固める
    
2. **3** で _ローカル_ 完結の **マイクロサービス通信** を習得
    
3. **4** で _クラウド**だけ**運用_ を経験し、AWS 基本サービスに慣れる
    
4. **5** で _k8s のオブジェクト & Helm テンプレ_ に手を動かす
    
5. **6** で _EKS + GitOps_ ＝ 本番相当の運用モデルへ
