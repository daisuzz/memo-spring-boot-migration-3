Spring Securityチームは、Spring Security 6.0へのアップグレードを簡素化するために、5.8リリースを準備しました。5.8とその準備手順を使用して、6.0へのアップデートを簡素化しましょう。

5.8へのアップデート後、このガイドに従って、残りの移行やクリーンアップのステップを実行してください。

また、万が一トラブルが発生した場合、準備ガイドには5.xの動作に戻すためのオプトアウト手順が記載されていることを思い出してください。

# Update to Spring Security 6.0

まず、Spring Boot 3.0の最新パッチリリースであることを確認します。次に、Spring Security 6.0の最新パッチリリースであることを確認する必要があります。Spring Security
6.0にアップデートする方法については、リファレンスガイドの[Getting Spring Security](https://docs.spring.io/spring-security/reference/getting-spring-security.html)セクションを参照してください。

# Update Package Names
アップデートされたので、javax importをjakarta importに変更する必要があります。

# Perform Application-Specific Steps
次に、[Servlet](https://docs.spring.io/spring-security/reference/migration/servlet/index.html)アプリケーションかReactiveアプリケーションかによって、実行すべき手順があります。
