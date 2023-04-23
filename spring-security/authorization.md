# Use AuthorizationManager for Method Security

[AuthorizationManager API](https://docs.spring.io/spring-security/site/docs/5.8.3/api/org/springframework/security/authorization/AuthorizationManager.html)
とSpring
AOPの直接利用により、[Method Security](https://docs.spring.io/spring-security/reference/5.8/servlet/authorization/method-security.html)
が[簡素化](https://docs.spring.io/spring-security/reference/5.8/servlet/authorization/method-security.html#jc-enable-method-security)
されています。

これらの変更を行う際に問題が発生した場合、@EnableGlobalMethodSecurityは非推奨ですが、6.0では削除されないので、古いアノテーションに固執することで回避することができることに注意してください。

## Replace global method security with method security

EnableGlobalMethodSecurityと<global-method-security>はそれぞれ@EnableMethodSecurityと<method-security>
に代わって非推奨となりました。新しいアノテーションとXML要素は、デフォルトでSpringのpre-postアノテーションを有効にし、AuthorizationManagerを内部で使用します。

これは、以下の2つのリストが機能的に同等であることを意味します：

```java
@EnableGlobalMethodSecurity(prePostEnabled = true)
```

```java
@EnableMethodSecurity
```

プリポストアノテーションを使用しないアプリケーションでは、不要な動作を起動させないために、必ずオフにしてください。

例えば、以下があります：

```java
@EnableGlobalMethodSecurity(securedEnabled = true)
```

↑のコードは以下に変えてください

```java
@EnableMethodSecurity(securedEnabled = true, prePostEnabled = false)
```

## Use a Custom @Bean instead of subclassing DefaultMethodSecurityExpressionHandler

パフォーマンスの最適化として、MethodSecurityExpressionHandlerに、Authenticationの代わりにSupplier<Authentication>を受け取る新しいメソッドが導入された。

これにより、Spring
SecurityはAuthenticationのルックアップを延期することができ、@EnableGlobalMethodSecurityの代わりに@EnableMethodSecurityを使用すると自動的に活用される。

しかし、あなたのコードが DefaultMethodSecurityExpressionHandler を拡張し、カスタム SecurityExpressionRoot インスタンスを返すために
createSecurityExpressionRoot(Authentication, MethodInvocation) をオーバーライドしたとしましょう。これは、@EnableMethodSecurity が設定する配置が、代わりに
createEvaluationContext(Supplier<Authentication>, MethodInvocation) を呼び出すため、もはや機能しない。

嬉しいことに、このようなレベルのカスタマイズは不要なことが多い。その代わりに、必要な認可メソッドを持つカスタムBeanを作成することができます。

例えば、@PostAuthorize("hasAuthority('ADMIN')")のカスタム評価を希望しているとします。このようなカスタム@Beanを作成することができます：

```java
class MyAuthorizer {
    boolean isAdmin(MethodSecurityExpressionOperations root) {
        boolean decision = root.hasAuthority("ADMIN");
        // custom work ...
        return decision;
    }
}
```

以下のように、アノテーションの中で参照します：

```java
@PreAuthorize("@authz.isAdmin(#root)")
```

### I’d still prefer to subclass DefaultMethodSecurityExpressionHandler

DefaultMethodSecurityExpressionHandler のサブクラス化を継続する必要がある場合は、まだそうすることができます。その代わりに、createEvaluationContext(
Supplier<Authentication>, MethodInvocation)メソッドを次のようにオーバーライドします：

```java

@Component
class MyExpressionHandler extends DefaultMethodSecurityExpressionHandler {
    @Override
    public EvaluationContext createEvaluationContext(
            Supplier<Authentication> authentication, MethodInvocation mi) {
        StandardEvaluationContext context = (StandardEvaluationContext) super.createEvaluationContext(authentication, mi);
        MySecurityExpressionRoot root = new MySecurityExpressionRoot(authentication, invocation);
        root.setPermissionEvaluator(getPermissionEvaluator());
        root.setTrustResolver(new AuthenticationTrustResolverImpl());
        root.setRoleHierarchy(getRoleHierarchy());
        context.setRootObject(root);
        return context;
    }
}
```

### Opt-out Steps

これらの変更をオプトアウトする必要がある場合は、@EnableMethodSecurityの代わりに@EnableGlobalMethodSecurityを使用することができます。

## Publish a MethodSecurityExpressionHandler instead of a PermissionEvaluator

EnableMethodSecurityは、PermissionEvaluatorを拾わない。これは、そのAPIをシンプルに保つのに役立ちます。

カスタムの[PermissionEvaluator](https://docs.spring.io/spring-security/site/docs/5.8.3/api/org/springframework/security/access/PermissionEvaluator.html)
@Beanをお持ちの方は、そちらから変更してください：

変更前

```java
@Bean
static PermissionEvaluator permissionEvaluator(){
        // ... your evaluator
        }
```

変更後

```java
@Bean
static MethodSecurityExpressionHandler expressionHandler(){
        var expressionHandler=new DefaultMethodSecurityExpressionHandler();
        expressionHandler.setPermissionEvaluator(myPermissionEvaluator);
        return expressionHandler;
        }
```

## Replace any custom method-security AccessDecisionManagers

お客様のアプリケーションでは、カスタム[AccessDecisionManager](https://docs.spring.io/spring-security/site/docs/5.8.3/api/org/springframework/security/access/AccessDecisionManager.html)
または[AccessDecisionVoter](https://docs.spring.io/spring-security/site/docs/5.8.3/api/org/springframework/security/access/AccessDecisionVoter.html)
の配置がある場合があります。準備戦略は、それぞれの配置を行う理由によって異なります。あなたの状況に最適なものを見つけるために、お読みください。

### I use UnanimousBased

アプリケーションのデフォルトの投票者がUnanimousBasedを使用している場合、@EnableMethodSecurityではUnanimous-basedがデフォルトの動作なので、何もする必要はないでしょう。

しかし、デフォルトの認可マネージャを受け入れることができないことが判明した場合、AuthorizationManagers.allOfを使用して、独自の配置を構成することができます。その後、リファレンスマニュアルの詳細に従って、[カスタムのAuthorizationManagerを追加](https://docs.spring.io/spring-security/reference/5.8/servlet/authorization/method-security.html#jc-method-security-custom-authorization-manager)
してください。

### I use AffirmativeBased

もし、あなたのアプリケーションがAffirmativeBasedを使用しているなら、次のように同等のAuthorizationManagerを構築することができます：

```java
AuthorizationManager<MethodInvocation> authorization=AuthorizationManagers.anyOf(
        // ... your list of authorization managers
        )
```

AuthorizationManagerを実装した後、リファレンスマニュアルの詳細に従って、[カスタムAuthorizationManagerを追加](https://docs.spring.io/spring-security/reference/5.8/servlet/authorization/method-security.html#jc-method-security-custom-authorization-manager)
してください。

### I use ConsensusBased

ConsensusBasedに相当するフレームワークが提供されていません。その場合、デリゲートAuthorizationManagerのセットを考慮した複合AuthorizationManagerを実装してください。

AuthorizationManagerを実装した後、リファレンスマニュアルの詳細に従って、[カスタムAuthorizationManagerを追加](https://docs.spring.io/spring-security/reference/5.8/servlet/authorization/method-security.html#jc-method-security-custom-authorization-manager)
してください。

### I use a custom AccessDecisionVoter

AuthorizationManagerを実装するクラスに変更するか、アダプタを作成する必要があります。

あなたのカスタム投票者が何をしているのかを知らなければ、汎用的な解決策を推奨することは不可能です。しかし、例として、SecurityMetadataSourceとAccessDecisionVoterを@PreAuthorizeに適応させるとどのようになるかを紹介します：

```java
public final class PreAuthorizeAuthorizationManagerAdapter implements AuthorizationManager<MethodInvocation> {
    private final SecurityMetadataSource metadata;
    private final AccessDecisionVoter voter;

    public PreAuthorizeAuthorizationManagerAdapter(MethodSecurityExpressionHandler expressionHandler) {
        ExpressionBasedAnnotationAttributeFactory attributeFactory =
                new ExpressionBasedAnnotationAttributeFactory(expressionHandler);
        this.metadata = new PrePostAnnotationSecurityMetadataSource(attributeFactory);
        ExpressionBasedPreInvocationAdvice expressionAdvice = new ExpressionBasedPreInvocationAdvice();
        expressionAdvice.setExpressionHandler(expressionHandler);
        this.voter = new PreInvocationAuthorizationAdviceVoter(expressionAdvice);
    }

    public AuthorizationDecision check(Supplier<Authentication> authentication, MethodInvocation invocation) {
        List<ConfigAttribute> attributes = this.metadata.getAttributes(invocation, AopUtils.getTargetClass(invocation.getThis()));
        int decision = this.voter.vote(authentication.get(), invocation, attributes);
        if (decision == ACCESS_GRANTED) {
            return new AuthorizationDecision(true);
        }
        if (decision == ACCESS_DENIED) {
            return new AuthorizationDecision(false);
        }
        return null; // abstain
    }
}
```

AuthorizationManagerを実装した後、リファレンスマニュアルの詳細に従って、カスタムAuthorizationManagerを追加してください。

### I use AfterInvocationManager or AfterInvocationProvider

AfterInvocationManagerとAfterInvocationProviderは、呼び出しの結果について認可の判断を行います。例えば、メソッド呼び出しの場合、これらはメソッドの戻り値に関する認可決定を行う。

Spring Security
3.0では、認可の判断は@PostAuthorizeと@PostFilterのアノテーションに標準化されました。PostAuthorizeは、戻り値全体を返すことが許可されているかどうかを決定するためのものです。PostFilterは、返されたコレクション、配列、またはストリームから個々のエントリをフィルタリングするためのものです。

AfterInvocationProviderとAfterInvocationManagerは現在非推奨となっているため、この2つのアノテーションでほとんどのニーズに対応できるはずです。

AfterInvocationManagerやAfterInvocationProviderを独自に実装した場合、まず、それが何をしようとしているのか自問する必要があります。戻り値の型を認可しようとしているのであれば、AuthorizationManager<MethodInvocationResult>
を実装してAfterMethodAuthorizationManagerInterceptorを使うことを検討してください。または、カスタムBeanを発行し、@PostAuthorize("@myBean.authorize(
#root)")を使用します。

フィルタリングしようとしているのであれば、カスタムBeanを発行して@PostFilter("@mybean.authorize(#root)")
を使うことを検討してください。あるいは、必要であれば、独自のMethodInterceptorを実装することもできます。例として、PostFilterAuthorizationMethodInterceptorとPrePostMethodSecurityConfigurationを見てみてください。

### I use RunAsManager

現在、RunAsManagerに代わるものはないが、検討中である。

しかし、必要であれば、RunAsManagerをAuthorizationManager APIに適応させることは非常に簡単である。

ここでは、いくつかの疑似コードを紹介します：

```java
public final class RunAsAuthorizationManagerAdapter<T> implements AuthorizationManager<T> {
    private final RunAsManager runAs = new RunAsManagerImpl();
    private final SecurityMetadataSource metadata;
    private final AuthorizationManager<T> authorization;

    // ... constructor

    public AuthorizationDecision check(Supplier<Authentication> authentication, T object) {
        Supplier<Authentication> wrapped = (auth) -> {
            List<ConfigAttribute> attributes = this.metadata.getAttributes(object);
            return this.runAs.buildRunAs(auth, object, attributes);
        };
        return this.authorization.check(wrapped, object);
    }
}
```

AuthorizationManagerを実装した後、リファレンスマニュアルの詳細に従って、カスタムAuthorizationManagerを追加してください。

## Check for AnnotationConfigurationExceptions

EnableMethodSecurityと<method-security>は、Spring
Securityの繰り返し不可能なアノテーション、あるいは互換性のないアノテーションをより厳格に実施するようにします。どちらかに移行した後、ログにAnnotationConfigurationExceptionが表示されたら、例外メッセージの指示に従って、アプリケーションのメソッドセキュリティアノテーションの使い方をクリーンアップしてください。

# Use AuthorizationManager for Message Security

AuthorizationManager APIとSpring AOPの直接利用により、メッセージセキュリティが向上しました。

これらの変更に問題がある場合は、このセクションの最後にあるオプトアウトの手順を実行することができます。

## Ensure all messages have defined authorization rules

現在では非推奨のメッセージセキュリティサポートは、デフォルトですべてのメッセージを許可します。新しいサポートは、すべてのメッセージを拒否するという、より強力なデフォルトを備えています。

これに備えて、リクエストごとに認可ルールが存在することを確実に宣言しておく。

例えば、こんな感じのアプリケーション構成です：

```java
@Override
protected void configureInbound(MessageSecurityMetadataSourceRegistry messages){
        messages
        .simpDestMatchers("/user/queue/errors").permitAll()
        .simpDestMatchers("/admin/**").hasRole("ADMIN");
        }
```

これは以下のように変更してください

```java
@Override
protected void configureInbound(MessageSecurityMetadataSourceRegistry messages){
        messages
        .simpTypeMatchers(CONNECT,DISCONNECT,UNSUBSCRIBE).permitAll()
        .simpDestMatchers("/user/queue/errors").permitAll()
        .simpDestMatchers("/admin/**").hasRole("ADMIN")
        .anyMessage().denyAll();
        }
```

## Add @EnableWebSocketSecurity

[NOTE]
CSRFを無効にしたい場合で、Javaのコンフィギュレーションを使用している場合は、移行手順が若干異なります。EnableWebSocketSecurityを使用する代わりに、WebSocketMessageBrokerConfigurerの適切なメソッドを自分でオーバーライドすることになります。このステップの詳細については、[リファレンスマニュアル](https://docs.spring.io/spring-security/reference/5.8/servlet/integrations/websocket.html#websocket-sameorigin-disable)
を参照してください。

Java Configurationを使用している場合は、@EnableWebSocketSecurityを追加してください。

例えば、次のようにwebsocketのセキュリティ設定クラスに追加します：

```java

@EnableWebSocketSecurity
@Configuration
public class WebSocketSecurityConfig extends AbstractSecurityWebSocketMessageBrokerConfigurer {
    // ...
}
```

これにより、MessageMatcherDelegatingAuthorizationManager.Builderのプロトタイプインスタンスが利用可能になり、拡張ではなく、構成による構成を推奨します。

## Use an AuthorizationManager<Message<?>> instance

AuthorizationManagerの利用を開始するには、XMLでuse-authorization-manager属性を設定するか、JavaでAuthorizationManager<Message<?>>
@Beanを発行する必要があります。

例えば、次のようなアプリケーション構成です：

```java
@Override
protected void configureInbound(MessageSecurityMetadataSourceRegistry messages){
        messages
        .simpTypeMatchers(CONNECT,DISCONNECT,UNSUBSCRIBE).permitAll()
        .simpDestMatchers("/user/queue/errors").permitAll()
        .simpDestMatchers("/admin/**").hasRole("ADMIN")
        .anyMessage().denyAll();
        }
```

以下のように変更できます。

```java
@Bean
AuthorizationManager<Message<?>>messageSecurity(MessageMatcherDelegatingAuthorizationManager.Builder messages){
        messages
        .simpTypeMatchers(CONNECT,DISCONNECT,UNSUBSCRIBE).permitAll()
        .simpDestMatchers("/user/queue/errors").permitAll()
        .simpDestMatchers("/admin/**").hasRole("ADMIN")
        .anyMessage().denyAll();
        return messages.build();
        }
```

## Stop Implementing AbstractSecurityWebSocketMessageBrokerConfigurer

Javaの設定を使用している場合は、WebSocketMessageBrokerConfigurerを拡張するだけで済むようになりました。

例えば、AbstractSecurityWebSocketMessageBrokerConfigurerを継承したクラスがWebSocketSecurityConfigという名前であった場合：

```java

@EnableWebSocketSecurity
@Configuration
public class WebSocketSecurityConfig extends AbstractSecurityWebSocketMessageBrokerConfigurer {
    // ...
}
```

以下のコードに変更できます。

```java

@EnableWebSocketSecurity
@Configuration
public class WebSocketSecurityConfig implements WebSocketMessageBrokerConfigurer {
    // ...
}
```

## Opt-out Steps

また、オプトアウトに最適なシナリオもご紹介しています：

### I cannot declare an authorization rule for all requests

anyRequestの認可ルールにdenyAllを設定するのが面倒な場合は、次のようにpermitAllを代わりに使用してください：

```java
@Bean
AuthorizationManager<Message<?>>messageSecurity(MessageMatcherDelegatingAuthorizationManager.Builder messages){
        messages
        .simpDestMatchers("/user/queue/errors").permitAll()
        .simpDestMatchers("/admin/**").hasRole("ADMIN")
        // ...
        .anyMessage().permitAll();
        return messages.build();
        }
```

### I cannot get CSRF working, need some other AbstractSecurityWebSocketMessageBrokerConfigurer feature, or am having trouble with AuthorizationManager

Javaの場合、AbstractMessageSecurityWebSocketMessageBrokerConfigurerを使い続けることができます。非推奨とはいえ、6.0でも削除されることはないでしょう。

XMLの場合、use-authorization-manager="false "とすることで、AuthorizationManagerを使わないようにすることができます：

変更前

```xml

<websocket-message-broker>
    <intercept-message pattern="/user/queue/errors" access="permitAll"/>
    <intercept-message pattern="/admin/**" access="hasRole('ADMIN')"/>
</websocket-message-broker>
```

変更後

```xml

<websocket-message-broker use-authorization-manager="false">
    <intercept-message pattern="/user/queue/errors" access="permitAll"/>
    <intercept-message pattern="/admin/**" access="hasRole('ADMIN')"/>
</websocket-message-broker>
```

# Use AuthorizationManager for Request Security

AuthorizationManager APIにより、HTTPリクエストのセキュリティが簡素化されました。

これらの変更に問題がある場合は、このセクションの最後にあるオプトアウトの手順を実行することができます。

## Ensure that all requests have defined authorization rules

Spring Security
5.8以前では、認可ルールがないリクエストはデフォルトで許可されます。デフォルトで拒否する方がセキュリティ上強い立場なので、エンドポイントごとに認可ルールを明確に定義する必要があります。そのため、6.0では、Spring
Securityは認可ルールがないリクエストをデフォルトで拒否します。

この変更に備える最も簡単な方法は、最後の認可ルールとして適切なanyRequestルールを導入することです。推奨はdenyAllで、これは暗黙の6.0デフォルトだからです。

[NOTE]
すでにanyRequestルールが定義されていて、それに満足している場合は、この手順を省略することができます。

最後にdenyAllを追加すると以下になる：

変更前

```java
http
        .authorizeRequests((authorize)->authorize
        .filterSecurityInterceptorOncePerRequest(true)
        .mvcMatchers("/app/**").hasRole("APP")
        // ...
        )
// ...
```

変更後

```java
http
        .authorizeRequests((authorize)->authorize
        .filterSecurityInterceptorOncePerRequest(true)
        .mvcMatchers("/app/**").hasRole("APP")
        // ...
        .anyRequest().denyAll()
        )
// ...
```

すでにauthorizeHttpRequestsに移行している場合、推奨する変更方法は同じです。

## Switch to AuthorizationManager

AuthorizationManagerの使用をオプトインするには、JavaではauthorizeHttpRequestsを、XMLではuse-authorization-managerをそれぞれ使用することができます。

変更前

```java
http
        .authorizeRequests((authorize)->authorize
        .filterSecurityInterceptorOncePerRequest(true)
        .mvcMatchers("/app/**").hasRole("APP")
        // ...
        .anyRequest().denyAll()
        )
// ...
```

変更後

```java
http
        .authorizeHttpRequests((authorize)->authorize
        .shouldFilterAllDispatcherTypes(false)
        .mvcMatchers("/app/**").hasRole("APP")
        // ...
        .anyRequest().denyAll()
        )
// ...
```

## Migrate SpEL expressions to AuthorizationManager

認可ルールについては、SpELよりもJavaの方がテストやメンテナンスがしやすい傾向にあります。そのため、authorizeHttpRequestsには、SpELのStringを宣言するメソッドは用意されていない。

代わりに、独自の AuthorizationManager の実装を行うか、WebExpressionAuthorizationManager を使用することができます。

完全性を期すため、両方のオプションのデモンストレーションを行います。

まず、以下のSpELをお持ちの方：

```java
http
        .authorizeRequests((authorize)->authorize
        .filterSecurityInterceptorOncePerRequest(true)
        .mvcMatchers("/complicated/**").access("hasRole('ADMIN') || hasAuthority('SCOPE_read')")
        // ...
        .anyRequest().denyAll()
        )
// ...
```

そして、Spring Securityの認可プリミティブを使った独自のAuthorizationManagerを次のように構成することができます：

```java
http
        .authorizeHttpRequests((authorize)->authorize
        .shouldFilterAllDispatcherTypes(false)
        .mvcMatchers("/complicated/**").access(anyOf(hasRole("ADMIN"),hasAuthority("SCOPE_read"))
        // ...
        .anyRequest().denyAll()
        )
// ...
```

または、以下の方法でWebExpressionAuthorizationManagerを使用することができます：

```java
http
        .authorizeRequests((authorize)->authorize
        .filterSecurityInterceptorOncePerRequest(true)
        .mvcMatchers("/complicated/**").access(
        new WebExpressionAuthorizationManager("hasRole('ADMIN') || hasAuthority('SCOPE_read')")
        )
        // ...
        .anyRequest().denyAll()
        )
// ...
```

## Switch to filter all dispatcher types

Spring Security 5.8以前は、リクエストごとに一度だけ認可を実行します。つまり、REQUESTの後に実行されるFORWARDやINCLUDEなどのディスパッチャタイプは、デフォルトではセキュリティが確保されていません。

Spring Securityはすべてのディスパッチタイプをセキュアにすることが推奨されています。そのため、6.0では、Spring Securityはこのデフォルトを変更します。

そこで、最後に、すべてのディスパッチャタイプをフィルタリングするように認可ルールを変更します。

そのためには、変更する必要があります：

変更前

```java
http
        .authorizeHttpRequests((authorize)->authorize
        .shouldFilterAllDispatcherTypes(false)
        .mvcMatchers("/app/**").hasRole("APP")
        // ...
        .anyRequest().denyAll()
        )
// ...
```

変更後

```java
http
        .authorizeHttpRequests((authorize)->authorize
        .shouldFilterAllDispatcherTypes(true)
        .mvcMatchers("/app/**").hasRole("APP")
        // ...
        .anyRequest().denyAll()
        )
// ...
```

また、FilterChainProxyは、すべてのディスパッチャ・タイプに対して登録する必要があります。Spring
Bootを使用している場合は、spring.security.filter.dispatcher-typesプロパティを変更して、すべてのディスパッチャータイプを含める必要があります：

```properties
spring.security.filter.dispatcher-types=request,async,error,forward,include
```

AbstractSecurityWebApplicationInitializer を使用している場合は、getSecurityDispatcherTypes
メソッドをオーバーライドして、すべてのディスパッチャー型を返す必要があります：

```java
import org.springframework.security.web.context.*;

public class SecurityWebApplicationInitializer extends AbstractSecurityWebApplicationInitializer {

    @Override
    protected EnumSet<DispatcherType> getSecurityDispatcherTypes() {
        return EnumSet.of(DispatcherType.REQUEST, DispatcherType.ERROR, DispatcherType.ASYNC,
                DispatcherType.FORWARD, DispatcherType.INCLUDE);
    }

}
```

### Permit FORWARD when using Spring MVC

ビュー名の解決にSpring MVCを使用している場合、FORWARDリクエストを許可する必要があります。これは、Spring
MVCがビュー名と実際のビューの間のマッピングを検出したときに、ビューへの転送を実行するためです。前のセクションで見たように、Spring Security
6.0はデフォルトでFORWARDリクエストに認可を適用します。

次のような一般的な構成を考えてみましょう：

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http)throws Exception{
        http
        .authorizeHttpRequests((authorize)->authorize
        .shouldFilterAllDispatcherTypes(true)
        .requestMatchers("/").authenticated()
        .anyRequest().denyAll()
        )
        .formLogin((form)->form
        .loginPage("/login")
        .permitAll()
        ));
        return http.build();
        }
```

と、それに相当する以下のMVCビューマッピングの設定のいずれかを行う：

```java

@Controller
public class MyController {

    @GetMapping("/login")
    public String login() {
        return "login";
    }

}
```

```java

@Configuration
public class MyWebMvcConfigurer implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/login").setViewName("login");
    }

}
```

どちらの構成でも、/loginへのリクエストがあると、Spring
MVCはloginビューへの転送を実行し、デフォルトの構成では、src/main/resources/templates/login.htmlパスの下にあるビューです。セキュリティ設定では、/loginへのリクエストは許可されますが、それ以外のリクエストは拒否されます。これは、/templates/login.htmlのビューへのFORWARDリクエストも同様です。

この問題を解決するには、FORWARDリクエストを許可するようにSpring Securityを設定する必要があります：

```java
http
        .authorizeHttpRequests((authorize)->authorize
        .shouldFilterAllDispatcherTypes(true)
        .dispatcherTypeMatchers(DispatcherType.FORWARD).permitAll()
        .anyRequest().denyAll()
        )
// ...
```

## Replace any custom filter-security AccessDecisionManagers

お客様のアプリケーションでは、カスタムAccessDecisionManagerまたはAccessDecisionVoterの配置がある場合があります。準備戦略は、それぞれの配置を行う理由によって異なります。あなたの状況に最適なものを見つけるために、お読みください。

### I use UnanimousBased

アプリケーションでUnanimousBasedを使用する場合は、まずAccessDecisionVotersを適応または置換してから、AuthorizationManagerをこのように構築する必要があります：

```java
@Bean
AuthorizationManager<RequestAuthorizationContext> requestAuthorization(){
        PolicyAuthorizationManager policy=...;
        LocalAuthorizationManager local=...;
        return AuthorizationMangers.allOf(policy,local);
        }
```

というように、DSLに配線します：

```java
http
        .authorizeHttpRequests((authorize)->authorize.anyRequest().access(requestAuthorization))
// ...
```

[NOTE]

authorizeHttpRequestsは、任意のurlパターンにカスタムAuthorizationManagerを適用できるように設計されています。詳しくは[リファレンス](https://docs.spring.io/spring-security/reference/5.8/servlet/authorization/authorize-http-requests.html#custom-authorization-manager)
を参照してください。

### I use AffirmativeBased

もし、あなたのアプリケーションがAffirmativeBasedを使用しているなら、次のように同等のAuthorizationManagerを構築することができます：

```java
@Bean
AuthorizationManager<RequestAuthorizationContext> requestAuthorization(){
        PolicyAuthorizationManager policy=...;
        LocalAuthorizationManager local=...;
        return AuthorizationMangers.anyOf(policy,local);
        }
```

というように、DSLに配線します：

```java
http
        .authorizeHttpRequests((authorize)->authorize.anyRequest().access(requestAuthorization))
// ...
```

[NOTE]
authorizeHttpRequestsは、任意のurlパターンにカスタムAuthorizationManagerを適用できるように設計されています。詳しくは[リファレンス](https://docs.spring.io/spring-security/reference/5.8/servlet/authorization/authorize-http-requests.html#custom-authorization-manager)
を参照してください。

### I use ConsensusBased

ConsensusBasedに相当するフレームワークが提供されていません。その場合、デリゲートAuthorizationManagerのセットを考慮した複合AuthorizationManagerを実装してください。

AuthorizationManagerを実装した後、リファレンスマニュアルの詳細に従って、カスタムAuthorizationManagerを追加してください。

### I use a custom AccessDecisionVoter

AuthorizationManagerを実装するクラスに変更するか、アダプタを作成する必要があります。

あなたのカスタム投票者が何をしているのかを知らなければ、汎用的な解決策を推奨することは不可能です。しかし、例として、SecurityMetadataSource と AccessDecisionVoter
を anyRequest().authenticated() に適応させると、次のようになります：

```java
public final class AnyRequestAuthenticatedAuthorizationManagerAdapter implements AuthorizationManager<RequestAuthorizationContext> {
    private final SecurityMetadataSource metadata;
    private final AccessDecisionVoter voter;

    public PreAuthorizeAuthorizationManagerAdapter(SecurityExpressionHandler expressionHandler) {
        Map<RequestMatcher, List<ConfigAttribute>> requestMap = Collections.singletonMap(
                AnyRequestMatcher.INSTANCE, Collections.singletonList(new SecurityConfig("authenticated")));
        this.metadata = new DefaultFilterInvocationSecurityMetadataSource(requestMap);
        WebExpressionVoter voter = new WebExpressionVoter();
        voter.setExpressionHandler(expressionHandler);
        this.voter = voter;
    }

    public AuthorizationDecision check(Supplier<Authentication> authentication, RequestAuthorizationContext context) {
        List<ConfigAttribute> attributes = this.metadata.getAttributes(context);
        int decision = this.voter.vote(authentication.get(), invocation, attributes);
        if (decision == ACCESS_GRANTED) {
            return new AuthorizationDecision(true);
        }
        if (decision == ACCESS_DENIED) {
            return new AuthorizationDecision(false);
        }
        return null; // abstain
    }
}
```

AuthorizationManagerを実装した後、リファレンスマニュアルの詳細に従って、カスタムAuthorizationManagerを追加してください。

## Opt-out Steps

また、オプトアウトに最適なシナリオもご紹介しています：

### I cannot secure all dispatcher types

すべてのディスパッチャタイプを保護することができない場合は、まず、どのディスパッチャタイプに認証を必要としないかを次のように宣言してください：

```java
http
        .authorizeHttpRequests((authorize)->authorize
        .shouldFilterAllDispatcherTypes(true)
        .dispatcherTypeMatchers(FORWARD,INCLUDE).permitAll()
        .mvcMatchers("/app/**").hasRole("APP")
        // ...
        .anyRequest().denyAll()
        )
// ...
```

また、それがうまくいかない場合は、filter-all-dispatcher-types と filterAllDispatcherTypes を false に設定して、明示的に動作を停止させることもできます：

```java
http
        .authorizeHttpRequests((authorize)->authorize
        .filterAllDispatcherTypes(false)
        .mvcMatchers("/app/**").hasRole("APP")
        // ...
        )
// ...
```

または、まだauthorizeRequestsやuse-authorization-manager="false "を使用している場合は、oncePerRequestをtrueに設定してください：

```java
http
        .authorizeRequests((authorize)->authorize
        .filterSecurityInterceptorOncePerRequest(true)
        .mvcMatchers("/app/**").hasRole("APP")
        // ...
        )
// ...
```

### I cannot declare an authorization rule for all requests

anyRequestの認可ルールにdenyAllを設定するのが面倒な場合は、次のようにpermitAllを代わりに使用してください：

```java
http
        .authorizeHttpReqeusts((authorize)->authorize
        .mvcMatchers("/app/*").hasRole("APP")
        // ...
        .anyRequest().permitAll()
        )
```

### I cannot migrate my SpEL or my AccessDecisionManager

SpELやAccessDecisionManager、あるいは<http>やauthorizeRequestsで使い続けなければならない機能がある場合、以下のことを試してみてください。

まず、authorizeRequestsがまだ必要であれば、使い続けていただいて結構です。非推奨とはいえ、6.0で削除されるわけではありません。

次に、カスタムのaccess-decision-manager-refがまだ必要な場合、またはAuthorizationManagerをオプトアウトする他の理由がある場合は、そうしてください：

```xml

<http use-authorization-manager="false">
    <intercept-url pattern="/app/*" access="hasRole('APP')"/>
    <!-- ... -->
</http>
```
