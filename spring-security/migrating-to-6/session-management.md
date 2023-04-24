次の手順は、セッション管理サポートの移行を完了する方法に関するものです。

# Require Explicit Saving of SecurityContextRepository

Spring Security
5では、SecurityContextはSecurityContextPersistenceFilterを使ってSecurityContextRepositoryに自動的に保存されるのがデフォルトの挙動です。保存は、HttpServletResponseがコミットされる直前、SecurityContextPersistenceFilterの直前に行う必要があります。残念ながら、SecurityContextの自動永続化は、リクエストが完了する前（つまり、HttpServletResponseをコミットする直前）に行われると、ユーザーを驚かせてしまう。また、保存が必要かどうかを判断するために状態を追跡するのは複雑で、SecurityContextRepository（つまりHttpSession）への書き込みが不要になることもあります。

Spring Security
6では、SecurityContextHolderFilterはSecurityContextRepositoryからSecurityContextを読み取り、SecurityContextHolderに入力するだけというのがデフォルトの動作になっています。リクエスト間でSecurityContextを持続させたい場合、ユーザーは明示的にSecurityContextRepositoryでSecurityContextを保存しなければならなくなった。これにより、曖昧さがなくなり、必要なときだけSecurityContextRepository（つまりHttpSession）への書き込みが必要となり、パフォーマンスが向上します。

[NOTE]
また、ログアウト時など、コンテキストを消去する際にも、コンテキストを保存する必要があります。詳しくは、[このセクション](https://docs.spring.io/spring-security/reference/servlet/authentication/session-management.html#properly-clearing-authentication)
を参照してください。

Spring Security 6の新しいデフォルトを明示的にオプトインする場合、以下の設定を削除することでSpring Security 6のデフォルトを受け入れることが可能です。

```java
public SecurityFilterChain filterChain(HttpSecurity http){
        http
        // ...
        .securityContext((securityContext)->securityContext
        .requireExplicitSave(true)
        );
        return http.build();
        }
```

設定を使用する際、SecurityContextHolderにSecurityContextを設定するコードは、リクエスト間でSecurityContextを持続させる必要がある場合、SecurityContextRepositoryにSecurityContextを保存することも重要である。

例えば、次のようなコードです：

変更前

```java
SecurityContextHolder.setContext(securityContext);
```

変更後

```java
SecurityContextHolder.setContext(securityContext);
        securityContextRepository.saveContext(securityContext,httpServletRequest,httpServletResponse);
```

# Multiple SecurityContextRepository

Spring Security 5では、デフォルトのSecurityContextRepositoryはHttpSessionSecurityContextRepositoryでした。

Spring Security
6では、デフォルトのSecurityContextRepositoryはDelegatingSecurityContextRepositoryです。6.0へのアップデートのためだけにSecurityContextRepositoryを設定したのであれば、完全に削除することが可能です。

# Deprecation in SecurityContextRepository

この非推奨に伴う、さらなる移行手順はございません。

# Optimize Querying of RequestCache

Spring Security
5では、デフォルトの動作として、保存されたリクエストをリクエストごとに問い合わせるようになっています。これは、典型的なセットアップでは、RequestCacheを使用するために、HttpSessionがリクエストごとにクエリされることを意味します。

Spring Security 6では、HTTPパラメータcontinueが定義されている場合のみ、RequestCacheがキャッシュリクエストの問い合わせを行うことがデフォルトとなっています。これにより、Spring
Securityは不必要にRequestCacheでHttpSessionを読み込むことを回避できる。

Spring Security 5では、デフォルトではHttpSessionRequestCacheを使用し、リクエストごとにキャッシュされたリクエストを問い合わせるようになっています。デフォルトを上書きしない場合(
NullRequestCacheを使用する場合)、以下の設定を使用すると、Spring Security 5.8で明示的にSpring Security 6の動作を選択することができます：

```java
@Bean
DefaultSecurityFilterChain springSecurity(HttpSecurity http) throws Exception {
	HttpSessionRequestCache requestCache = new HttpSessionRequestCache();
	requestCache.setMatchingRequestParameterName("continue");
	http
		// ...
		.requestCache((cache) -> cache
			.requestCache(requestCache)
		);
	return http.build();
}
```
