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

⑤. `authorizationGrantType`：OAuth2.0授权框架定义了四种认证授权类型。支持的值是`authorization_code`，`client_credentials`，`password`和`implicit`。

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

**ClientRegistrationRepository**

`ClientRegistrationRepository`是作为OAuth2.0/OpenID Connect 1.0 `ClientRegistration`(s)的存储库。

> ![](https://docs.spring.io/spring-security/site/docs/5.2.2.BUILD-SNAPSHOT/reference/htmlsingle/images/tip.png)客户端注册信息最终由关联性的授权服务器存储和拥有。这个存储库提供了检索主要客户端注册信息子集的能力，这些信息存储在授权服务器中。

Spring Boot 2.x自动配置将`spring.security.oauth2.client.registration.[registrationId]`下的每个属性绑定到`ClientRegistration`实例，然后在`ClientRegistrationRepository`中组合每个`ClientRegistration`实例。

> ![](https://docs.spring.io/spring-security/site/docs/5.2.2.BUILD-SNAPSHOT/reference/htmlsingle/images/tip.png)`ClientRegistrationRespository`的默认实现是`InMemoryClientRegistrationRepository`。

自动配置还在`ApplicationContext`中将`ClientRegistrationRepository`注册为`@Bean`，以便在应用程序需要时，可以对其进行依赖注入。

下面的清单列举了一个例子：

```java
@Controller
public class OAuth2ClientController {

    @Autowired
    private ClientRegistrationRepository clientRegistrationRepository;

    @GetMapping("/")
    public String index() {
        ClientRegistration oktaRegistration =
            this.clientRegistrationRepository.findByRegistrationId("okta");
        ...
        return "index";
    }
}
```



**OAuth2AuthorizedClient**

`OAuth2AuthorizedClient`是一个授权客户端的表示。当最终用户（资源所有者）授予客户端访问其受保护资源的权限时，客户端被视为已授权。

`OAuth2AuthorizedClient`的作用是将`OAuth2AccessToken`（可选`OAuth2RefreshToken`）与`ClientRegistration`（client）和资源所有者（授予授权的主要最终用户）相关联。

**OAuth2AuthorizedClientRepository / OAuth2AuthorizedClientService**

`OAuth2AuthorizedClientRepository`负责的是在Web请求之间持久化`OAuth2AuthorizedClient`(s)。然而，`OAuth2AuthorizedClientService`的主要作用是在应用程序级管理`OAuth2AuthorizedClient`(s)。

从开发人员的角度来看，`OAuth2AuthorizedClientRepository`或`OAuth2AuthorizedClientService`提供了查找与客户端关联的`OAuth2AccessToken`的功能，以便可以使用它来发起受保护资源的请求。

下面的清单显示了一个例子：

```java
@Controller
public class OAuth2ClientController {

    @Autowired
    private OAuth2AuthorizedClientService authorizedClientService;

    @GetMapping("/")
    public String index(Authentication authentication) {
        OAuth2AuthorizedClient authorizedClient =
            this.authorizedClientService.loadAuthorizedClient("okta", authentication.getName());

        OAuth2AccessToken accessToken = authorizedClient.getAccessToken();

        ...

        return "index";
    }
}
```

> ![](https://docs.spring.io/spring-security/site/docs/5.2.2.BUILD-SNAPSHOT/reference/htmlsingle/images/tip.png)Spring Boot 2.x自动配置在`ApplicationContext`中注册一个`OAuth2AuthorizedClientRepository`和/或`OAuth2AuthorizedClientService` `@Bean`。但是，应用程序可以选择覆盖和注册自定义`OAuth2AuthorizedClientRepository`或`OAuth2AuthorizedClientService` `@Bean`。

**OAuth2AuthorizedClientManager / OAuth2AuthorizedClientProvider**

`OAuth2AuthorizedClientManager`负责的是全面管理`OAth2AuthorizedClient`(s)。

主要负责包括：

- 使用`OAuth2AuthorizedClientProvider`授权（或重新授权）OAuth2.0客户端。
- 通过使用委派的方式持久化`OAuth2AuthorizedClient`，通常委派对象使用`OAuth2AuthorizedClientService`或`OAuth2AuthorizedClientRepository`。

`OAuth2AuthorizedClientProvider`实现了OAuth 2.0客户端授权（或重新授权）策略。通常实现的授予授权类型：`authorization_code`,`refresh_token`,`client_credentials`和`password`等。

`OAuth2AuthorizedClientManager`的默认实现是`DefaultOAuth2AuthorizedClientManager`，它与一个`OAuth2AuthorizedClientProvider`相关联，这个`OAuth2AuthorizedClientProvider`可以使用委派组合模式支持多个授权授予类型。可以使用`OAuth2AuthorizedClientProviderBilder`配置和构建委派组合模式。

下面的代码展示了一个如何配置和构建`OAuth2AuthorizedClientProvider`组合的示例，该组合支持`authorization_code`,`refresh_token`,`client_credentials`和`password`授予授权类型。

```java
@Bean
public OAuth2AuthorizedClientManager authorizedClientManager(
        ClientRegistrationRepository clientRegistrationRepository,
        OAuth2AuthorizedClientRepository authorizedClientRepository) {

    OAuth2AuthorizedClientProvider authorizedClientProvider =
            OAuth2AuthorizedClientProviderBuilder.builder()
                    .authorizationCode()
                    .refreshToken()
                    .clientCredentials()
                    .password()
                    .build();

    DefaultOAuth2AuthorizedClientManager authorizedClientManager =
            new DefaultOAuth2AuthorizedClientManager(
                    clientRegistrationRepository, authorizedClientRepository);
    authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);

    return authorizedClientManager;
}
```

`DefaultOAuth2AuthorizedClientManager` 还与类型为`Function<OAuth2AuthorizeRequest, Map<String, Object>>`的`contextAttributesMapper`相关联，后者负责将`OAuth2AuthorizeRequest`中的属性映射到与`OAuth2AuthorizationContext`中相关联的`Map`属性。当您需要为`OAuth2AuthorizedClientProvider`提供所需要的属性（支持的属性）时，这可能很有用，例如，`PasswordOAuth2AuthorizedClientProvider`要求资源所有者的`username`和`passwword`可以在`OAuth2AuthorizationContext.getAttributes()`中获得。

下面的代码展示了一个`contextAttributesMapper`的一个示例：

```java
@Bean
public OAuth2AuthorizedClientManager authorizedClientManager(
        ClientRegistrationRepository clientRegistrationRepository,
        OAuth2AuthorizedClientRepository authorizedClientRepository) {

    OAuth2AuthorizedClientProvider authorizedClientProvider =
            OAuth2AuthorizedClientProviderBuilder.builder()
                    .password()
                    .refreshToken()
                    .build();

    DefaultOAuth2AuthorizedClientManager authorizedClientManager =
            new DefaultOAuth2AuthorizedClientManager(
                    clientRegistrationRepository, authorizedClientRepository);
    authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);

    // Assuming the `username` and `password` are supplied as `HttpServletRequest` parameters,
    // map the `HttpServletRequest` parameters to `OAuth2AuthorizationContext.getAttributes()`
    authorizedClientManager.setContextAttributesMapper(contextAttributesMapper());

    return authorizedClientManager;
}

private Function<OAuth2AuthorizeRequest, Map<String, Object>> contextAttributesMapper() {
    return authorizeRequest -> {
        Map<String, Object> contextAttributes = Collections.emptyMap();
        HttpServletRequest servletRequest = authorizeRequest.getAttribute(HttpServletRequest.class.getName());
        String username = servletRequest.getParameter(OAuth2ParameterNames.USERNAME);
        String password = servletRequest.getParameter(OAuth2ParameterNames.PASSWORD);
        if (StringUtils.hasText(username) && StringUtils.hasText(password)) {
            contextAttributes = new HashMap<>();

            // `PasswordOAuth2AuthorizedClientProvider` requires both attributes
            contextAttributes.put(OAuth2AuthorizationContext.USERNAME_ATTRIBUTE_NAME, username);
            contextAttributes.put(OAuth2AuthorizationContext.PASSWORD_ATTRIBUTE_NAME, password);
        }
        return contextAttributes;
    };
}
```

#### 12.2.2. Authorization Grant Support

##### Authorization Code

**Initiating the Authorization Request**

`OAuth2AuthorizationRequestRedirectFilter`使用`OAuth2AuthorizationRequestResolver`解析`OAuth2AuthorizationRequest`，通过将最终用户的用户代理重定向到授权服务器的授权端点来启动授权代码授予流。

`OAuth2AuthorizationRequestResolver`的主要作用是解析从web请求中提供`OAuth2AuthorizationRequest`。默认实现`DefaultOAuth2AuthorizationRequestResolver`提取`registrationId`匹配（默认）路径`/oauth2/authorization/{registrationId}`并使用它为相关的`ClientRegistration`构建`OAuth2AuthorizationRequest`。

为OAuth 2.0客户端注册Spring Boot 2.x提供了以下属性：

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          okta:
            client-id: okta-client-id
            client-secret: okta-client-secret
            authorization-grant-type: authorization_code
            redirect-uri: "{baseUrl}/authorized/okta"
            scope: read, write
        provider:
          okta:
            authorization-uri: https://dev-1234.oktapreview.com/oauth2/v1/authorize
            token-uri: https://dev-1234.oktapreview.com/oauth2/v1/token
```

使用基本路径`/`的请求将启动由`OAuth2AuthorizationRequestRegirectFilter`重定向的授权请求，并最终启动授权代码授予流。

如果OAuth 2.0客户端是公共客户端，则按以下方式配置OAuth 2.0客户端注册：

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          okta:
            client-id: okta-client-id
            client-authentication-method: none
            authorization-grant-type: authorization_code
            redirect-uri: "{baseUrl}/authorized/okta"
            ...
```

公共客户端支持使用Proof Key for Code Exchange（PKCE）。如果客户端允许在不受信任的环境中，因此无法保证其凭证的机密性，则当以下条件满足时，将自动使用PKCE：

1. `client-secret`缺省（或为空）
2. `client-authentication-method`被设置为`none`（`ClientAuthenticationMethod.NONE`）

`DefaultOAuth2AuthorizationRequestResolver`还支持使用`UriComponentBuilder`解析`regirect-uri`的`URI`模板变量。

下面的配置使用所有受支持的`URI`模板变量：

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          okta:
            ...
            redirect-uri: "{baseScheme}://{baseHost}{basePort}{basePath}/authorized/{registrationId}"
            ...
```

> `{baseUrl}`解析为`{baseScheme}://{baseHost}{basePort}{basePath}`

当OAuth 2.0客户端运行在代理服务器后面时，使用`URI`模板变量配置`regirect-uri`特别有用。这确保在展开`redirect-uri`时使用`X-Forwarded-*`头。

**Customizing the Authorization Request**

`OAuth2AuthorizationRequestResolver`可以实现的主要用例之一是能够使用OAuth 2.0授权框架中定义的标准参数之外的其他参数自定义授权请求。

例如，OpenID Connect为Authorization Code Flow定义了附加的OAuth 2.0请求参数，这些请求参数扩展了OAuth 2.0授权框架定义的标准参数。其中一个扩展参数是`prompt`。

> 可选. 空格分隔符，区分大小写的ACII字符串值列表，用于指定授权服务器是否提示最终用户进行重新认证和同意。定义的值是：none，login，consent，select_account

下面的例子显示了如何实现一个`OAuth2AuthorizationRequestResolver`，它为`oauth2Login()`自定义了授权请求，包含的请求参数`prompt=consent`。

```java
@EnableWebSecurity
public class OAuth2LoginSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private ClientRegistrationRepository clientRegistrationRepository;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests(authorizeRequests ->
                authorizeRequests
                    .anyRequest().authenticated()
            )
            .oauth2Login(oauth2Login ->
                oauth2Login
                    .authorizationEndpoint(authorizationEndpoint ->
                        authorizationEndpoint
                            .authorizationRequestResolver(
                                new CustomAuthorizationRequestResolver(
                                        this.clientRegistrationRepository))    ①
                    )
            );
    }
}

public class CustomAuthorizationRequestResolver implements OAuth2AuthorizationRequestResolver {
    private final OAuth2AuthorizationRequestResolver defaultAuthorizationRequestResolver;

    public CustomAuthorizationRequestResolver(
            ClientRegistrationRepository clientRegistrationRepository) {

        this.defaultAuthorizationRequestResolver =
                new DefaultOAuth2AuthorizationRequestResolver(
                        clientRegistrationRepository, "/oauth2/authorization");
    }

    @Override
    public OAuth2AuthorizationRequest resolve(HttpServletRequest request) {
        OAuth2AuthorizationRequest authorizationRequest =
                this.defaultAuthorizationRequestResolver.resolve(request);  ②

        return authorizationRequest != null ?   ③
                customAuthorizationRequest(authorizationRequest) :
                null;
    }

    @Override
    public OAuth2AuthorizationRequest resolve(
            HttpServletRequest request, String clientRegistrationId) {

        OAuth2AuthorizationRequest authorizationRequest =
                this.defaultAuthorizationRequestResolver.resolve(
                    request, clientRegistrationId);    ④

        return authorizationRequest != null ?   ⑤
                customAuthorizationRequest(authorizationRequest) :
                null;
    }

    private OAuth2AuthorizationRequest customAuthorizationRequest(
            OAuth2AuthorizationRequest authorizationRequest) {

        Map<String, Object> additionalParameters =
                new LinkedHashMap<>(authorizationRequest.getAdditionalParameters());
        additionalParameters.put("prompt", "consent");  ⑥

        return OAuth2AuthorizationRequest.from(authorizationRequest)    ⑦
                .additionalParameters(additionalParameters) ⑧
                .build();
    }
}
```

①、配置自定义`OAuth2AuthorizationRequestResolver`

②④、使用`DefaultOAuth2AuthorizationRequestResolver`尝试解析`OAuth2AuthorizationRequest`

③⑤、如果`OAuth2AuthorizationRequest`已经解析，则返回自定义版本，否则返回`null`

⑥、将自定义参数添加到已有的`OAuth2AuthorizationRequest.additionalParameters`中

⑦、创建`OAuth2AuthorizationRequest`的默认副本，它将返回`OAuth2AuthorizationRequest.Builder`，以作进一步修改。

⑧、重写默认的`additionalParameters`

> 

对于简单的用例，附加的请求参数对于特定的提供者始终是相同的，可以直接将其添加到`authoruzation_uri`中。

例如，如果提供者`okta`的请求参数`prompt`的值始终是`consent`，那么简单的配置如下：

```yaml
spring:
  security:
    oauth2:
      client:
        provider:
          okta:
            authorization-uri: https://dev-1234.oktapreview.com/oauth2/v1/authorize?prompt=consent
```

前面的示例展示了在标准参数之上添加自定义参数的常见用例。或者，如果你的要求更高级，那么只需要覆盖`OAuth2AuthorizationRequest.authorizationRequestUri`属性就可以完全控制构建授权请求URI。

下面的示例显示了`customAuthorizationRequest()`方法与前面示例不同的形式，而是覆盖了`OAuth2AuthorizationRequest.authorizationRequestUri`属性。

```java
private OAuth2AuthorizationRequest customAuthorizationRequest(
        OAuth2AuthorizationRequest authorizationRequest) {

    String customAuthorizationRequestUri = UriComponentsBuilder
            .fromUriString(authorizationRequest.getAuthorizationRequestUri())
            .queryParam("prompt", "consent")
            .build(true)
            .toUriString();

    return OAuth2AuthorizationRequest.from(authorizationRequest)
            .authorizationRequestUri(customAuthorizationRequestUri)
            .build();
}
```

**Storing the Authorization Request**

`AuthorizationRequestRepository`负责从发起授权请求到收到授权响应之间`OAuth2AuthorizationRequest`的持久化。

> `OAuth2AuthorizationRequest`用于关联和验证授权响应。

`AuthorizationRequestRepository`的默认实现是`HttpSessionOAuth2AuthorizationRequestRepository`，它将`OAth2AuthorizationRequest`存储在`HttpSeesion`。

如果你想实现自定义的`AuthorizationRequestRepository`，你可以根据如下示例配置：

```java
@EnableWebSecurity
public class OAuth2ClientSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .oauth2Client(oauth2Client ->
                oauth2Client
                    .authorizationCodeGrant(authorizationCodeGrant ->
                        authorizationCodeGrant
.authorizationRequestRepository(this.authorizationRequestRepository())
                            ...
                    )
            );
    }
}
```

**Requesting an Access Token**

Authorization Code授予的`OAuth2AccessTokenResponseClient`的默认实现是`DefaultAuthorizationCodeTokenResponseClient`，它使用`RestOperations`通过授权服务令牌端点为访问令牌交换授权码。

`DefaultAuthorizationCodeTokenResponseClient`非常灵活，因为它允许您自定义令牌请求的预处理和/或令牌响应的后处理。

**Customizing the Access Token Request**

如果你需要自定义令牌请求的预处理，你可以自定义一个`Converter<OAUth2AuthorizationCodeGrantRequest,RequestEntity<?>>`提供`DefaultAuthorizationCodeTokenResponseClient.setRequestEntityConverter()`。默认实现`OAuth2AuthorizationCodeGrantRequestEntityConverter`是构建标准OAuth 2.0访问令牌请求`RequestEntity`的表示。但是，提供自定义`Converter`，将允许扩展标准令牌请求并添加自定义参数。

> ![](https://docs.spring.io/spring-security/site/docs/5.2.2.BUILD-SNAPSHOT/reference/htmlsingle/images/important.png)**重要**
>
> ​			自定义`Converter`必须返回OAuth 2.0访问令牌请求的有效`RequestEntity`表示，该请求可被预期的OAuth 			2.0提供者理解。

**Customizing the Access Token Response**

另一方面，如果你需要自定义Token响应的后处理，你将需要使用自定义配置的`RestOperation`提供`DefaultAuthorizationCodeTokenResponseClient.setRestOperations()`。默认的`RestOperations`配置如下：

```java
RestTemplate restTemplate = new RestTemplate(Arrays.asList(
        new FormHttpMessageConverter(),
        new OAuth2AccessTokenResponseHttpMessageConverter()));

restTemplate.setErrorHandler(new OAuth2ErrorResponseErrorHandler());
```

> ![](https://docs.spring.io/spring-security/site/docs/5.2.2.BUILD-SNAPSHOT/reference/htmlsingle/images/tip.png)Spring MVC`FormHttpMessageConverter`是必须得，因为它是在OAuth 2.0发送访问令牌请求时使用的。

`OAuth2AccessTokenResponseHttpMessageConverter`是一个用于OAuth 2.0访问令牌响应的`HttpMessageConverter`。你可以自定义一个`Converter<Map<String,String>,OAuth2AccessTokenResponse>`提供`OAuth2AccessTokenResponseHttpMessageConverter.setTokenResponseConverter()`，它用于将OAuth 2.0访问令牌响应参数转换为`OAuth2AccessTokenResponse`。

`OAuth2ErrorResponseErrorHandler`是一个可以处理OAuth 2.0错误的`ResponseErrorHandler`，例如. 400 Bad Request。它使用`OAuth2ErrorHeepMessageConverter`将OAuth 2.0错误参数转换为`OAuth2Error`。

无论你是自定义`DefaultAuthorizationCodeTokenResponseClient`，还是提供你自己的`OAuth2AccessTokenResponseClient`的实现，你都需要按如下示例进行配置：

```java
@EnableWebSecurity
public class OAuth2ClientSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .oauth2Client(oauth2Client ->
                oauth2Client
                    .authorizationCodeGrant(authorizationCodeGrant ->
                        authorizationCodeGrant
                            .accessTokenResponseClient(this.accessTokenResponseClient())
                            ...
                     )
            );
    }
}
```

##### Refresh Token

Refreshing an Access Token

Customizing the Access Token Request

Customizing the Access Token Response

##### Client Credentials

Requesting an Access Token

Customizing the Access Token Request

Customizing the Access Token Response

Using the Access Token









#### 12.2.3. Additional Features

**Resloving an Authorized Client**

`@RegisterdOAuth2AuthorizedClient`注解提供了将方法参数解析为`OAuth2AuthorizedClient`类型的参数值的功能。与使用`OAuth2AuthorizedClientManager` 或`OAuth2AuthorizedClientService`访问`OAuthAuthorizedClient`相比，这是一种方便的替代方法。

```java
@Controller
public class OAuth2ClientController {

    @GetMapping("/")
    public String index(@RegisteredOAuth2AuthorizedClient("okta") OAuth2AuthorizedClient authorizedClient) {
        OAuth2AccessToken accessToken = authorizedClient.getAccessToken();

        ...

        return "index";
    }
}
```

`@RegisterdOAuth2AuthorizedClient`注解使用`OAuthAuthorizedClientArgumentResolver`处理。它直接使用`OAuth2AuthorizedClientManager`，因此继承了它的功能。





#### 12.2.4. WebClient integration for Servlet Environments



