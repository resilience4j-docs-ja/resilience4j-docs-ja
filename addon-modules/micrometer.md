Micrometer
==========
resilience4j-micrometerの利用

[トップページに戻る](../index.md)

Resilience4jは、InfluxDBやPrometheusのような最も人気の監視システムをサポートしているMicrometerのためのモジュールを提供します。

このモジュールは、実行時に `micrometer-core` が既に提供されていることを期待します。Spring Reactorは透過的依存性ではありません。

```groovy
repositories {
    jCenter()
}

dependencies {
    compile "io.github.resilience4j:resilience4j-micrometer:${resilience4jVersion}"
}
```

# CircuitBreakerメトリクス
下記のコードは `MeterRegistry` にCircuitBreakerメトリクスをバインドする方法を示しています。これは全CircuitBreakerインスタンスを一度にバインドし、新規作成されたインスタンスに動的にバインドするためのイベントコンシューマーを登録します。

```java
MeterRegistry meterRegistry = new SimpleMeterRegistry();
CircuitBreakerRegistry circuitBreakerRegistry =
  CircuitBreakerRegistry.ofDefaults();
CircuitBreaker foo = circuitBreakerRegistry
  .circuitBreaker("backendA");
CircuitBreaker boo = circuitBreakerRegistry
  .circuitBreaker("backendB");

TaggedCircuitBreakerMetrics
  .ofCircuitBreakerRegistry(circuitBreakerRegistry)
  .bindTo(meterRegistry)
```

下記のメトリクスがエクスポートされます:

| メトリクス名 | 型 | タグ | 説明 |
|------------|---|------|-----|
| resilience4j.circuitbreaker.calls | Timer | name="backendA" | 成功・失敗した呼び出しの合計回数 |
| resilience4j.circuitbreaker.max.buffered.calls | Gauge | name="backendA" | 現在のスライディングウィンドウ内に保存可能なバッファーされた呼び出しの最大数 |
| resilience4j.circuitbreaker.state | Gauge <br> 0 - Not active <br> 1 - Active | 下記のいずれか1つ: <br>- state="closed"<br>- state="open"<br>- state="half_open"<br>- state="forced_open"<br>- state="disabled"<br>name="backendA" | CircuitBreakerの状態 |
| resilience4j.circuitbreaker.failure.rate | Gauge |name="backendA" | CircuitBreakerの失敗率 |
| resilience4j.circuitbreaker.buffered.calls | Gauge | 下記のいずれか1つ: <br>- kind="failed"<br>- kind="successful"<br>name="backendA" | スライディングウィンドウに保存されている、バッファーされた成功・失敗した呼び出しの回数 |
| resilience4.circuitbreaker.calls | Counter | kind="not_permitted"<br>name="backendA" | 失敗したが例外が無視された呼び出しの回数 |
| resilience4j.circuitbreaker.slow.call.rate | Gauge | name="backendA" | CircuitBreakerの遅い呼び出し |

# Retryメトリクス
下記のコードは `MeterRegistry` にRetryメトリクスをバインドする方法を示しています。これは全Retryインスタンスを一度にバインドし、新規作成されたインスタンスに動的にバインドするためのイベントコンシューマーを登録します。

```java
MeterRegistry meterRegistry = new SimpleMeterRegistry();
RetryRegistry retryRegistry = RetryRegistry.ofDefaults();
Retry retry = retryRegistry.retry("backendA");

// 全Retryを一度に登録
TaggedRetryMetrics
  .ofRetryRegistry(retryRegistry)
  .bindTo(meterRegistry);
```

下記のメトリクスがエクスポートされます:

| メトリクス名 | 型 | タグ | 説明 |
|------------|---|------|-----|
| resilience4j.retry.calls | Gauge | 下記のいずれか1つ:<br>- kind="successful.without.retry"<br>- kind="successful.with.retry"<br>- kind="failed.with.retry"<br>- kind="failed.without.retry"<br>name="backendA" | kindの呼び出し回数 |

# Bulkheadメトリクス
下記のコードは `MeterRegistry` にBulkheadメトリクスをバインドする方法を示しています。これは全Bulkheadインスタンスを一度にバインドし、新規作成されたインスタンスに動的にバインドするためのイベントコンシューマーを登録します。

```java
MeterRegistry meterRegistry = new SimpleMeterRegistry();
BulkheadRegistry bulkheadRegistry = BulkheadRegistry.ofDefaults();
Bulkhead bulkhead = bulkheadRegistry.bulkhead("backendA");

// 全Bulkheadを一度に登録
TaggedBulkheadMetrics
  .ofBulkheadRegistry(bulkheadRegistry)
  .bindTo(meterRegistry);
```

下記のメトリクスがエクスポートされます:

| メトリクス名 | 型 | タグ | 説明 |
|------------|---|------|-----|
| resilience4j.bulkhead.available.concurrent.calls | Gauge | name="backendA" | 利用可能なパーミッション数 |
| resilience4j.bulkhead.max.allowed.concurrent.calls | Gauge | name="backendA" | 利用可能なパーミッションの最大数 |

# RateLimiterメトリクス
下記のコードは `MeterRegistry` にRateLimiterメトリクスをバインドする方法を示しています。これは全RateLimiterインスタンスを一度にバインドし、新規作成されたインスタンスに動的にバインドするためのイベントコンシューマーを登録します。

```java
MeterRegistry meterRegistry = new SimpleMeterRegistry();
RateLimiterRegistry rateLimiterRegistry = RateLimiterRegistry.ofDefaults();
RateLimiter rateLimiter = rateLimiterRegistry
  .rateLimiter("backendA");

// 全RateLimiterを一度に登録
TaggedRateLimiterMetrics
  .ofRateLimiterRegistry(rateLimiterRegistry)
  .bindTo(meterRegistry);
```

下記のメトリクスがエクスポートされます:

| メトリクス名 | 型 | タグ | 説明 |
|------------|---|------|-----|
| resilience4j.ratelimiter.available.permissions | Gauge | name="backendA" | 利用可能なパーミッション数 |
| resilience4j.ratelimiter.waiting.threads | Gauge | name="backendA" | 待機中のスレッド数 |

# Prometheus
Prometheusに公開したい場合は、下記の依存性を追加してください:

```groovy
dependencies {
    compile "io.micrometer:micrometer-registry-prometheus"
}
```

下記のメトリクスがCircuitBreakerごとにエクスポートされます:

```
# HELP resilience4j_circuitbreaker_buffered_calls The number of buffered failed calls stored in the ring buffer
# TYPE resilience4j_circuitbreaker_buffered_calls gauge
resilience4j_circuitbreaker_buffered_calls{kind="failed",name="backendA",} 0.0
resilience4j_circuitbreaker_buffered_calls{kind="successful",name="backendA",} 0.0

# HELP resilience4j_circuitbreaker_calls_total Total number of not permitted calls
# TYPE resilience4j_circuitbreaker_calls_total counter
resilience4j_circuitbreaker_calls_total{kind="not_permitted",name="backendA",} 0.0

# HELP resilience4j_circuitbreaker_state The states of the circuit breaker
# TYPE resilience4j_circuitbreaker_state gauge
resilience4j_circuitbreaker_state{name="backendA",state="half_open",} 0.0
resilience4j_circuitbreaker_state{name="backendA",state="forced_open",} 0.0
resilience4j_circuitbreaker_state{name="backendA",state="disabled",} 0.0
resilience4j_circuitbreaker_state{name="backendA",state="closed",} 1.0
resilience4j_circuitbreaker_state{name="backendA",state="open",} 0.0
resilience4j_circuitbreaker_state{name="backendA",} 0.0

# HELP resilience4j_circuitbreaker_failure_rate The failure rate of the circuit breaker
# TYPE resilience4j_circuitbreaker_failure_rate gauge
resilience4j_circuitbreaker_failure_rate{name="backendA",} 20.0

# HELP resilience4j_circuitbreaker_max_buffered_calls The maximum number of buffered calls which can be stored in the ring buffer
# TYPE resilience4j_circuitbreaker_max_buffered_calls gauge
resilience4j_circuitbreaker_max_buffered_calls{name="backendA",} 5.0

# HELP resilience4j_circuitbreaker_calls_seconds_max Total duration of calls
# TYPE resilience4j_circuitbreaker_calls_seconds_max gauge
resilience4j_circuitbreaker_calls_seconds_max{kind="successful",name="backendA",} 0.0
resilience4j_circuitbreaker_calls_seconds_max{kind="failed",name="backendA",} 0.0
resilience4j_circuitbreaker_calls_seconds_max{kind="ignored",name="backendA",} 0.0

resilience4j_circuitbreaker_calls_seconds_sum{kind="ignored",name="backendA",} 0.0

# HELP resilience4j_circuitbreaker_calls_seconds Total number of successful calls
# TYPE resilience4j_circuitbreaker_calls_seconds histogram
resilience4j_circuitbreaker_calls_seconds_count{kind="successful",name="backendA",} 0.0
resilience4j_circuitbreaker_calls_seconds_sum{kind="successful",name="backendA",} 0.0

# HELP resilience4j_circuitbreaker_calls_seconds Total number of failed calls
# TYPE resilience4j_circuitbreaker_calls_seconds histogram
resilience4j_circuitbreaker_calls_seconds_count{kind="failed",name="backendA",} 0.0
resilience4j_circuitbreaker_calls_seconds_sum{kind="failed",name="backendA",} 0.0
```

下記のメトリクスがBulkheadごとにエクスポートされます:

```
# HELP resilience4j_bulkhead_available_concurrent_calls The number of available permissions
# TYPE resilience4j_bulkhead_available_concurrent_calls gauge
resilience4j_bulkhead_available_concurrent_calls{name="backendA",} 10.0

# HELP resilience4j_bulkhead_max_allowed_concurrent_calls The maximum number available permissions
# TYPE resilience4j_bulkhead_max_allowed_concurrent_calls gauge
resilience4j_bulkhead_max_allowed_concurrent_calls{name="backendA",} 10.0
```

下記のメトリクスがRetryごとにエクスポートされます:

```
# HELP resilience4j_retry_calls The number of successful calls without a retry attempt
# TYPE resilience4j_retry_calls gauge
resilience4j_retry_calls{kind="failed_with_retry",name="backendA",} 0.0
resilience4j_retry_calls{kind="failed_without_retry",name="backendA",} 0.0
resilience4j_retry_calls{kind="successful_without_retry",name="backendA",} 0.0
resilience4j_retry_calls{kind="successful_with_retry",name="backendA",} 0.0
```

下記のメトリクスがRateLimiterごとにエクスポートされます:

```
# HELP resilience4j_ratelimiter_waiting_threads The number of waiting threads
# TYPE resilience4j_ratelimiter_waiting_threads gauge
resilience4j_ratelimiter_waiting_threads{name="backendA",} 0.0

# HELP resilience4j_ratelimiter_available_permissions The number of available permissions
# TYPE resilience4j_ratelimiter_available_permissions gauge
resilience4j_ratelimiter_available_permissions{name="backendA",} 50.0
```

# リンク
- [トップページ](../index.md)

