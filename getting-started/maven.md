Maven
==============
Mavenでの利用

# リリース
このプロジェクトにはJDK 8が必要です。プロジェクトはJCenterおよびMaven Centralで公開されています。Mavenを使う場合は、Resilience4jモジュールを下記のように含めることが出来ます。

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-circuitbreaker</artifactId>
    <version>${resilience4jVersion}</version>
</dependency>
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-ratelimiter</artifactId>
    <version>${resilience4jVersion}</version>
</dependency>
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-retry</artifactId>
    <version>${resilience4jVersion}</version>
</dependency>
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-bulkhead</artifactId>
    <version>${resilience4jVersion}</version>
</dependency>
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-cache</artifactId>
    <version>${resilience4jVersion}</version>
</dependency>
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-timelimiter</artifactId>
    <version>${resilience4jVersion}</version>
</dependency>
```

# スナップショット

```xml
<repositories>
    <repository>
      <id>jcenter-snapshots</id>
      <name>jcenter</name>
      <url>http://oss.jfrog.org/artifactory/oss-snapshot-local/</url>
    </repository>
</repositories>
```

# リンク
- [トップページ](../index.md)
