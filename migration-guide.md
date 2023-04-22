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
