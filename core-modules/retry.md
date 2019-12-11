リトライ
=======
resilience4j-retryの利用

[トップページに戻る](../index.md)

# RetryRegistryの作成
CircuitBreakerモジュールと同様に、このモジュールはインメモリーの `RetryRegistry` を提供します。これを利用して、Retryインスタンスの管理（作成および取得）を行うことができます。

```java
RetryRegistry retryRegistry = RetryRegistry.ofDefaults();
```

# Retryの作成および設定
カスタムのグローバルRetryConfigを作ることができます。カスタムのグローバルRetryConfigを作成するには、RateLimiterConfigビルダーを利用できます。ビルダーでは下記のプロパティを設定できます:

- 最大リトライ回数
- 試行間の待ち時間
- どのようなレスポンスがリトライをするべきか評価するカスタムPredicate
- どのような例外がリトライをするべきか評価するカスタムPredicate
- リトライするべき例外のリスト
- 無視してリトライすべきでない例外のリスト

| プロパティ | デフォルト値 | 説明 |
|-----|------------|-----|
| maxAttempts | 3 | 最大リトライ回数 |
| waitDuration | 500 [ms] | 試行間の待ち時間 |
| intervalFunction | numOfAttempts -> waitDuration | 失敗後の待ち時間間隔を変更する関数。デフォルトでは一定のwaitDuration |
| retryOnResultPredicate | result -> false | リトライされるべき結果か評価するPredicateを設定する。Predicateは、結果がリトライされるべき場合にtrue、そうでなければfalseを返す |
| retryOnExceptionPredicate | throwable -> true | リトライされるべき例外か評価するPredicateを設定する。Predicateは、例外がリトライされるべき場合にtrue、そうでなければfalseを返す |
| retryExceptions | empty | 失敗と記録されリトライされるべきエラークラスのリストを設定する |
| ignoreExceptions | empty | 無視されリトライされないべきエラークラスのリストを設定する |

```java
RetryConfig config = RetryConfig.custom()
  .maxAttempts(2)
  .waitDuration(Duration.ofMillis(1000))
  .retryOnResult(response -> response.getStatus() == 500)
  .retryOnException(e -> e instanceof WebServiceException)
  .retryExceptions(IOException.class, TimeoutException.class)
  .ignoreExceptions(BunsinessException.class, OtherBunsinessException.class)
  .build();

// カスタムのグローバル設定でRetryRegistryを作成する
RetryRegistry registry = RetryRegistry.of(config);

// レジストリーからRetryを取得または作成する
// （Retryはデフォルト設定を持つ）
Retry retryWithDefaultConfig = registry.retry("name1");

// レジストリーからBulkheadを取得または作成する
// （バルクヘッド作成時にはカスタムの設定を使う）
RetryConfig custom = RetryConfig.custom()
    .waitDuration(Duration.ofMillis(100))
    .build();

Retry retryWithCustomConfig = registry.retry("name2", custom);
```

# 関数型インタフェースのデコレートと実行
ご想像の通り、サーキットブレイカーのようにRetryはあらゆる種類の高階デコレーター関数があります。Retryでデコレートできるのは `Callable` 、 `Supplier` 、 `Runnable` 、 `Consumer` 、 `CheckedRunnable` 、 `CheckedSupplier` 、 `CheckedConsumer` 、 `CompletionStage` です。

```java
// 例外をスローするHelloWorldServiceがあると仮定する
HelloWorldService  helloWorldService = mock(HelloWorldService.class);
given(helloWorldService.sayHelloWorld())
  .willThrow(new WebServiceException("BAM!"));

// デフォルト設定でRetryを作成する
Retry retry = Retry.ofDefaults("id");
// Decorate the invocation of the HelloWorldService
CheckedFunction0<String> retryableSupplier = Retry
  .decorateCheckedSupplier(retry, helloWorldService::sayHelloWorld);

// 関数を実行する
Try<String> result = Try.of(retryableSupplier)
  .recover((throwable) -> "Hello world from recovery function");

// helloWorldServiceは3回実行される
BDDMockito.then(helloWorldService).should(times(3)).sayHelloWorld();
// 例外はrecovery関数によってハンドリングされる
assertThat(result.get()).isEqualTo("Hello world from recovery function");
```

# 発行されたRegistryEventsの消費
RetryRegistryにイベントコンシューマーを登録し、Retryが作成・置き換え・削除された際に処理を実行できます。

```java
RetryRegistry registry = RetryRegistry.ofDefaults();
registry.getEventPublisher()
  .onEntryAdded(entryAddedEvent -> {
    Retry addedRetry = entryAddedEvent.getAddedEntry();
    LOG.info("Retry {} added", addedRetry.getName());
  })
  .onEntryRemoved(entryRemovedEvent -> {
    Retry removedRetry = entryRemovedEvent.getRemovedEntry();
    LOG.info("Retry {} removed", removedRetry.getName());
  });
```

# カスタムIntervalFunctionの利用
リトライ試行間の待ち時間間隔を一定にしたくない場合は、代わりに `IntervalFunction` を使って全試行の待ち時間間隔を計算することができます。Resilience4jは `IntervalFunction` の単純な生成を行ういくつかのファクトリーメソッドを提供しています。

```java
IntervalFunction defaultWaitInterval = IntervalFunction
  .ofDefaults();

// このIntervalFunctionは待ち時間間隔の設定のみ行う
IntervalFunction fixedWaitInterval = IntervalFunction
  .of(Duration.ofSeconds(5));

IntervalFunction intervalWithExponentialBackoff = IntervalFunction
  .ofExponentialBackoff();

IntervalFunction intervalWithCustomExponentialBackoff = IntervalFunction
  .ofExponentialBackoff(IntervalFunction.DEFAULT_INITIAL_INTERVAL, 2d);

IntervalFunction randomWaitInterval = IntervalFunction
  .ofRandomized();

// デフォルトのIntervalFunctionをカスタムのもので上書きする
RetryConfig retryConfig = RetryConfig.custom()
  .intervalFunction(intervalWithExponentialBackoff)
  .build();
```

# リンク
- [トップページ](../index.md)
- [Retryのサンプルコード](retry-examples.md)