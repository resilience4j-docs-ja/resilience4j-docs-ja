サンプルコード
============
resilience4j-retryのサンプルコード

[トップページに戻る](../index.md)

# RetryRegistryの作成
カスタムのRetryConfigでRetryRegistryを作成します。

```java
RetryConfig config = RetryConfig.custom()
    .maxAttempts(2)
    .waitDuration(Duration.ofMillis(100))
    .retryOnResult(response -> response.getStatus() == 500)
    .retryOnException(e -> e instanceof WebServiceException)
    .retryExceptions(IOException.class, TimeoutException.class)
    .ignoreExceptions(BunsinessException.class, OtherBunsinessException.class)
    .build();
    
// カスタムのグローバル設定でRetryRegistryを作成する
RetryRegistry registry = RetryRegistry.of(config);
```

# リンク
- [トップページ](../index.md)
- [Retry](retry.md)