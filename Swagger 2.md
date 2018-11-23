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

### *properties*

- `value`：描述类的作用
- `tags`：说明该类的作用，非空时将覆盖value的值
- `description`：对api资源的描述，在1.5版本后不再支持
- `basePath`：基本路径可以不配置，在1.5版本后不再支持
- `position`：如果配置多个Api 想改变显示的顺序位置，在1.5版本后不再支持
- `produces`：设置MIME类型的列表（output），例如：“`application/json, application/xml`”，默认为空。
- `consumes`：设置MIME类型的列表（output），例如：“`application/json, application/xml`”，默认为空。
- `protocols`：设置特定协议，多个协议用逗号分隔。例如：可选值:`http`、`https`、`ws`、`wss`。
- `authorizations`：获取授权列表（安全声明），如果未设置，则返回一个空的授权值。
- `hidden`：默认为false，设置为true，将在文档中隐藏。

### *sound code*

```java
package io.swagger.annotations;

import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 将一个类标记为一个Swagger资源。
 * <p>
 * By default, Swagger-Core will only include and introspect only classes that are annotated
 * with {@code @Api} and will ignore other resources (JAX-RS endpoints, Servlets and
 * so on).默认情况下，Swagger-Core只包含并内省了用{@code @Api}注释的类，并且忽略了其他资源(JAX-RS端点、servlet等)。
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface Api {
    /**
     * 隐式设置操作的标签，遗留支持(读描述)。
     * 在swagger-core 1.3.X，它被用作“path”，用于托管资源的API声明。
     * 这与swagger-core 1.5.X不再相关。
     * 如果{@link #tags()}未被使用，此值将用于设置此资源描述的操作的标记。否则,该值将被忽略。
     * leading/(如果存在)将被删除。
     *
     * @return 除非定义了{@link #tags()}，否则此资源下操作的标记名。
     */
    String value() default "";

    /**
     * API文档控制的标记列表。
     * 标记可用于根据资源或任何其他限定符对操作进行逻辑分组。
     * 非空值将覆盖{@link #value()}中提供的值。
     *
     * @return 标记值的字符串数组
     * @since 1.5.2-M1
     */
    String[] tags() default "";

    /**
     * 1.5.X没有使用，用于遗留支持。
     *
     * @return 关于这个API的更长的描述，不再使用。
     */
    @Deprecated String description() default "";

    /**
     * 1.5.X没有使用，用于遗留支持。
     * @return 这个操作的basePath，不再使用。
     */
    @Deprecated String basePath() default "";

    /**
     * 1.5.X没有使用，用于遗留支持。
     * @return 此API在资源列表中的位置，不再使用。
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



## @ApiOperation

用在请求的方法上，说明方法的用途、作用。

### *properties*

- `value`：说明方法的用途，作用
- `notes`：方法的备注说明
- `tags`：操作标签，非空时将覆盖value值
- `response`：响应类型（即返回对象）
- `responseContainer`：声明包装的响应容器（返回对象类型），有效值是“List”、“Set”或“Map”。其他任何值都将被忽略。
- `responseReference`：指定对响应类型的引用，将覆盖任何指定的response()类。
- `httpMethod`：请求方式，可选值：`GET`、`HEAD`、`POST`、`PUT`、`DELETE`、`OPTIONS`和`PATCH`。
- `position`：如果配置多个Api，想要改变显示的顺序位置。在1.5.x版本都不在支持。
- `nickname`：第三方工具唯一标识，默认为空。
- `produces`：设置MIME类型的列表（output），例如：“`application/json, application/xml`”，默认为空。
- `consumes`：设置MIME类型的列表（output），例如：“`application/json, application/xml`”，默认为空。
- `protocols`：设置特定协议，多个协议用逗号分隔。例如：可选值:`http`、`https`、`ws`、`wss`。
- `authorizations`：获取授权列表（安全声明），如果未设置，则返回一个空的授权值。
- `hidden`：默认为false，设置为true，将在文档中隐藏。
- `responseHeaders`：响应头列表
- `code`：响应的HTTP状态码，默认为200。
- `extensions`：扩展属性列表数组。

### *sound code*

```java
package io.swagger.annotations;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 描述针对特定路径的操作或HTTP方法。
 * <p>
 * 具有等效路径的操作在单个操作对象中分组。
 * HTTP方法和路径的组合创建了一个独特的操作。
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ApiOperation {
    /**
     * 对应于操作的“摘要”字段。
     * <p>
     * 提供此操作的简要描述。应该是120个字符或更少，以适当的能见度在Swagger-UI。
     */
    String value();

    /**
     * 对应于操作的“notes”字段。
     * <p>
     * 操作的详细描述。
     */
    String notes() default "";

    /**
     * API文档控制的标记列表。
     * 标记可用于根据资源或任何其他限定符对操作进行逻辑分组。
     * 非空值将覆盖该操作从{@link Api#value()}或{@link Api#tags()}收到的值。
     * @since 1.5.2-M1
     */
    String[] tags() default "";

    /**
     * 操作的响应类型。
     * <p>
     * 在JAX-RS应用程序中，将自动使用方法的返回类型， 除非是{@code javax.ws.rs.core.Response}。      * 在这种情况下，操作返回类型默认为“void”，因为无法知道实际的响应类型。
     * <p>
     * 设置此属性将覆盖任何自动派生的数据类型。
     * <p>
     * 如果使用的值是一个基本类型({@code Integer}， {@code Long}，…)的类，
     * 则将使用相应的基本类型。
     */
    Class<?> response() default Void.class;

    /**
     * 声明包装响应的容器。
     * <p>
     * 有效值是“List”、“Set”或“Map”。其他任何值都将被忽略。
     */
    String responseContainer() default "";

    /**
     * 指定对响应类型的引用。指定的引用可以是本地引用，也可以是远程引用，并将按原样使用， 
     * 并将覆盖任何指定的response()类。
     */

    String responseReference() default "";

    /**
     * 与使用的HTTP方法对应的“method”字段。
     * <p>
     * 如果没有说明，在JAX-RS应用程序中，将扫描并使用以下JAX-RS注释:{@code @GET}、
     * {@code @HEAD}、{@code @POST}、{@code @PUT}、{@code @DELETE}和{@code @OPTIONS}。
     * 注意，即使不是JAX-RS规范的一部分，如果您创建并使用{@code @PATCH}注释，
     * 它也将被解析和使用。如果设置了httpMethod属性，它将覆盖JAX-RS注释。
     * <p>
     * 对于servlet，必须手动指定HTTP方法。
     * <p>
     * 可接受的值是“GET”、“HEAD”、“POST”、“PUT”、“DELETE”、“OPTIONS”和“PATCH”。
     */
    String httpMethod() default "";

    /**
     * 1.5.X未使用，用于遗留支持。
     */
    @Deprecated int position() default 0;

    /**
     * 对应于“operationId”字段。
     * <p>
     * 第三方工具使用operationId来惟一地标识该操作。在时髦的2.0中,
     * 这不再是强制性的，如果不提供将保持为空。
     */
    String nickname() default "";

    /**
     * 对应于操作的“produces”字段。
     * <p>
     * 接受内容类型的逗号分隔值。
     * 例如，“application/json, application/xml”将建议此操作生成json和xml输出。
     * <p>
     * 对于JAX-RS资源，如果存在{@code @Produces}注释，则会自动获取{@code @Produces}注释的值。
     * 它还可以用于覆盖Swagger文档的{@code @Produces}值。
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
     * 为该操作设置特定的协议(方案)。
     * 多个协议用逗号分隔。可能的值:http、https、ws、wss。
     * @return 资源下的操作支持的协议。
     */
    String protocols() default "";

    /**
     * 对应于操作对象的“security”字段。
     * 在该操作的授权(安全要求)的列表中获取。
     * @return 服务器需要的授权数组，或未设置的单个空授权值。
     * @see Authorization
     */
    Authorization[] authorizations() default @Authorization(value = "");

    /**
     * 从操作列表中隐藏操作。
     */
    boolean hidden() default false;

    /**
     * A list of possible headers provided alongside the response.
     * 响应旁边可能提供的标头列表。
     * @return 一个响应头列表。
     */
    ResponseHeader[] responseHeaders() default @ResponseHeader(name = "", response = Void.class);

    /**
     * 响应的HTTP状态代码。
     * <p>
     * 值的形式应该是<a target="_blank" href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html">HTTP状态代码定义</a>.
     */
    int code() default 200;

    /**
     * @return 可选的扩展数组
     */

    Extension[] extensions() default @Extension(properties = @ExtensionProperty(name = "", value = ""));
}

```

### *example*





## @ApiImplicitParams

一个包装器，以允许多个`@ApiImplicitParam`对象的列表。





>example :

-------------------------------------------


```java
@ApiImplicitParams({
    @ApiImplicitParam(name = "offset",defaultValue = "0",required = false,paramType = "query",dataType = "int"),
    @ApiImplicitParam(name = "limit",defaultValue = "5",required = false,paramType = "query",dataType = "int")
})
```

## @ApiImplicitParam



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

## @ApiIgnore

使标注的接口不显示在swagger文档中不显示。



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