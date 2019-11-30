## 12. OAuth2

### 12.1. OAuth 2.0 Login

#### 12.1.3. CommonOAuth2Provider

`CommonOauth2Provider`为许多著名的提供商预先定义了一组默认的客户端属性：Google，GitHup，Facebook和Okta。

例如，对于提供商的`authorization-uri`，`token-uri`和`user-info-uri`不会经常变更。因此，提供默认值以减少配置是有意义的。

正如前面所演示的，在配置Google客户端时，只需要配置`client-id`和`client-secret`属性。

下面的清单列举了一个例子：

```properties
spring.security.oauth2.client.registration.google.client-id=google-client-id
spring.security.oauth2.client.registration.google.client-secret=google-client-secret
```

> ![](https://docs.spring.io/spring-security/site/docs/5.2.2.BUILD-SNAPSHOT/reference/htmlsingle/images/tip.png)配置如上客户端属性就可以自动工作，因为`registrationId`(`google`)与`CommonOauth2Provider`中的`GOOGLE` `enum`（不区分大小写）匹配。

在某些情况下，我们可能会需要配置两个以上的客户端（不同的`registrationId`，例如`Google`），仍然可以通过`provider`属性来利用预先定义的默认客户端属性。

下面的清单列举了一个例子：

```properties
spring.security.oauth2.client.registration.google-login.provider=google
spring.security.oauth2.client.registration.google-login.client-id=google-client-id
spring.security.oauth2.client.registration.google-login.client-secret=google-client-secret
```

- `registrationId`被设置为`google-login`。
- `propvider`属性设置为`google`，这将利用`CommonOAuth2Provider.GOOGLE.getBuilder()`中设置Google的默认客户端属性。

#### 12.1.4. Configuring Custom Provider Properties

有一些OAuth 2.0提供商支持多租户，每一个租户（或子域）使用不同的协议端点。

例如，注册的Okta OAuth客户端将被分配到特定的子域，并且具有自己的协议端点。

对于这种情况，Spring Boot 2.x为配置自定义提供商属性提供了一下基本属性：

```properties
spring.security.oauth2.client.provider.[providerId]
```

下面的清单列举了一个例子：

```properties
spring.security.oauth2.client.registration.okta.client-id=okta-client-id
spring.security.oauth2.client.registration.okta.client-secret=okta-client-secret

spring.security.oauth2.client.provider.okta.authorization-uri=authorization-uri
spring.security.oauth2.client.provider.okta.token-uri=token-uri
spring.security.oauth2.client.provider.okta.user-info-uri=user-info-uri
spring.security.oauth2.client.provider.okta.user-name-attribute=sub
spring.security.oauth2.client.provider.okta.jwk-set-uri=jwk-set-uri
```

- 基本属性`spring.security.oauth2.client.provider.okta`允许对协议端点进行自定义配置。



### 12.2 OAuth 2.0 Client

#### 12.2.1. Core Interfaces / Classes

**ClientRegistration**

`ClientRegistration`表示OAuth 2.0或OpenID Connect 1.0客户端注册的信息。

客户端注册包含如下信息，比如客户端ID，客户端密码，授权授予类型，重定向URI，范围，授权URI，令牌URI和其他详细信息。

`ClientRegistration`属性定义如下：

```java
public final class ClientRegistration {
    private String registrationId;  ①
    private String clientId;    ②
    private String clientSecret;    ③
    private ClientAuthenticationMethod clientAuthenticationMethod;  ④
    private AuthorizationGrantType authorizationGrantType;  ⑤
    private String redirectUriTemplate; ⑥
    private Set<String> scopes; ⑦
    private ProviderDetails providerDetails;
    private String clientName;  ⑧

    public class ProviderDetails {
        private String authorizationUri;    ⑨
        private String tokenUri;    ⑩
        private UserInfoEndpoint userInfoEndpoint;
        private String jwkSetUri;   ①①
        private Map<String, Object> configurationMetadata;  ①②

        public class UserInfoEndpoint {
            private String uri; ①③
            private AuthenticationMethod authenticationMethod;  ①④
            private String userNameAttributeName;   ①⑤
        }
    }
}
```

①. `registrationId`：标识`ClientRegistration`的唯一ID。

②. `clientId`：客户端标识符。

③. `clientSecret`：客户端密码。

④. `clientAuthenticationMethod`：用于向提供者验证客户端的方法。支持的值是**basic**，**post**和**none**。

⑤. `authorizationGrantType`：OAuth2.0授权框架定义了四种认证授权类型。支持的值是`autjorization_code`，`client_credentials`，`password`和`implicit`。

⑥. `redirectUriTemplate`：客户端注册的重定向URI，在终端用户对客户端进行身份验证和授权访问之后，授权服务器最终用户的用户代理重定向到该URI。

⑦. `scopes`：授权请求流程期间客户端请求的范围，如openid，email或profile。

⑧. `clientName`：用于客户端的描述性名称。该名称可用于某些场景，例如在自动生成的登录页面中显示客户端名称时

⑨. `authorizationUri`：授权服务的授权端点URI。

⑩. `tokenUri`：授权服务的令牌端点URI。

①①. `jwkSetUri`：用于从授权服务器检索JSON Web秘钥（JWK）集的URI，其中包含用于验证ID token的JSON Web签名（JWS）和UserInfo响应（可选）的加密秘钥。

①②. `configurationMetadata`：OpenId提供者配置信息。此信息只有在Spring Boot 2.x属性`spring.security.oauth2.client.provider.[providerId].issuerUri`被配置时才可用。

①③. `(UserInfoEndpoint)uri`：用于访问经过身份验证的最终用户的 声明/属性 的端点URI。

①④. `(UserInfoEndpoint)authenticationMethod`：将访问令牌发送到UserInfo端点时使用的身份验证方法。支持的值是**header**，**form**和**query**

①⑤. `userNameAttributeName`：UserInfo响应中返回的属性名，该属性引用最终用户的名称或标识符。

`ClientRegistration`可以使用OpenId Connect提供者的配置端点或授权服务的元数据端点的发现来初始化的配置。

`ClientRegistration`以这种方式 — 为配置`ClientRegistration`提供了方便的方法，如下面的示例所示：

```java
ClientRegistration clientRegistration =
    ClientRegistrations.fromIssuerLocation("https://idp.example.com/issuer").build();
```

上面的代码将连续查询 `https://idp.example.com/issuer/.well-known/openid-configuration`, 然后是 `https://idp.example.com/.well-known/openid-configuration/issuer`, 最后是 `https://idp.example.com/.well-known/oauth-authorization-server/issuer`

作为替代方案，你可以使用`ClientRegistrations.fromOidcIssuerLocation()`只查询OpenID Connect提供者的配置端点。

**ClientRegistrationRespository**

**OAuth2AuthorizedClient**

**OAuth2AuthorizedClientRepository / OAuth2AuthorizedClientService**

**OAuth2AuthorizedClientManager / OAuth2AuthorizedClientProvider**

#### 12.2.2. Authorization Grant Support











