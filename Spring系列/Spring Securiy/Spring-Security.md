# Spring Security源码解读与源码分析

温馨提示，阅读此文章需要对Spring过滤其模块和Http Servlet Wrapper有所了解，关于这一块的知识点，请参考附录。

Spring Securty的认证是基于过滤器链实现的

1. 过滤器链如何创建？
2. 过滤器链如何注册？
3. 过滤器链如何调用？
4. 提供了那些过滤器
5. 这些过滤器的作用如何
6. Spring Security如何保存登陆状态

## 前言

在现在的互联网公司中，Web开发，授权认证是一个避不开的话题，基本上90%的系统都需要登录认证，无论简单用户名密码登录，还是相对比较复杂的OAuth2单点登录。为此，业界出现了很多有名开源的权限框架，其中最为被广泛使用的就是Shiro和Spring Security。

既然Shiro和Spring Security最为出名，那么孰优孰劣？其实在笔者看来它们没有高下之分，如果你对Spring Security非常熟悉，并且精通其实现原理，而对Shiro不甚了解，那么你又有什么理由不选择Spring Security呢。与此相反，道理也是一样的。

Spring Security是Spring体系的框架，它可以完美的与Spring集成，至于Spring的重要性，那也不言而喻了。

## 测试环境搭建



## 架构

学习一个知识，最好的方式之一就是先理解它是什么，然后再看它的整体结构是什么，最后才去关注想要关心的细节。如果反其道而行之，正所谓是不熟庐山真面目，只缘身在此山中。

Spring security与Spring boot一样，都是构建spring之上。要分析spring boot，其方式之一就是spring boot的启动入口一点一点地分析，而spring security却不一样，从入口分析你也仅仅只能知道它是如何启动的，仍然不知道它是怎么回事，有一叶障目之感，甚至，在不理解spring security核心原理及其相关组件架构的情况下，即使从入口分析，也可能不明如它是怎么回事。所以我们将会从大到小，从广到细的进行分析。

Spring security是通过什么样的方式实现其安全认证的呢？其架构又是怎样的呢？

过滤器，一切皆是过滤器，Spring security是通过过滤器是实现一个一个的安全认证组件，拦截需要认证的请求。而这里的过滤器也就是Servlet的Filter。不过需要注意的是，Spring security实际上只向Servlet注册了一个过滤器，这个过滤器称为过滤器链代理，其真正的认证过滤器组合成过滤器链的形式被过滤器链代理代理访问。

*Spring security架构*

![]()

如上图所示，客户端请求首先经过防火墙，然后进入到过滤器链代理中，通过请求的URL依次与过滤器链的匹配器进行匹配，以映射到对应的过滤器链。在过滤器链中包含了多个用于授权认证过滤器。待过滤器链中所有的过滤器都认证完成之后，就正式进入到业务代码层次了。

过滤器链中的过滤器，并不是所有的过滤器都是做授权访问认证的，还有包括一些与授权访问认证附属功能相关的，比如退出登录，CORSR认证，CSRF安全，安全上下文等。对于授权访问认证的过滤器，其认证的处理交由认证管理器去处理。在认证管理器中包含多个认证提供器，真正的认证处理是由认证提供器实现。认证管理器是有层次结构的，即认证管理器可以有父认证管理器，认证的时候首先会通过子认证管理器查找匹配的认证提供器进行认证，如果没有找到，就到父认证管理器中去查找，一直到找到或者到没有父认证管理器为止。

其中给一个案例

`UsernamePasswordAuthenticationFilter`过滤器就是用于用户名密码登录认证的，它的认证工作就是交由对应的认证提供器进行认证，在此认证提供器中，完成了用户密码的查询，密码校验匹配，认证成功和失败的处理等。

## 安全组件

通过前面对Spring Security结构的分析，可以知道Spring Security包含了三大核心组件，分别是安全过滤器链、过滤器链代理以及身份验证管理器。

### 安全过滤器链

安全过滤器`org.springframework.security.web.SecurityFilterChain`是一个接口，其源码如下：

```java
public interface SecurityFilterChain {
	boolean matches(HttpServletRequest request);
	List<Filter> getFilters();
}
```

可以看到，其包含了两个方法，一个是matches，即是用于匹配当前请求是否要应用当前过滤器链；getFilters就是用于获取当前链条中的所有过滤器。

在Spring security中只提供了一个默认实现：`org.springframework.security.web.DefaultSecurityFilterChain`。其源码如下：

```java
public final class DefaultSecurityFilterChain implements SecurityFilterChain {
	private static final Log logger = LogFactory.getLog(DefaultSecurityFilterChain.class);
	private final RequestMatcher requestMatcher;
	private final List<Filter> filters;

	public DefaultSecurityFilterChain(RequestMatcher requestMatcher, Filter... filters) {
		this(requestMatcher, Arrays.asList(filters));
	}

	public DefaultSecurityFilterChain(RequestMatcher requestMatcher, List<Filter> filters) {
		logger.info("Creating filter chain: " + requestMatcher + ", " + filters);
		this.requestMatcher = requestMatcher;
		this.filters = new ArrayList<>(filters);
	}

	public RequestMatcher getRequestMatcher() {
		return requestMatcher;
	}

	public List<Filter> getFilters() {
		return filters;
	}

	public boolean matches(HttpServletRequest request) {
		return requestMatcher.matches(request);
	}

	@Override
	public String toString() {
		return "[ " + requestMatcher + ", " + filters + "]";
	}
}
```

实现很简单，主要就是包含一个用于匹配请求的RequestMatcher，和一个用于存储过滤器的List集合。

### 过滤器链代理

过滤器链代理`org.springframework.security.web.FilterChainProxy`继承了`org.springframework.web.filter.GenericFilterBean`，它是Spring security正真被注册到Servlet中的过滤器。

```java
public class FilterChainProxy extends GenericFilterBean {
    ......
}
```

FilterChainProxy主要包含了三个成员变量，如下所示：

```java
public class FilterChainProxy extends GenericFilterBean {
    ......
    private List<SecurityFilterChain> filterChains;
	private FilterChainValidator filterChainValidator = new NullFilterChainValidator();
	private HttpFirewall firewall = new StrictHttpFirewall();
    ......
}
```

第一个是filterChains，存储过滤器链的。需要注意的是它是一个List集合，也就说明了在Spring security中可以有多个过滤器链。

第二个参数是filterChainValidator，？？？

第三个参数是防火墙了。在后续会详细说明。

doFilter

看FilterChainProxy的实现，入口是doFilter。

```java
public void doFilter(ServletRequest request, ServletResponse response,
                     FilterChain chain) throws IOException, ServletException {
    boolean clearContext = request.getAttribute(FILTER_APPLIED) == null;
    if (clearContext) {
        try {
            request.setAttribute(FILTER_APPLIED, Boolean.TRUE);
            doFilterInternal(request, response, chain);
        }
        finally {
            SecurityContextHolder.clearContext();
            request.removeAttribute(FILTER_APPLIED);
        }
    }
    else {
        doFilterInternal(request, response, chain);
    }
}
```

分析上面的代码，if...else...上下两段唯一的不同是，如果clearContext为true，即在ServletRequest上下文中存在`FILTER_APPLIED`的值，那么在最后需要执行安全上下文的清理，并且从ServletRequest上下文中删除`FILTER_APPLIED`。看一下clearContext为true的条件，`request.getAttribute(FILTER_APPLIED) == null`，即一个新的请求过来，clearContext必定为true，从这一点儿来看，没有问题，执行完成请求之后，清理上下文资源，以防止垃圾数据堆积。可以这里为什么要这这样的判定呢？想象一下：当一个请求首次过来的时候，clearContext为true，当在执行后续的逻辑的时候重定向到一个新的链接了，此时ServletRequest还是同一个上下文对象，如果在这一次请求当中将安全上下文清理了，重定向之前的请求，如果还要使用安全上下文怎么办。（此处存疑）

继续看doFilterInternal

```java
private void doFilterInternal(ServletRequest request, ServletResponse response,
                              FilterChain chain) throws IOException, ServletException {

    FirewalledRequest fwRequest = firewall
        .getFirewalledRequest((HttpServletRequest) request);
    HttpServletResponse fwResponse = firewall
        .getFirewalledResponse((HttpServletResponse) response);

    List<Filter> filters = getFilters(fwRequest);

    if (filters == null || filters.size() == 0) {
        if (logger.isDebugEnabled()) {
            logger.debug(UrlUtils.buildRequestUrl(fwRequest)
                         + (filters == null ? " has no matching filters"
                            : " has an empty filter list"));
        }

        fwRequest.reset();

        chain.doFilter(fwRequest, fwResponse);

        return;
    }

    VirtualFilterChain vfc = new VirtualFilterChain(fwRequest, chain, filters);
    vfc.doFilter(fwRequest, fwResponse);
}
```

首先是应用对应的防火墙，然后调用getFilters获取对应过滤器链中的所有过滤器，并判断是否为空。这里需要注意的是getFilters()方法，源码如下：

```java
private List<Filter> getFilters(HttpServletRequest request) {
    for (SecurityFilterChain chain : filterChains) {
        if (chain.matches(request)) {
            return chain.getFilters();
        }
    }
    return null;
}
```

实现很简单，无非就是依次迭代所有的过滤器链，并与当前请求进行匹配，如果匹配到了则直接返回，不在进行后续匹配。所以多个过滤器链的顺序就非常重要了。

VirtualFilterChain是位于FilterChainProxy中的一个内部类，它的主要作用就是用于循环调用过滤器链中的过滤器。源码如下：

```java
private static class VirtualFilterChain implements FilterChain {
    private final FilterChain originalChain;
    private final List<Filter> additionalFilters;
    private final FirewalledRequest firewalledRequest;
    private final int size;
    private int currentPosition = 0;

    private VirtualFilterChain(FirewalledRequest firewalledRequest,
                               FilterChain chain, List<Filter> additionalFilters) {
        this.originalChain = chain;
        this.additionalFilters = additionalFilters;
        this.size = additionalFilters.size();
        this.firewalledRequest = firewalledRequest;
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response)
        throws IOException, ServletException {
        if (currentPosition == size) {
            if (logger.isDebugEnabled()) {
                logger.debug(UrlUtils.buildRequestUrl(firewalledRequest)
                             + " reached end of additional filter chain; proceeding with original chain");
            }

            // Deactivate path stripping as we exit the security filter chain
            this.firewalledRequest.reset();

            originalChain.doFilter(request, response);
        }
        else {
            currentPosition++;

            Filter nextFilter = additionalFilters.get(currentPosition - 1);

            if (logger.isDebugEnabled()) {
                logger.debug(UrlUtils.buildRequestUrl(firewalledRequest)
                             + " at position " + currentPosition + " of " + size
                             + " in additional filter chain; firing Filter: '"
                             + nextFilter.getClass().getSimpleName() + "'");
            }

            nextFilter.doFilter(request, response, this);
        }
    }
}
```

其关键在于VirtualFilterChain的doFilter方法，当currentPosition等于size的时候表示已经迭代执行完了所有的过滤器，如果不相等，自然是没有迭代完，那么继续进行迭代。你可能会很奇怪，这里没有for循环或者while等循环，那么它是如何迭代的呢？注意到最后一行代码`nextFilter.doFilter(request, response, this);`，nextFilter自然是表示当前过滤器的下一个过滤器，其第三个参数是将当前引用this，即VirtualFilterChain对象传递给了下一个过滤器，当下一个过滤器执行完毕之后，就会调用`chain.doFilter(request, response);`，也就是又调用到了当前VirtualFilterChain对象doFilter方法，也就完成了其循环。可以看到VirtualFilterChain实现了`javax.servlet.FilterChain`接口。

### 验证管理器

验证管理器`org.springframework.security.authentication.AuthenticationManager`是一个接口，源码如下：

```java
public interface AuthenticationManager {
	Authentication authenticate(Authentication authentication)
			throws AuthenticationException;
}
```

其是一个函数式接口，只有一个抽象方法，其认证工作就是交由它来完成，至于内部会不会再次分发，那就看起具体的实现了。

在Spring Security中，不考虑内部类，只提供了一个`AuthenticationManager`的实现`org.springframework.security.authentication.ProviderManager`。在后续的安全组件构建时，也是构建的这个对象。

还记得在Spring security架构一节说到在认证管理器中包含多个包含多个认证提供器，真正的认证处理是由认证提供器实现。`ProviderManager`是对认证提供器的管理器。

`ProviderManager`的实现也并不复杂，其包含5个主要的成员变量，如下：

```java
public class ProviderManager implements AuthenticationManager, MessageSourceAware,
		InitializingBean {
	......
	private AuthenticationEventPublisher eventPublisher = new NullEventPublisher();
	private List<AuthenticationProvider> providers = Collections.emptyList();
	protected MessageSourceAccessor messages = SpringSecurityMessageSource.getAccessor();
	private AuthenticationManager parent;
	private boolean eraseCredentialsAfterAuthentication = true;
	......        
}
```

- `eventPublisher`：事件发布器，当认证成功或失败时发布对应的事件。
- `providers`：认证提供器，具体的认证工作就是交由它来实现。
- `messages`：？？？
- `parent`：父级认证管理器。
- `eraseCredentialsAfterAuthentication`：控制是否在认证之后擦除凭证，默认true。

主要的实现就是`authenticate`方法，方法稍有点长，忽略其中的日志以及异常等信息的处理，其实现还是很简单的，源码如下：

```java
public Authentication authenticate(Authentication authentication)
    throws AuthenticationException {
    Class<? extends Authentication> toTest = authentication.getClass();
    AuthenticationException lastException = null;
    AuthenticationException parentException = null;
    Authentication result = null;
    Authentication parentResult = null;
    boolean debug = logger.isDebugEnabled();
	//迭代当前认证管理器中的所有认证提供器
    for (AuthenticationProvider provider : getProviders()) {
        //判断当前认证提供器是否支持当前请求
        if (!provider.supports(toTest)) {
            //不支持就直接下一个
            continue;
        }

        if (debug) {
            logger.debug("Authentication attempt using "
                         + provider.getClass().getName());
        }

        try {
            //认证
            result = provider.authenticate(authentication);

            if (result != null) {
                copyDetails(authentication, result);
                break;
            }
        }
        catch (AccountStatusException e) {
            //发布认证失败的异常
            prepareException(e, authentication);
            // SEC-546: Avoid polling additional providers if auth failure is due to
            // invalid account status
            throw e;
        }
        catch (InternalAuthenticationServiceException e) {
            //发布认证失败的异常
            prepareException(e, authentication);
            throw e;
        }
        catch (AuthenticationException e) {
            lastException = e;
        }
    }
	//如果认证结果为null，并且存在父级的认证管理器，那么就去父级认证管理器中查找是否有支持当前请求的
    //认证提供器进行认证。
    if (result == null && parent != null) {
        // Allow the parent to try.
        try {
            //父级认证管理器认证
            result = parentResult = parent.authenticate(authentication);
        }
        catch (ProviderNotFoundException e) {
            // ignore as we will throw below if no other exception occurred prior to
            // calling parent and the parent
            // may throw ProviderNotFound even though a provider in the child already
            // handled the request
        }
        catch (AuthenticationException e) {
            lastException = parentException = e;
        }
    }

    if (result != null) {
        //擦除凭证
        if (eraseCredentialsAfterAuthentication
            && (result instanceof CredentialsContainer)) {
            // Authentication is complete. Remove credentials and other secret data
            // from authentication
            // 认证成功，从authentication中删除凭证和其他秘钥
            ((CredentialsContainer) result).eraseCredentials();
        }

        // If the parent AuthenticationManager was attempted and successful than it will publish an AuthenticationSuccessEvent
        // This check prevents a duplicate AuthenticationSuccessEvent if the parent AuthenticationManager already published it
        //发布认证成功事件
        if (parentResult == null) {
            eventPublisher.publishAuthenticationSuccess(result);
        }
        return result;
    }

    // Parent was null, or didn't authenticate (or throw an exception).

    if (lastException == null) {
        lastException = new ProviderNotFoundException(messages.getMessage(
            "ProviderManager.providerNotFound",
            new Object[] { toTest.getName() },
            "No AuthenticationProvider found for {0}"));
    }

    // If the parent AuthenticationManager was attempted and failed than it will publish an AbstractAuthenticationFailureEvent
    // This check prevents a duplicate AbstractAuthenticationFailureEvent if the parent AuthenticationManager already published it
    //发布认证失败的事件
    if (parentException == null) {
        prepareException(lastException, authentication);
    }

    throw lastException;
}
```

首先依次迭代所有的认证提供器，判断当前迭代的提供器是否支持当前请求的认证，如果不支持，则直接下一个，如果支持，则下一步直接开始认证，对于认证的结果而言，可能认证成功，也可能认证失败，如果认证成功，则返回的是一个`Authentication`对象，如果认证失败，则返回是的`null`。`AuthenticationProvider`是一个接口，其源码如下：

```java
public interface AuthenticationProvider {
   Authentication authenticate(Authentication authentication) throws AuthenticationException;
   boolean supports(Class<?> authentication);
}
```

其中对于`authenticate`方法的返回值是这样描述的：

```
* @return a fully authenticated object including credentials. May return
* <code>null</code> if the <code>AuthenticationProvider</code> is unable to support
* authentication of the passed <code>Authentication</code> object. In such a case,
* the next <code>AuthenticationProvider</code> that supports the presented
* <code>Authentication</code> class will be tried.

@return 安全经过认证的对象包括凭证。如果AuthenticationProvider不支持通过Authentication对象的身份验证，可以返回null。在这种情况下，将尝试下一个支持Authentication类的AuthenticationProvider。
```

所以，`authenticate`方法认证失败返回的结果就是`null`，后续的代码也可以论证这一点。

在认证完成之后，如果认证失败，那么就要尝试在父级的认证管理器上去认证（前提是父级认证管理器存在）。如果认证成功，就需要擦除凭证（这为了防止将凭证带入后续的业务逻辑当中，影响安全。凭证可以认为就是密码），并发布认证成功的事件。如果是认证失败的话（以返回null，或抛出异常形式表现出来），后续就是处理异常，并发布认证失败事件了。

发布事件的时候需要注意的是`eventPublisher`的默认类型是`NullEventPublisher`，这是一个空实现，也就是说在默认情况下，是不会触发事件发布的。还好的是Spring Security提供了一个默认的实现：`org.springframework.security.authentication.DefaultAuthenticationEventPublisher`，其中认证成功发布`AuthenticationSuccessEvent`，而认证失败发布的是`AbstractAuthenticationEvent`子类实例事件，`AbstractAuthenticationEvent`有很多的实现，基本上针对每一种认证失败的情形都有一个相应的实现（认证成功只有一种情况，认证失败却可能是各种各样的）。

我们可以通过配置的方式设置`eventPublisher`，可以是`DefaultAuthenticationEventPublisher`，也可以是我们自定义的扩展，我们可以使用它处理一些业务，比如记录用户登录认证成功或失败的日志。

配置方式如下：

```java
@Configuration
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.authenticationEventPublisher(new DefaultAuthenticationEventPublisher());
        super.configure(auth);
    }
}
```

## 安全组件构建

Spring Security提供了三个核心的安全组件，分别是验证管理器（AuthenticationProvider）、安全过滤器链（SecurityFilterChain）、过滤器链代理（FilterChainProxy），为了能对这三个核心组件进行构建，又提供了`AuthenticationManagerBuilder`、`HttpSecurity`、`WebSecurity`分别对其进行构建。这三个构建器它们都实现了SecurityBuilder接口。

相关模型如下：

![](SecurityBuilder.jpg)

类说明：

- `SecurityBuilder`：用于安全组件的构建。
- `AbstractSecurityBuilder`：
- `AbstractConfiguredSecurityBuilder`：
- `HttpSecurityBuilder`：
- `ProviderManagerBuilder`：
- `WebSecurity`：构建过滤器链代理Filter
- `HttpSecurity`：构建过滤器链
- `AuthenticationManagerBuilder`：构建认证提供管理器





*AbstractSecurityBuilder源码如下：*

```java
public abstract class AbstractSecurityBuilder<O> implements SecurityBuilder<O> {
	private AtomicBoolean building = new AtomicBoolean();

	private O object;

	public final O build() throws Exception {
		if (this.building.compareAndSet(false, true)) {
			this.object = doBuild();
			return this.object;
		}
		throw new AlreadyBuiltException("This object has already been built");
	}

	public final O getObject() {
		if (!this.building.get()) {
			throw new IllegalStateException("This object has not been built");
		}
		return this.object;
	}

	protected abstract O doBuild() throws Exception;
}
```

如上面的代码所示，AbstractSecurityBuilder主要完成两件工作：

1. 通过CAS限制了安全组件有且只能构建一次。
2. 存储构建对象，并设置了`Getter`方法用于获取。

AbstractSecurityBuilder实现了SecurityBuilder，并实现了`build()`方法，在方法中仅仅通过CAS限制了安全组件有且只能构建一次，而真正的构建工作转交给了自身的另一个抽象方法`doBuild()`。

至于`getObject()`方法，是用于获取构建的对象，构建对象还没有被构建完成的时候获取将会报错。构建对象被当前成员变量`object`所引用，其是在`build()`方法中调用`doBuild()`方法完成构建之后关联上的。

### AbstractConfiguredSecurityBuilder

AbstractConfiguredSecurityBuilder是一个很关键的抽象，它是被AuthenticationManagerBuilder、HttpSecurity、WebSecurity所直接继承的抽象类。

在AbstractConfiguredSecurityBuilder抽象类当中，主要以下三点：

1. 安全组件配置类应用。
2. 构建流程的模板定义。
3. 用于辅助增强对象。

AbstractConfiguredSecurityBuilder抽象类定义了六个成员变量，其所有的实现都是围绕这六个成员变量而展开。

AbstractConfiguredSecurityBuilder抽象类成员变量如下：

```java
public abstract class AbstractConfiguredSecurityBuilder<O, B extends SecurityBuilder<O>>
    extends AbstractSecurityBuilder<O> {
    private final Log logger = LogFactory.getLog(getClass());

    private final LinkedHashMap<Class<? extends SecurityConfigurer<O, B>>, List<SecurityConfigurer<O, B>>> configurers = new LinkedHashMap<Class<? extends SecurityConfigurer<O, B>>, List<SecurityConfigurer<O, B>>>();
    private final List<SecurityConfigurer<O, B>> configurersAddedInInitializing = new ArrayList<SecurityConfigurer<O, B>>();

    private final Map<Class<? extends Object>, Object> sharedObjects = new HashMap<Class<? extends Object>, Object>();

    private final boolean allowConfigurersOfSameType;

    private BuildState buildState = BuildState.UNBUILT;

    private ObjectPostProcessor<Object> objectPostProcessor;
    
    ......
}
```

- `configurers`：用于存储configurer类的class对象与对应configurer类实例的映射关系。configurer类实例可以有多个。
- `configurersAddedInInitializing`：用于存储在构建初始化阶段添加的新的configurer类实例。
- `sharedObjects`：用户构建相关的全局共享对象存储，即存储在sharedObjects中的数据，可以在相关的任何地方获取。
- `allowConfigurersOfSameType`：是否允许相同类型的配置，这里的相同类型的配置是指configurers属性，class对象与configurer类实例的映射是，是否可以有多个相同的configurer类实例。默认为false。
- `buildState`：构建状态。
- `objectPostProcessor`：

了解了属性，现在来分析一下相关代码，AbstractConfiguredSecurityBuilder中最核心的有两个方法，其他的大部分都是类似与Getter/Setter这样的方法。

1、添加配置：add()

```java
private <C extends SecurityConfigurer<O, B>> void add(C configurer) throws Exception {
    Assert.notNull(configurer, "configurer cannot be null");

    Class<? extends SecurityConfigurer<O, B>> clazz = (Class<? extends SecurityConfigurer<O, B>>) configurer
        .getClass();
    synchronized (configurers) {
        if (buildState.isConfigured()) {
            throw new IllegalStateException("Cannot apply " + configurer
                                            + " to already built object");
        }
        List<SecurityConfigurer<O, B>> configs = allowConfigurersOfSameType ? this.configurers
            .get(clazz) : null;
        if (configs == null) {
            configs = new ArrayList<SecurityConfigurer<O, B>>(1);
        }
        configs.add(configurer);
        this.configurers.put(clazz, configs);
        if (buildState.isInitializing()) {
            this.configurersAddedInInitializing.add(configurer);
        }
    }
}
```

如上代码所示，add()方法被apply()方法所调用，而apply()方法并没有太复杂的实现。

`add()`初次看可能不太理解，现在来依次分析：

1. 获取configurer的class对象。
2. 加锁，防止因为并发的原因出现在构建的配置阶段之后添加的新的configurer配置。
3. 根据`allowConfigurersOfSameType`属性确定是否允许出现相同类型的配置。如果允许，则直接从configurers中获取之前已经添加过的configurer配置的集合，否则，直接返回null。
4. 根据第三部返回结果是否是null，而创建一个新的LIst集合。这里有两重语义：1.如果在第三步是允许出现相同类型的配置，那当前这个configurers可能是第一次添加，也就说，这个时候，configs是等于null的，所以这里创建一个新的LIst集合相当于是为当前configurer首次初始化。2.如果在第三步不允许出现相同类型的配置，那么configs必定等于null，也就是说，当前所添加的configurer无论是否已经在configurers中已经存在，都会为其创建一个新的List集合用于存放。故而也就相当于一个新的配置，如此也就不重复了。
5. 将配置添加到configurers当中存放。
6. 如果添加当前的configurer的时候是构建的初始化阶段，则将当前configurer添加到configurersAddedInInitializing集合中，以便在configurers初始化完成之后，再次初始化构建初始化阶段添加的configurer，即configurersAddedInInitializing集合中的configurer。

关于第六步的相关代码如下：

```java
private void init() throws Exception {
    Collection<SecurityConfigurer<O, B>> configurers = getConfigurers();

    for (SecurityConfigurer<O, B> configurer : configurers) {
        configurer.init((B) this);
    }

    for (SecurityConfigurer<O, B> configurer : configurersAddedInInitializing) {
        configurer.init((B) this);
    }
}

private Collection<SecurityConfigurer<O, B>> getConfigurers() {
    List<SecurityConfigurer<O, B>> result = new ArrayList<SecurityConfigurer<O, B>>();
    for (List<SecurityConfigurer<O, B>> configs : this.configurers.values()) {
        result.addAll(configs);
    }
    return result;
}
```

如上代码所示，`init()`方法首先会调用`getConfigurers()`获取所有的`SecurityConfigurer`，在`getConfigurers()`方法中可以看到，这里所有的`SecurityConfigurer`仅仅指`configurers`当中的`SecurityConfigurer`，并且它返回的是一个新的`List`集合。所以说`init()`方法迭代初始化的时候，仍然也可以在此时调用`apply()`方法添加新的`SecurityConfigurer`（`configurer#init()`中添加`configurer`），但是，这里迭代的是一个新的List集合，所以即使在此时调用`apply()`方法添加的新的`SecurityConfigurer`仍会添加到`configurers`中，但是却不会被初始化了，所以需要一个新的集合来存储补偿，这个集合即是`configurersAddedInInitializing`。

二、构建安全组件：doBuild()

doBuild()方法是实现于AbstractSecurityBuilder的抽象方法，在之前已经讲过，AbstractSecurityBuilder的build()方法仅仅使用CAS控制了不允许重复构建，真正构建工作交由了其所属的抽象方法doBuild()去完成。实际上AbstractConfiguredSecurityBuilder实现了doBuild()方法也没有去完成真正的构建，因为AbstractConfiguredSecurityBuilder也是一个抽象类，而真正的构建肯定交由具体的实现。在doBuild()方法主要定义了构建的流程，其具体的构建又再次交由了AbstractConfiguredSecurityBuilder的performBuild()抽象方法去完成。这是典型模板方法模式的应用。

*doBuild()相关源码如下：*

```java
@Override
protected final O doBuild() throws Exception {
    synchronized (configurers) {
        buildState = BuildState.INITIALIZING;

        beforeInit();
        init();

        buildState = BuildState.CONFIGURING;

        beforeConfigure();
        configure();

        buildState = BuildState.BUILDING;

        O result = performBuild();

        buildState = BuildState.BUILT;

        return result;
    }
}
```

如上代所示，构建流程分别三个步骤：

1. 初始化：beforeInit()是一个空方法，用于子类扩展，可以实现自己的初始化操作。在Spring Security中没有任何类重写扩展。init()方法是真正的初始化方法，其是转交由`SecurityConfigurer`的init()方法初始化的。
2. 配置：beforeConfigure()是一个空方法，用于子类扩展，可以实现自己的初始化操作。在Spring Security中只被HttpSecurity类重写扩展。仅仅将AuthenticationManager添加到`sharedObjects`共享对象中存放。configure()方法是真正的配置方法，其是转交由`SecurityConfigurer`的configure()方法配置的。
3. 构建：performBuild()方法是抽象的方法，由子类实现。

在每一步流程开始之前，都会更新为对应的构建状态。



### AuthenticationManagerBuilder



### HttpSecurity



### WebSecurity







- AuthenticationManager：身份验证管理器

- SecurityFilterChain：安全过滤器链

- FilterChainProxy：过滤器链代理

  









需要注意的是`HttpSecurity`的核心目的仅仅是构建过滤器链，而过滤器链中的Filter需要通过其他方式组装。那么Spring Security是如何去创建Filter的呢？

## 安全组件配置

在Spring Security中SecurityBuilder仅仅完成了身份验证管理器、安全过滤器链、过滤器链代理的构建，但实际上，它们都还依赖很多其他外部的配置以控制器行为，其中最为主要的就是安全过滤器链中的过滤器。

Spring Security是通过定义一个安全组件构建器的配置器，通过将安全组件构建器传入到配置器，然后再通过配置器对其进行配置，

SecurityConfigurer（安全组件配置器）就是用于对SecurityBuilder（安全组件构建器）进行配置的，它定义了配置SecurityBuilder的通用行为，可以配置安全过滤器链中的过滤器、身份验证管理器中的验证管理器等等。

SecurityConfigurer是一个接口，源码如下：

```java
public interface SecurityConfigurer<O, B extends SecurityBuilder<O>> {
    void init(B builder) throws Exception;
    void configure(B builder) throws Exception;
}
```

上如代码所示，SecurityConfigurer定义个两个抽象方法，init()方法用于初始化SecurityBuilder，configure()用于配置SecurityBuilder。

>  注意SecurityConfigurer中的泛型，O和B，其中B必须是SecurityBuilder子类型，而0是SecurityBuilder构建的类型。
>
> 关于SecurityBuilder，还记得前面所讲的安全组件构建吗，`WebSecurity`、`HttpSecurity`以及`AuthenticationManagerBuilder`都是SecurityBuilder的子类型，所以其实就可以理解为SecurityConfigurer就是用于配置`WebSecurity`、`HttpSecurity`和`AuthenticationManagerBuilder`。

SecurityConfigurer还是一个比较非常抽象的类型，虽然它限制了仅仅用于配置SecurityBuilder。所以它应该还有更加具体的抽象类型对象去定义公共的行为以及更具体的构建对象。

SecurityConfigurer类结构体系如下：

```shell
- SecurityConfigurer
  - SecurityConfigurerAdapter
    - AbstractHttpConfigurer
      - AbstractInterceptUrlConfigurer
      - AbstractAuthenticationFilterConfigurer
    - UserDetailsAwareConfigurer
      - AbstractDaoAuthenticationConfigurer
  - WebSecurityConfigurer
    - WebSecurityConfigurerAdapter
  - GlobalAuthenticationConfigurerAdapter
```

- SecurityConfigurerAdapter：安全组件配置的适配器，定义了所有安全配置通用操作。
  - AbstractHttpConfigurer：安全过滤器链配置
  - UserDetailsAwareConfigurer：身份验证管理器配置
- WebSecurityConfigurer：过滤器链代理配置
- GlobalAuthenticationConfigurerAdapter：

SecurityConfigurerAdapter是安全组件配置的适配器，定义了所有安全配置的通用操作。

*SecurityConfigurerAdapter源码如下：*

```java
public abstract class SecurityConfigurerAdapter<O, B extends SecurityBuilder<O>>
    implements SecurityConfigurer<O, B> {
    private B securityBuilder;

    private CompositeObjectPostProcessor objectPostProcessor = new CompositeObjectPostProcessor();

    public void init(B builder) throws Exception {
    }

    public void configure(B builder) throws Exception {
    }

    public B and() {
        return getBuilder();
    }

    protected final B getBuilder() {
        if (securityBuilder == null) {
            throw new IllegalStateException("securityBuilder cannot be null");
        }
        return securityBuilder;
    }

    @SuppressWarnings("unchecked")
    protected <T> T postProcess(T object) {
        return (T) this.objectPostProcessor.postProcess(object);
    }

    public void addObjectPostProcessor(ObjectPostProcessor<?> objectPostProcessor) {
        this.objectPostProcessor.addObjectPostProcessor(objectPostProcessor);
    }

    public void setBuilder(B builder) {
        this.securityBuilder = builder;
    }

    private static final class CompositeObjectPostProcessor implements
        ObjectPostProcessor<Object> {
        private List<ObjectPostProcessor<? extends Object>> postProcessors = new ArrayList<ObjectPostProcessor<?>>();

        @SuppressWarnings({ "rawtypes", "unchecked" })
        public Object postProcess(Object object) {
            for (ObjectPostProcessor opp : postProcessors) {
                Class<?> oppClass = opp.getClass();
                Class<?> oppType = GenericTypeResolver.resolveTypeArgument(oppClass,
                                                                           ObjectPostProcessor.class);
                if (oppType == null || oppType.isAssignableFrom(object.getClass())) {
                    object = opp.postProcess(object);
                }
            }
            return object;
        }

        private boolean addObjectPostProcessor(
            ObjectPostProcessor<? extends Object> objectPostProcessor) {
            boolean result = this.postProcessors.add(objectPostProcessor);
            Collections.sort(postProcessors, AnnotationAwareOrderComparator.INSTANCE);
            return result;
        }
    }
}
```

如上代码所示，SecurityConfigurerAdapter主要完成了两件工作：

1. 存储SecurityBuilder对象。

   SecurityConfigurerAdapter定了一个securityBuilder属性，它的类型是B，其实就是SecurityBuilder类型。并且定义了securityBuilder的Getter和Setter方法，看到这里就已经印证了我们前面所说过的：定义一个配置对象，然后将安全组件构建器传入到配置对象中，然后就可以控制安全组件构建器的行为，对安全组件进行配置了。所以，安全组件配置就是通过这儿的`setBuilder()`方法将SecurityBuilder传入的。

2. 初始化对象。

   另一个属性是objectPostProcessor，它是SecurityConfigurerAdapter中的内部类CompositeObjectPostProcessor，CompositeObjectPostProcessor是组合类，组合了对ObjectPostProcessor的操作。

   ????????????????????????????

   *ObjectPostProcessor源码如下：*

   ```java
   public interface ObjectPostProcessor<T> {
   	<O extends T> O postProcess(O object);
   }
   ```

   

### AbstractHttpConfigurer

AbstractHttpConfigurer是用于对安全过滤器链构建器进行配置，它所继承的类`SecurityConfigurerAdapter<DefaultSecurityFilterChain, B>`，已经在泛型中声明了这一点。

其配置对应的构建对象是DefaultSecurityFilterChain，总所周知，在Spring Security中只有HttpSecurity是对其进行构建的，所以这里就可以认为AbstractHttpConfigurer就是用于对HttpSecurity进行配置的。虽然如此，但是AbstractHttpConfigurer并没有限制其配置的构建器，仍然还是用泛型B表示构建器对象，所以这里还是一个仅仅针对安全过滤器链构建器配置抽象配置对象，留给了我们可扩展的空间。因此，我们完全可以定义自定义的安全过滤器链构建器。

*AbstractHttpConfigurer源码如下：*

```java
public abstract class AbstractHttpConfigurer<T extends AbstractHttpConfigurer<T, B>, B extends HttpSecurityBuilder<B>>
		extends SecurityConfigurerAdapter<DefaultSecurityFilterChain, B> {

	@SuppressWarnings("unchecked")
	public B disable() {
		getBuilder().removeConfigurer(getClass());
		return getBuilder();
	}

	@SuppressWarnings("unchecked")
	public T withObjectPostProcessor(ObjectPostProcessor<?> objectPostProcessor) {
		addObjectPostProcessor(objectPostProcessor);
		return (T) this;
	}
}
```

在AbstractHttpConfigurer中有两个方法：

1. `disable`：从语义上是禁用某一个配置，实质就是删除不需要的配置。
2. `withObjectPostProcessor`：用于添加ObjectPostProcessor。



### UserDetailsAwareConfigurer









### WebSecurityConfigurer

WebSecurityConfigurer是用于对过滤器链代理构建器进行配置的，它与AbstractHttpConfigurer模式是一样的，限定了构建对象是Filter，即在Spring Security中对应的构建器就是`WebSecurity`。

*WebSecurityConfigurer源码如下：*

```java
public interface WebSecurityConfigurer<T extends SecurityBuilder<Filter>> extends
		SecurityConfigurer<Filter, T> {
}
```

WebSecurityConfigurer有些特别，在说明它的特别之处之前，先回顾一下安全组件构建器。

Spring Security提供了三个核心的安全组件，分别是身份验证管理器、安全过滤器链、过滤器链代理，它们分别由`AuthenticationManagerBuilder`、`HttpSecurity`、`WebSecurity`构建，各自独立。但是过滤器链代理包含安全过滤器链，安全过滤器链包含身份验证管理器，它们之间是层层递进的包含关系，也就是说，必须先完成身份验证管理器的构建，再完成安全过滤器链的构建，最后完成过滤器链代理的构建，如此才算完事儿。

那么WebSecurityConfigurer特别在哪儿呢？有两个点：

1. 在通过WebSecurityConfigurer对过滤器链代理进行配合的时候，必须先完成身份验证管理器、安全过滤器链的配置。
2. Spring Security提供了WebSecurityConfigurerAdapter类，它实现了WebSecurityConfigurer，提供很多默认的安全配置。

如果你对Spring Security比较熟悉的话，你就会明白为什么第二点比较特别。在项目中集成Spring Security的时候，通常其默认的安全配置是不能满足要求的，都需要在其默认的安全配置的基础上订制个性化的安全策略，而这个订制的方式就是创建一个配置类，并继承WebSecurityConfigurerAdapter，然后重写其中的configure方法。

*配置示例：*

```java
@Order(0)
@Configuration
public class DemoSecurityConfigurer extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        super.configure(auth);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            .anyRequest().authenticated()
            .and()
            .httpBasic();
    }

    @Override
    public void configure(WebSecurity web) throws Exception {
        super.configure(web);
    }
}
```

下面单独花一小节来说明WebSecurityConfigurerAdapter默认提供了那些安全配置。

### WebSecurityConfigurerAdapter





## 内置安全过滤器

### `SecurityContextPersistenceFilter`

`org.springframework.security.web.context.SecurityContextPersistenceFilter`是用于安全上下文`org.springframework.security.core.context.SecurityContext`持久化的过滤器。`SecurityContext`是一个接口，其主要的作用就是用于存储认证的安全信息，而认证的安全信息是由`org.springframework.security.core.Authentication`对象表示，接口如下：

```java
public interface SecurityContext extends Serializable {
	Authentication getAuthentication();
	void setAuthentication(Authentication authentication);
}
```

可以看到，仅仅提供了`Authentication`的Getter和Setter方法。在Spring security中只提供了一个默认的实现`org.springframework.security.core.context.SecurityContextImpl`，排除其中的非关键代码，源码如下：

```java
public class SecurityContextImpl implements SecurityContext {
	......
    private Authentication authentication;
    ......
	@Override
	public Authentication getAuthentication() {
		return authentication;
	}
	......
	@Override
	public void setAuthentication(Authentication authentication) {
		this.authentication = authentication;
	}
	......
}
```



`SecurityContextPersistenceFilter`源码如下：

```java
public class SecurityContextPersistenceFilter extends GenericFilterBean {
	static final String FILTER_APPLIED = "__spring_security_scpf_applied";
	private SecurityContextRepository repo;
	private boolean forceEagerSessionCreation = false;

	public SecurityContextPersistenceFilter() {
		this(new HttpSessionSecurityContextRepository());
	}

	public SecurityContextPersistenceFilter(SecurityContextRepository repo) {
		this.repo = repo;
	}

	public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
			throws IOException, ServletException {
		HttpServletRequest request = (HttpServletRequest) req;
		HttpServletResponse response = (HttpServletResponse) res;
        //确保每个请求只应用一次当前filter
		if (request.getAttribute(FILTER_APPLIED) != null) {
			chain.doFilter(request, response);
			return;
		}
		final boolean debug = logger.isDebugEnabled();
		request.setAttribute(FILTER_APPLIED, Boolean.TRUE);
		//强制即时会话创建
		if (forceEagerSessionCreation) {
			HttpSession session = request.getSession();
			if (debug && session.isNew()) {
				logger.debug("Eagerly created session: " + session.getId());
			}
		}
		//创建HttpRequest和HttpResponse持有器对象
		HttpRequestResponseHolder holder = new HttpRequestResponseHolder(request,
				response);
        //加载当前安全上下文，默认情况下是从当前会话（HttpSession）中加载
		SecurityContext contextBeforeChainExecution = repo.loadContext(holder);
		try {
            //存储当前安全上下文，以便于后续通过静态方法获取。默认情况下存储在ThreadLocal中
			SecurityContextHolder.setContext(contextBeforeChainExecution);
			chain.doFilter(holder.getRequest(), holder.getResponse());
		}
		finally {
            //从SecurityContextHolder中获取当前安全上下文
			SecurityContext contextAfterChainExecution = SecurityContextHolder
					.getContext();
			// Crucial removal of SecurityContextHolder contents - do this before anything
			// else.
            //当前请求已经完成，清理掉SecurityContextHolder中的上下文
			SecurityContextHolder.clearContext();
            //保存安全上文到会话中
			repo.saveContext(contextAfterChainExecution, holder.getRequest(),
					holder.getResponse());
			request.removeAttribute(FILTER_APPLIED);

			if (debug) {
				logger.debug("SecurityContextHolder now cleared, as request processing completed");
			}
		}
	}
	......
}
```

如上代码所示，首先是一个当前Filter是否已被应用的判定，之所以做这个判定，是因为？？？。然后强制即时会话创建？？。下一步就是从`repo`加载当前安全上下文，并将其存储在`SecurityContextHolder`中，`SecurityContextHolder`提供了用于获取当前安全上下文的静态方法，如此后续业务代码需要使用的时候，就可以很方便的获取到`SecurityContext`。`repo`的类型是`SecurityContextRepository`，从名字就可以看出，它是安全上下文的存储仓库。默认情况下`SecurityContext`存储在当前会话（`HttpSession`）中，可以看到在`SecurityContextPersistenceFilter`的构造器中，提供了默认实现`HttpSessionSecurityContextRepository`实例。

`SecurityContextRepository`是一个接口，其源码如下：

```java
public interface SecurityContextRepository {
	SecurityContext loadContext(HttpRequestResponseHolder requestResponseHolder);
	void saveContext(SecurityContext context, HttpServletRequest request,
			HttpServletResponse response);
	boolean containsContext(HttpServletRequest request);
}
```

如上代码所示，`SecurityContextRepository`提供了三个抽象方法：

- `loadContext`用于获取当前安全上下文。
- `saveContext`用于存储当前安全上下文。
- `containsContext`用于判断档期安全上下文仓库中是否包含当前安全上下文。

`HttpSessionSecurityContextRepository#loadContext`

```java
public SecurityContext loadContext(HttpRequestResponseHolder requestResponseHolder) {
    HttpServletRequest request = requestResponseHolder.getRequest();
    HttpServletResponse response = requestResponseHolder.getResponse();
    HttpSession httpSession = request.getSession(false);
	//从HttpSession会话中获取当前安全上下文
    SecurityContext context = readSecurityContextFromSession(httpSession);
	
    if (context == null) {
        if (logger.isDebugEnabled()) {
            logger.debug("No SecurityContext was available from the HttpSession: "
                         + httpSession + ". " + "A new one will be created.");
        }
        //没有获取到创建一个新的安全上下文
        context = generateNewContext();

    }

    SaveToSessionResponseWrapper wrappedResponse = new SaveToSessionResponseWrapper(
        response, request, httpSession != null, context);
    requestResponseHolder.setResponse(wrappedResponse);

    if (isServlet3) {
        requestResponseHolder.setRequest(new Servlet3SaveToSessionRequestWrapper(
            request, wrappedResponse));
    }

    return context;
}
```





`SecurityContextHolder`

`SecurityContextHolder`是`SecurityContex`持有器对象，其提供了用于设置，获取，清理、创建`SecurityContext`的静态方法，其源码如下：

```java
public class SecurityContextHolder {
	public static final String MODE_THREADLOCAL = "MODE_THREADLOCAL";
	public static final String MODE_INHERITABLETHREADLOCAL = "MODE_INHERITABLETHREADLOCAL";
	public static final String MODE_GLOBAL = "MODE_GLOBAL";
	public static final String SYSTEM_PROPERTY = "spring.security.strategy";
	private static String strategyName = System.getProperty(SYSTEM_PROPERTY);
	private static SecurityContextHolderStrategy strategy;
	private static int initializeCount = 0;

	static {
		initialize();
	}

    //清理安全上下问
	public static void clearContext() {
		strategy.clearContext();
	}
    //获取安全上下文
	public static SecurityContext getContext() {
		return strategy.getContext();
	}
	//获取安全上下文持有者存储策略的初始化次数
	public static int getInitializeCount() {
		return initializeCount;
	}
	private static void initialize() {
        //如果没有设置自定义的策略，则设置默认策略为ThreadLocalSecurityContextHolderStrategy
		if (!StringUtils.hasText(strategyName)) {
			strategyName = MODE_THREADLOCAL;
		}
		if (strategyName.equals(MODE_THREADLOCAL)) {
			strategy = new ThreadLocalSecurityContextHolderStrategy();
		}
		else if (strategyName.equals(MODE_INHERITABLETHREADLOCAL)) {
			strategy = new InheritableThreadLocalSecurityContextHolderStrategy();
		}
		else if (strategyName.equals(MODE_GLOBAL)) {
			strategy = new GlobalSecurityContextHolderStrategy();
		}
		else {
			// 尝试加载自定义策略
			try {
				Class<?> clazz = Class.forName(strategyName);
				Constructor<?> customStrategy = clazz.getConstructor();
				strategy = (SecurityContextHolderStrategy) customStrategy.newInstance();
			}
			catch (Exception ex) {
				ReflectionUtils.handleReflectionException(ex);
			}
		}
		initializeCount++;
	}
	//清理安全上下文
	public static void setContext(SecurityContext context) {
		strategy.setContext(context);
	}
    //设置自定义策略
	public static void setStrategyName(String strategyName) {
		SecurityContextHolder.strategyName = strategyName;
		initialize();
	}
	//获取当前应用的策略
	public static SecurityContextHolderStrategy getContextHolderStrategy() {
		return strategy;
	}
	//创建一个空的安全上下文
	public static SecurityContext createEmptyContext() {
		return strategy.createEmptyContext();
	}
	......
}
```

本质上，存储`SecurityContex`由不同的策略（`SecurityContextHolderStrategy`）实现，如上代码所示，在`SecurityContextHolder`中默认提供了三种策略：

1. `ThreadLocalSecurityContextHolderStrategy`：使用ThreadLocal存储安全上下文。
2. `InheritableThreadLocalSecurityContextHolderStrategy`：使用`InheritableThreadLocal`存储安全上下文
3. `GlobalSecurityContextHolderStrategy`：使用静态字段存储安全上下文件。

`SecurityContextHolder`中提供了静态代码块初始化存储策略，默认情况下使用的是`ThreadLocalSecurityContextHolderStrategy`策略。我们可以通过参数`spring.security.strategy`实现自定义的策略。

不同的策略实现实现了`SecurityContextHolderStrategy`接口，其源码如下：

```java
public interface SecurityContextHolderStrategy {
	void clearContext();
	SecurityContext getContext();
	void setContext(SecurityContext context);
	SecurityContext createEmptyContext();
}
```

其提供了四个方法，分别对应着`SecurityContextHolder`中设置，获取，清理、创建`SecurityContext`的静态方法。

SecurityContextHolderStrategy的三个策略

`ThreadLocalSecurityContextHolderStrategy`、`InheritableThreadLocalSecurityContextHolderStrategy`和`GlobalSecurityContextHolderStrategy`的实现方式都是一样的，其唯一的不同点在于其用于存储`SecurityContext`方式不一样。前面已经描述了其存储的方式。关键源码如下：

```java
final class ThreadLocalSecurityContextHolderStrategy implements
		SecurityContextHolderStrategy {
	private static final ThreadLocal<SecurityContext> contextHolder = new ThreadLocal<>();
    ......
}

final class InheritableThreadLocalSecurityContextHolderStrategy implements
		SecurityContextHolderStrategy {
	private static final ThreadLocal<SecurityContext> contextHolder = new InheritableThreadLocal<>();
	......
}

final class GlobalSecurityContextHolderStrategy implements SecurityContextHolderStrategy {
	private static SecurityContext contextHolder;
	......
}
```

这三种方式各有优劣，对于使用`ThreadLocal`存储，好处在于每个请求线程都是各自独立的，互不影响，缺点在于只能被当前请求的主线程获取，如果有子线程的话，就自能通过传参的方式设置进去了。对于`InheritableThreadLocal`？？？，

对于第三种静态字段的方式，它的最大好处在于全局可用，无论在哪儿，无论是主线程还是子线程。如果你对JVM的内存结构有所了解的话，那你就应该知道，对象虽然是存在堆内存当中的，但是静态字段是存储在常量池中的，当静态字段对堆内存中的对象产生引用时，意味着此对象永远不会被GC。虽然这个特性看起来很好，但是其有极大的局限性，如果我们的系统有两个及以上的用户，那么它们产生的`SecurityContext`必然不同，这个时候就没有办法同时存储了，只能存储一个。如果此时A用户的请求还没有完成，B用户的请求过来了，就会将A用户的`SecurityContext`替换，与此同时A用户需要使用自己的`SecurityContext`，它就会发现获取的`SecurityContext`不是自己的。所以，如果你的系统有且只有一个用户，那么就可以使用这种策略，即使如此，也强烈建议不要使用这种策略。

上述的三种Spring security提供的策略只能针对单体应用，对于集群应用就不适用了。此时我们就可以实现自己的存储策略，比如将`SecurityContext`存储在分布式缓存当中。

### `HeaderWriterFilter`

此过滤器实现了添加headers到当前响应中。可以添加某些有用的响应头，以便启用浏览器保护。例如`X-Frame-Options`，`X-XSS-Protection*`或`X-Content-Type-Options`。

上面一段话是其相关源码的注释说明。换一个更通俗的说法，就是在HttpServletResponse向客户端响应之前添加响应头。仅此而已，只是这个是Spring安全框架的是过滤器，所以一般添加的响应头都是用于安全保护相关的。

`HeaderWriterFilter`的实现并不复杂，只是有几点需要注意

1. `HeaderWriterFilter`实现的`OncePerRequestFilter`抽象类，也就是说它在一次请求的生命周期中，只会被执行一次。很正常，既然是向客户端写响应头，写一次够了，没有必要重复写。

2. 在`HeaderWriterFilter`中真正执行写响应头的逻辑操作是有具体的HttpServletResponseWrapper中完成，在`HeaderWriterFilter`中定义为内部类。

   相关源码如下：

   ```java
   @Override
   protected void doFilterInternal(HttpServletRequest request,
                                   HttpServletResponse response, FilterChain filterChain)
       throws ServletException, IOException {
       //创建HeaderWriterResponse Wrapper(真正的响应头在这里面实现)
       HeaderWriterResponse headerWriterResponse = new HeaderWriterResponse(
           request,response, this.headerWriters);
       //创建HeaderWriterRequest Wrapper
       HeaderWriterRequest headerWriterRequest = new HeaderWriterRequest(
           request,headerWriterResponse);
       try {
           filterChain.doFilter(headerWriterRequest, headerWriterResponse);
       }
       finally {
           //finally块中执行响应头的写操作(这里必定会执行，但不一定是有效的)
           headerWriterResponse.writeHeaders();
       }
   }
   
   static class HeaderWriterResponse extends OnCommittedResponseWrapper {
       private final HttpServletRequest request;
       private final List<HeaderWriter> headerWriters;
   
       HeaderWriterResponse(HttpServletRequest request, HttpServletResponse response,
                            List<HeaderWriter> headerWriters) {
           super(response);
           this.request = request;
           this.headerWriters = headerWriters;
       }
   
       @Override
       protected void onResponseCommitted() {
           writeHeaders();
           this.disableOnResponseCommitted();
       }
   
       protected void writeHeaders() {
           if (isDisableOnResponseCommitted()) {
               return;
           }
           for (HeaderWriter headerWriter : this.headerWriters) {
               headerWriter.writeHeaders(this.request, getHttpResponse());
           }
       }
   
       private HttpServletResponse getHttpResponse() {
           return (HttpServletResponse) getResponse();
       }
   }
   
   static class HeaderWriterRequest extends HttpServletRequestWrapper {
       private final HeaderWriterResponse response;
   
       HeaderWriterRequest(HttpServletRequest request, HeaderWriterResponse response) {
           super(request);
           this.response = response;
       }
   
       @Override
       public RequestDispatcher getRequestDispatcher(String path) {
           return new HeaderWriterRequestDispatcher(super.getRequestDispatcher(path), this.response);
       }
   }
   
   static class HeaderWriterRequestDispatcher implements RequestDispatcher {
       private final RequestDispatcher delegate;
       private final HeaderWriterResponse response;
   
       HeaderWriterRequestDispatcher(RequestDispatcher delegate, HeaderWriterResponse response) {
           this.delegate = delegate;
           this.response = response;
       }
   
       @Override
       public void forward(ServletRequest request, ServletResponse response) throws ServletException, IOException {
           this.delegate.forward(request, response);
       }
   
       @Override
       public void include(ServletRequest request, ServletResponse response) throws ServletException, IOException {
           this.response.onResponseCommitted();
           this.delegate.include(request, response);
       }
   }
   ```

   上述代码初始看起来可能有些困惑，如果不是对Servlet所有了解的话，可能不太明白，为什么不将需要写的响应头直接放在finally块中写，而要通过Servlet Wrapper包装一次。这里是因为需要保证HttpServletResponse在向客户端响应之前，必须把响应头设置到ServletResponse中，否则将不会生效。说到这里你可能会理解它是通过Servlet Wrapper来保证这一点的，没错，事实就是如此。但是Servlet Wrapper本身并不能保证这一点，Spring security是通过OnCommittedResponseWrapper来完成这一点。你应该已经注意到了`HeaderWriterResponse`继承了它。

   OnCommittedResponseWrapper

   OnCommittedResponseWrapper是一个比较通用的抽象类，它的主要作用是包保证在HttpServletResponse在向客户端响应之前，执行必要的操作。源码如下：

   ```java
   
   ```

   









### HttpRequestWrapper

### Spring过滤器









现在我们已经知道了安全组件，并且知道了安全组件如何构建以及如何配置，也就是真正实现验证逻辑的代码。

还记得在分析安全组件过滤器连构建是的过滤器吗

```java
private FilterComparator comparator = new FilterComparator();
```



### BasicAuthenticationFilter

 

### SecurityContextPersistenceFilter







```java
org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter
org.springframework.security.web.context.SecurityContextPersistenceFilter
org.springframework.security.web.header.HeaderWriterFilter
org.springframework.security.web.authentication.logout.LogoutFilter
org.springframework.security.oauth2.provider.authentication.OAuth2AuthenticationProcessingFilter
org.springframework.security.web.savedrequest.RequestCacheAwareFilter
org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter
org.springframework.security.web.authentication.AnonymousAuthenticationFilter
org.springframework.security.web.session.SessionManagementFilter
org.springframework.security.web.access.ExceptionTranslationFilter
org.springframework.security.web.access.intercept.FilterSecurityInterceptor

org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter
```



AuthenticationManager  ProviderManager   AuthenticationProvider











- HttpSecurity：过滤器链构建，组成元素Filter
- WebSecurity：安全过滤器构建，组成元素SecurityFilterChain
- AuthenticationManagerBuilder：ProviderManager构建，组成元素AuthenticationProvider



AuthenticationManagerBuilder仅仅提供了ProviderManager的构建，而在ProviderManager中是包含了多个AuthenticationProvider，真正的验证工作是由`AuthenticationProvider`去完成，而ProviderManager仅仅是对其提供的管理工作．实际上，Spring Security也只提供了`ProviderManager`管理器（内部类除外）．我们完全可以创建自己的AuthenticationManager，同时创建与`AuthenticationProvider`等价的验证器，完成验证．

> 为什么是说＂同时创建与`AuthenticationProvider`等价的验证器＂，而不是通过实现`AuthenticationProvider`接口呢，是因为`AuthenticationProvider`接口提供的验证方法入参是`Authentication`，而有时我们用不到这个强大的功能，可能我们需要验证的数据就是一个简简单单的字符串，所以完全可以以相同的模式提供自定义的实现。Spring Security Oauth2就是如此：
>
> ```
> OAuth2AuthenticationManager -> ResourceServerTokenServices
> ```
>
> 







- AbstractConfiguredSecurityBuilder
  - ObjectPostProcessor
  - SecurityConfigurer
  - SecurityConfigurerAdapter







```java
org.springframework.security.access.intercept.AbstractSecurityInterceptor
org.springframework.security.web.access.intercept.FilterSecurityInterceptor
org.springframework.security.access.intercept.aopalliance.MethodSecurityInterceptor
org.springframework.security.access.intercept.aspectj.AspectJMethodSecurityInterceptor


org.springframework.security.access.SecurityMetadataSource

org.springframework.security.web.access.intercept.FilterInvocationSecurityMetadataSource

org.springframework.security.web.access.intercept.DefaultFilterInvocationSecurityMetadataSource



org.springframework.security.access.AccessDecisionManager

org.springframework.security.access.vote.AbstractAccessDecisionManager

org.springframework.security.access.vote.AffirmativeBased



org.springframework.security.access.AccessDecisionVoter

org.springframework.security.access.vote.RoleVoter



org.springframework.security.access.ConfigAttribute
org.springframework.security.web.access.expression.WebExpressionConfigAttribute


org.springframework.security.access.intercept.InterceptorStatusToken
```