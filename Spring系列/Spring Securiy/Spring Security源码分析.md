### 

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
ObjectPostProcessor
```









