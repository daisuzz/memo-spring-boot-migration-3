次のステップは、認可サポートの移行を完了する方法に関するものです。

# Use AuthorizationManager for Method Security

この機能については、これ以上の移行手順はありません。

# Use AuthorizationManager for Message Security

6.0では、<websocket-message-broker>
のデフォルトはuse-authorization-manager=trueです。そのため、移行を完了するには、websocket-message-broker@use-authorization-manager=true属性をすべて削除してください。

例.変更前

```xml

<websocket-message-broker use-authorization-manager="true"/>
```

変更後

```xml

<websocket-message-broker/>
```

この機能のために、JavaやKotlinのマイグレーションステップを増やすことはありません。

# Use AuthorizationManager for Request Security

6.0では、<http>
のデフォルトは、once-per-requestがfalse、filter-all-dispatcher-typesがtrue、use-authorization-managerがtrueになっています。また、authorizeRequests#filterSecurityInterceptorOncePerRequestのデフォルトはfalse、authorizeHttpRequests#filterAllDispatcherTypesのデフォルトはtrueになります。そのため、移行を完了するためには、デフォルトの値をすべて削除することができます。

例えば、6.0のデフォルトのfilter-all-dispatcher-typesやauthorizeHttpRequests#filterAllDispatcherTypesをこのようにオプトインした場合です：

```java
http
        .authorizeHttpRequests((authorize)->authorize
        .filterAllDispatcherTypes(true)
        // ...
        )
```

以下のようにデフォルトはできる。

```java
http
        .authorizeHttpRequests((authorize)->authorize
        // ...
        )
```

[NOTE]
once-per-request は use-authorization-manager="false" のときのみ適用され、filter-all-dispatcher-types は
use-authorization-manager="true" のときのみ適用されます。
