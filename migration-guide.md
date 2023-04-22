この文書は、あなたが所持するアプリケーションをSpring Boot 3.0に移行するための支援をすることを目的としています。

# Before You Start
## Upgrade to the Latest 2.7.x Version
アップグレードを開始する前に、利用可能な最新の 2.7.x バージョンにアップグレードすることを確認してください。これにより、そのラインの最新の依存関係に対してビルドしていることが確認されます。

## Review Dependencies
Spring Boot 3への移行により、多くの依存関係がアップグレードされ、あなたの側で作業が必要になる可能性があります。[2.7.xの依存関係管理](https://docs.spring.io/spring-boot/docs/2.7.x/reference/html/dependency-versions.html#appendix.dependency-versions)と[3.0.xの依存関係管理](https://docs.spring.io/spring-boot/docs/3.0.x/reference/html/dependency-versions.html#appendix.dependency-versions)を確認し、あなたのプロジェクトがどのように影響されるかを判断することができます。

また、Spring Bootで管理されていない依存関係（例：Spring Cloud）を使用することもあります。プロジェクトではそれらに対して明示的なバージョンを定義しているので、アップグレードの前にまず互換性のあるバージョンを特定する必要があります。

## Spring Security
Spring Boot 3.0はSpring Security 6.0を使用しています。Spring Securityチームは、Spring Security 6.0へのアップグレードを簡素化するためにSpring Security 5.8をリリースしました。Spring Boot 3.0にアップグレードする前に、Spring Boot 2.7アプリケーションをSpring Security 5.8にアップグレードすることを検討してください。Spring Securityチームは、そのための[移行ガイド](https://docs.spring.io/spring-security/reference/5.8/migration/index.html)を作成しました。そこから、Spring Boot 3.0にアップグレードする際には、[5.8から6.0への移行ガイド](https://docs.spring.io/spring-security/reference/migration/index.html)に従うことができます。

### Dispatch types
Servletアプリケーションでは、Spring Security 6.0はすべてのディスパッチタイプに認可を適用します。これに合わせて、Spring BootではディスパッチタイプごとにSpring Securityのフィルタが呼び出されるように設定されるようになりました。タイプは`spring.security.filter.dispatcher-types`プロパティを使って設定できます。

## Review System Requirements
Spring Boot 3.0は、Java 17以降が必要です。Java 8はサポートされなくなりました。また、Spring Framework 6.0が必要です。

## Review Deprecations from Spring Boot 2.x
Spring Boot 2.xで非推奨となったクラス、メソッド、プロパティは、このリリースで削除されました。アップグレードする前に、非推奨のメソッドを呼び出していないことを確認してください。

# Upgrade to Spring Boot 3
プロジェクトとその依存関係の状態を確認したら、Spring Boot 3.0の最新のメンテナンスリリースにアップグレードしてください。

## Configuration Properties Migration
Spring Boot 3.0では、いくつかの設定プロパティが改名/削除され、開発者はそれに応じて`application.properties/application.yml`を更新する必要があります。それを支援するために、Spring Bootは`spring-boot-properties-migrator`モジュールを提供します。プロジェクトに依存関係として追加すると、アプリケーションの環境を分析し、起動時に診断を表示するだけでなく、実行時に一時的にプロパティを移行してくれます。

Mavenの`pom.xml`に以下を追加することで、Migratorを追加することができます：
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-properties-migrator</artifactId>
	<scope>runtime</scope>
</dependency>
```
あるいは、Gradleを使う場合：
```gradle
runtime("org.springframework.boot:spring-boot-properties-migrator")
```
[Note]備考移行が完了したら、プロジェクトの依存関係からこのモジュールを削除することを確認してください。

## Spring Framework 6.0
Spring Boot 3.0はSpring Framework 6.0をベースに構築されています。続行する前に、その[アップグレードガイド](https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-6.x)を確認した方がよいでしょう。

## Jakarta EE
Spring BootがJakarta EE仕様に依存している場合、Spring Boot 3.0はJakarta EE 10に含まれるバージョンにアップグレードしています。例えば、Spring Boot 3.0はServlet 6.0とJPA 3.1仕様を使用しています。

もし、あなたが独自の依存関係を管理しており、スターターPOMに依存していないのであれば、MavenまたはGradleファイルを適切に更新していることを確認する必要があります。特に、古いJava EE依存関係がビルドで直接または過渡的に使用されないように注意する必要があります。例えば、`javax.servlet:javax.servlet-api`ではなく、`jakarta.servlet:jakarta.servlet-api`を常に使用する必要があるとします。

依存関係の調整と同様に、Jakarta EEでは、`javax`ではなく`jakarta`パッケージを使用するようになりました。依存関係を更新したら、プロジェクト内の`import`文を更新する必要があることがわかるかもしれません。

マイグレーションを支援するツールは、以下の通りです：
- [OpenRewrite recipes](https://docs.openrewrite.org/reference/recipes/java/migrate/jakarta/javaxmigrationtojakarta).
- [The Spring Boot Migrator project](https://github.com/spring-projects-experimental/spring-boot-migrator).
- [Migration support in IntelliJ IDEA](https://blog.jetbrains.com/idea/2021/06/intellij-idea-eap-6/).

## Core Changes
Spring Bootのコアには、ほとんどのアプリケーションに関連するいくつかの変更が加えられています。

### Image Banner Support Removed
`banner.gif`、`banner.jpg`、`banner.png`は無視され、テキストベースのbanner.txtに置き換える必要があります。

### Logging Date Format
Logback と Log4j2 のログメッセージの日付と時間のコンポーネントのデフォルトフォーマットが、ISO-8601 標準に合わせるために変更されました。新しいデフォルトフォーマット `yyyy-MM-dd'T'HH:mm:ss.SSSXXX` は、スペース文字の代わりに `T` を使用して日付と時刻を区切り、タイムゾーンオフセットを末尾に追加しています。`LOG_DATEFORMAT_PATTERN`環境変数または`logging.pattern.dateformat`プロパティを使用すると、以前のデフォルト値である`yyyy-MM-dd HH:mm:ss.SSS` に戻すことができます。

### @ConstructingBinding No Longer Needed at the Type Level
`@ConstructorBinding`は、`@ConfigurationProperties`クラスの型レベルでは不要になったので、削除する必要があります。クラスまたはレコードに複数のコンストラクタがある場合、プロパティ・バインディングに使用すべきコンストラクタを示すために、コンストラクタ上で使用することができます。

### YamlJsonParser Has Been Removed
SnakeYAMLのJSONパーシングが他のパーサー実装と矛盾していたため、`YamlJsonParser`は削除されました。万が一、`YamlJsonParser`を直接使用していた場合は、他の`JsonParser`の実装のいずれかに移行してください。

### Auto-configuration Files
Spring Boot 2.7では、spring.factoryでの登録との後方互換性を維持しつつ、自動設定を登録するための新しいMETA-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.importsファイルを[導入](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.7-Release-Notes#changes-to-auto-configuration)しています。このリリースでは、spring.factoryでの自動設定登録のサポートは削除され、importsファイルでの登録が優先されます。

## Web Application Changes
Webアプリケーションをアップグレードする場合、次のセクションを確認する必要があります。

### Spring MVC and WebFlux URL Matching Changes
Spring Framework 6.0では、トレイリングスラッシュのマッチング設定オプションは非推奨となり、そのデフォルト値は`false`に設定されました。つまり、以前は以下のコントローラは「GET /some/greeting」と「GET /some/greeting/」の両方にマッチしていたことになります：
```java
@RestController
public class MyController {

  @GetMapping("/some/greeting")
  public String greeting() {
    return "Hello";
  }

}
```
今回の[Spring Frameworkの変更](https://github.com/spring-projects/spring-framework/issues/28552)で、「GET /some/greeting/」はデフォルトではもうマッチせず、HTTP 404エラーになります。

開発者はその代わりに、プロキシやServlet/Webフィルタを通して明示的にリダイレクト/リライトを設定する必要があります。また、より限定的なケースでは、コントローラハンドラで明示的に追加ルートを宣言（@GetMapping("/some/greeting", "/some/greeting/")のように）することもできます。

アプリケーションがこの変更に完全に適応するまでは、以下のSpring MVCのグローバル設定でデフォルトを変更することができます：
```java
@Configuration
public class WebConfiguration implements WebMvcConfigurer {

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
      configurer.setUseTrailingSlashMatch(true);
    }

}
```
あるいは、Spring WebFluxを使用している場合：
```java
@Configuration
public class WebConfiguration implements WebFluxConfigurer {

    @Override
    public void configurePathMatching(PathMatchConfigurer configurer) {
      configurer.setUseTrailingSlashMatch(true);
    }

}
```
### 'server.max-http-header-size'
これまで、server.max-http-header-sizeは、サポートする4つの組み込みWebサーバー間で一貫性のない扱いを受けていました。Jetty、Netty、またはUndertowを使用する場合は、最大HTTPリクエストヘッダーサイズが設定されます。Tomcatを使用する場合は、HTTPリクエストとレスポンスのヘッダーサイズの最大値を設定します。

この矛盾に対処するため、server.max-http-header-sizeは非推奨となり、代わりにserver.max-http-request-header-sizeが導入されました。両プロパティは、基礎となるウェブサーバーに関係なく、リクエストヘッダーサイズにのみ適用されるようになりました。

TomcatやJetty（このような設定をサポートする唯一の2つのサーバー）でHTTPレスポンスの最大ヘッダーサイズを制限するには、WebServerFactoryCustomizerを使用します。

### Updated Phases for Graceful Shutdown
`SmartLifecycle`の実装がグレースフルシャットダウンに使用するフェーズが更新されました。グレースフル・シャットダウンは、`SmartLifecycle.DEFAULT_PHASE - 2048`のフェーズで開始し、Webサーバーは`SmartLifecycle.DEFAULT_PHASE - 1024`のフェーズで停止するようになりました。グレースフル・シャットダウンに参加していた`SmartLifecycle`の実装は、それに応じて更新する必要があります。

### Jetty
JettyはまだServlet 6.0をサポートしていません。Spring Boot 3.0でJettyを使用するには、Servlet APIを5.0にダウングレードする必要があります。そのためには、`jakarta-servlet.version`プロパティを使用します。

### Apache HttpClient in RestTemplate
Apache HttpClientのサポートは@Spring Framework 6.0で削除](https://github.com/spring-projects/spring-framework/issues/28925)され、すぐに`org.apache.httpcomponents.client5:httpclient5`（注意：この依存関係は異なるgroupIdを持っています）に置き換えられました。HTTPクライアントの動作に問題がある場合、`RestTemplate`がJDKクライアントにフォールバックしている可能性があります。 `org.apache.httpcomponents:httpclient` は他の依存関係によって推移的にもたらされることがあるので、アプリケーションは宣言せずにこの依存関係に依存しているかもしれません。

## Actuator Changes
### JMX Endpoint Exposure
### 'httptrace' Endpoint Renamed to 'httpexchanges'
### Actuator JSON
### Actuator Endpoints Sanitization
### Auto-configuration of Micrometer’s JvmInfoMetrics
### Actuator Metrics Export Properties
### Mongo Health Check

## Micrometer and Metrics Changes
### Deprecation of the Spring Boot 2.x instrumentation
### Tag providers and contributors migration

## Data Access Changes
### Changes to Data properties
### Cassandra Properties
### Redis Properties
### Flyway
### Liquibase
### Hibernate 6.1
### Embedded MongoDB
### R2DBC 1.0
### Elasticsearch Clients and Templates
### MySQL JDBC Driver
## Spring Security Changes
### ReactiveUserDetailsService
### SAML2 Relying Party Configuration
## Spring Batch Changes
### @EnableBatchProcessing is now discouraged
### Multiple Batch Jobs
## Spring Session Changes
### Spring Session Store Type

## Gradle Changes
### Simplified Main Class Name Resolution With Gradle
### Configuring Gradle Tasks
### Excluding Properties From 'build-info.properties' With Gradle

## Maven Changes
### Running Your Application in the Maven Process
### Git Commit ID Maven Plugin

## Dependency Management Changes
### JSON-B
### ANTLR 2
### RxJava
### Hazelcast Hibernate Removed
### Ehcache3
### Other Removals
