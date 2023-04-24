# Upgrading to Version 6.0

## Core Container

JSR-330ベースの@Injectアノテーションは、現在、jakarta.injectにあるようです。対応するJSR-250ベースのアノテーション@PostConstructと@PreDestroyは、jakarta.annotationにあります。当分の間、Springは、コンパイル済みバイナリでの一般的な使用をカバーするために、javaxの同等物をも検出し続けます。

コアコンテナは、デフォルトでjava.bean.Introspectorを使用せずに基本的なBeanプロパティ決定を行う。洗練されたJavaBeansを使用する場合、5.3.xとの完全な後方互換性のために、META-INF/spring.worksファイルに次の内容を指定して、5.3スタイルの完全なjava.bean.Introspector使用を可能にします：
org.springframework.beans.BeanInfoFactory=org.springframework.beans.ExtendedBeanInfoFactory

当面5.3.xを使用する場合、カスタムのMETA-INF/spring.factsファイルによって、6.0スタイルのプロパティ決定（およびイントロスペクションのパフォーマンス向上！）との互換性を強制できます：
org.springframework.beans.BeanInfoFactory=org.springframework.beans.SimpleBeanInfoFactory

LocalVariableTableParameterNameDiscoverer は現在非推奨で、解決に成功するたびに警告が記録されます (
StandardReflectionParameterNameDiscoverer が名前を見つけられなかった場合にのみ起動します)。この警告を回避するために、パラメータ名保持のための一般的なJava
8+の-parametersフラグ（-debugコンパイラフラグに依存するのではなく）でJavaソースをコンパイルするか、影響を受けるコードのメンテナに報告します。Kotlinコンパイラでは、完全性のために-java-parametersフラグを推奨します。

LocalValidatorFactoryBeanは、現在Bean Validation 3.0の標準的なパラメータ名解決に依存しており、Kotlinが存在する場合は追加のKotlin反射を構成するだけです。Bean
Validationのセットアップでパラメータ名を参照する場合は、Java 8+の-parametersフラグでJavaソースをコンパイルすることを確認してください。

ListenableFutureはCompletableFutureに取って代わられ、非推奨となりました。[#27780](https://github.com/spring-projects/spring-framework/issues/27780)
を参照してください。

Asyncでアノテーションされたメソッドは、Futureかvoidのどちらかを返さなければなりません。これは長い間文書化されていましたが、現在では積極的にチェックされ、強制されるようになっており、それ以外の戻り値の場合は例外がスローされます。
[#27734](https://github.com/spring-projects/spring-framework/issues/27734)を参照してください。

SimpleEvaluationContextは、通常のコンストラクタの解決に合わせ、配列の割り当てを無効にするようになりました。

## Data Access and Transactions

Jakarta EEへの移行に伴い、hibernate-core-jakartaアーティファクトを使用してHibernate ORM
5.6.xにアップグレードし、同時にjavax.persistenceインポートをjakarta.persistence（ Jakarta EE 9 ）に切り替えてください。あるいは、Spring Boot
3.0に搭載されているHibernateのバージョンであるHibernate ORM 6.1（jakarta.persistence のみベース、EE 9とEE 10に対応）にすぐに移行することも検討します。

対応するHibernate Validatorの世代は、jakarta.validation（Jakarta EE 9）に基づく7.0.xです。また、すぐにHibernate Validator
8.0にアップグレードすることも可能です（Jakarta EE 10に準拠）。

永続化プロバイダとしてEclipseLinkを選択した場合、リファレンスバージョンは3.0.x（Jakarta EE 9）、最新のサポートバージョンとしてEclipseLink 4.0（Jakarta EE
10）があります。

SpringのデフォルトのJDBC例外トランスレータは、JDBC
4ベースのSQLExceptionSubclassTranslatorになり、JDBCドライバのサブクラスと一般的なSQL状態の表示を検出します（実行時にデータベース製品名の解決なし）。6.0.3では、DuplicateKeyExceptionの共通SQL状態のチェックが含まれ、SQL状態のマッピングとレガシーのデフォルトエラーコードマッピングの間の長年の相違に対処しています。

PessimisticLockingFailureExceptionベースクラスと、その共通のCannotAcquireLockExceptionサブクラス（JPA/Hibernateに準拠）の一貫したセマンティクスが、すべてのデフォルト例外変換シナリオで優先されるため、JDBCセマンティクスとの不整合により、CannotSerializeTransactionException
と DeadlockLoserDataAccessExceptionは6.0.3で非推奨となります。

データベース固有のエラーコードの完全な後方互換性を確保するために、従来のSQLErrorCodeSQLExceptionTranslatorを再度有効にしてください。このトランスレータは、ユーザが提供したsql-error-codes.xmlファイルに対してキックされます。クラスパスのルートにあるユーザー提供の空のファイルがトリガーとなり、Springのレガシーなデフォルトエラーコードマッピングを単純にピックアップすることもできます。

## Web Applications

Jakarta EEへの移行に伴い、Tomcat 10、Jetty 11、またはUndertow 2.2.19にundertow-servlet-jakartaアーティファクトをアップグレードし、同時にjavax.servlet
importをjakarta.servlet（ Jakarta EE 9）へと切り替えておく必要があることを確認します。最新のサーバー世代では、Tomcat 10.1とUndertow 2.3 (Jakarta EE
10)を検討してください。

例えば、Apache Commons FileUpload (org.springframework.web.multipart.commons.CommonsMultipartResolver) や Apache Tiles、対応する
org.springframework.web.servlet.view サブパッケージの FreeMarker JSP サポートなどがあります。マルチパートファイルのアップロードには
org.springframework.web.multipart.support.StandardServletMultipartResolver を、必要であれば通常の FreeMarker テンプレートビュー、そして REST
指向のウェブアーキテクチャに一般的に焦点を当てることを推奨します。

Spring MVCとSpring
WebFluxは、型レベルの@RequestMappingアノテーションだけではコントローラを検出しなくなった。つまり、WebコントローラのインターフェイスベースのAOPプロキシは、もはや機能しないかもしれません。そのようなコントローラにはクラスベースのプロキシを有効にしてください。そうでなければ、インターフェイスも
@Controller でアノテーションする必要があります。[#22154](https://github.com/spring-projects/spring-framework/issues/22154) を参照してください。

HttpMethod はクラスとなり、enum ではなくなっています。パブリック API は維持されていますが、いくつかの移行が必要な場合があります（EnumSet<HttpMethod> から
Set<HttpMethod> への変更、switch の代わりに if else
を使用する、など）。この決定の根拠については、[#27697](https://github.com/spring-projects/spring-framework/issues/27697)を参照してください。

WebTestClient.ResponseSpec::expectBodyのKotlin拡張関数がJava BodySpec型を返すようになり、回避型のKotlinBodySpecを使用しなくなりました。Spring
6.0ではKotlin 1.6を使用しており、このワークアラウンドが必要だったバグ（[KT-5464](https://youtrack.jetbrains.com/issue/KT-5464)
）が修正されました。つまり、consumeWithは使えなくなったということです。

RestTemplateというか、HttpComponentsClientHttpRequestFactoryは、Apache HttpClient 5を必要とするようになりました。

Springが提供するServletモック（MockHttpServletRequest、MockHttpSession）は、Servlet 5.0 と 6.0 API jarの間のブレークチェンジのため、Servlet 6.0
が必要になりました。これらは、Servlet 5.0ベースのコードのテストに使用できますが、テストのクラスパスでServlet 6.0 API（または新しいもの）に対して実行する必要があります。モックベースのテストはServlet
6.0 API jarに対して実行される必要があります。

Spring MVCとRestTemplateでは、SourceHttpMessageConverterはデフォルトで設定されなくなった。その結果、javax.xml.transform.Sourceを使用するSpring
Webアプリケーションは、SourceHttpMessageConverterを明示的に設定する必要がある。コンバーターの登録順序は重要で、SourceHttpMessageConverterは通常、MappingJackson2HttpMessageConverterなどの「キャッチオール」コンバーターの前に登録する必要があることに注意。
