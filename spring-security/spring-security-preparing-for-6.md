Spring Securityチームは、Spring Security
6.0へのアップグレードを簡略化するために5.8リリースを準備しました。5.8と以下の手順を使用して、[6.0へのアップデート](https://docs.spring.io/spring-security/reference/6.0.2/migration/index.html)
の際の変更を最小限に抑えてください。

# Update to Spring Security 5.8

まず、Spring Boot 2.7の最新パッチリリースであることを確認します。次に、Spring Security 5.8の最新パッチリリースであることを確認する必要があります。Spring
Bootを使用している場合は、Spring BootのバージョンをSpring Security 5.7から5.8にオーバーライドする必要があります。Spring Security 5.8はSpring Security
5.7と完全な互換性があり、Spring Boot 2.7にも対応しています。Spring Security
5.8へのアップデート方法については、リファレンスガイドの[Getting Spring Security](https://docs.spring.io/spring-security/reference/5.8/getting-spring-security.html)
のセクションを参照してください。

# Update Password Encoding

6.0では、PBKDF2、SCrypt、Argon2のパスワードエンコードの最小値が更新されています。

[NOTE] デフォルトのパスワードエンコーダを使用している場合は、準備する手順はありませんので、このセクションはスキップしてかまいません。

## Update Pbkdf2PasswordEncoder

[Pbkdf2PasswordEncoderを使用している](https://docs.spring.io/spring-security/reference/5.8/features/authentication/password-storage.html#authentication-password-storage-pbkdf2)
場合、コンストラクタは、与えられた設定が適用されるSpring Securityのバージョンを参照する静的ファクトリに置き換えられます。

### Replace Deprecated Constructor Usage

デフォルトのコンストラクタを使用している場合は、まず変更する必要があります：

変更前

```java
@Bean
PasswordEncoder passwordEncoder(){
        return new Pbkdf2PasswordEncoder();
        }
```

変更後

```java
@Bean
PasswordEncoder passwordEncoder(){
        return Pbkdf2PasswordEncoder.defaultsForSpringSecurity_v5_5();
        }
```

また、カスタム設定がある場合は、以下のようにすべての設定を指定するコンストラクタに変更します：

```java
@Bean
PasswordEncoder passwordEncoder(){
        PasswordEncoder current=new Pbkdf2PasswordEncoder("mysecret".getBytes(UTF_8),320000);
        return current;
        }
```

以下のように、完全に指定されたコンストラクタを使用するように変更します：

```java
@Bean
PasswordEncoder passwordEncoder(){
        PasswordEncoder current=new Pbkdf2PasswordEncoder("mysecret".getBytes(UTF_8),16,185000,256);
        return current;
        }
```

### Use DelegatingPasswordEncoder

非推奨のコンストラクタを使用しないようになったら、次のステップとして、DelegatingPasswordEncoderを使用して、最新の標準にアップグレードするためのコードを準備します。次のコードは、カレントを使用しているパスワードを検出し、最新のものに置き換えるようにデリゲートエンコーダを設定します：

```java
@Bean
PasswordEncoder passwordEncoder(){
        String prefix="pbkdf2@5.8";
        PasswordEncoder current= // ... see previous step
        PasswordEncoder upgraded=Pbkdf2PasswordEncoder.defaultsForSpringSecurity_v5_8();
        DelegatingPasswordEncoder delegating=new DelegatingPasswordEncoder(prefix,Map.of(prefix,upgraded));
        delegating.setDefaultPasswordEncoderForMatches(current);
        return delegating;
        }
```

## Update SCryptPasswordEncoder

SCryptPasswordEncoderを使用している場合、コンストラクタは、与えられた設定が適用されるSpring Securityのバージョンを参照する静的ファクトリに置き換えられます。

### Replace Deprecated Constructor Usage

デフォルトのコンストラクタを使用している場合は、まず変更する必要があります：

変更前

```java
@Bean
PasswordEncoder passwordEncoder(){
        return new SCryptPasswordEncoder();
        }
```

変更後

```java
@Bean
PasswordEncoder passwordEncoder(){
        return SCryptPasswordEncoder.defaultsForSpringSecurity_v4_1();
        }
```

### Use DelegatingPasswordEncoder

非推奨のコンストラクタを使用しないようになったら、次のステップとして、DelegatingPasswordEncoderを使用して、最新の標準にアップグレードするためのコードを準備します。次のコードは、カレントを使用しているパスワードを検出し、最新のものに置き換えるようにデリゲートエンコーダを設定します：

```java
@Bean
PasswordEncoder passwordEncoder(){
        String prefix="scrypt@5.8";
        PasswordEncoder current= // ... see previous step
        PasswordEncoder upgraded=SCryptPasswordEncoder.defaultsForSpringSecurity_v5_8();
        DelegatingPasswordEncoder delegating=new DelegatingPasswordEncoder(prefix,Map.of(prefix,upgraded));
        delegating.setDefaultPasswordEncoderForMatches(current);
        return delegating;
        }
```

## Update Argon2PasswordEncoder

[Argon2PasswordEncoderを使用している](https://docs.spring.io/spring-security/reference/5.8/features/authentication/password-storage.html#authentication-password-storage-argon2)
場合、コンストラクタは、与えられた設定が適用されるSpring Securityのバージョンを参照する静的ファクトリに置き換えられます。

### Replace Deprecated Constructor Usage

デフォルトのコンストラクタを使用している場合は、まず変更する必要があります：

変更前

```java
@Bean
PasswordEncoder passwordEncoder(){
        return new Argon2PasswordEncoder();
        }
```

変更後

```java
@Bean
PasswordEncoder passwordEncoder(){
        return Argon2PasswordEncoder.defaultsForSpringSecurity_v5_2();
        }
```

### Use DelegatingPasswordEncoder

非推奨のコンストラクタを使用しないようになったら、次のステップとして、DelegatingPasswordEncoderを使用して、最新の標準にアップグレードするためのコードを準備します。次のコードは、カレントを使用しているパスワードを検出し、最新のものに置き換えるようにデリゲートエンコーダを設定します：

```java
@Bean
PasswordEncoder passwordEncoder(){
        String prefix="argon@5.8";
        PasswordEncoder current= // ... see previous step
        PasswordEncoder upgraded=Argon2PasswordEncoder.defaultsForSpringSecurity_v5_8();
        DelegatingPasswordEncoder delegating=new DelegatingPasswordEncoder(prefix,Map.of(prefix,upgraded));
        delegating.setDefaultPasswordEncoderForMatches(current);
        return delegating;
        }
```

# Stop using Encryptors.queryableText

Encryptors.queryableText(CharSequence,CharSequence)
は、[同じ入力データから同じ出力が得られるため](https://tanzu.vmware.com/security/cve-2020-5408)
、安全とは言えません。これは非推奨であり、6.0で削除される予定です。Spring
Securityはこの方法でのデータの暗号化をサポートしなくなったからです。

アップグレードするには、対応するメカニズムで再暗号化するか、復号化して保存する必要があります。

暗号化された各エントリーをテーブルから読み出し、復号化し、サポートされているメカニズムを使用して再暗号化する次の疑似コードを考えてみましょう：

```java
TextEncryptor deprecated=Encryptors.queryableText(password,salt);
        BytesEncryptor aes=new AesBytesEncryptor(password,salt,KeyGenerators.secureRandom(12),CipherAlgorithm.GCM);
        TextEncryptor supported=new HexEncodingTextEncryptor(aes);
        for(MyEntry entry:entries){
        String value=deprecated.decrypt(entry.getEncryptedValue());  // 1
        entry.setEncryptedValue(supported.encrypt(value));             // 2
        entryService.save(entry)
        }
```

1. 上記では、非推奨のqueryableTextを使用して、値を平文に変換しています。
2. その後、サポートされているSpring Securityのメカニズムで値を再暗号化します。

3. [Spring Securityがサポートしている暗号化メカニズム](https://docs.spring.io/spring-security/reference/5.8/features/integrations/cryptography.html)
   については、リファレンスマニュアルを参照してください。

# Perform Application-Specific Steps

次に、[Servlet](https://docs.spring.io/spring-security/reference/5.8/migration/servlet/index.html)
アプリケーションか[Reactive](https://docs.spring.io/spring-security/reference/5.8/migration/reactive.html)
アプリケーションかによって、実行すべき手順があります。
