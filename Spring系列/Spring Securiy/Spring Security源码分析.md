# Spring Security源码分析

## 前言



## 模块介绍



## 环境准备



## 原理介绍



## SecurityBuilder

### HttpSecurity

### WebSecurity

### AuthenticationManagerBuilder

#### AuthenticationManager
##### ProviderManager
#### AuthenticationProvider



## WebSecurityConfigurer

### GlobalAuthenticationConfigurerAdapter

### SecurityConfigurer

#### AbstractHttpConfigurer
#### UserDetailsAwareConfigurer
#### LdapAuthenticationProviderConfigurer

### SecurityConfigurerAdapter

## Filter

### FilterSecurityInterceptor

org.springframework.security.web.access.intercept.FilterSecurityInterceptor

#### AccessDecisionManager

org.springframework.security.access.AccessDecisionManager



org.springframework.security.access.vote.AffirmativeBased

org.springframework.security.access.vote.UnanimousBased

org.springframework.security.access.vote.ConsensusBased

现在已经明确了它们的作用，来看看它们的规则：

- AffirmativeBased：只要有一个同意，则同意；
- UnanimousBased：只要有一个拒绝，则决绝；
- ConsensusBased：如果同意数大于等于拒绝数，则通意，反之，则拒绝；



#### AccessDecisionVoter

org.springframework.security.access.AccessDecisionVoter



```


```



```
ObjectPostProcessor
```





No AuthenticationProvider found for org.springframework.security.authentication.UsernamePasswordAuthenticationToken





AbstractConfiguredSecurityBuilder#add(C)

它的Javadoc描述如下：

添加SecurityConfigurer，确保它是允许的，并在必要时立即调用SecurityConfigurer.init(SecurityBuilder)。



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





