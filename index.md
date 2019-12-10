Java™向けフォールトトレランスライブラリResilience4j
====================================================
Resilience4jは、Netflix Hystrixにインスパイアされた軽量なフォールトトレランスライブラリですが、関数型プログラミング向けにデザインされています。このライブラリはVavrしか使っておらず、その他の外部依存ライブラリが無いため、軽量です。それに対してNetflix Hystrixは、コンパイル依存性にArcaiusがあります。これは、GuavaやApache Commons Configurationのような多くの外部依存ライブラリを持っています。

Resilience4jは関数型インタフェース、ラムダ式、メソッド参照をサーキットブレイカー、流量制限、リトライ、バルクヘッドで拡張するための高階関数（デコレーター）を提供します。関数型インタフェース、ラムダ式、メソッド参照には、1つ以上のデコレーターをスタックすることができます。これにより、必要なデコレーターを使うか、何も使わないかを選ぶことができます。

<!-- FIXME 「スタックする」をうまい日本語にする-->

```java
Supplier<String> supplier = () -> backendService.doSomething(param1, param2);

Supplier<String> decoratedSupplier = Decorators.ofSupplier(supplier)
  .withRetry(Retry.ofDefaults("name"))
  .withCircuitBreaker(CircuitBreaker.ofDefaults("name"))
  .withBulkhead(Bulkhead.ofDefaults("name"));  

String result = Try.ofSupplier(decoratedSupplier)
  .recover(throwable -> "Hello from Recovery").get();
```

Resilience4jでは、全てを使う必要はありません。必要なものだけを選ぶことができます。