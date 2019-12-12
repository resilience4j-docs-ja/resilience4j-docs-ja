サンプルコード
============
resilience4j-bulkheadのサンプルコード

[トップページに戻る](../index.md)

# BulkheadRegistryの作成
カスタムのBulkheadConfigを利用したBulkheadRegistryの作成

```java
// バルクヘッドのカスタム設定を作成する
BulkheadConfig config = BulkheadConfig.custom()
        .maxConcurrentCalls(10)
        .maxWaitDuration(Duration.ofMillis(1))
        .build();

// カスタムのグローバル設定でBulkheadRegistryを作成する
BulkheadRegistry bulkheadRegistry =
        BulkheadRegistry.of(config);
```

# Bulkheadの作成
グローバルなデフォルト設定を持ったBulkheadRegistryからのBulkhead取得

```java
Bulkhead bulkhead = bulkheadRegistry 
  .bulkhead("name");
```

# 関数型インタフェースのデコレート
`BackendService.doSomething()` の呼び出しをBulkheadでデコレートし、デコレートされたSupplierを実行、全ての例外からリカバリーする

```java
Supplier<String> decoratedSupplier = Bulkhead
    .decorateSupplier(retry, backendService::doSomething);

String result = Try.ofSupplier(decoratedSupplier)
    .recover(throwable -> "Hello from Recovery").get(); 
```

# リンク
- [トップページ](../index.md)
- [Bulkhead](bulkhead.md)
