Gradle
=============
Gradleでの利用

# リリース
このプロジェクトにはJDK 8が必要です。プロジェクトはJCenterおよびMaven Centralで公開されています。Gradleを使う場合は、Resilience4jモジュールを下記のように含めることが出来ます。

```groovy
repositories {
    jCenter()
}

dependencies {
  compile "io.github.resilience4j:resilience4j-circuitbreaker:${resilience4jVersion}"
  compile "io.github.resilience4j:resilience4j-ratelimiter:${resilience4jVersion}"
  compile "io.github.resilience4j:resilience4j-retry:${resilience4jVersion}"
  compile "io.github.resilience4j:resilience4j-bulkhead:${resilience4jVersion}"
  compile "io.github.resilience4j:resilience4j-cache:${resilience4jVersion}"
  compile "io.github.resilience4j:resilience4j-timelimiter:${resilience4jVersion}"
}
```

# スナップショット

```groovy
repositories {
   maven { url 'http://oss.jfrog.org/artifactory/oss-snapshot-local/' }
}
```