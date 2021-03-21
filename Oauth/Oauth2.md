# Oauth2

本文大部分内容引用自阮一峰博客：[OAuth 2.0 的四种方式](http://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html)

## RFC 6749

OAuth 2.0的标准是[RFC 6749](https://tools.ietf.org/html/rfc6749)文件。该文件解释了OAuth是什么。

```
OAuth引入了一个授权层，用来分离两种不同的角色：客户端和资源所有者。......资源所有者同意以后，资源服务器可以向客户端颁发令牌。客户端通过令牌去请求数据。
```

这段话的意思是：OAuth的核心就是向第三方应用颁发令牌。然后，RFC 6749接着写到：

```
由于互联网有多种场景，本标准定义了获得令牌的四种授权方式（authorization grant）。
```

OAuth 2.0获得令牌的四种授权方式：

| 名称       | 编码               |
| ---------- | ------------------ |
| 授权码     | authorization_code |
| 隐藏式     | implicit           |
| 密码式     | password           |
| 客户端凭证 | client_credentials |

## 第一种方式：授权码

**授权码（authorization code）方法，指的是第三方应用先申请一个授权码，然后再用该码获取令牌。**

这种方式是最常用的流程，安全性也最高，它适用于那些有后端的Web应用。授权码通过前端传送，令牌则是存储在后端，而且所有与资源服务器的通信都在后端完成。这样的前后端分离，可以避免令牌泄露。

1. 第一步，A网站提供一个链接，用户点击后就会跳转到B网站，授权用户数据给A网站使用。

   下面就是A网站跳转B网站的一个示意链接。

   ```http
   https://b.com/oauth/authorize?response_type=code&client_id=CLIENT_ID&redirect_uri=CALLBACK_URL&scope=read
   ```

   > 实际上这个链接是由B网站提供的，A网站将这个链接嵌入到自己的网站上，用户点击就可以跳转到B网站上，只是其中有些参数携带了A网站的信息。

   URL中需要携带四个参数：

   - `response_type`：表示要求返回授权码（code）。
   - `client_id`：客户端ID，表示让B知道是谁在请求（这里的客户端指的就是A网站，每个客户端都有为一个属于自己的客户端ID）。
   - `redirect_uri`：B接收或拒绝请求后的跳转网址。
   - `scope`：表示要求的授权范围（这里是只读）。

2. 第二步，用户跳转后，B万扎安会要求用户登录，然后询问是否同意给予A网站授权。用户表示同意，这时B网站就会跳回`redirect_uri`参数指定的网址。跳转时，会传回一个授权码，例如：

   ```http
   https://a.com/callback?code=AUTHORIZATION_CODE
   ```

   上面的URL中，`code`参数就是授权码。

3. 第三步，A网站拿到授权码以后，就可以在后端，向B网站请求令牌。

   ```http
   https://b.com/oauth/token?
   	client_id=CLIENT_ID&
   	client_secret=CLIENT_SECRET&
   	grant_type=authorization_code&
   	code=AUTHORIZATION_CODE&
   	redirect_uri=CALLBACK_URL
   ```

   URL中需要携带五个参数：

   - `client_id`：客户端ID，相当于客户端的用户名，用于让B确认A的身份。
   - `client_secret`：客户端凭证，相当于客户端的密码，用于让B确认A的身份（客户端凭证是保密的，因此只能在后端保存和发送请求）。
   - `grant_type`：参数值是`authorization_code`，表示采用的授权方式是授权码。
   - `code`：上一步拿到的授权码。
   - `redirect_uri`：令牌颁发后的回调网址。

4. 第四步，B网站收到请求以后，就会颁发令牌。具体做法是向`redirect_uri`参数指定的网址发送一段JSON数据。

   ```json
   {
       "access_token":"ACCESS_TOKEN",
       "token_type":"bearer",
       "expires_in":259200,
       "refresh_token":"REFRESH_TOKEN",
       "scope":"read",
       "uid":100101,
       "info":{...}
   }
   ```

   上面的JSON数据中，`access_token`就是拿到的令牌。

授权码模式就是通过第三方登录的单点登录模式，例如我们常常使用的通过QQ，微信，微博，GitHub登录等等。

![img](images/bg2019040905.jpg)

## 第二种方式：隐藏式

有些Web应用是纯前端应用，没有后端。这时就不能用上面的方式了，必须将令牌存储在前端。**RFC 6749就规定了第二种方式，允许直接向前端颁发令牌。这种方式没有授权码这个中间步骤，所以称为“隐藏式”（implicit）。**

1. 第一步，A网站提供一个链接，要求用户跳转到B网站，授权用户数据给A网站使用。

   ```http
   https://b.com/oauth/authorize?
   	response_type=token&
   	client_id=CLIENT_ID&
   	redirect_uri=CALLBACK_URL&
   	scope=read
   ```

   上面 URL 中，`response_type`参数为`token`，表示要求直接返回令牌。

2. 第二步，用户跳转到B网站，登录后同意给予A网站授权。这时，B网站就会跳回到`redirect_uri`参数指定的网址，并发令牌作为URL参数，传给A网站。

   ```http
   https://a.com/callback#token=ACCESS_TOKEN
   ```

   > Node：令牌的位置是URL锚点（fragment），而不是查询字符换（querystring），这是因为OAuth 2.0允许跳转网址是HTTP协议，因此存在“中间人攻击”的风险，而浏览器跳转时，锚点不会发到服务器，就减少了泄露令牌的风险。

在“隐藏式”的模式下，redirect_uri可以是一个前端地址，也可以是一个后端地址，但一般是一个前端地址，因为如果是后端地址的话，就没有后续的交互操作可以把Token返回给前端，而Token最终还是要给前端用于请求资源操作的。

但这种直接把令牌传给前端的方式是很不安全的。因此，只能用于一些安全要求不高的场景，并且令牌的有效期必须非常短，通常就是会话期间（session）有效，浏览器关掉，令牌就失效了。

![img](images/bg2019040906.jpg)

## 第三种方式：密码式

**如果你高度信任某个应用，RFC 6749也允许用户把用户名和密码，直接告诉该应用。该应用就使用你的密码申请令牌，这种方式称为“密码式”（password）。**

1. 第一步，A网站要求用户提供B网站的用户名和密码，拿到以后，A就直接向B请求令牌。

   ```http
   https://oauth.b.com/token?
   	grant_type=password&
   	username=USERNAME&
   	password=PASSWORD&
   	client_id=CLIENT_ID
   ```

   URL中需要携带四个参数：

   - `grant_type`：授权方式，这里为`password`，表示“密码式”。
   - `username`：B的用户名。
   - `password`：B的密码。
   - `client_id`：A在B哪儿的客户端ID。

2. 第二步，B网站验证身份通过后，直接给出令牌。注意，这时不需要跳转。而是把令牌放在JSON数据里面，作为HTTP响应，A因此拿到令牌再返回到客户端。

这种方式需要用户给出自己的用户名和密码，因此肯定不能用于第三方应用，只适用于相互之间高度信任的应用，例如公司内部的应用。

## 第四种方式：凭证式

**凭证式（client credentials）只适用于没有前端的命令行应用，即在命令行下请求令牌。**

1. 第一步，A应用在命令行向B发出请求。

   ```http
   https://oauth.b.com/token?
   	grant_type=client_credentials&
   	client_id=CLENT_ID&
   	client_secret=CLIENT_SECRET
   ```

   `grant_type`参数等于`client_credentials`表示采用凭证式，`client_id`和`client_secret`就相当于是应用A在应用B处的用户名和密码。

2. 第二步，B网站验证通过以后，直接返回令牌。

这种方式给出的令牌是针对第三方应用的，而不是针对用户，即有可能有多个用户共享同一个令牌。

## 令牌的使用

A网站拿到令牌以后，就可以向B网站的API请求数据了。

此时，每个发到API的请求都必须携带有令牌。具体做法就是在请求的头信息里加上一个`Authorization`字段，令牌就放在该字段中。

```shell
$ curl -H "Authorization: Bearer ACCESS_TOKEN" https://api.b.com
```

上面的命令中，`ACCESS_TOKEN`就是令牌。

## 更新令牌

令牌的有效期到了，如果让用户重新走一遍上面的流程，再申请一个新的令牌，很可能体验不好，而且也没有必要。OAuth 2.0允许用户自动更新令牌。

具体方法是，B网站颁发令牌的时候，一次性颁发两个令牌，一个用于获取数据，另一个用于获取新的令牌（refresh token）。令牌到期前，用户使用refresh_token发一个请求，去更新令牌。

```http
https://b.com/oauth/token?
	grant_type=refresh_type&
	client_id=CLIENT_ID&
	client_secret=CLIENT_SECRET&
	refresh_token=REFRESH_TOKEN
```

更新令牌请求需要携带四个参数：

- `grant_type`：`grant_type`参数为`refresh_type`表示要求更新令牌。
- `client_id`：客户端ID，相当于客户端的用户名，用于让B确认A的身份。
- `client_secret`：客户端凭证，相当于客户端的密码，用于让B确认A的身份。
- `refresh_token`：用于更新令牌的令牌。

这个更新令牌的请求动作一般是在后端进行，客户端只需要传递`refresh_token`就可以了，然后在后端组装这些参数。因为客户端凭证是保密的，因此只能在后端保存和发送请求

网站B验证通过之后，就会颁发新的令牌。