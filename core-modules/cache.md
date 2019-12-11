キャッシュ
========
resilience4j-cacheの利用

# キャッシュの作成および設定
下記の例は、キャッシュ抽象化でラムダ式をデコレートする方法を示しています。キャッシュ抽象化はラムダ式の結果をキャッシュインスタンス（JCache）します。そして、ラムダ式実行の前にキャッシュから前回キャッシュされた結果の取得を試みます。分散キャッシュからのキャッシュ取得に失敗した場合、例外がスローされラムダ式が呼び出されます。

```java
// JCacheをラップしたCacheContextインスタンスを生成する
javax.cache.Cache<String, String> cacheInstance = Caching
  .getCache("cacheName", String.class, String.class);
Cache<String, String> cacheContext = Cache.of(cacheInstance);

// BackendService.doSomething()をデコレートする
CheckedFunction1<String, String> cachedFunction = Decorators
    .ofCheckedSupplier(() -> backendService.doSomething())
    .withCache(cacheContext)
    .decorate();
String value = Try.of(() -> cachedFunction.apply("cacheKey")).get();
```

# 発行されたCacheEventの消費
キャッシュはCacheEventのストリームを発行します。イベントはキャッシュのヒット、キャッシュのミス、エラーです。

```java
cacheContext.getEventPublisher()
    .onCacheHit(event -> logger.info(...))
    .onCacheMiss(event -> logger.info(...))
    .onError(event -> logger.info(...));
```

# Ehcacheの例

```groovy:build.gradle
compile 'org.ehcache:ehcache:3.7.1'
```

```java
// キャッシュを設定する（一度）
this.cacheManager = Caching.getCachingProvider().getCacheManager();
this.cache = Cache.of(cacheManager
    .createCache("booksCache", new MutableConfiguration<>()));

// キャッシュを使ってBookを取得
List<Book> books = Cache.decorateSupplier(cache, library::getBooks)
    .apply(BOOKS_CACHE_KEY);
```

> 並行性の問題があるため、JCacheの参照実装を本番で使うことは非推奨です。Ehcache、Caffeine、Redisson、Hazelcast、Igniteなど他の実装を使ってください。