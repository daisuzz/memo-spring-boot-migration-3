以下の手順は、HttpSecurity、WebSecurity、AuthenticationManagerの構成方法の変更に関するものです。

# Add @Configuration annotation

6.0では、@EnableWebSecurity、@EnableMethodSecurity、@EnableGlobalMethodSecurity、@EnableGlobalAuthenticationから@Configurationが削除されています。

これに備えて、これらのアノテーションのいずれかを使用している場所では、@Configurationを追加する必要がある場合があります。例えば、@EnableMethodSecurityは、から変わります：

変更前

```java

@EnableMethodSecurity
public class MyConfiguration {
    // ...
}
```

変更後

```java

@Configuration
@EnableMethodSecurity
public class MyConfiguration {
    // ...
}
```

# Use the new requestMatchers methods

Spring Security 5.8では、
antMatchers、mvcMatchers、regexMatchersメソッドは非推奨となり、[新しいrequestMatchersメソッド](https://docs.spring.io/spring-security/reference/5.8/servlet/authorization/authorize-http-requests.html#_request_matchers)
に変更されました。

新しいrequestMatchersメソッドは、[authorizeHttpRequests](https://docs.spring.io/spring-security/reference/5.8/servlet/authorization/authorize-http-requests.html)
、authorizeRequests、CSRF設定、WebSecurityCustomizerなど、専用のRequestMatcherメソッドを持っていた場所に追加されました。非推奨のメソッドは、Spring
Security 6で削除されます。

これらの新しいメソッドは、アプリケーションに最も適したRequestMatcherの実装を選択するため、より安全なデフォルトを備えています。要約すると、新しいメソッドは、アプリケーションがクラスパスにSpring
MVCを持つ場合、MvcRequestMatcher実装を選択し、Spring MVCが存在しない場合はAntPathRequestMatcher実装に戻る（Kotlinの同等メソッドと動作を合わせる）。

新しいメソッドの使用を開始するには、非推奨のメソッドを新しいメソッドに置き換えることができます。例えば、次のようなアプリケーション構成です：

```java

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests((authz) -> authz
                        .antMatchers("/api/admin/**").hasRole("ADMIN")
                        .antMatchers("/api/user/**").hasRole("USER")
                        .anyRequest().authenticated()
                );
        return http.build();
    }

}
```

↑のコードを以下のように変更できる。

```java

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests((authz) -> authz
                        .requestMatchers("/api/admin/**").hasRole("ADMIN")
                        .requestMatchers("/api/user/**").hasRole("USER")
                        .anyRequest().authenticated()
                );
        return http.build();
    }

}
```

クラスパスにSpring MVCがあり、mvcMatchersのメソッドを使用している場合、新しいメソッドに置き換えれば、Spring
SecurityがMvcRequestMatcherの実装を選択してくれます。以下のような構成です：

```java

@Configuration
@EnableWebSecurity
@EnableWebMvc
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests((authz) -> authz
                        .mvcMatchers("/admin/**").hasRole("ADMIN")
                        .anyRequest().authenticated()
                );
        return http.build();
    }

}
```

↑のコードは以下のように書くことができる

```java

@Configuration
@EnableWebSecurity
@EnableWebMvc
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests((authz) -> authz
                        .requestMatchers("/admin/**").hasRole("ADMIN")
                        .anyRequest().authenticated()
                );
        return http.build();
    }

}
```

MvcRequestMatcherのservletPathプロパティをカスタマイズしている場合、MvcRequestMatcher.Builderを使用して、同じサーブレットパスを共有するMvcRequestMatcherインスタンスを作成できるようにしました：

```java

@Configuration
@EnableWebSecurity
@EnableWebMvc
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests((authz) -> authz
                        .mvcMatchers("/admin").servletPath("/path").hasRole("ADMIN")
                        .mvcMatchers("/user").servletPath("/path").hasRole("USER")
                        .anyRequest().authenticated()
                );
        return http.build();
    }

}
```

上記のコードは、MvcRequestMatcher.BuilderとrequestMatchersメソッドを使用して書き換えることができます：

```java

@Configuration
@EnableWebSecurity
@EnableWebMvc
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http, HandlerMappingIntrospector introspector) throws Exception {
        MvcRequestMatcher.Builder mvcMatcherBuilder = new MvcRequestMatcher.Builder(introspector).servletPath("/path");
        http
                .authorizeHttpRequests((authz) -> authz
                        .requestMatchers(mvcMatcherBuilder.pattern("/admin")).hasRole("ADMIN")
                        .requestMatchers(mvcMatcherBuilder.pattern("/user")).hasRole("USER")
                        .anyRequest().authenticated()
                );
        return http.build();
    }

}
```

新しいrequestMatchersメソッドに問題がある場合、いつでも今まで使っていたRequestMatcher実装に戻すことができます。例えば、AntPathRequestMatcherやRegexRequestMatcherの実装をまだ使いたい場合は、RequestMatcherのインスタンスを受け付けるrequestMatchersメソッドを使用することができます：

```java
import static org.springframework.security.web.util.matcher.AntPathRequestMatcher.antMatcher;
import static org.springframework.security.web.util.matcher.RegexRequestMatcher.regexMatcher;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests((authz) -> authz
                        .requestMatchers(antMatcher("/user/**")).hasRole("USER")
                        .requestMatchers(antMatcher(HttpMethod.POST, "/user/**")).hasRole("ADMIN")
                        .requestMatchers(regexMatcher(".*\\?x=y")).hasRole("SPECIAL") // matches /any/path?x=y
                        .anyRequest().authenticated()
                );
        return http.build();
    }

}
```

なお、上記のサンプルでは、可読性を高めるために、AntPathRequestMatcherとRegexRequestMatcherの静的ファクトリーメソッドを使用しています。

WebSecurityCustomizer インターフェースを使用している場合、非推奨の antMatchers メソッドを置き換えることができます：

```java
@Bean
public WebSecurityCustomizer webSecurityCustomizer(){
        return(web)->web.ignoring().antMatchers("/ignore1","/ignore2");
        }
```

↑のコードをrequestMatchersに対応するものに変更します：

```java
@Bean
public WebSecurityCustomizer webSecurityCustomizer(){
        return(web)->web.ignoring().requestMatchers("/ignore1","/ignore2");
        }
```

同じように、CSRFの設定をカスタマイズして一部のパスを無視する場合、非推奨のメソッドをrequestMatchersメソッドに置き換えることができます：

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http)throws Exception{
        http
        .csrf((csrf)->csrf
        .ignoringAntMatchers("/no-csrf")
        );
        return http.build();
        }
```

↑のコードは以下のように変更できる

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http)throws Exception{
        http
        .csrf((csrf)->csrf
        .ignoringRequestMatchers("/no-csrf")
        );
        return http.build();
        }
```

# Use the new securityMatchers methods

Spring Security 5.8では、HttpSecurityのantMatchers、mvcMatchers、requestMatchersメソッドが非推奨となり、新しいsecurityMatchersメソッドに変更されました。

これらのメソッドは、requestMatchers
メソッドを優先して[非推奨となった](https://docs.spring.io/spring-security/reference/5.8/migration/servlet/config.html#use-new-requestmatchers)
authorizeHttpRequests メソッドとは異なることに注意してください。しかし、securityMatchers
メソッドは、アプリケーションに最も適した RequestMatcher の実装を選択するという意味では、requestMatchers
メソッドと似ています。要約すると、新しいメソッドは、アプリケーションがクラスパスにSpring MVCを持つ場合、MvcRequestMatcher実装を選択し、Spring
MVCが存在しない場合はAntPathRequestMatcher実装に戻る（Kotlinの同等メソッドと動作を合わせる）。securityMatchersメソッドを追加したもう一つの理由は、authorizeHttpRequestsのrequestMatchersメソッドとの混同を避けるためです。

以下の構成です：

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http)throws Exception{
        http
        .antMatcher("/api/**","/app/**")
        .authorizeHttpRequests((authz)->authz
        .requestMatchers("/api/admin/**").hasRole("ADMIN")
        .anyRequest().authenticated()
        );
        return http.build();
        }
```

は、securityMatchersのメソッドを使用して書き換えることができます：

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http)throws Exception{
        http
        .securityMatcher("/api/**","/app/**")
        .authorizeHttpRequests((authz)->authz
        .requestMatchers("/api/admin/**").hasRole("ADMIN")
        .anyRequest().authenticated()
        );
        return http.build();
        }
```

HttpSecurityの設定でカスタムRequestMatcherを使用している場合：

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http)throws Exception{
        http
        .requestMatcher(new MyCustomRequestMatcher())
        .authorizeHttpRequests((authz)->authz
        .requestMatchers("/api/admin/**").hasRole("ADMIN")
        .anyRequest().authenticated()
        );
        return http.build();
        }

public class MyCustomRequestMatcher implements RequestMatcher {
    // ...
}
```

を使えば、securityMatcherを使ったのと同じことができる：

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http)throws Exception{
        http
        .securityMatcher(new MyCustomRequestMatcher())
        .authorizeHttpRequests((authz)->authz
        .requestMatchers("/api/admin/**").hasRole("ADMIN")
        .anyRequest().authenticated()
        );
        return http.build();
        }

public class MyCustomRequestMatcher implements RequestMatcher {
    // ...
}
```

HttpSecurityの設定において、複数のRequestMatcherの実装を組み合わせている場合：

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http)throws Exception{
        http
        .requestMatchers((matchers)->matchers
        .antMatchers("/api/**","/app/**")
        .mvcMatchers("/admin/**")
        .requestMatchers(new MyCustomRequestMatcher())
        )
        .authorizeHttpRequests((authz)->authz
        .requestMatchers("/admin/**").hasRole("ADMIN")
        .anyRequest().authenticated()
        );
        return http.build();
        }
```

securityMatchersを使用することで変更することができます：

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http)throws Exception{
        http
        .securityMatchers((matchers)->matchers
        .requestMatchers("/api/**","/app/**","/admin/**")
        .requestMatchers(new MyCustomRequestMatcher())
        )
        .authorizeHttpRequests((authz)->authz
        .requestMatchers("/admin/**").hasRole("ADMIN")
        .anyRequest().authenticated()
        );
        return http.build();
        }
```

もし、securityMatchersメソッドがRequestMatcherの実装を選んでくれるのに問題がある場合は、いつでも自分でRequestMatcherの実装を選択することができます：

```java
import static org.springframework.security.web.util.matcher.AntPathRequestMatcher.antMatcher;

@Bean
public SecurityFilterChain filterChain(HttpSecurity http)throws Exception{
        http
        .securityMatchers((matchers)->matchers
        .requestMatchers(antMatcher("/api/**"),antMatcher("/app/**"))
        )
        .authorizeHttpRequests((authz)->authz
        .requestMatchers(antMatcher("/api/admin/**")).hasRole("ADMIN")
        .anyRequest().authenticated()
        );
        return http.build();
        }
```

# Stop Using WebSecurityConfigurerAdapter

## Publish a SecurityFilterChain Bean

Spring Security 5.4では、WebSecurityConfigurerAdapterを拡張する代わりに、SecurityFilterChain
Beanを公開する機能が導入されました。6.0では、WebSecurityConfigurerAdapterは削除されます。この変更に備えるために、以下のような構成要素を置き換えることができます：

変更前

```java

@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests((authorize) -> authorize
                        .anyRequest().authenticated()
                )
                .httpBasic(withDefaults());
    }

}
```

変更後

```java

@Configuration
public class SecurityConfiguration {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests((authorize) -> authorize
                        .anyRequest().authenticated()
                )
                .httpBasic(withDefaults());
        return http.build();
    }

}
```

## Publish a WebSecurityCustomizer Bean

Spring Security 5.4では、WebSecurityConfigurerAdapterのconfigure(WebSecurity web)
を置き換えるために[WebSecurityCustomizer](https://github.com/spring-projects/spring-security/issues/8978)
を導入しました。その廃止に備え、以下のようなコードに置き換えておくとよいでしょう：

変更前

```java

@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Override
    public void configure(WebSecurity web) {
        web.ignoring().antMatchers("/ignore1", "/ignore2");
    }

}
```

変更後

```java

@Configuration
public class SecurityConfiguration {

    @Bean
    public WebSecurityCustomizer webSecurityCustomizer() {
        return (web) -> web.ignoring().antMatchers("/ignore1", "/ignore2");
    }

}
```

## Publish an AuthenticationManager Bean

WebSecurityConfigurerAdapeterの削除に伴い、configure(AuthenticationManagerBuilder)も削除されます。削除の準備は、使用する理由によって異なります。

### LDAP Authentication

[LDAP認証のサポート](https://docs.spring.io/spring-security/reference/5.8/servlet/authentication/passwords/ldap.html)
にauth.ldapAuthentication()を使用している場合、置き換えることができます：

変更前

```java

@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
                .ldapAuthentication()
                .userDetailsContextMapper(new PersonContextMapper())
                .userDnPatterns("uid={0},ou=people")
                .contextSource()
                .port(0);
    }

}
```

変更後

```java

@Configuration
public class SecurityConfiguration {
    @Bean
    public EmbeddedLdapServerContextSourceFactoryBean contextSourceFactoryBean() {
        EmbeddedLdapServerContextSourceFactoryBean contextSourceFactoryBean =
                EmbeddedLdapServerContextSourceFactoryBean.fromEmbeddedLdapServer();
        contextSourceFactoryBean.setPort(0);
        return contextSourceFactoryBean;
    }

    @Bean
    AuthenticationManager ldapAuthenticationManager(BaseLdapPathContextSource contextSource) {
        LdapBindAuthenticationManagerFactory factory =
                new LdapBindAuthenticationManagerFactory(contextSource);
        factory.setUserDnPatterns("uid={0},ou=people");
        factory.setUserDetailsContextMapper(new PersonContextMapper());
        return factory.createAuthenticationManager();
    }
}
```

### JDBC Authentication

[JDBC認証対応](https://docs.spring.io/spring-security/reference/5.8/servlet/authentication/passwords/jdbc.html)
でauth.jdbcAuthentication()を使っている場合は、置き換えることができます：

変更前

```java

@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .build();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        UserDetails user = User.withDefaultPasswordEncoder()
                .username("user")
                .password("password")
                .roles("USER")
                .build();
        auth.jdbcAuthentication()
                .withDefaultSchema()
                .dataSource(this.dataSource)
                .withUser(user);
    }
}
```

変更後

```java

@Configuration
public class SecurityConfiguration {
    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .addScript(JdbcDaoImpl.DEFAULT_USER_SCHEMA_DDL_LOCATION)
                .build();
    }

    @Bean
    public UserDetailsManager users(DataSource dataSource) {
        UserDetails user = User.withDefaultPasswordEncoder()
                .username("user")
                .password("password")
                .roles("USER")
                .build();
        JdbcUserDetailsManager users = new JdbcUserDetailsManager(dataSource);
        users.createUser(user);
        return users;
    }
}
```

### In-Memory Authentication

[インメモリ認証対応](https://docs.spring.io/spring-security/reference/5.8/servlet/authentication/passwords/in-memory.html)
でauth.inMemoryAuthentication()を使用している場合は、置き換えることができます：

変更前

```java

@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        UserDetails user = User.withDefaultPasswordEncoder()
                .username("user")
                .password("password")
                .roles("USER")
                .build();
        auth.inMemoryAuthentication()
                .withUser(user);
    }
}
```

変更後

```java

@Configuration
public class SecurityConfiguration {
    @Bean
    public InMemoryUserDetailsManager userDetailsService() {
        UserDetails user = User.withDefaultPasswordEncoder()
                .username("user")
                .password("password")
                .roles("USER")
                .build();
        return new InMemoryUserDetailsManager(user);
    }
}
```

# Add @Configuration to @Enable* annotations

6.0では、Spring Securityの@Enable*アノテーションはすべて、@Configurationが削除されました。便利ではありますが、Springプロジェクトの他の部分、特にSpring
Frameworkの@Enable*アノテーションとの整合性がとれていませんでした。さらに、Spring Frameworkで@Configuration(proxyBeanMethods=false)
のサポートが導入されたことも、Spring Securityの@Enable*アノテーションから@Configurationメタアノテーションを削除して、ユーザーが好みの設定モードを選べるようにする理由となっています。

以下のアノテーションは、@Configurationが削除されました：

- @EnableGlobalAuthentication
- @EnableGlobalMethodSecurity
- @EnableMethodSecurity
- @EnableReactiveMethodSecurity
- @EnableWebSecurity
- @EnableWebFluxSecurity

例えば、@EnableWebSecurityを使用している場合は、変更する必要があります：

変更前

```java

@EnableWebSecurity
public class SecurityConfig {
    // ...
}
```

変更後

```java

@Configuration
@EnableWebSecurity
public class SecurityConfig {
    // ...
}
```

また、上記に挙げた他のすべてのアノテーションについても同様です。

### Other Scenarios

AuthenticationManagerBuilderをより高度なものに使用する場合は、
[独自のAuthenticationManager @Beanを発行](https://docs.spring.io/spring-security/reference/5.8/servlet/authentication/architecture.html#servlet-authentication-authenticationmanager)
するか、[HttpSecurity#authenticationManager](https://docs.spring.io/spring-security/site/docs/5.8.3/api/org/springframework/security/config/annotation/web/builders/HttpSecurity.html#authenticationManager(org.springframework.security.authentication.AuthenticationManager))
でAuthenticationManagerインスタンスをHttpSecurity DSLに配線することができます。
