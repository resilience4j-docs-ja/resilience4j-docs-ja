サンプルコード
============
resilience4j-ratelimiterのサンプルコード

[トップページに戻る](../index.md)

# RateLimiterRegistryの作成
カスタムのRateLimiterConfigを利用したRateLimiterRegistryの作成

```java
// RateLimiterのカスタム設定を作成する
RateLimiterConfig config = RateLimiterConfig.custom()
  .timeoutDuration(TIMEOUT)
  .limitRefreshPeriod(REFRESH_PERIOD)
  .limitForPeriod(LIMIT)
  .build();

// カスタムのグローバル設定でRateLimiterRegistryを作成する
RateLimiterRegistry registry = RateLimiterRegistry.of(config);
```

# RegistryStoreの上書き
インメモリのRegistryStoreを独自実装で上書きすることができます。例えば、ある時間が経過した後に利用されていないインスタンスを削除するようなキャッシュを使いたい場合です。

```java
RateLimiterRegistry rateLimiterRegistry = RateLimiterRegistry.custom()
  .withRegistryStore(new CacheRateLimiterRegistryStore())
  .build();
```

# リンク
- [トップページ](../index.md)
- [RateLimiter](ratelimiter.md)
