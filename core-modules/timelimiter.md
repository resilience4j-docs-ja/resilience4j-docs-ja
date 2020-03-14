TimeLimiter
===========
resilience4j-timelimiterの利用

[トップページに戻る](../index.md)

# TimeLimiterRegistryの作成
CircuitBreakerモジュールと同様に、このモジュールはインメモリーの `TimeLimiterRegistry` を提供します。これを利用して、TimeLimiterインスタンスの管理（作成および取得）を行うことができます。

```java
TimeLimiterRegistry timeLimiterRegistry = TimeLimiterRegistry.ofDefaults();
```

# TimeLimiterの作成および設定
カスタムのグローバルTimeLimiterConfigを作ることができます。カスタムのグローバルTimeLimiterConfigを作成するには、TimeLimiterConfigビルダーを利用できます。ビルダーでは下記のプロパティを設定できます:

- タイムアウト時間
- 実行中のFutureがキャンセルされるべきか否か

```java
TimeLimiterConfig config = TimeLimiterConfig.custom()
   .cancelRunningFuture(true)
   .timeoutDuration(Duration.ofMillis(500))
   .build();

// カスタムのグローバル設定でTimeLimiterRegistryを作成する
TimeLimiterRegistry timeLimiterRegistry = TimeLimiterRegistry.of(config);

// TimeLimiterRegistryからTimeLimiterを作成（デフォルトの設定）
TimeLimiter timeLimiterWithDefaultConfig = registry.timeLimiter("name1");

// TimeLimiterRegistryからTimeLimiterを作成（カスタムの設定）
TimeLimiterConfig config = TimeLimiterConfig.custom()
   .cancelRunningFuture(false)
   .timeoutDuration(Duration.ofMillis(1000))
   .build();

TimeLimiter timeLimiterWithCustomConfig = registry.retry("name2", custom);
```

# 関数型インタフェースのデコレートおよび実行
TimeLimiterは、実行時間を制限するために `CompletionStage` または `Future` をデコレートする高階デコレーター関数を持っています。

```java
// 時間がかかるhelloWorldService.sayHelloWorld()メソッドがあると仮定
HelloWorldService helloWorldService = mock(HelloWorldService.class);

// TimeLimiterの作成
TimeLimiter timeLimiter = TimeLimiter.of(Duration.ofSeconds(1));
// ノンブロッキングなCompletableFutureにタイムアウトをスケジュールするために必要なスケジューラー
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(3);

// CompletableFutureによるノンブロッキング
CompletableFuture<String> result = timeLimiter.executeCompletionStage(
  scheduler, () -> CompletableFuture.supplyAsync(helloWorldService::sayHelloWorld)).toCompletableFuture();

// future.get(timeoutDuration, MILLISECONDS)によるブロッキング
String result = timeLimiter.executeFutureSupplier(
  () -> CompletableFuture.supplyAsync(() -> helloWorldService::sayHelloWorld));
```

# リンク
- [トップページ](../index.md)
