次の手順は、認証サポートの移行を完了する方法に関するものです。

# Propagate AuthenticationServiceExceptions

AuthenticationFilter は、AuthenticationServiceExceptions を AuthenticationEntryPoint に伝搬させる。AuthenticationServiceExceptions
はクライアント側のエラーではなく、サーバー側のエラーを表すため、6.0 ではコンテナに伝播するように変更されました。

そのため、rethrowAuthenticationServiceExceptionをtrueに設定してこの動作をオプトインした場合、次のように削除することができます：

変更前

```java
AuthenticationFilter authenticationFilter=new AuthenticationFilter(...);
        AuthenticationEntryPointFailureHandler handler=new AuthenticationEntryPointFailureHandler(...);
        handler.setRethrowAuthenticationServiceException(true);
        authenticationFilter.setAuthenticationFailureHandler(handler);
```

変更後

```java
AuthenticationFilter authenticationFilter=new AuthenticationFilter(...);
        AuthenticationEntryPointFailureHandler handler=new AuthenticationEntryPointFailureHandler(...);
        authenticationFilter.setAuthenticationFailureHandler(handler);
```

# Use SHA-256 in Remember Me

6.0では、TokenBasedRememberMeServicesはSHA-256を使用してトークンをエンコードし、マッチングします。移行を完了するには、デフォルト値をすべて削除することができます。

例えば、encodingAlgorithmとmatchingAlgorithmを6.0のデフォルトでオプトインした場合、次のようになります：

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

以下のようにデフォルトは削除することができます。

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
        return new TokenBasedRememberMeServices(myKey, userDetailsService);
    }
}
```

# Default authorities for oauth2Login()

Spring Security 5では、OAuth2またはOpenID Connect 1.0プロバイダで認証する（oauth2Login()
で認証する）ユーザーに与えられるデフォルトのGrantedAuthorityはROLE_USERです。

Spring Security 6では、OAuth2プロバイダーで認証するユーザーに与えられるデフォルトの権限はOAUTH2_USERです。OpenID Connect
1.0プロバイダで認証するユーザーに与えられるデフォルトの権限は、OIDC_USERです。6.0へのアップデートのためだけにGrantedAuthoritiesMapperを構成した場合は、完全に削除することができます。
