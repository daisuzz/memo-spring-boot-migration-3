以下の手順は、認証方法の変更に関するものです。

# Use SHA-256 in Remember Me

TokenBasedRememberMeServicesの実装がRemember MeトークンにSHA-256をサポートするようになり、これはSpring Security
6のデフォルトとなりました。MD5はすでに弱いハッシュアルゴリズムであることが証明されており、衝突攻撃やモジュール差分攻撃に対して脆弱であるため、この変更はデフォルトで実装をより安全なものにするものです。

新しく生成されたトークンは、どのアルゴリズムで生成されたかという情報を持つようになり、その情報がトークンの照合に使われるようになります。アルゴリズム名が存在しない場合は、matchingAlgorithmプロパティがトークンのチェックに使用されます。これにより、MD5からSHA-256への移行がスムーズに行えるようになりました。

トークンをエンコードする新しいSpring Security
6のデフォルトを選択しつつ、MD5でエンコードされたトークンをデコードできるようにするには、encodingAlgorithmプロパティをSHA-256に、matchingAlgorithmプロパティをMD5に設定すればよいでしょう。詳しくは、[リファレンス・ドキュメント](https://docs.spring.io/spring-security/reference/5.8/servlet/authentication/rememberme.html#_tokenbasedremembermeservices)
と[APIドキュメント](https://docs.spring.io/spring-security/site/docs/5.8.3/api/org/springframework/security/web/authentication/rememberme/TokenBasedRememberMeServices.html)
を参照してください。

```java

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http, RememberMeServices rememberMeServices) throws Exception {
        http
                // ...
                .rememberMe((remember) -> remember
                        .rememberMeServices(rememberMeServices)
                );
        return http.build();
    }

    @Bean
    RememberMeServices rememberMeServices(UserDetailsService userDetailsService) {
        RememberMeTokenAlgorithm encodingAlgorithm = RememberMeTokenAlgorithm.SHA256;
        TokenBasedRememberMeServices rememberMe = new TokenBasedRememberMeServices(myKey, userDetailsService, encodingAlgorithm);
        rememberMe.setMatchingAlgorithm(RememberMeTokenAlgorithm.MD5);
        return rememberMe;
    }

}
```

ある時点で、Spring Security 6のデフォルトに完全に移行したいと思うでしょう。
しかし、いつそうするのが安全なのか、どうやって知ることができるのでしょうか？例えば、11月1日にエンコーディングアルゴリズムとしてSHA-256を使用してアプリケーションをデプロイしたとします（ここで行ったように）。tokenValiditySecondsプロパティの値をN日（14がデフォルト）に設定していれば、11月1日からN日後（この例では11月15日）、SHA-256へ移行できます。その頃には、MD5で生成されたトークンはすべて失効しています。

```java

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http, RememberMeServices rememberMeServices) throws Exception {
        http
                // ...
                .rememberMe((remember) -> remember
                        .rememberMeServices(rememberMeServices)
                );
        return http.build();
    }

    @Bean
    RememberMeServices rememberMeServices(UserDetailsService userDetailsService) {
        RememberMeTokenAlgorithm encodingAlgorithm = RememberMeTokenAlgorithm.SHA256;
        TokenBasedRememberMeServices rememberMe = new TokenBasedRememberMeServices(myKey, userDetailsService, encodingAlgorithm);
        rememberMe.setMatchingAlgorithm(RememberMeTokenAlgorithm.SHA256);
        return rememberMe;
    }

}
```

Spring Security 6のデフォルトで問題がある場合、以下の設定を使用して5.8のデフォルトを明示的にオプトインすることができます：

```java

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http, RememberMeServices rememberMeServices) throws Exception {
        http
                // ...
                .rememberMe((remember) -> remember
                        .rememberMeServices(rememberMeServices)
                );
        return http.build();
    }

    @Bean
    RememberMeServices rememberMeServices(UserDetailsService userDetailsService) {
        RememberMeTokenAlgorithm encodingAlgorithm = RememberMeTokenAlgorithm.MD5;
        TokenBasedRememberMeServices rememberMe = new TokenBasedRememberMeServices(myKey, userDetailsService, encodingAlgorithm);
        rememberMe.setMatchingAlgorithm(RememberMeTokenAlgorithm.MD5);
        return rememberMe;
    }

}
```

# Propagate AuthenticationServiceExceptions

[AuthenticationFilter](https://docs.spring.io/spring-security/site/docs/5.8.3/api/org/springframework/security/web/authentication/AuthenticationFilter.html)
は、
[AuthenticationServiceExceptions](https://docs.spring.io/spring-security/site/docs/5.8.3/api/org/springframework/security/authentication/AuthenticationServiceException.html)
を
AuthenticationEntryPoint に伝搬させる。AuthenticationServiceExceptions はクライアント側のエラーではなく、サーバー側のエラーを表すため、6.0
ではコンテナに伝播するように変更されました。

## Configure AuthenticationFailureHandler to rethrow AuthenticationServiceExceptions
6.0のデフォルトに備えるため、AuthenticationServiceExceptionをrethrowsするAuthenticationFailureHandlerでAuthenticationFilterインスタンスを以下のように配線します：
```java
AuthenticationFilter authenticationFilter = new AuthenticationFilter(...);
AuthenticationEntryPointFailureHandler handler = new AuthenticationEntryPointFailureHandler(...);
handler.setRethrowAuthenticationServiceException(true);
authenticationFilter.setAuthenticationFailureHandler(handler);
```
## Opt-out Steps
AuthenticationServiceExceptionsの再投入が面倒な場合は、6.0のデフォルトを採用する代わりに、次のように値をfalseに設定することができます：
```java
AuthenticationFilter authenticationFilter = new AuthenticationFilter(...);
AuthenticationEntryPointFailureHandler handler = new AuthenticationEntryPointFailureHandler(...);
handler.setRethrowAuthenticationServiceException(false);
authenticationFilter.setAuthenticationFailureHandler(handler);
```
