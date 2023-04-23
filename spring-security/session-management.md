# Require Explicit Saving of SecurityContextRepository

Spring Security 5では、
[SecurityContext](https://docs.spring.io/spring-security/reference/5.8/servlet/authentication/architecture.html#servlet-authentication-securitycontext)
は[SecurityContextPersistenceFilter](https://docs.spring.io/spring-security/reference/5.8/servlet/authentication/persistence.html#securitycontextpersistencefilter)
を使って[SecurityContextRepository](https://docs.spring.io/spring-security/reference/5.8/servlet/authentication/persistence.html#securitycontextrepository)
に自動的に保存されるのがデフォルトの挙動です。保存は、HttpServletResponseがコミットされる直前、SecurityContextPersistenceFilterの直前に行う必要があります。残念ながら、SecurityContextの自動永続化は、リクエストが完了する前（つまり、HttpServletResponseをコミットする直前）に行われると、ユーザーを驚かせてしまう。また、保存が必要かどうかを判断するために状態を追跡するのは複雑で、SecurityContextRepository（つまりHttpSession）への書き込みが不要になることもあります。

Spring Security 6では、
[SecurityContextHolderFilter](https://docs.spring.io/spring-security/reference/5.8/servlet/authentication/persistence.html#securitycontextholderfilter)
はSecurityContextRepositoryからSecurityContextを読み取り、SecurityContextHolderに入力するだけというのがデフォルトの動作になっています。リクエスト間でSecurityContextを持続させたい場合、ユーザーは明示的にSecurityContextRepositoryでSecurityContextを保存しなければならなくなった。これにより、曖昧さがなくなり、必要なときだけSecurityContextRepository（つまりHttpSession）への書き込みが必要となり、パフォーマンスが向上します。

[NOTE]
また、ログアウト時など、コンテキストを消去する際にも、コンテキストを保存する必要があります。詳しくは、[このセクション](https://docs.spring.io/spring-security/reference/5.8/servlet/authentication/session-management.html#properly-clearing-authentication)
を参照してください。

新しいSpring Security 6のデフォルトにオプトインするためには、次のような構成が考えられます。

```java
public SecurityFilterChain filterChain(HttpSecurity http){
        http
        // ...
        .securityContext((securityContext)->securityContext.requireExplicitSave(true));
        return http.build();
        }
```

設定を使用する際、SecurityContextHolderにSecurityContextを設定するコードは、リクエスト間でSecurityContextを持続させる必要がある場合、SecurityContextRepositoryにSecurityContextを保存することも重要である。

例えば、次のようなコードは：

```java
SecurityContextHolder.setContext(securityContext);
```

↓に置き換える必要があります。

```java
SecurityContextHolder.setContext(securityContext);
        securityContextRepository.saveContext(securityContext,httpServletRequest,httpServletResponse);
```

# Change HttpSessionSecurityContextRepository to DelegatingSecurityContextRepository

Spring Security
5では、デフォルトの[SecurityContextRepository](https://docs.spring.io/spring-security/reference/5.8/servlet/authentication/persistence.html#securitycontextrepository)
はHttpSessionSecurityContextRepositoryです。

Spring Security 6では、SecurityContextRepositoryのデフォルトはDelegatingSecurityContextRepositoryです。新しいSpring Security
6のデフォルトにオプトインするためには、以下のような構成にすればよい。

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http)throws Exception{
        http
        // ...
        .securityContext((securityContext)->securityContext
        .securityContextRepository(new DelegatingSecurityContextRepository(
        new RequestAttributeSecurityContextRepository(),
        new HttpSessionSecurityContextRepository()
        ))
        );
        return http.build();
        }
```

[important] 	
すでにHttpSessionSecurityContextRepository以外の実装を使用している場合は、RequestAttributeSecurityContextRepositoryと一緒に使用されるように、上記の例で選択した実装に置き換える必要があります。

# Address SecurityContextRepository Deprecations

Spring Security
5.7では、[SecurityContextRepository](https://docs.spring.io/spring-security/reference/5.8/servlet/authentication/persistence.html#securitycontextrepository)
にシグネチャを持つ新しいメソッドが追加されました：

```java
Supplier<SecurityContext> loadContext(HttpServletRequest request)
```

Spring Security 5.8で
[DelegatingSecurityContextRepository](https://docs.spring.io/spring-security/reference/5.8/servlet/authentication/persistence.html#delegatingsecuritycontextrepository)
が追加されたことにより、このメソッドは非推奨となり、このシグネチャを持つ新しいメソッドに変更されました：

```java
DeferredSecurityContext loadDeferredContext(HttpServletRequest request)
```

Spring Security 6では、非推奨のメソッドは削除されました。SecurityContextRepositoryを自分で実装し、loadContext(request)
メソッドの実装を追加した場合、そのメソッドの実装を削除し、代わりに新しいメソッドを実装することでSpring Security 6への準備ができます。

新しいメソッドの実装を始めるには、以下の例でDeferredSecurityContextを提供します：

```java
@Override
public DeferredSecurityContext loadDeferredContext(HttpServletRequest request){
        return new DeferredSecurityContext(){
private SecurityContext securityContext;
private boolean isGenerated;

@Override
public SecurityContext get(){
        if(this.securityContext==null){
        this.securityContext=getContextOrNull(request);
        if(this.securityContext==null){
        SecurityContextHolderStrategy strategy=SecurityContextHolder.getContextHolderStrategy();
        this.securityContext=strategy.createEmptyContext();
        this.isGenerated=true;
        }
        }
        return this.securityContext;
        }

@Override
public boolean isGenerated(){
        get();
        return this.isGenerated;
        }
        };
        }
```

# Optimize Querying of RequestCache

Spring Security 5では、
デフォルトの動作として、[保存されたリクエスト](https://docs.spring.io/spring-security/reference/5.8/servlet/architecture.html#savedrequests)
をリクエストごとに問い合わせるようになっています。これは、典型的なセットアップでは、[RequestCache](https://docs.spring.io/spring-security/reference/5.8/servlet/architecture.html#requestcache)
を使用するために、HttpSessionがリクエストごとにクエリされることを意味します。

Spring Security 6では、HTTPパラメータcontinueが定義されている場合のみ、RequestCacheがキャッシュリクエストの問い合わせを行うことがデフォルトとなっています。これにより、Spring
Securityは不必要にRequestCacheでHttpSessionを読み込むことを回避できる。

Spring Security 5では、デフォルトではHttpSessionRequestCacheを使用し、リクエストごとにキャッシュされたリクエストを問い合わせるようになっています。デフォルトを上書きしない場合(
NullRequestCacheを使用する場合)、以下の設定を使用すると、Spring Security 5.8で明示的にSpring Security 6の動作を選択することができます：

```java
@Bean
DefaultSecurityFilterChain springSecurity(HttpSecurity http)throws Exception{
        HttpSessionRequestCache requestCache=new HttpSessionRequestCache();
        requestCache.setMatchingRequestParameterName("continue");
        http
        // ...
        .requestCache((cache)->cache
        .requestCache(requestCache)
        );
        return http.build();
        }
```

# Require Explicit Invocation of SessionAuthenticationStrategy

Spring Security
5では、デフォルトの設定はSessionManagementFilterに依存して、ユーザーが認証されたかどうかを検出し、SessionAuthenticationStrategyを呼び出すようになっています。この問題は、典型的なセットアップでは、リクエストごとにHttpSessionを読み込まなければならないことを意味することです。

Spring Security
6では、認証機構自身がSessionAuthenticationStrategyを呼び出さなければならないというのがデフォルトです。つまり、認証が完了したことを検出する必要がないので、リクエストごとにHttpSessionを読み込む必要がない。

新しいSpring Security 6のデフォルトにオプトインするためには、次のような構成が考えられます。

```java
@Bean
DefaultSecurityFilterChain springSecurity(HttpSecurity http)throws Exception{
        http
        // ...
        .sessionManagement((sessions)->sessions
        .requireExplicitAuthenticationStrategy(true)
        );
        return http.build();
        }
```

もし、これがあなたのアプリケーションを壊してしまうのであれば、以下の設定を使用して、5.8のデフォルトを明示的に選択することができます：

```java
@Bean
DefaultSecurityFilterChain springSecurity(HttpSecurity http)throws Exception{
        http
        // ...
        .sessionManagement((sessions)->sessions
        .requireExplicitAuthenticationStrategy(false)
        );
        return http.build();
        }
```
