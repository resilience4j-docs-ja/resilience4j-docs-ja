バルクヘッド
==========
resilience4j-bulkheadの利用

[トップページに戻る](../index.md)

# イントロダクション
Resilience4jは、同時実行数を制御するために使われるバルクヘッドパターンの実装を2つ提供します。

- セマフォを利用する `SemaphoreBulkhead`
- 境界づけられたキューと固定スレッドプールを利用する `FixedThreadPoolBulkhead`

`SemaphoreBulkhead` は様々なスレッドとI/Oモデルで動作します。Hystrixと違うセマフォベースで、"シャドウ"スレッドプールの選択肢を提供しません。スレッドプールのサイズがバルクヘッドの設定と一貫性があるようにすることは、クライアントの責任です。

# BulkheadRegistryの作成
CircuitBreakerモジュールと同様に、このモジュールはインメモリーの `BulkheadRegistry` と `ThreadPoolBulkheadRegistry` を提供します。これを利用して、Bulkheadインスタンスの管理（作成および取得）を行うことができます。

```java
BulkheadRegistry bulkheadRegistry = BulkheadRegistry.ofDefaults();

ThreadPoolBulkheadRegistry threadPoolBulkheadRegistry = 
  ThreadPoolBulkheadRegistry.ofDefaults();
```

# Bulkheadの作成と設定
カスタムのグローバルBulkheadConfigを作ることができます。カスタムのグローバルBulkheadConfigを作成するには、BulkheadConfigビルダーを利用できます。ビルダーでは下記のプロパティを設定できます。

| プロパティ | デフォルト値 | 説明 |
|-----|------------|-----|
| maxConcurrentCalls | 25 | バルクヘッドにより許可される最大並行実行数 |
| maxWaitDuration | 0 | 飽和したバルクヘッドに入ろうとした時のスレッドの最大待ち時間 |

```java
// バルクヘッドのカスタム設定を作成する
BulkheadConfig config = BulkheadConfig.custom()
  	.maxConcurrentCalls(150)
  	.maxWaitDuration(Duration.ofMillis(500))
  	.build();

// カスタムのグローバル設定でBulkheadRegistryを作成する
BulkheadRegistry registry = BulkheadRegistry.of(config);

// レジストリーからバルクヘッドを取得または作成する
// （バルクヘッドはデフォルト設定を持つ）
Bulkhead bulkheadWithDefaultConfig = registry.bulkhead("name1");

// レジストリーからバルクヘッドを取得または作成する
// （バルクヘッド作成時にはカスタムの設定を使う）
Bulkhead bulkheadWithCustomConfig = registry.bulkhead("name2", custom);
```

# ThreadPoolBulkheadの作成および設定
カスタムのグローバルThreadPoolBulkheadConfigを作ることができます。カスタムのグローバルThreadPoolBulkheadConfigを作成するには、ThreadPoolBulkheadConfigビルダーを利用できます。ビルダーでは下記のプロパティを設定できます。

| プロパティ | デフォルト値 | 説明 |
|-----|------------|-----|
| maxThreadPoolSize | Runtime.getRuntime().availableProcessors() | 最大スレッドプールサイズを設定します。 |
| coreThreadPoolSize | Runtime.getRuntime().availableProcessors() - 1 | コアスレッドプールサイズを設定します。 |
| queueCapacity | 100 | キューの容量を設定します。 |
| keepAliveDuration | 20 [ms] | スレッド数がコアを超えた場合、これは余分な待機スレッドが終了する前に新しいタスクを待機する最大時間です。 |

```java
ThreadPoolBulkheadConfig config = ThreadPoolBulkheadConfig.custom()
  .maxThreadPoolSize(10)
  .coreThreadPoolSize(2)
  .queueCapacity(20)
  .build();
        
// カスタムのグローバル設定でBulkheadRegistryを作成する
ThreadPoolBulkheadRegistry registry = ThreadPoolBulkheadRegistry.of(config);

// レジストリーからThreadPoolBulkheadを取得または作成する
// （バルクヘッドはデフォルト設定を持つ）
ThreadPoolBulkhead bulkheadWithDefaultConfig = registry.bulkhead("name1");

// レジストリーからThreadPoolBulkheadを取得または作成する
// （バルクヘッド作成時にはカスタムの設定を使う）
ThreadPoolBulkheadConfig custom = BulkheadConfig.custom()
  .maxThreadPoolSize(5)
  .build();

ThreadPoolBulkhead bulkheadWithCustomConfig = registry.bulkhead("name2", custom);
```

# 関数型インタフェースのデコレートと実行
ご想像の通り、サーキットブレイカーのようにバルクヘッドはあらゆる種類の高階デコレーター関数があります。バルクヘッドでデコレートできるのは `Callable` 、 `Supplier` 、 `Runnable` 、 `Consumer` 、 `CheckedRunnable` 、 `CheckedSupplier` 、 `CheckedConsumer` 、 `CompletionStage` です。

```java
// 与えられた設定
Bulkhead bulkhead = Bulkhead.of("name", config);

// 関数をデコレートする
CheckedFunction0<String> decoratedSupplier = Bulkhead
  .decorateCheckedSupplier(bulkhead, () -> "This can be any method which returns: 'Hello");

// 他の関数をmapでチェインする
Try<String> result = Try.of(decoratedSupplier)
  .map(value -> value + " world'");

// 全関数が成功すると、TryモナドはSuccess<String>を返す
assertThat(result.isSuccess()).isTrue();
assertThat(result.get()).isEqualTo("This can be any method which returns: 'Hello world'");
assertThat(bulkhead.getMetrics().getAvailableConcurrentCalls()).isEqualTo(1);
```

```java
ThreadPoolBulkheadConfig config = ThreadPoolBulkheadConfig.custom()
    .maxThreadPoolSize(10)
    .coreThreadPoolSize(2)
    .queueCapacity(20)
    .build();

ThreadPoolBulkhead bulkhead = ThreadPoolBulkhead.of("name", config);

CompletionStage<String> supplier = ThreadPoolBulkhead
    .executeSupplier(bulkhead, backendService::doSomething);
```

# 発行されたRegistryEventsの消費
BulkheadRegistryにイベントコンシューマーを登録し、Bulkheadが作成・置き換え・削除された際に処理を実行できます。

```java
BulkheadRegistry registry = BulkheadRegistry.ofDefaults();
registry.getEventPublisher()
  .onEntryAdded(entryAddedEvent -> {
    Bulkhead addedBulkhead = entryAddedEvent.getAddedEntry();
    LOG.info("Bulkhead {} added", addedBulkhead.getName());
  })
  .onEntryRemoved(entryRemovedEvent -> {
    Bulkhead removedBulkhead = entryRemovedEvent.getRemovedEntry();
    LOG.info("Bulkhead {} removed", removedBulkhead.getName());
  });
```

# 発行されたBulkheadEventsの消費
バルクヘッドはBulkheadEventのストリームを発行します。発行されるイベントは2種類あります: 実行の許可、実行の拒否、実行の完了です。これらのイベントを消費したい場合は、イベントコンシューマーを登録する必要があります。

```java
bulkhead.getEventPublisher()
    .onCallPermitted(event -> logger.info(...))
    .onCallRejected(event -> logger.info(...))
    .onCallFinished(event -> logger.info(...));
```

# リンク
- [トップページ](../index.md)
- [Bulkheadのサンプルコード](bulkhead-examples.md)
