RateLimiter
===========
resilience4j-ratelimiterの利用

[トップページに戻る](../index.md)

# イントロダクション
流量制御はAPIのスケールおよびサービスの高い可用性、信頼性の確立に必須の技術です。それだけではなく、どうやって検知された制限超過をハンドリングするか、どのようなリクエストを制限するかなど、とても多くの異なる選択肢が付属しています。単に制限を超えたリクエストを拒否することもできますし、後で実行できるようにキューを作ることも、これら2つのアプローチを何らかの方法で組み合わせることもできます。

## 内部
Resilience4jは、エポック開始からの全ナノ秒をサイクルに分割するRateLimiterを提供します。各サイクルは `RateLimiterConfig.limitRefreshPeriod` で設定された間隔を持っています。各サイクルの最初では、RateLimiterは有効なパーミッション数を `RateLimiterConfig.limitForPeriod` にセットします。RateLimiterの呼び出し元には実際にはこのように見えます。しかし、 `AtomicRateLimiter` 実装には、RateLimiterがアクティブに使われていない場合にこのリフレッシュはスキップする念入りな最適化があります。

![RateLimiterの内部](../images/rate_limiter.png "RateLimiterの内部")

RateLimiterのデフォルト実装である `AtomicRateLimiter` は、自身の状態をAtomicReferenceによって管理します。AtomicRateLimiter.Stateは完全にイミュータブルかつ下記のフィールドを持ちます:

- activeCycle - 最後の呼び出しで使われたサイクル番号
- activePermissions - 最後の呼び出し後に利用可能なパーミッション数。パーミッションが予約されている場合は負になる
- nanosToWait - 最後の呼び出しのパーミッションを待つ時間（ナノ秒）

`SemaphoreBasedRateLimiter` も用意されています。これはセマフォと、各 `RateLimiterConfig#limitRefreshPeriod` 後にパーミッションをリフレッシュするスケジューラーを使っています。

# RateLimiterRegistryの作成
CircuitBreakerモジュールと同様に、このモジュールはインメモリーの `RateLimiterRegistry` を提供します。これを利用して、RateLimiterインスタンスの管理（作成および取得）を行うことができます。

```java
RateLimiterRegistry rateLimiterRegistry = RateLimiterRegistry.ofDefaults();
```

# RateLimiterの作成および設定
カスタムのグローバルRateLimiterConfigを作ることができます。カスタムのグローバルRateLimiterConfigを作成するには、RateLimiterConfigビルダーを利用できます。ビルダーでは下記のプロパティを設定できます。

| プロパティ | デフォルト値 | 説明 |
|-----|------------|-----|
| timeoutDuration | 5 [s] | スレッドがパーミッションを待つデフォルトの待ち時間 |
| limitRefreshPeriod | 500 [ns] | 制限をリフレッシュする期間。各期間の後、RateLimiterは自身のパーミッション数をlimitForPeriodの値に戻す |
| limitForPeriod | 50 | 1つの制限リフレッシュ期間の間に利用可能なパーミッション数 |

メソッドの呼び出し率を10リクエスト/msを超えないように制限する例です。

```java
RateLimiterConfig config = RateLimiterConfig.custom()
  .limitRefreshPeriod(Duration.ofMillis(1))
  .limitForPeriod(10)
  .timeoutDuration(Duration.ofMillis(25))
  .build();

// Registryを作成する
RateLimiterRegistry rateLimiterRegistry = RateLimiterRegistry.of(config);

// Registryを利用する
RateLimiter rateLimiterWithDefaultConfig = rateLimiterRegistry
  .rateLimiter("name1");

RateLimiter rateLimiterWithCustomConfig = rateLimiterRegistry
  .rateLimiter("name2", config);
```

# 関数型インタフェースのデコレートと実行
ご想像の通り、サーキットブレイカーのようにRateLimiterはあらゆる種類の高階デコレーター関数があります。RateLimiterでデコレートできるのは `Callable` 、 `Supplier` 、 `Runnable` 、 `Consumer` 、 `CheckedRunnable` 、 `CheckedSupplier` 、 `CheckedConsumer` 、 `CompletionStage` です。

```java
// BackendService.doSomething()の呼び出しをデコレートする
CheckedRunnable restrictedCall = RateLimiter
    .decorateCheckedRunnable(rateLimiter, backendService::doSomething);

Try.run(restrictedCall)
    .andThenTry(restrictedCall)
    .onFailure((RequestNotPermitted throwable) -> LOG.info("Wait before call it again :)"));
```

changeTimeoutDurationおよびchangeLimitForPeriodメソッドで、実行時にRateLimiterのパラメーターを変更できます。新しいタイムアウト間隔（timeoutDuration）は現在パーミッション待ちのスレッドに影響しません。新しいリミット（limitForPeriod）は現在間隔のパーミッションには影響せず、次のものにのみ適用されます。

```java
// BackendService.doSomething()の呼び出しをデコレートする
CheckedRunnable restrictedCall = RateLimiter
    .decorateCheckedRunnable(rateLimiter, backendService::doSomething);

// 次回のリフレッシュサイクルでは、RateLimiterのパーミッション数は100
rateLimiter.changeLimitForPeriod(100);
```

# 発行されたRegistryEventsの消費
RateLimiterRegistryにイベントコンシューマーを登録し、RateLimiterが作成・置き換え・削除された際に処理を実行できます。

```java
RateLimiterRegistry registry = RateLimiterRegistry.ofDefaults();
registry.getEventPublisher()
  .onEntryAdded(entryAddedEvent -> {
    RateLimiter addedRateLimiter = entryAddedEvent.getAddedEntry();
    LOG.info("RateLimiter {} added", addedRateLimiter.getName());
  })
  .onEntryRemoved(entryRemovedEvent -> {
    RateLimiter removedRateLimiter = entryRemovedEvent.getRemovedEntry();
    LOG.info("RateLimiter {} removed", removedRateLimiter.getName());
  });
```

# 発行されたRateLimiterEventsの消費
RateLimiterはRateLimiterEventのストリームを発行します。イベントはパーミッション取得成功および取得失敗です。全てのイベントは、イベント作成日時やRateLimiter名などの追加情報を含んでいます。これらのイベントを消費したい場合は、イベントコンシューマーを登録する必要があります。

```java
rateLimiter.getEventPublisher()
    .onSuccess(event -> logger.info(...))
    .onFailure(event -> logger.info(...));
```

EventPublisherをReactive Streamに変換するために、RxJava、RxJava2、Project Reactorアダプターを利用できます。

```java
ReactorAdapter.toFlux(rateLimiter.getEventPublisher())
    .filter(event -> event.getEventType() == FAILED_ACQUIRE)
    .subscribe(event -> logger.info(...))
```

# リンク
- [トップページ](../index.md)
- [RateLimiterのサンプルコード](ratelimiter-examples.md)