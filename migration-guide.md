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
## Configuration Properties Migration
## Spring Framework 6.0
## Jakarta EE
## Core Changes
### Image Banner Support Removed
### Logging Date Format
### @ConstructingBinding No Longer Needed at the Type Level
### YamlJsonParser Has Been Removed
### Auto-configuration Files

## Web Application Changes
### Spring MVC and WebFlux URL Matching Changes
### 'server.max-http-header-size'
### Updated Phases for Graceful Shutdown
### Jetty
### Apache HttpClient in RestTemplate
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
