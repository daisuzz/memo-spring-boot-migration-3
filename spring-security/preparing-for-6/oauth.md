# Change Default oauth2Login() Authorities

Spring Security 5では、OAuth2またはOpenID Connect 1.0プロバイダで認証する（oauth2Login()
で認証する）ユーザーに与えられるデフォルトのGrantedAuthorityはROLE_USERです。
[NOTE]
詳しくは、[「ユーザー権限のマッピング」](https://docs.spring.io/spring-security/reference/5.8/servlet/oauth2/login/advanced.html#oauth2login-advanced-map-authorities)
を参照してください。

Spring Security 6では、OAuth2プロバイダーで認証するユーザーに与えられるデフォルトの権限はOAUTH2_USERです。OpenID Connect
1.0プロバイダで認証したユーザーに与えられるデフォルトの権限は、OIDC_USERです。これらのデフォルト値により、OAuth2またはOpenID Connect
1.0プロバイダーで認証されたユーザーをより明確に区別することができます。

hasRole("USER")やhasAuthority("ROLE_USER")などの認可ルールや式を使用して、この特定の権限を持つユーザーを認可している場合、Spring Security
6の新しいデフォルトは、あなたのアプリケーションに影響を与えます。

新しいSpring Security 6のデフォルトにオプトインするためには、次のような構成があります。

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http)throws Exception{
        http
        // ...
        .oauth2Login((oauth2Login)->oauth2Login
        .userInfoEndpoint((userInfo)->userInfo
        .userAuthoritiesMapper(grantedAuthoritiesMapper())
        )
        );
        return http.build();
        }

private GrantedAuthoritiesMapper grantedAuthoritiesMapper(){
        return(authorities)->{
        Set<GrantedAuthority> mappedAuthorities=new HashSet<>();

        authorities.forEach((authority)->{
        GrantedAuthority mappedAuthority;

        if(authority instanceof OidcUserAuthority){
        OidcUserAuthority userAuthority=(OidcUserAuthority)authority;
        mappedAuthority=new OidcUserAuthority(
        "OIDC_USER",userAuthority.getIdToken(),userAuthority.getUserInfo());
        }else if(authority instanceof OAuth2UserAuthority){
        OAuth2UserAuthority userAuthority=(OAuth2UserAuthority)authority;
        mappedAuthority=new OAuth2UserAuthority(
        "OAUTH2_USER",userAuthority.getAttributes());
        }else{
        mappedAuthority=authority;
        }

        mappedAuthorities.add(mappedAuthority);
        });

        return mappedAuthorities;
        };
        }
```

## Opt-out Steps

新しいオーソリティを設定するのが面倒な場合は、以下の設定で5.8のROLE_USERのオーソリティを明示的に使用することができます。

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http)throws Exception{
        http
        // ...
        .oauth2Login((oauth2Login)->oauth2Login
        .userInfoEndpoint((userInfo)->userInfo
        .userAuthoritiesMapper(grantedAuthoritiesMapper())
        )
        );
        return http.build();
        }

private GrantedAuthoritiesMapper grantedAuthoritiesMapper(){
        return(authorities)->{
        Set<GrantedAuthority> mappedAuthorities=new HashSet<>();

        authorities.forEach((authority)->{
        GrantedAuthority mappedAuthority;

        if(authority instanceof OidcUserAuthority){
        OidcUserAuthority userAuthority=(OidcUserAuthority)authority;
        mappedAuthority=new OidcUserAuthority(
        "ROLE_USER",userAuthority.getIdToken(),userAuthority.getUserInfo());
        }else if(authority instanceof OAuth2UserAuthority){
        OAuth2UserAuthority userAuthority=(OAuth2UserAuthority)authority;
        mappedAuthority=new OAuth2UserAuthority(
        "ROLE_USER",userAuthority.getAttributes());
        }else{
        mappedAuthority=authority;
        }

        mappedAuthorities.add(mappedAuthority);
        });

        return mappedAuthorities;
        };
        }
```

# Address OAuth2 Client Deprecations

Spring Security
6では、[OAuth2 Client](https://docs.spring.io/spring-security/reference/5.8/servlet/oauth2/client/index.html)
から非推奨のクラスとメソッドが削除されました。それぞれの非推奨を以下に列挙し、直接の代替となるものを示します。

## ServletOAuth2AuthorizedClientExchangeFilterFunction

setAccessTokenExpiresSkew(...) メソッドは、以下のいずれかに置き換えることができます：

- ClientCredentialsOAuth2AuthorizedClientProvider#setClockSkew(…​)
- RefreshTokenOAuth2AuthorizedClientProvider#setClockSkew(…​)
- JwtBearerOAuth2AuthorizedClientProvider#setClockSkew(…​)

メソッド setClientCredentialsTokenResponseClient(...) は、コンストラクタ ServletOAuth2AuthorizedClientExchangeFilterFunction(
OAuth2AuthorizedClientManager) と置き換えることができます。

[NOTE]
詳しくは、[「クライアント認証情報」](https://docs.spring.io/spring-security/reference/5.8/servlet/oauth2/client/authorization-grants.html#oauth2Client-client-creds-grant)
を参照してください。

## OidcUserInfo

phoneNumberVerified(String)メソッドは、phoneNumberVerified(Boolean)に置き換えることができます。

## OAuth2AuthorizedClientArgumentResolver

メソッド setClientCredentialsTokenResponseClient(...) は、コンストラクタ OAuth2AuthorizedClientArgumentResolver(
OAuth2AuthorizedClientManager) と置き換えることができる。

[NOTE]
詳しくは、[「クライアント認証情報」](https://docs.spring.io/spring-security/reference/5.8/servlet/oauth2/client/authorization-grants.html#oauth2Client-client-creds-grant)
を参照してください。

## ClaimAccessor

メソッド containsClaim(...) は hasClaim(...) と置き換えることができる。

## OidcClientInitiatedLogoutSuccessHandler

setPostLogoutRedirectUri(URI)メソッドは、setPostLogoutRedirectUri(String)に置き換えることができます。

## HttpSessionOAuth2AuthorizationRequestRepository

メソッドsetAllowMultipleAuthorizationRequests(...)に直接の代替はない。

## AuthorizationRequestRepository

removeAuthorizationRequest(HttpServletRequest) メソッドは removeAuthorizationRequest(HttpServletRequest, HttpServletResponse)
に置き換えることができます。

## ClientRegistration

getRedirectUriTemplate()メソッドは、getRedirectUri()に置き換えることができます。

## ClientRegistration.Builder

メソッド redirectUriTemplate(...) は redirectUri(...) と置き換えることができます。

## AbstractOAuth2AuthorizationGrantRequest

コンストラクタ AbstractOAuth2AuthorizationGrantRequest(AuthorizationGrantType) は AbstractOAuth2AuthorizationGrantRequest(
AuthorizationGrantType, ClientRegistration) と置き換えることができます。

## ClientAuthenticationMethod

静的フィールドBASICは、CLIENT_SECRET_BASICに置き換えることができる。

静的フィールドPOSTは、CLIENT_SECRET_POSTに置き換えることができます。

## OAuth2AccessTokenResponseHttpMessageConverter

tokenResponseConverter フィールドは、直接の代替となるものがない。

setTokenResponseConverter(...) メソッドは setAccessTokenResponseConverter(...) と置き換えることができます。

tokenResponseParametersConverterというフィールドは、直接的に代替するものがない。

setTokenResponseParametersConverter(...) メソッドは setAccessTokenResponseParametersConverter(...) と置き換えることができます。

## NimbusAuthorizationCodeTokenResponseClient

NimbusAuthorizationCodeTokenResponseClient クラスは、DefaultAuthorizationCodeTokenResponseClient に置き換えることができます。

## NimbusJwtDecoderJwkSupport

NimbusJwtDecoderJwkSupport というクラスは、NimbusJwtDecoder や JwtDecoders で置き換えることができます。

## ImplicitGrantConfigurer

ImplicitGrantConfigurer クラスは、直接の代替はありません。

[WARNING]
暗黙のグラントタイプの使用は推奨されておらず、Spring Security 6では関連するすべてのサポートが削除されました。

## AuthorizationGrantType

静的フィールドIMPLICITに直接の代替はない。

[WARNING]
暗黙のグラントタイプの使用は推奨されておらず、Spring Security 6では関連するすべてのサポートが削除されました。

## OAuth2AuthorizationResponseType

静的フィールドTOKENに直接の代替はない。

[WARNING]
暗黙のグラントタイプの使用は推奨されておらず、Spring Security 6では関連するすべてのサポートが削除されました。

## OAuth2AuthorizationRequest

静的メソッドimplicit()は、直接の代替はありません。

[WARNING]
暗黙のグラントタイプの使用は推奨されておらず、Spring Security 6では関連するすべてのサポートが削除されました。

# Address JwtAuthenticationConverter Deprecation

extractAuthorities
メソッドは削除されます。
JwtAuthenticationConverterを拡張する代わりに、JwtAuthenticationConverter#setJwtGrantedAuthoritiesConverterでカスタム付与権限変換器を供給してください。

