サンプルコード
============
resilience4j-ratelimiterのサンプルコード

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
