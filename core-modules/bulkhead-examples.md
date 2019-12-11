サンプルコード
============
resilience4j-bulkheadのサンプルコード

# BulkheadRegistryの作成
カスタムのBulkheadConfigを利用したBulkheadRegistryの作成

> FIXME RateLimiterのコードなのでおそらく原文の間違い。修正依頼中

```java
// Create a custom configuration for a RateLimiter
RateLimiterConfig config = RateLimiterConfig.custom()
    .limitRefreshPeriod(Duration.ofMillis(1))
    .limitForPeriod(10)
    .timeoutDuration(Duration.ofMillis(25))
    .build();

// Create a RateLimiterRegistry with a custom global configuration
RateLimiterRegistry rateLimiterRegistry = RateLimiterRegistry.of(config);
```

# Bulkheadの作成
グローバルなデフォルト設定を持ったBulkheadRegistryからのBulkhead取得

> FIXME 原文でメソッド名の間違い。修正依頼中

```java
Bulkhead bulkhead = bulkheadRegistry 
  .circuitBreaker("name");
```

# 関数型インタフェースのデコレート
`BackendService.doSomething()` の呼び出しをBulkheadでデコレートし、デコレートされたSupplierを実行、全ての例外からリカバリーする

```java
Supplier<String> decoratedSupplier = Bulkhead
    .decorateSupplier(retry, backendService::doSomething);

String result = Try.ofSupplier(decoratedSupplier)
    .recover(throwable -> "Hello from Recovery").get(); 
```
