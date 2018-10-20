# Swagger 2

https://blog.csdn.net/lovesomnus/article/details/78873089

https://www.cnblogs.com/fengli9998/p/7921601.html

https://www.cnblogs.com/javahr/p/8423489.html

https://blog.csdn.net/u014231523/article/details/76522486

https://blog.csdn.net/u010889990/article/details/79441978

https://www.cnblogs.com/JealousGirl/p/swagger2.html



https://blog.csdn.net/jyfu2_12/article/details/79207643

https://blog.csdn.net/u011671022/article/details/79030588

https://blog.csdn.net/z28126308/article/details/71126677



https://blog.csdn.net/z28126308/article/details/71126677

[TOC]

## @Api

用在请求类上，标识对类的说明。

```java
@Api：用在请求的类上，表示对类的说明
   常用参数：
      tags="说明该类的作用，非空时将覆盖value的值"
       value="描述类的作用"
   其他参数：
      description           对api资源的描述，在1.5版本后不再支持
      basePath              基本路径可以不配置，在1.5版本后不再支持
      position              如果配置多个Api 想改变显示的顺序位置，在1.5版本后不再支持
      produces              设置MIME类型列表（output），例："application/json, application/xml"，默认为空
      consumes              设置MIME类型列表（input），例："application/json, application/xml"，默认为空
      protocols             设置特定协议，例：http， https， ws， wss。
      authorizations        获取授权列表（安全声明），如果未设置，则返回一个空的授权值。
      hidden                默认为false， 配置为true 将在文档中隐藏

   示例：
        @Api(tags="登录请求")
        @Controller
        @RequestMapping(value="/highPregnant")
        public class LoginController {}
```



```java
package io.swagger.annotations;

import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * Marks a class as a Swagger resource.
 * <p>
 * By default, Swagger-Core will only include and introspect only classes that are annotated
 * with {@code @Api} and will ignore other resources (JAX-RS endpoints, Servlets and
 * so on).
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface Api {
    /**
     * Implicitly sets a tag for the operations, legacy support (read description).
     * <p>
     * In swagger-core 1.3.X, this was used as the 'path' that is to host the API Declaration of the
     * resource. This is no longer relevant in swagger-core 1.5.X.
     * <p>
     * If {@link #tags()} is <i>not</i> used, this value will be used to set the tag for the operations described by this
     * resource. Otherwise, the value will be ignored.
     * <p>
     * The leading / (if exists) will be removed.
     *
     * @return tag name for operations under this resource, unless {@link #tags()} is defined.
     */
    String value() default "";

    /**
     * A list of tags for API documentation control.
     * Tags can be used for logical grouping of operations by resources or any other qualifier.
     * <p>
     * A non-empty value will override the value provided in {@link #value()}.
     *
     * @return a string array of tag values
     * @since 1.5.2-M1
     */
    String[] tags() default "";

    /**
     * Not used in 1.5.X, kept for legacy support.
     *
     * @return a longer description about this API, no longer used.
     */
    @Deprecated String description() default "";

    /**
     * Not used in 1.5.X, kept for legacy support.
     *
     * @return the basePath for this operation, no longer used.
     */
    @Deprecated String basePath() default "";

    /**
     * Not used in 1.5.X, kept for legacy support.
     *
     * @return the position of this API in the resource listing, no longer used.
     */
    @Deprecated int position() default 0;

    /**
     * Corresponds to the `produces` field of the operations under this resource.
     * <p>
     * Takes in comma-separated values of content types.
     * For example, "application/json, application/xml" would suggest the operations
     * generate JSON and XML output.
     * <p>
     * For JAX-RS resources, this would automatically take the value of the {@code @Produces}
     * annotation if such exists. It can also be used to override the {@code @Produces} values
     * for the Swagger documentation.
     *
     * @return the supported media types supported by the server, or an empty string if not set.
     */
    String produces() default "";

    /**
     * Corresponds to the `consumes` field of the operations under this resource.
     * <p>
     * Takes in comma-separated values of content types.
     * For example, "application/json, application/xml" would suggest the operations
     * accept JSON and XML input.
     * <p>
     * For JAX-RS resources, this would automatically take the value of the {@code @Consumes}
     * annotation if such exists. It can also be used to override the {@code @Consumes} values
     * for the Swagger documentation.
     *
     * @return the consumes value, or empty string if not set
     */
    String consumes() default "";

    /**
     * Sets specific protocols (schemes) for the operations under this resource.
     * <p>
     * Comma-separated values of the available protocols. Possible values: http, https, ws, wss.
     *
     * @return the protocols supported by the operations under the resource.
     */
    String protocols() default "";

    /**
     * Corresponds to the `security` field of the Operation Object.
     * <p>
     * Takes in a list of the authorizations (security requirements) for the operations under this resource.
     * This may be overridden by specific operations.
     *
     * @return an array of authorizations required by the server, or a single, empty authorization value if not set.
     * @see Authorization
     */
    Authorization[] authorizations() default @Authorization(value = "");

    /**
     * Hides the operations under this resource.
     *
     * @return true if the api should be hidden from the swagger documentation
     */
    boolean hidden() default false;
}

```







## Target
```
@Target(ElementType.TYPE)
```
## Attribute
### `description`
接口集合描述
>example : `description = "用户"`

# 

## @ApiOperation

用在请求方法上，说明方法的用途、作用

```java
/**
 * Copyright 2016 SmartBear Software
 * <p>
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 * <p>
 * http://www.apache.org/licenses/LICENSE-2.0
 * <p>
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package io.swagger.annotations;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * Describes an operation or typically a HTTP method against a specific path.
 * <p>
 * Operations with equivalent paths are grouped in a single Operation Object.
 * A combination of a HTTP method and a path creates a unique operation.
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ApiOperation {
    /**
     * Corresponds to the `summary` field of the operation.
     * <p>
     * Provides a brief description of this operation. Should be 120 characters or less
     * for proper visibility in Swagger-UI.
     */
    String value();

    /**
     * Corresponds to the 'notes' field of the operation.
     * <p>
     * A verbose description of the operation.
     */
    String notes() default "";

    /**
     * A list of tags for API documentation control.
     * <p>
     * Tags can be used for logical grouping of operations by resources or any other qualifier.
     * A non-empty value will override the value received from {@link Api#value()} or {@link Api#tags()}
     * for this operation.
     *
     * @since 1.5.2-M1
     */
    String[] tags() default "";

    /**
     * The response type of the operation.
     * <p>
     * In JAX-RS applications, the return type of the method would automatically be used, unless it is
     * {@code javax.ws.rs.core.Response}. In that case, the operation return type would default to `void`
     * as the actual response type cannot be known.
     * <p>
     * Setting this property would override any automatically-derived data type.
     * <p>
     * If the value used is a class representing a primitive ({@code Integer}, {@code Long}, ...)
     * the corresponding primitive type will be used.
     */
    Class<?> response() default Void.class;

    /**
     * Declares a container wrapping the response.
     * <p>
     * Valid values are "List", "Set" or "Map". Any other value will be ignored.
     */
    String responseContainer() default "";

    /**
     * Specifies a reference to the response type. The specified reference can be either local or remote and
     * will be used as-is, and will override any specified response() class.
     */

    String responseReference() default "";

    /**
     * Corresponds to the `method` field as the HTTP method used.
     * <p>
     * If not stated, in JAX-RS applications, the following JAX-RS annotations would be scanned
     * and used: {@code @GET}, {@code @HEAD}, {@code @POST}, {@code @PUT}, {@code @DELETE} and {@code @OPTIONS}.
     * Note that even though not part of the JAX-RS specification, if you create and use the {@code @PATCH} annotation,
     * it will also be parsed and used. If the httpMethod property is set, it will override the JAX-RS annotation.
     * <p>
     * For Servlets, you must specify the HTTP method manually.
     * <p>
     * Acceptable values are "GET", "HEAD", "POST", "PUT", "DELETE", "OPTIONS" and "PATCH".
     */
    String httpMethod() default "";

    /**
     * Not used in 1.5.X, kept for legacy support.
     */
    @Deprecated int position() default 0;

    /**
     * Corresponds to the `operationId` field.
     * <p>
     * The operationId is used by third-party tools to uniquely identify this operation. In Swagger 2.0, this is
     * no longer mandatory and if not provided will remain empty.
     */
    String nickname() default "";

    /**
     * Corresponds to the `produces` field of the operation.
     * <p>
     * Takes in comma-separated values of content types.
     * For example, "application/json, application/xml" would suggest this operation
     * generates JSON and XML output.
     * <p>
     * For JAX-RS resources, this would automatically take the value of the {@code @Produces}
     * annotation if such exists. It can also be used to override the {@code @Produces} values
     * for the Swagger documentation.
     */
    String produces() default "";

    /**
     * Corresponds to the `consumes` field of the operation.
     * <p>
     * Takes in comma-separated values of content types.
     * For example, "application/json, application/xml" would suggest this API Resource
     * accepts JSON and XML input.
     * <p>
     * For JAX-RS resources, this would automatically take the value of the {@code @Consumes}
     * annotation if such exists. It can also be used to override the {@code @Consumes} values
     * for the Swagger documentation.
     */
    String consumes() default "";

    /**
     * Sets specific protocols (schemes) for this operation.
     * <p>
     * Comma-separated values of the available protocols. Possible values: http, https, ws, wss.
     *
     * @return the protocols supported by the operations under the resource.
     */
    String protocols() default "";

    /**
     * Corresponds to the `security` field of the Operation Object.
     * <p>
     * Takes in a list of the authorizations (security requirements) for this operation.
     *
     * @return an array of authorizations required by the server, or a single, empty authorization value if not set.
     * @see Authorization
     */
    Authorization[] authorizations() default @Authorization(value = "");

    /**
     * Hides the operation from the list of operations.
     */
    boolean hidden() default false;

    /**
     * A list of possible headers provided alongside the response.
     *
     * @return a list of response headers.
     */
    ResponseHeader[] responseHeaders() default @ResponseHeader(name = "", response = Void.class);

    /**
     * The HTTP status code of the response.
     * <p>
     * The value should be one of the formal <a target="_blank" href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html">HTTP Status Code Definitions</a>.
     */
    int code() default 200;

    /**
     * @return an optional array of extensions
     */

    Extension[] extensions() default @Extension(properties = @ExtensionProperty(name = "", value = ""));
}

```



## Target

```
@Target(ElementType.METHOD)
```
## Attribute
### `value`
接口描述
>example : `value = "首字母排序全部商户"`

-------------------------------------------


# `@ApiImplicitParams`
一个包装器，以允许多个`@ApiImplicitParam`对象的列表。

## Target

```
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE, ElementType.TYPE})
```
## Attribute
### `value`
参数的集合
```
ApiImplicitParam[] value();
```
>example :

-------------------------------------------


```java
@ApiImplicitParams({
    @ApiImplicitParam(name = "offset",defaultValue = "0",required = false,paramType = "query",dataType = "int"),
    @ApiImplicitParam(name = "limit",defaultValue = "5",required = false,paramType = "query",dataType = "int")
})
```


# `@ApiImplicitParam`

## Target

```
@Target(ElementType.METHOD)
```
## Attribute
### `name`
参数的名称
```
String name() default "";
```
>example : `name = "limit"`

### `defaultValue`
参数的默认值

```
String defaultValue() default "";
```
>example : `defaultValue = "xxx"`
### `required`
指定是否需要参数。路径参数应该根据需要设置。

```
boolean required() default false;
```
>example : `required = false`
### `paramType`
参数的参数类型。可以取值 : 
- `path`
- `query`
- `body`
- `header`
- `form`
```
String paramType() default "";
```
>example : `paramType = "query"`
### `dataType`
参数的数据类型。这可以是类名或原语。

```
String dataType() default "";
```
>example : `dataType = "int"`

# `@ApiIgnore`
使标注的接口不显示在swagger文档中不显示。
## Target
```
@Target({ElementType.METHOD, ElementType.TYPE, ElementType.PARAMETER})
```
## Attribute
### `value`
简要说明为什么忽略此参数/操作。
```
String value() default "";
```

## @ApiModel

## @ApiModelProperty

## @ApiResponses

## @ApiResponse



POST请求方法的入参一般都是一个对象，默认在API文档中是以body的方式请求，也确实是符合我们需要的，但是默认情况下API文档上却并不会显示对应对象的JSON映射格式，添加Spring注解`org.springframework.web.bind.annotation.RequestBody`就可以将入参对象映射为JSON格式实例入参。

例如：

```java
@ApiOperation(value = "...")
@PostMapping("...")
public Object getUsers(@RequestBody` User user) {
    ...
}
```



有些时候我们的接口请求方法是GET方式，但是接口的入参却是一个对象，在API文档中默认显示的是body形式，而我们需要的却是query形式。这种情况下，需要在对象入参前添加Spring的`org.springframework.web.bind.annotation.ModelAttribute`注解。至于其原理，这里不做深入探讨。

> 可以使用`@ApiImplicitParams`注解手动标注需要的query入参。

例如：

```java
@ApiOperation(value = "...")
@GetMapping("...")
public Object getUsers(@ModelAttribute User user) {
    ...
}
```







```java


@ApiOperation：用在请求的方法上，说明方法的用途、作用
   常用参数：
      value="说明方法的用途、作用"
       notes="方法的备注说明"
   其他参数：
      tags                操作标签，非空时将覆盖value的值
      response            响应类型（即返回对象）
      responseContainer   声明包装的响应容器（返回对象类型）。有效值为 "List", "Set" or "Map"。
      responseReference  指定对响应类型的引用。将覆盖任何指定的response（）类
      httpMethod        指定HTTP方法，"GET", "HEAD", "POST", "PUT", "DELETE", "OPTIONS" and "PATCH"
      position            如果配置多个Api 想改变显示的顺序位置，在1.5版本后不再支持
      nickname         第三方工具唯一标识，默认为空
      produces            设置MIME类型列表（output），例："application/json, application/xml"，默认为空
      consumes            设置MIME类型列表（input），例："application/json, application/xml"，默认为空
      protocols           设置特定协议，例：http， https， ws， wss。
      authorizations      获取授权列表（安全声明），如果未设置，则返回一个空的授权值。
      hidden              默认为false， 配置为true 将在文档中隐藏
      responseHeaders       响应头列表
      code            响应的HTTP状态代码。默认 200
      extensions       扩展属性列表数组

   示例：
        @ResponseBody
        @PostMapping(value="/login")
        @ApiOperation(value = "登录检测", notes="根据用户名、密码判断该用户是否存在")
        public UserModel login(@RequestParam(value = "name", required = false) String account,
        @RequestParam(value = "pass", required = false) String password){}


@ApiImplicitParams：用在请求的方法上，表示一组参数说明
    @ApiImplicitParam：用在@ApiImplicitParams注解中，指定一个请求参数的各个方面
        name：参数名，参数名称可以覆盖方法参数名称，路径参数必须与方法参数一致
        value：参数的汉字说明、解释
        required：参数是否必须传，默认为false [路径参数必填]
        paramType：参数放在哪个地方
            · header --> 请求参数的获取：@RequestHeader
            · query --> 请求参数的获取：@RequestParam
            · path（用于restful接口）--> 请求参数的获取：@PathVariable
            · body（不常用）
            · form（不常用）
        dataType：参数类型，默认String，其它值dataType="Integer"
        defaultValue：参数的默认值

   示例：
       @ResponseBody
        @PostMapping(value="/login")
        @ApiOperation(value = "登录检测", notes="根据用户名、密码判断该用户是否存在")
        @ApiImplicitParams({
                    @ApiImplicitParam(name = "name", value = "用户名", required = false, paramType = "query", dataType = "String"),
                    @ApiImplicitParam(name = "pass", value = "密码", required = false, paramType = "query", dataType = "String")
            })
        public UserModel login(@RequestParam(value = "name", required = false) String account,
                @RequestParam(value = "pass", required = false) String password){}

   其他参数（@ApiImplicitParam）：
        allowableValues    限制参数的可接受值。1.以逗号分隔的列表   2、范围值  3、设置最小值/最大值
        access             允许从API文档中过滤参数。
        allowMultiple      指定参数是否可以通过具有多个事件接受多个值，默认为false
        example            单个示例
        examples         参数示例。仅适用于BodyParameters



@ApiModel：用于响应类上，表示一个返回响应数据的信息（这种一般用在post创建的时候，使用@RequestBody这样的场景，请求参数无法使用@ApiImplicitParam注解进行描述的时候）
    @ApiModelProperty：用在属性上，描述响应类的属性
   示例：
       @ApiModel(value="用户登录信息", description="用于判断用户是否存在")
        public class UserModel implements Serializable{

           private static final long serialVersionUID = 1L;

           /**
            * 用户名
            */
           @ApiModelProperty(value="用户名")
           private String account;

           /**
             * 密码
             */
            @ApiModelProperty(value="密码")
           private String password;


        }
   其他(@ApiModelProperty)：
       value                  此属性的简要说明。
        name                     允许覆盖属性名称
        allowableValues          限制参数的可接受值。1.以逗号分隔的列表   2、范围值  3、设置最小值/最大值
        access             允许从API文档中过滤属性。
           notes              目前尚未使用。
        dataType            参数的数据类型。可以是类名或者参数名，会覆盖类的属性名称。
        required            参数是否必传，默认为false
        position            允许在类中对属性进行排序。默认为0
        hidden             允许在Swagger模型定义中隐藏该属性。
        example                属性的示例。
        readOnly            将属性设定为只读。
        reference           指定对相应类型定义的引用，覆盖指定的任何参数值



@ApiResponses：用在请求的方法上，表示一组响应
    @ApiResponse：用在@ApiResponses中，一般用于表达一个错误的响应信息
        code：数字，例如400
        message：信息，例如"请求参数没填好"
        response：抛出异常的类

   示例：
       @ResponseBody
        @PostMapping(value="/update/{id}")
       @ApiOperation(value = "修改用户信息",notes = "打开页面并修改指定用户信息")
        @ApiResponses({
            @ApiResponse(code=400,message="请求参数没填好"),
            @ApiResponse(code=404,message="请求路径没有或页面跳转路径不对")
        })
        public JsonResult update(@PathVariable String id, UserModel model){}

@ApiParam： 用在请求方法中，描述参数信息
   name：参数名称，参数名称可以覆盖方法参数名称，路径参数必须与方法参数一致
   value：参数的简要说明。
   defaultValue：参数默认值
   required           属性是否必填，默认为false [路径参数必须填]

    示例：
         @ResponseBody
         @PostMapping(value="/login")
         @ApiOperation(value = "登录检测", notes="根据用户名、密码判断该用户是否存在")
         public UserModel login(@ApiParam(name = "name", value = "用户名", required = false) @RequestParam(value = "name", required = false) String account,
                @ApiParam(name = "pass", value = "密码", required = false) @RequestParam(value = "pass", required = false) String password){}


         或以实体类为参数：
         @ResponseBody
         @PostMapping(value="/login")
         @ApiOperation(value = "登录检测", notes="根据用户名、密码判断该用户是否存在")
         public UserModel login(@ApiParam(name = "model", value = "用户信息Model") UserModel model){}

    其他：
        allowableValues    限制参数的可接受值。1.以逗号分隔的列表   2、范围值  3、设置最小值/最大值
        access             允许从API文档中过滤参数。
        allowMultiple      指定参数是否可以通过具有多个事件接受多个值，默认为false
        hidden             隐藏参数列表中的参数。
        example            单个示例
        examples         参数示例。仅适用于BodyParameters
```