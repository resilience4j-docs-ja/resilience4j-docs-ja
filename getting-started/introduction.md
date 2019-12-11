イントロダクション
====================================================
Resilience4jは、Netflix Hystrixにインスパイアされた軽量なフォールトトレランスライブラリですが、関数型プログラミング向けにデザインされています。このライブラリはVavrしか使っておらず、その他の外部依存ライブラリが無いため、軽量です。それに対してNetflix Hystrixは、コンパイル依存性にArcaiusがあります。これは、GuavaやApache Commons Configurationのような多くの外部依存ライブラリを持っています。

Resilience4jは関数型インタフェース、ラムダ式、メソッド参照をサーキットブレイカー、流量制限、リトライ、バルクヘッドで拡張するための高階関数（デコレーター）を提供します。関数型インタフェース、ラムダ式、メソッド参照には、1つ以上のデコレーターをスタックすることができます。これにより、必要なデコレーターを使うか、何も使わないかを選ぶことができます。

<!-- FIXME 「スタックする」をうまい日本語にする -->

```java
Supplier<String> supplier = () -> backendService.doSomething(param1, param2);

Supplier<String> decoratedSupplier = Decorators.ofSupplier(supplier)
  .withRetry(Retry.ofDefaults("name"))
  .withCircuitBreaker(CircuitBreaker.ofDefaults("name"))
  .withBulkhead(Bulkhead.ofDefaults("name"));  

String result = Try.ofSupplier(decoratedSupplier)
  .recover(throwable -> "Hello from Recovery").get();
```

Resilience4jでは、全てを使う必要はありません。必要なものでだけを選ぶことができます。

# 概要
下記は、サーキットブレイカーおよびリトライ（例外発生時に最大3回まで）でラムダ式をデコレートする例です。リトライの待ち時間やバックオフアルゴリズムは設定することができます。この例では全リトライが失敗した際、Vavrの `Try` モナドを使って例外から回復し、フォールバックとしてもう1つのラムダ式を実行しています。

```java
// デフォルト設定のサーキットブレイカーを作成する
CircuitBreaker circuitBreaker = CircuitBreaker
  .ofDefaults("backendService");

// デフォルト設定のリトライを作成する
// 500msの固定間隔でリトライを試みる
Retry retry = Retry
  .ofDefaults("backendService");

// デフォルト設定のバルクヘッドを作成する
Bulkhead bulkhead = Bulkhead
  .ofDefaults("backendService");

Supplier<String> supplier = () -> backendService
  .doSomething(param1, param2)

// backendService.doSomething() をバルクヘッド、サーキットブレイカー、リトライでデコレートする
// **注意: resilience4j-allの依存性が必要です
Supplier<String> decoratedSupplier = Decorators.ofSupplier(supplier)
  .withRetry(retry)
  .withCircuitBreaker(circuitBreaker)
  .withBulkhead(bulkhead)
  .decorate();

// デコレートされたSupplierを実行、例外からリカバリーする
String result = Try.ofSupplier(decoratedSupplier)
  .recover(throwable -> "Hello from Recovery").get();

// ラムダ式をデコレートしたくない場合は、
// 単に実行してサーキットブレイカーで呼び出しを保護します
String result = circuitBreaker
  .executeSupplier(backendService::doSomething);
```

# モジュール化
Resilience4jでは、全てを使う必要はありません。必要なものだけを選ぶことができます。

## 全部入り
- resilience4j-all

## コアモジュール
- resilience4j-circuitbreaker: サーキットブレイカー
- resilience4j-ratelimiter: 流量制御
- resilience4j-bulkhead: バルクヘッド
- resilience4j-retry: 自動リトライ（同期、非同期）
- resilience4j-cache: キャッシュ
- resilience4j-timelimiter: タイムアウトハンドリング

## アドオンモジュール
- resilience4j-retrofit: Retrofitアダプター
- resilience4j-feign: Feignアダプター
- resilience4j-consumer: Circular Buffer Event consumer
- resilience4j-kotlin: Kotlin coroutineサポート

## フレームワークモジュール
- resilience4j-spring-boot: Spring Boot Starter
- resilience4j-spring-boot2: Spring Boot 2 Starter
- resilience4j-ratpack: Ratpack Starter
- resilience4j-vertx: Vertx Future decorator

## リアクティブモジュール
- resilience4j-rxjava2: RxJava2カスタムオペレーター
- resilience4j-reactor: Spring Reactorカスタムオペレーター

## メトリクスモジュール
- resilience4j-micrometer: Micrometer Metrics exporter
- resilience4j-metrics: Dropwizard Metrics exporter
- resilience4j-prometheus: Prometheus Metrics exporter

## サードパーティーモジュール
- [Camel Circuit Breaker](https://camel.apache.org/manual/latest/resilience4j-eip.html)
- [Spring Cloud Circuit Breaker](https://spring.io/projects/spring-cloud-circuitbreaker)
- [http4k resilience module](https://www.http4k.org/guide/modules/resilience/)

# リンク
- [トップページ](../index.md)
