サンプルコード
============
resilience4j-circuitbreakerのサンプルコード

[トップページに戻る](../index.md)

# CircuitBreakerRegistryの作成
カスタムのCircuitBreakerConfigを利用したCircuitBreakerRegistryの作成

```java
// CircuitBreakerのためのカスタムCircuitBreakerConfigを作成する
CircuitBreakerConfig circuitBreakerConfig = CircuitBreakerConfig.custom()
    .failureRateThreshold(50)
    .waitDurationInOpenState(Duration.ofMillis(1000))
    .permittedNumberOfCallsInHalfOpenState(2)
    .slidingWindowSize(2)
    .recordExceptions(IOException.class, TimeoutException.class)
    .ignoreExceptions(BusinessException.class, OtherBusinessException.class)
    .build();

// カスタムのグローバル設定でCircuitBreakerRegistryを作成する
CircuitBreakerRegistry circuitBreakerRegistry =
  CircuitBreakerRegistry.of(circuitBreakerConfig);
```

# CircuitBreakerの作成
グローバルなデフォルト設定を持ったCircuitBreakerRegistryからのCircuitBreaker取得

```java
CircuitBreaker circuitBreaker = circuitBreakerRegistry
  .circuitBreaker("name");
```

# 関数型インタフェースのデコレート
`BackendService.doSomething()` の呼び出しをCircuitBreakerでデコレートし、デコレートされたSupplierを実行、全ての例外からリカバリーする

```java
Supplier<String> decoratedSupplier = CircuitBreaker
    .decorateSupplier(circuitBreaker, backendService::doSomething);

String result = Try.ofSupplier(decoratedSupplier)
    .recover(throwable -> "Hello from Recovery").get(); 
```

# デコレートされた関数型インタフェースの実行
ラムダ式をデコレートしたくないが、単に実行およびCircuitBreakerによる呼び出しの保護をしたい場合

```java
String result = circuitBreaker
  .executeSupplier(backendService::doSomething);
```

# 例外からのリカバリー
CircuitBreakerが例外を失敗と記録しそれからリカバリーしたい場合は、 `Try.recover()` でメソッドをチェインできます。recoveryメソッドは `Try.of()` が `Failure<Throwable>` モナドを返した時のみ実行されます。

```java
// 与えられた設定
CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("testName");

// 関数をデコレートして実行する
CheckedFunction0<String> checkedSupplier =
  CircuitBreaker.decorateCheckedSupplier(circuitBreaker, () -> {
    throw new RuntimeException("BAM!");
});
Try<String> result = Try.of(checkedSupplier)
        .recover(throwable -> "Hello Recovery");

// 関数は成功する（例外がリカバリーされるため）
assertThat(result.isSuccess()).isTrue();
// 結果はリカバリー関数の戻り値と一致する
assertThat(result.get()).isEqualTo("Hello Recovery");
```

CircuitBreakerが例外を失敗と記録する前にリカバリーしたい場合は、下記のようにできます。

```java
Supplier<String> supplier = () -> {
            throw new RuntimeException("BAM!");
        };

Supplier<String> supplierWithRecovery = SupplierUtils
  .recover(supplier, (exception) -> "Hello Recovery");

String result = circuitBreaker.executeSupplier(supplierWithRecovery);

assertThat(result).isEqualTo("Hello Recovery");
```

`SupplierUtils` と `CallableUtils` は、関数をチェインするための `andThen` などのメソッドを含んでいます。例えば、例外をスローできるように、HTTPレスポンスのステータスコードを確認します。

```java
Supplier<String> supplierWithResultAndExceptionHandler = SupplierUtils
  .andThen(supplier, (result, exception) -> "Hello Recovery");

Supplier<HttpResponse> supplier = () -> httpClient.doRemoteCall();
Supplier<HttpResponse> supplierWithResultHandling = SupplierUtils.andThen(supplier, result -> {
    if (result.getStatusCode() == 400) {
       throw new ClientException();
    } else if (result.getStatusCode() == 500) {
       throw new ServerException();
    }
    return result;
});
HttpResponse httpResponse = circuitBreaker
  .executeSupplier(supplierWithResultHandling);
```

# CircuitBreakerのリセット
サーキットブレイカーは元の状態へのリセット、全メトリクスの消去、実質的にスライディングウィンドウをリセットすることをサポートしています。

```java
CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("testName");
circuitBreaker.reset();
```

# 手動での状態遷移

```java
CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("testName");
circuitBreaker.transitionToDisabledState();
// circuitBreaker.onFailure(...) は状態変更を引き起こしません
circuitBreaker.transitionToClosedState(); // CLOSEDに遷移して通常の振る舞いを再び可能にします。メトリクスは保持されます
circuitBreaker.transitionToForcedOpenState();
// circuitBreaker.onSuccess(...) は状態変更を引き起こしません
circuitBreaker.reset(); // CLOSEDに遷移して通常の振る舞いを再び可能にします。メトリクスは失われます
```

# リンク
- [トップページ](../index.md)
- [CircuitBreaker](circuitbreaker.md)
