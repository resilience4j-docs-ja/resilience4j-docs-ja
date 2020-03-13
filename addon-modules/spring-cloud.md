Getting Started
===============
resilience4j-spring-cloud2の利用

[トップページに戻る](../index.md)

# 準備
Resilience4jのSpring Cloud 2 Starterをコンパイル依存性に追加してください。Spring Cloud 2 Starterにより、Spring Cloud Configを利用した設定の一括管理や実行中の設定リフレッシュが可能になります。

このモジュールは、実行時に `org.springframework.boot:spring-boot-starter-actuator` および `org.springframework.boot:spring-boot-starter-aop` が既に提供されていることを期待します。

```groovy
repositories {
    jCenter()
}

dependencies {
    compile "io.github.resilience4j:resilience4j-spring-cloud2:${resilience4jVersion}"
    compile('org.springframework.boot:spring-boot-starter-actuator')
    compile('org.springframework.boot:spring-boot-starter-aop')
    compile('org.springframework.cloud:spring-cloud-starter-config')  
}
```

設定はSpring Boot 2 Starterと似ています。

# デモ
Spring Cloud 2における準備および利用は[demo](https://github.com/resilience4j/resilience4j-spring-cloud2-demo)で紹介されています。

# リンク
- [トップページ](../index.md)

