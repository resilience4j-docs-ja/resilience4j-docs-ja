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

# リンク
- [トップページ](../index.md)
- [RateLimiter](ratelimiter.md)
