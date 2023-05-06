
- [Before You Start](#before-you-start)
  - [Upgrade to the Latest 2.7.x Version](#upgrade-to-the-latest-27x-version)
  - [Review Dependencies](#review-dependencies)
  - [Spring Security](#spring-security)
    - [Dispatch types](#dispatch-types)
  - [Review System Requirements](#review-system-requirements)
  - [Review Deprecations from Spring Boot 2.x](#review-deprecations-from-spring-boot-2x)
- [Upgrade to Spring Boot 3](#upgrade-to-spring-boot-3)
  - [Configuration Properties Migration](#configuration-properties-migration)
  - [Spring Framework 6.0](#spring-framework-60)
  - [Jakarta EE](#jakarta-ee)
  - [Core Changes](#core-changes)
    - [Image Banner Support Removed](#image-banner-support-removed)
    - [Logging Date Format](#logging-date-format)
    - [@ConstructingBinding No Longer Needed at the Type Level](#constructingbinding-no-longer-needed-at-the-type-level)
    - [YamlJsonParser Has Been Removed](#yamljsonparser-has-been-removed)
    - [Auto-configuration Files](#auto-configuration-files)
  - [Web Application Changes](#web-application-changes)
    - [Spring MVC and WebFlux URL Matching Changes](#spring-mvc-and-webflux-url-matching-changes)
    - ['server.max-http-header-size'](#servermax-http-header-size)
    - [Updated Phases for Graceful Shutdown](#updated-phases-for-graceful-shutdown)
    - [Jetty](#jetty)
    - [Apache HttpClient in RestTemplate](#apache-httpclient-in-resttemplate)
  - [Actuator Changes](#actuator-changes)
    - [JMX Endpoint Exposure](#jmx-endpoint-exposure)
    - ['httptrace' Endpoint Renamed to 'httpexchanges'](#httptrace-endpoint-renamed-to-httpexchanges)
    - [Actuator JSON](#actuator-json)
    - [Actuator Endpoints Sanitization](#actuator-endpoints-sanitization)
  - [Micrometer and Metrics Changes](#micrometer-and-metrics-changes)
    - [Deprecation of the Spring Boot 2.x instrumentation](#deprecation-of-the-spring-boot-2x-instrumentation)
    - [Tag providers and contributors migration](#tag-providers-and-contributors-migration)
    - [Auto-configuration of Micrometer’s JvmInfoMetrics](#auto-configuration-of-micrometers-jvminfometrics)
    - [Actuator Metrics Export Properties](#actuator-metrics-export-properties)
    - [Mongo Health Check](#mongo-health-check)
  - [Data Access Changes](#data-access-changes)
    - [Changes to Data properties](#changes-to-data-properties)
    - [Cassandra Properties](#cassandra-properties)
    - [Redis Properties](#redis-properties)
    - [Flyway](#flyway)
    - [Liquibase](#liquibase)
    - [Hibernate 6.1](#hibernate-61)
    - [Embedded MongoDB](#embedded-mongodb)
    - [R2DBC 1.0](#r2dbc-10)
    - [Elasticsearch Clients and Templates](#elasticsearch-clients-and-templates)
    - [MySQL JDBC Driver](#mysql-jdbc-driver)
  - [Spring Security Changes](#spring-security-changes)
    - [ReactiveUserDetailsService](#reactiveuserdetailsservice)
    - [SAML2 Relying Party Configuration](#saml2-relying-party-configuration)
  - [Spring Batch Changes](#spring-batch-changes)
    - [@EnableBatchProcessing is now discouraged](#enablebatchprocessing-is-now-discouraged)
    - [Multiple Batch Jobs](#multiple-batch-jobs)
  - [Spring Session Changes](#spring-session-changes)
    - [Spring Session Store Type](#spring-session-store-type)
  - [Gradle Changes](#gradle-changes)
    - [Simplified Main Class Name Resolution With Gradle](#simplified-main-class-name-resolution-with-gradle)
    - [Configuring Gradle Tasks](#configuring-gradle-tasks)
    - [Excluding Properties From 'build-info.properties' With Gradle](#excluding-properties-from-build-infoproperties-with-gradle)
  - [Maven Changes](#maven-changes)
    - [Running Your Application in the Maven Process](#running-your-application-in-the-maven-process)
    - [Git Commit ID Maven Plugin](#git-commit-id-maven-plugin)
  - [Dependency Management Changes](#dependency-management-changes)
    - [JSON-B](#json-b)
    - [ANTLR 2](#antlr-2)
    - [RxJava](#rxjava)
    - [Hazelcast Hibernate Removed](#hazelcast-hibernate-removed)
    - [Ehcache3](#ehcache3)
    - [Other Removals](#other-removals)

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
Apache HttpClientのサポートは[Spring Framework 6.0で削除](https://github.com/spring-projects/spring-framework/issues/28925)され、すぐに`org.apache.httpcomponents.client5:httpclient5`（注意：この依存関係は異なるgroupIdを持っています）に置き換えられました。HTTPクライアントの動作に問題がある場合、`RestTemplate`がJDKクライアントにフォールバックしている可能性があります。 `org.apache.httpcomponents:httpclient` は他の依存関係によって推移的にもたらされることがあるので、アプリケーションは宣言せずにこの依存関係に依存しているかもしれません。

## Actuator Changes
Spring Bootのactuatorモジュールを使用している方は、以下のアップデートに注意してください。

### JMX Endpoint Exposure
デフォルトでは、ヘルスエンドポイントのみがJMX上で公開され、デフォルトのWebエンドポイント公開と一致するようになりました。これは、management.endpoints.jmx.exposure.includeおよびmanagement.endpoints.jmx.exposure.excludeプロパティを構成することで変更できます。

### 'httptrace' Endpoint Renamed to 'httpexchanges'
httptraceエンドポイントおよび関連するインフラストラクチャは、最近のHTTPリクエスト-レスポンス交換に関する情報を記録し、その情報へのアクセスを提供します。[Micrometer Tracing](https://micrometer.io/docs/tracing)のサポートが導入された後、httptraceという名前は混乱を引き起こす可能性があります。この可能な混乱を減らすために、エンドポイントはhttpexchangesに名前が変更されました。エンドポイントのレスポンスの内容も、この名称変更によって影響を受けています。詳細については、[Actuator APIドキュメント](https://docs.spring.io/spring-boot/docs/3.0.x/actuator-api/htmlsingle/#httpexchanges)を参照してください。

### Actuator JSON
Spring Bootで出荷されたアクチュエータエンドポイントからの応答は、結果が一貫していることを保証するために、分離されたObjectMapperインスタンスを使用します。もし、以前の動作に戻してアプリケーションObjectMapperを使用したい場合は、management.endpoints.jackson.isolated-object-mapperをfalseに設定することができます。

独自のエンドポイントを開発した場合、レスポンスが OperationResponseBody インターフェースを実装していることを確認したい場合があります。これにより、レスポンスをJSONとしてシリアライズする際に、孤立したObjectMapperが考慮されるようになります。

### Actuator Endpoints Sanitization
envと/configpropsのエンドポイントには機密性の高い値が含まれることがあるため、デフォルトですべての値が常にマスクされます。以前は、機密性の高いキーにのみ適用されていました。

その代わりに、このリリースでは、より安全なデフォルトを選択しました。キーベースのアプローチは削除され、ヘルスエンドポイントの詳細と同様に、ロールベースのアプローチが採用されています。サニタイズされていない値を表示するかどうかは、プロパティ management.endpoint.env.show-values または management.endpoint.configprops.show-values を使用して設定することができ、以下の値を持つことができます：
- NEVER - すべての値がサニタイズされます（デフォルト）。
- ALWAYS - すべての値が出力されます（サニタイズ機能が適用されます）。
- WHEN_AUTHORIZED - ユーザーが許可された場合のみ、値が出力されます（サニタイズ機能が適用されます）。

JMXの場合、ユーザーは常に認可されているとみなされます。HTTPでは、認証され、指定されたロールを持つ場合、ユーザーは認可されたとみなされます。

QuartzEndpointのサニタイズも、同様にプロパティ management.endpoint.quartz.show-values で設定可能です。

## Micrometer and Metrics Changes
Spring Boot 3.0では、Micrometer 1.10をベースに構築されています。もしあなたのアプリケーションがメトリックスの収集とエクスポートを行うのであれば、以下の変更に注意する必要があります。

### Deprecation of the Spring Boot 2.x instrumentation
Observationサポートとの統合の結果、以前のインスツルメンテーションは非推奨となりました。クラス全体のバグを解決できず、重複したインストルメンテーションのリスクが高すぎるため、実際のインストルメンテーションを実行するフィルタ、インターセプターは完全に削除されました。例えば、WebMvcMetricsFilterは完全に削除され、Spring FrameworkのServerHttpObservationFilterで実質的に置き換えられています。対応する *TagProvider *TagContributor と *Tags クラスは、非推奨となりました。これらは、オブザベーションインストルメンテーションでデフォルトではもう使われません。開発者が既存のインフラを新しいものに移行できるように、非推奨フェーズの間、これらを残しています。

### Tag providers and contributors migration
あなたのアプリケーションがメトリックスをカスタマイズしている場合、あなたのコードベースでは新しい非推奨が表示されるかもしれません。新しいモデルでは、タグプロバイダとコントリビュータの両方が、観測規約に置き換えられています。Spring Boot 2.xのSpring MVC "http.server.requests" metrics instrumentation supportを例にとって説明しましょう。

TagContributorで追加のTagを提供したり、TagProviderを部分的にオーバーライドするだけなら、おそらくDefaultServerRequestObservationConventionを要件に合わせて拡張すべきです：
```java
public class ExtendedServerRequestObservationConvention extends DefaultServerRequestObservationConvention {

  @Override
  public KeyValues getLowCardinalityKeyValues(ServerRequestObservationContext context) {
    // here, we just want to have an additional KeyValue to the observation, keeping the default values
    return super.getLowCardinalityKeyValues(context).and(custom(context));
  }

  protected KeyValue custom(ServerRequestObservationContext context) {
    return KeyValue.of("custom.method", context.getCarrier().getMethod());
  }

}
```
メトリクス・タグを大幅に変更する場合、おそらく WebMvcTagsProvider をカスタム実装に置き換えて、Bean として寄贈することになるでしょう。この場合、気になる観測のための規約を実装しておくとよいでしょう。ここでは ServerRequestObservationConvention を実装します。これは ServerRequestObservationContext を使用して、現在のリクエストに関する情報を抽出します。あとは、自分の要求を考慮したメソッドを実装すればよい：
```java
public class CustomServerRequestObservationConvention implements ServerRequestObservationConvention {

  @Override
  public String getName() {
    // will be used for the metric name
    return "http.server.requests";
  }

  @Override
  public String getContextualName(ServerRequestObservationContext context) {
    // will be used for the trace name
    return "http " + context.getCarrier().getMethod().toLowerCase();
  }

  @Override
  public KeyValues getLowCardinalityKeyValues(ServerRequestObservationContext context) {
    return KeyValues.of(method(context), status(context), exception(context));
  }

  @Override
  public KeyValues getHighCardinalityKeyValues(ServerRequestObservationContext context) {
    return KeyValues.of(httpUrl(context));
  }

  protected KeyValue method(ServerRequestObservationContext context) {
    // You should reuse as much as possible the corresponding ObservationDocumentation for key names
    return KeyValue.of(ServerHttpObservationDocumentation.LowCardinalityKeyNames.METHOD, context.getCarrier().getMethod());
  }

  //...
}
```
どちらの場合も、それらをBeanとしてアプリケーションコンテキストに寄与させれば、自動構成によってピックアップされ、事実上デフォルトのものに取って代わられます。
```java
@Configuration
public class CustomMvcObservationConfiguration {

  @Bean
  public ExtendedServerRequestObservationConvention extendedServerRequestObservationConvention() {
    return new ExtendedServerRequestObservationConvention();
  }

}
```
また、カスタムObservationFilterを使用して、オブザベーションのキーバリューを追加または削除することで、同様のゴールを達成することができます。フィルターはデフォルトの規約を置き換えるものではなく、後処理コンポーネントとして使用されます。
```java
public class ServerRequestObservationFilter implements ObservationFilter {

  @Override
  public Observation.Context map(Observation.Context context) {
    if (context instanceof ServerRequestObservationContext serverContext) {
      context.addLowCardinalityKeyValue(KeyValue.of("project", "spring"));
      String customAttribute = (String) serverContext.getCarrier().getAttribute("customAttribute");
      context.addLowCardinalityKeyValue(KeyValue.of("custom.attribute", customAttribute));
    }
    return context;
  }
}
```
### Auto-configuration of Micrometer’s JvmInfoMetrics
MicrometerのJvmInfoMetricsが自動構成されるようになりました。手動で設定されたJvmInfoMetricsのBean定義は、削除することができます。

### Actuator Metrics Export Properties
[アクチュエータ・メトリクス・エクスポート](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#actuator.metrics)を制御するプロパティを移動しました。古いスキーマは management.metrics.export.<product> で、新しいスキーマは management.<product>.metrics.export です（例：prometheus のプロパティは management.metrics.export.prometheus から management.prometheus.metrics.export に移動しました）。spring-boot-properties-migratorを使用している場合、起動時に通知されます。

詳しくは[issue#30381](https://github.com/spring-projects/spring-boot/issues/30381)をご覧ください。

### Mongo Health Check
MongoDB用のHealthIndicatorがMongoDBのStable APIをサポートするようになりました。buildInfo クエリは isMaster に置き換えられ、レスポンスには version の代わりに maxWireVersion が含まれるようになりました。[MongoDB のドキュメント](https://www.mongodb.com/docs/v4.2/reference/command/isMaster/)にあるように、クライアントは MongoDB との互換性を交渉するために maxWireVersion を使用することができます。maxWireVersion は整数値であることに注意しましょう。

## Data Access Changes
アプリケーションがデータを扱う場合、以下の変更点を確認する必要があります。

### Changes to Data properties
spring.dataプレフィックスはSpring Data用に予約されており、プレフィックスの下にあるプロパティはクラスパス上でSpring Dataが必要であることを意味する。

### Cassandra Properties
Cassandraの設定プロパティは、spring.data.cassandra.からspring.cassandra.に移動しました。

### Redis Properties
Redisの自動設定にはSpring Dataがクラスパスに存在する必要があるため、Redisの設定プロパティはspring.redis.からspring.data.redis.に移動しました。

### Flyway
Spring Boot 3.0では、デフォルトでFlyway 9.0を使用します。これがあなたのアプリケーションにどのような影響を与えるかについては、Flywayの[リリースノート](https://flywaydb.org/documentation/learnmore/releaseNotes#9.0.0)と[ブログポスト](https://flywaydb.org/blog/version-9-is-coming-what-developers-need-to-know)を参照してください。

FlywayConfigurationCustomizerビーンズは、CallbackおよびJavaMigrationビーンズが構成に追加された後に、FluentConfigurationをカスタマイズするために呼ばれるようになりました。Callback および JavaMigration Bean を定義し、カスタマイザーを使用してコールバックおよび Java マイグレーションを追加するアプリケーションは、意図したコールバックおよび Java マイグレーションが使用されるように更新する必要がある場合があります。

### Liquibase
Spring Boot 3.0 は、デフォルトで Liquibase 4.17.x を使用しています。[4.17.xではいくつか問題が報告されている](https://github.com/spring-projects/spring-boot/issues/35010)ので、もし、あなたのアプリケーションが影響を受けたら、アプリケーションのニーズに合わせて Liquibase のバージョンをオーバーライドすることを検討してください。

### Hibernate 6.1
Spring Boot 3.0では、デフォルトでHibernate 6.1が使用されます。これがアプリケーションにどのような影響を与えるかについては、Hibernate [6.0](https://docs.jboss.org/hibernate/orm/6.0/migration-guide/migration-guide.html)および[6.1](https://docs.jboss.org/hibernate/orm/6.1/migration-guide/migration-guide.html)移行ガイドを参照してください。

依存関係管理と spring-boot-starter-data-jpa starter が更新され、Hibernate の依存関係に新しい org.hibernate.orm グループ ID が使用されるようになりました。

Spring.jpa.hibernate.use-new-id-generator-mappings 設定プロパティは、Hibernate が古い ID ジェネレーターマッピングへの切り替えをサポートしなくなったため、削除されました。

### Embedded MongoDB
Flapdoodleの埋め込みMongoDBの自動設定と依存関係管理は削除されました。テストに埋め込みMongoDBを使用する場合は、[Flapdoodleプロジェクト](https://github.com/flapdoodle-oss/de.flapdoodle.embed.mongo.spring)が提供する自動設定ライブラリを使用するか、埋め込みMongoDBの代わりに[Testcontainersプロジェクト](https://www.testcontainers.org/)を使用するようにテストを修正します。

### R2DBC 1.0
Spring Boot 3.0では、デフォルトでR2DBC 1.0を使用しています。1.0のリリースにより、R2DBCは部品表（bom）を発行しなくなり、Spring Bootの依存関係管理に影響を与えました。r2dbc-bom.versionを使用してR2DBCのバージョンをオーバーライドすることはできなくなりました。その代わりに、個別にバージョン管理されたモジュールのためのいくつかの新しいプロパティが利用できるようになりました：
- oracle-r2dbc.version (com.oracle.database.r2dbc:oracle-r2dbc)
- r2dbc-h2.version (io.r2dc:r2dbc-h2)
- r2dbc-pool.version (io.r2dc:r2dbc-pool)
- r2dbc-postgres.version (io.r2dc:r2dbc-postgres)
- r2dbc-proxy.version (io.r2dc:r2dbc-proxy)
- r2dbc-spi.version (io.r2dc:r2dbc-spi)

### Elasticsearch Clients and Templates
Elasticsearch のハイレベルな REST クライアントのサポートが削除されました。その代わりに、Elasticsearch の新しい Java クライアントの自動設定が導入されました。同様に、高レベルのRESTクライアントの上に構築されたSpring Data Elasticsearchテンプレートのサポートが削除されました。その代わりに、新しいJavaクライアントの上に構築された新しいテンプレートのための自動設定が導入されました。詳細は[リファレンスドキュメントのElasticsearchのセクション](https://docs.spring.io/spring-boot/docs/3.0.x/reference/html/data.html#data.nosql.elasticsearch)を参照してください。

ReactiveElasticsearchRestClientAutoConfiguration は ReactiveElasticsearchClientAutoConfiguration に名称変更され、 org.springframework.boot.autoconfigure.data.elasticsearch から org.springframework.boot.autoconfigure.elasticsearch に移行されました。自動設定の除外や順序は、それに応じて更新する必要があります。

### MySQL JDBC Driver
MySQL JDBCドライバの座標が、mysql:mysql-connector-java から com.mysql:mysql-connector-j に変更されました。MySQL JDBCドライバを使用している場合は、Spring Boot 3.0にアップグレードする際に、その座標を適宜更新してください。

## Spring Security Changes
Spring Boot 3.0は[Spring Security 6.0](https://docs.spring.io/spring-security/reference/migration/index.html)にアップグレードしました。以下の項目に加え、Spring Security 6.0移行ガイドをご確認ください。

### ReactiveUserDetailsService
AuthenticationManagerResolver が存在する場合、ReactiveUserDetailsService は自動設定されなくなりました。AuthenticationManagerResolver が存在するにもかかわらず、アプリケーションが ReactiveUserDetailService に依存している場合、そのニーズを満たす独自の ReactiveUserDetailsService ビーンを定義します。

### SAML2 Relying Party Configuration
spring.security.saml2.relyingparty.registration.{id}.identity-provider のプロパティのサポートが削除されました。代わりに spring.security.saml2.relyingparty.registration.{id}.asserting-party にある新しいプロパティを使用します。

## Spring Batch Changes
Spring Boot 3.0はSpring Batch 5.0にアップグレードしました。以下の項目に加え、Spring Batch 5.0移行ガイドをご確認ください。

### @EnableBatchProcessing is now discouraged
以前は@EnableBatchProcessingを使用することで、Spring BootのSpring Batchの自動設定を有効にすることができました。これはもはや必須ではなく、Bootの自動設定を使用したいアプリケーションからは削除する必要があります。EnableBatchProcessingでアノテーションされたBeanや、BatchのDefaultBatchConfigurationを拡張したBeanを定義することで、自動設定を停止するように指示できるようになり、アプリケーションがBatchの設定方法を完全に制御できるようになりました。

### Multiple Batch Jobs
複数のバッチジョブを実行することはサポートされなくなりました。自動設定によって単一のジョブが検出された場合、そのジョブがスタートアップ時に実行されます。コンテキストで複数のジョブが検出された場合、起動時に実行するジョブ名を spring.batch.job.name プロパティを使用してユーザーが指定する必要があります。

## Spring Session Changes
以下は、Spring Sessionをご利用の方に関連する項目です。

### Spring Session Store Type
Spring セッションのストアタイプを spring.session.store-type で明示的に設定することは、サポートされなくなりました。クラスパス上で複数のセッションストアリポジトリの実装が検出された場合、どのSessionRepositoryを自動設定すべきかを決定するために、[固定された順序](https://docs.spring.io/spring-boot/docs/3.0.x/reference/html/web.html#web.spring-session)が使用されます。Spring Bootで定義された順序がニーズに合わない場合は、独自のSessionRepository Beanを定義して、自動設定を後退させることができます。

## Gradle Changes
Spring BootプロジェクトをGradleでビルドするユーザーは、以下のセクションを確認してください。

### Simplified Main Class Name Resolution With Gradle
Gradleでアプリケーションをビルドするとき、アプリケーションのメインクラスの名前の解決が簡素化され、一貫したものになりました。 bootJar、bootRun、bootWarはすべて、メインソースセットの出力でそれを探すことによってメインクラス名の名前を解決するようになりました。これにより、タスクがデフォルトで同じメインクラス名を使用していないかもしれないという小さなリスクが取り除かれました。メインクラスがメインソースセットの出力以外の場所から解決されることに依存していた場合は、Gradleの設定を更新して、springBoot DSLのmainClassプロパティを使用してメインクラス名を構成するようにしてください：
```gradle
springBoot {
    mainClass = "com.example.Application"
}
```
また、resolveMainClassName タスクの classpath プロパティを設定して、メイン・ソース・セットの出力ディレクトリー以外の場所を検索することも可能です。

### Configuring Gradle Tasks
Spring BootのGradleタスクは、その設定にGradleのPropertyサポートを一貫して使用するように更新されました。その結果、プロパティの値を参照する方法を変更する必要がある場合があります。例えば、bootBuildImageのimageNameプロパティの値は、imageName.get()を使用してアクセスできるようになりました。さらに、Kotlin DSLを使用している場合、プロパティを設定する方法を変更する必要があるかもしれません。例えば、Spring Boot 2.xでは、bootJarタスクのレイヤリングを以下のように無効化することができます：
```gradle
tasks.named<BootJar>("bootJar") {
	layered {
		isEnabled = false
	}
}
```
3.0では、以下を使用する必要があります：
```gradle
tasks.named<BootJar>("bootJar") {
	layered {
		enabled.set(false)
	}
}
```
詳しい例については、[Gradleプラグインのリファレンスドキュメント](https://docs.spring.io/spring-boot/docs/3.0.x/gradle-plugin/reference/htmlsingle/)を参照してください。

### Excluding Properties From 'build-info.properties' With Gradle
先に説明したGradleタスクの設定変更の一環として、生成されたbuild-info.propertiesファイルからプロパティを除外する仕組みも変更されました。以前は、プロパティをnullに設定することで除外することができました。これはもはや機能せず、名前ベースのメカニズムに置き換えられています：
```gradle
springBoot {
	buildInfo {
		excludes = ['time']
	}
}
```
Gradle Kotlin DSLでは以下の通りです：
```kotlin
springBoot {
	buildInfo {
		excludes.set(setOf("time"))
	}
}
```
## Maven Changes
MavenでSpring Bootプロジェクトを構築するユーザーは、以下のセクションを確認してください。

### Running Your Application in the Maven Process
Spring Boot 2.7で非推奨とされていたspring-boot:runとspring-boot:startのfork属性が削除されました。

### Git Commit ID Maven Plugin
Git Commit ID Maven Pluginがバージョン5に更新され、座標が変更になりました。以前の座標は pl.project13.maven:git-commit-id-plugin でした。新しい座標は io.github.git-commit-id:git-commit-id-maven-plugin です。pom.xmlファイル内の<plugin>宣言もそれに合わせて更新する必要があります。

## Dependency Management Changes
Spring Bootで管理される依存関係において、以下の変更がありました。

### JSON-B
Apache Johnzonの依存関係管理は、Eclipse Yassonを優先して削除されました。Jakarta EE 10互換のApache JohnzonはSpring Boot 3で使用できますが、依存性宣言でバージョンを指定する必要が出てきました。

### ANTLR 2
ANTLR 2の依存関係管理（antlr:antlr）は、不要になったため削除しました。アプリケーションでANTLR 2を使用する場合は、ニーズに合ったバージョンを指定してください。

### RxJava
RxJava 1.x および 2.x の依存性管理が削除され、代わりに RxJava 3 の依存性管理が追加されています。

### Hazelcast Hibernate Removed
Spring BootはHazelcast Hibernateに依存しないので、そのバージョンについて意見する必要はありません。そのため、Hazelcast Hibernateの依存性管理は削除されました。Hazelcast Hibernateの使用を継続したい場合は、ニーズに合ったバージョンを指定してください。また、代わりに org.hibernate.orm:hibernate-jcache を使用することを検討してください。

### Ehcache3
Jakarta EE 9以降に対応するため、Ehcacheのehcacheモジュールとehcache-transactionsモジュールの依存関係管理をjakarta classifierで宣言するようになりました。pom.xmlやbuild.gradleスクリプトの依存性宣言も同様に更新する必要があります。

### Other Removals
Spring Boot 3.0では、以下の依存関係のサポートが削除されました：
- Apache ActiveMQ
- Atomikos
- EhCache 2
- Hazelcast 3

Apache Solrのサポートは、そのJettyベースのクライアントであるHttp2SolrClientがJetty 11と互換性がないため、削除されました。
