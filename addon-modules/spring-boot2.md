Getting Started
===============
resilience4j-spring-boot2の利用

[トップページに戻る](../index.md)

# 準備
Resilience4jのSpring Boot Starterをコンパイル依存性に追加してください。

このモジュールは、実行時に `org.springframework.boot:spring-boot-starter-actuator` および `org.springframework.boot:spring-boot-starter-aop` が既に提供されていることを期待します。

```groovy
repositories {
    jCenter()
}

dependencies {
    compile "io.github.resilience4j:resilience4j-spring-boot2:${resilience4jVersion}"
    compile('org.springframework.boot:spring-boot-starter-actuator')
    compile('org.springframework.boot:spring-boot-starter-aop')
}
```

# デモ
Spring Boot 2における準備および利用は[demo](https://github.com/resilience4j/resilience4j-spring-boot2-demo)で紹介されています。

# 設定
CircuitBreaker・Retry・RateLimiter・Bulkhead・ThreadPoolBulkhead・TimeLimiterのインスタンスは、Spring Bootのapplication.yml設定ファイルで設定できます。

例

```yaml
resilience4j.circuitbreaker:
    instances:
        backendA:
            registerHealthIndicator: true
            slidingWindowSize: 100
        backendB:
            registerHealthIndicator: true
            slidingWindowSize: 10
            permittedNumberOfCallsInHalfOpenState: 3
            slidingWindowType: TIME_BASED
            minimumNumberOfCalls: 20
            waitDurationInOpenState: 50s
            failureRateThreshold: 50
            eventConsumerBufferSize: 10
            recordFailurePredicate: io.github.robwin.exception.RecordFailurePredicate
            
resilience4j.retry:
    instances:
        backendA:
            maxRetryAttempts: 3
            waitDuration: 10s
            enableExponentialBackoff: true
            exponentialBackoffMultiplier: 2
            retryExceptions:
                - org.springframework.web.client.HttpServerErrorException
                - java.io.IOException
            ignoreExceptions:
                - io.github.robwin.exception.BusinessException
        backendB:
            maxRetryAttempts: 3
            waitDuration: 10s
            retryExceptions:
                - org.springframework.web.client.HttpServerErrorException
                - java.io.IOException
            ignoreExceptions:
                - io.github.robwin.exception.BusinessException
                
resilience4j.bulkhead:
    instances:
        backendA:
            maxConcurrentCalls: 10
        backendB:
            maxWaitDuration: 10ms
            maxConcurrentCalls: 20
            
resilience4j.thread-pool-bulkhead:
    instances:
        backendC:
            maxThreadPoolSize: 1
            coreThreadPoolSize: 1
            queueCapacity: 1
        
resilience4j.ratelimiter:
    instances:
        backendA:
            limitForPeriod: 10
            limitRefreshPeriod: 1s
            timeoutDuration: 0
            registerHealthIndicator: true
            eventConsumerBufferSize: 100
        backendB:
            limitForPeriod: 6
            limitRefreshPeriod: 500ms
            timeoutDuration: 3s
            
resilience4j.timelimiter:
    instances:
        backendA:
            timeoutDuration: 2s
            cancelRunningFuture: true
        backendB:
            timeoutDuration: 1s
            cancelRunningFuture: false
```

デフォルトの設定を上書きすることも可能です。Spring Bootのapplication.yml設定ファイルに共有設定を定義し、上書きます。

例

```yaml
resilience4j.circuitbreaker:
    configs:
        default:
            slidingWindowSize: 100
            permittedNumberOfCallsInHalfOpenState: 10
            waitDurationInOpenState: 10000
            failureRateThreshold: 60
            eventConsumerBufferSize: 10
            registerHealthIndicator: true
        someShared:
            slidingWindowSize: 50
            permittedNumberOfCallsInHalfOpenState: 10
    instances:
        backendA:
            baseConfig: default
            waitDurationInOpenState: 5000
        backendB:
            baseConfig: someShared
```

特定のCircuitBreaker・Retry・RateLimiter・Bulkhead・TimeLimiterのインスタンスの設定を上書きするには、特定のインスタンス名のCustomizerを利用します。下記は、上記のYAMLで設定されたCircuitBreaker **backendA** の設定を上書きする例です:

```java
@Bean
public CircuitBreakerConfigCustomizer testCustomizer() {
    return CircuitBreakerConfigCustomizer
        .of("backendA", builder -> builder.slidingWindowSize(100));
}
```

Resilience4jは上記のように利用できるCustomizer型を持っています:

| Resilience4j型 | インスタンスCustomizerクラス |
|----------------|---------------------------|
| Circuit breaker | CircuitBreakerConfigCustomizer |
| Retry | RetryConfigCustomizer |
| RateLimiter | RateLimiterConfigCustomizer |
| Bulkhead | BulkheadConfigCustomizer |
| ThreadPoolBulkhead | ThreadPoolBulkheadConfigCustomizer |
| TimeLimiter | TimeLimiterConfigCustomizer |

# アノテーション
Spring Boot 2 Starterは、アノテーションとAuto Configuration済みのAOPアスペクトを提供します。RateLimiter・Retry・CircuitBreaker・Bulkheadアノテーションは、同期戻り値型およびCompletableFutureやSpring ReactorのFluxやMonoのようなリアクティブ型（ `resilience4j-reactor` などの適切なパッケージをインポートしている場合）をサポートしています。

Bulkheadアノテーションには、利用されるバルクヘッド実装を定義するtype属性があります。デフォルトはセマフォですが、アノテーションのtype属性を設定することでスレッドプールに変更できます:

```java
@Bulkhead(name = BACKEND, type = Bulkhead.Type.THREADPOOL)
public CompletableFuture<String> doSomethingAsync() throws InterruptedException {
    Thread.sleep(500);
    return CompletableFuture.completedFuture("Test");
}
```

全てのResilience4jがサポートしているSpringアスペクトの例

```java
@CircuitBreaker(name = BACKEND, fallbackMethod = "fallback")
@RateLimiter(name = BACKEND)
@Bulkhead(name = BACKEND)
@Retry(name = BACKEND, fallbackMethod = "fallback")
@TimeLimiter(name = BACKEND)
public Mono<String> method(String param1) {
    return Mono.error(new NumberFormatException());
}

private Mono<String> fallback(String param1, IllegalArgumentException e) {
    return Mono.just("test");
}

private Mono<String> fallback(String param1, RuntimeException e) {
    return Mono.just("test");
}
```

フォールバックメソッドは同一クラス内に置かれ、かつ（ただ1つのターゲット例外の引数を追加した）同一シグネチャでなければならないことに注意してください。

> 訳注: 一般に「シグネチャ」と言った場合「メソッド名＋引数の型の並び順」を指しますが、この文脈では「戻り値の型＋引数の型の並び順」を指しています。

複数のフォールバックメソッドがあった場合、最も近くマッチしたメソッドが実行されます。例えば:

`NumberFormatException` からリカバリーする場合、 `String fallback(String parameter, IllegalArgumentException exception)` シグネチャのメソッドが実行されます。

例外引数を持った1つのグローバルなフォールバックメソッドを作成することも可能ですが、複数のメソッドが同じ戻り値の型で、かつそれら全ての同じフォールバックメソッドを定義したい時のみです。

# アスペクトの順番
Resilience4jアスペクトの順番は下記のとおりです:

Retry ( CircuitBreaker ( RateLimiter ( TimeLimiter ( Bulkhead ( Function ) ) ) ) )

違う順番が必要な場合は、Springアノテーションスタイルの代わりに関数チェーンスタイルを使ってください。

# Metricsエンドポイント
CircuitBreaker・Retry・RateLimiter・Bulkhead・TimeLimiterメトリクスはメトリクスエンドポイントから自動的に高階されます。利用可能なメトリクスの名前を取得したい場合は、 `/actuator/metrics` にGETリクエストしてください。詳細は[Actuator Metrics documentation](https://docs.spring.io/spring-boot/docs/current/actuator-api/html/#metrics)を参照してください。

```json
{
    "names": [
        "resilience4j.circuitbreaker.calls",
        "resilience4j.circuitbreaker.buffered.calls",
        "resilience4j.circuitbreaker.state",
        "resilience4j.circuitbreaker.failure.rate"
    ]
}
```

メトリクスのを取得するには、 `/actuator/metrics/{metric.name}` にGETリクエストしてください。

例: `/actuator/metrics/resilience4j.circuitbreaker.calls`

```json
{
    "name": "resilience4j.circuitbreaker.calls",
    "measurements": [
        {
            "statistic": "VALUE",
            "value": 3
        }
    ],
    "availableTags": [
        {
            "tag": "kind",
            "values": [
                "not_permitted",
                "successful",
                "failed"
            ]
        },
        {
            "tag": "name",
            "values": [
                "backendB",
                "backendA"
            ]
        }
    ]
}
```

PrometheusエンドポイントでCircuitBreakerエンドポイントを公開したい場合は、 `io.micrometer:micrometer-registry-prometheus` 依存性を追加してください。

メトリクスを取得するには、 `/actuator/prometheus` にGETリクエストしてください。詳細は[Micrometer Getting Started](https://resilience4j.readme.io/docs/micrometer)を参照してください。

# Healthエンドポイント
Spring Boot Actuatorのhealth情報は実行中アプリケーションの状態をチェックするために使われます。モニタリングソフトウェアが、本番のシステムに深刻な問題があるかを誰かに警告するためにしばしば使われます。

デフォルトではCircuitBreakerとRateLimiterのhealthインジケーターは無効化されていますが、設定で有効化することができます。healthインジケーターが無効化されているのは、CircuitBreakerはOPENの時、アプリケーションの状態はDOWNだからです。これは達成したいことではないでしょう。

```yaml
management.health.circuitbreakers.enabled: true
management.health.ratelimiters.enabled: true

resilience4j.circuitbreaker:
  configs:
    default:
      registerHealthIndicator: true


resilience4j.ratelimiter:
  configs:
    default:
      registerHealthIndicator: true
```

CLOSEDなCircuitBreakerがUPとマッピングされ、OPEN状態はDOWN、HALF-OPENはUNKNOWNになります。

例:

```json
{
  "status": "UP",
  "details": {
    "circuitBreakers": {
      "status": "UP",
      "details": {
        "backendB": {
          "status": "UP",
          "details": {
            "failureRate": "-1.0%",
            "failureRateThreshold": "50.0%",
            "slowCallRate": "-1.0%",
            "slowCallRateThreshold": "100.0%",
            "bufferedCalls": 0,
            "slowCalls": 0,
            "slowFailedCalls": 0,
            "failedCalls": 0,
            "notPermittedCalls": 0,
            "state": "CLOSED"
          }
        },
        "backendA": {
          "status": "UP",
          "details": {
            "failureRate": "-1.0%",
            "failureRateThreshold": "50.0%",
            "slowCallRate": "-1.0%",
            "slowCallRateThreshold": "100.0%",
            "bufferedCalls": 0,
            "slowCalls": 0,
            "slowFailedCalls": 0,
            "failedCalls": 0,
            "notPermittedCalls": 0,
            "state": "CLOSED"
          }
        }
      }
    }
  }
}
```

# Eventsエンドポイント
放出されたCircuitBreaker・Retry・RateLimiter・Bulkhead・TimeLimiterイベントは別々の循環イベントコンシューマーバッファーに保存されます。イベントコンシューマーバッファーのサイズはapplication.ymlファイルで設定できます（eventConsumerBufferSize）。

`/actuator/circuitbreakers` エンドポイントは全CircuitBreakerインスタンスの名前を一覧します。Retry・RateLimiter・Bulkhead・TimeLimiterのエンドポイントも利用可能です。

例:

```json
{
    "circuitBreakers": [
      "backendA",
      "backendB"
    ]
}
```

`/actuator/circuitbreakerevents` エンドポイントは、デフォルトでは全CircuitBreakerインスタンスの直近100イベントを一覧します。Retry・RateLimiter・Bulkhead・TimeLimiterのエンドポイントも利用可能です。

```json
{
    "circuitBreakerEvents": [
        {
            "circuitBreakerName": "backendA",
            "type": "ERROR",
            "creationTime": "2017-01-10T15:39:17.117+01:00[Europe/Berlin]",
            "errorMessage": "org.springframework.web.client.HttpServerErrorException: 500 This is a remote exception",
            "durationInMs": 0
        },
        {
            "circuitBreakerName": "backendA",
            "type": "SUCCESS",
            "creationTime": "2017-01-10T15:39:20.518+01:00[Europe/Berlin]",
            "durationInMs": 0
        },
        {
            "circuitBreakerName": "backendB",
            "type": "ERROR",
            "creationTime": "2017-01-10T15:41:31.159+01:00[Europe/Berlin]",
            "errorMessage": "org.springframework.web.client.HttpServerErrorException: 500 This is a remote exception",
            "durationInMs": 0
        },
        {
            "circuitBreakerName": "backendB",
            "type": "SUCCESS",
            "creationTime": "2017-01-10T15:41:33.526+01:00[Europe/Berlin]",
            "durationInMs": 0
        }
    ]
}
```

# リンク
- [トップページ](../index.md)
