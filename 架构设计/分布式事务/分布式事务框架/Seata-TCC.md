# Seata TCC

[TOC]

## 前置准备

1. 启动Seata Server

    ```shell
     $ seata-server.sh -h 127.0.0.1 -p 8091
    ```

    > 本文主要讲述Seata-TCC的使用，因此启动Seata Server仅简单的以本地File模式启动。关于Seata Server更详细的描述请参考[Seata Server]()。

2. 添加依赖

    ```xml
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-alibaba-seata</artifactId>
        <version>${seata-version}</version>
    </dependency>
    ```

3. 在`resources`目录下添加`file.conf`和`registry.conf`文件

	`file.conf`和`registry.conf`文件位于`seata/script/client/conf`目录下，直接拷贝即可。

4. 指定"事务服务组"

    ```properties
    spring.cloud.alibaba.seata.tx-service-group=default_tx_group
    ```

    > 如果不设置"事务服务组"，将报错：endpoint format should like ip:port。

    > 如果没有启动Seata Server，将报错：can not register RM,err:can not connect to services-server。

## 实现案例

定义了三个服务：订单，账户，积分。业务流程是下订单→扣账户→加积分，对应三个分支事务。

### 开启全局事务

首先在调用方`OrderController`的接口上标注`@GlobalTransactional`注解开启全局事务。代码如下所示：

```java
@RestController
@RequestMapping("/order")
public class OrderController {

    @Autowired
    private IOrderService orderService;

    @PostMapping
    @GlobalTransactional
    public void createOrder() {
        orderService.createOrder(null);
    }
}
```

### 开启分支事务Order

分支事务服务接口：

1. 在接口上标注`@LocalTCC`注解
2. 在接口中声明Try，Confirm，Cancel抽象方法
3. 在Try抽象方法上标注`@TwoPhaseBusinessAction`注解，并指明名称（`name`），Confirm方法（`commitMethod`）和Cancel方法（`rollbackMethod`）。
4. Try，Confirm，Cancel方法的第一个参数必须是`BusinessActionContext`参数。

```java
@LocalTCC
public interface IOrderService extends IService<Order> {

    @TwoPhaseBusinessAction(name = "createOrder", commitMethod = "commitCreateOrder", rollbackMethod = "rollbackCreateOrder")
    void createOrder(BusinessActionContext context);

    void commitCreateOrder(BusinessActionContext context);

    void rollbackCreateOrder(BusinessActionContext context);
}
```

实现`IOrderService`接口，Try，Confirm，Cancel实现方法根据需要开启本地事务。

```java
@Service
public class OrderServiceImpl extends ServiceImpl<OrderMapper, Order> implements IOrderService {

    @Autowired
    private AccountFeign accountFeign;

    @Autowired
    private IntegralFeign integralFeign;

    @Override
    @Transactional
    public void createOrder(BusinessActionContext context) {
        accountFeign.deductAmount();
        integralFeign.addIntegral();
        System.out.println("Order try");
    }

    @Override
    @Transactional
    public void commitCreateOrder(BusinessActionContext context) {
        System.out.println("Order commit");
    }

    @Override
    @Transactional
    public void rollbackCreateOrder(BusinessActionContext context) {
        System.out.println("Order rollback");
    }
}
```
### 开启分支事务Integal

积分服务Controller接口。代码如下所示：

```java
@RestController
@RequestMapping("/integral")
public class IntegralController {

    @Autowired
    private IIntegralService iIntegralService;

    @PostMapping
    public void addIntegral() {
        iIntegralService.addIntegral(null);
    }
}
```

分支事务服务接口：等同`IOrderService`。

```java
@LocalTCC
public interface IIntegralService extends IService<Integral> {

    @TwoPhaseBusinessAction(name = "addIntegral", commitMethod = "commitAddIntegral", rollbackMethod = "rollbackAddIntegral")
    void addIntegral(BusinessActionContext context);

    void commitAddIntegral(BusinessActionContext context);

    void rollbackAddIntegral(BusinessActionContext context);
}

```

实现`IIntegralService`接口，Try，Confirm，Cancel实现方法根据需要开启本地事务。代码如下所示：

```java
@Service
public class IntegralServiceImpl extends ServiceImpl<IntegralMapper, Integral> implements IIntegralService {

    @Override
    @Transactional
    public void addIntegral(BusinessActionContext context) {
        System.out.println("Integral try");
    }

    @Override
    @Transactional
    public void commitAddIntegral(BusinessActionContext context) {
        System.out.println("Integral commit");
    }

    @Override
    @Transactional
    public void rollbackAddIntegral(BusinessActionContext context) {
        System.out.println("Integral rollback");
    }
}
```

### 开启分支事务Account

账户服务Controller接口。代码如下所示：

```java
@RestController
@RequestMapping("/account")
public class AccountController {

    @Autowired
    private IAccountService iAccountService;

    @PutMapping("/deduct")
    public void deductAmount(){
        iAccountService.deductAmount(null);
    }
}
```

分支事务服务接口：等同`IOrderService`。

```java
@LocalTCC
public interface IAccountService extends IService<Account> {

    @TwoPhaseBusinessAction(name = "deductAmount", commitMethod = "commitDeductAmount", rollbackMethod = "rollbackDeductAmount")
    void deductAmount(BusinessActionContext context);

    void commitDeductAmount(BusinessActionContext context);

    void rollbackDeductAmount(BusinessActionContext context);
}
```

实现`IAccountService`接口，Try，Confirm，Cancel实现方法根据需要开启本地事务。代码如下所示：

```java
@Service
public class AccountServiceImpl extends ServiceImpl<AccountMapper, Account> implements IAccountService {

    @Override
    @Transactional
    public void deductAmount(BusinessActionContext context) {
        System.out.println("Account try");
    }

    @Override
    @Transactional
    public void commitDeductAmount(BusinessActionContext context) {
        System.out.println("Account commit");
    }

    @Override
    @Transactional
    public void rollbackDeductAmount(BusinessActionContext context) {
        System.out.println("Account rollback");
    }
}

```

### Feign接口

Feign接口就是普通的Feign接口，不需要做额外的工作。

- 调用账户服务的Feign接口：

  ```java
  @FeignClient("sadness-tcc-account")
  @RequestMapping("/account")
  public interface AccountFeign {
  
      @PutMapping("/deduct")
      void deductAmount();
  }
  ```

- 调用积分服务的Feign接口：

  ```java
  @FeignClient("sadness-tcc-integral")
  @RequestMapping("/integral")
  public interface IntegralFeign {
  
      @PostMapping
      void addIntegral();
  }
  ```

## 注解说明

使用Seata-TCC，应用了三个注解，分别是：

- `@LocalTCC`：标注在TCC接口上的注解，表示该接口的实现类被Seata管理，Seata根据事务的状态，自动调用相应的Confirm方法或Cancel方法。

- `@TwoPhaseBusinessAction`：标注在TCC接口中Try方法上的注解。

  - `name`：TCC的Bean名称，必须唯一；一般指定方法名即可。
  - `commitMethod`：Confirm方法名称。
  - `rollbackMethod`：Cancel方法名称。

- `@BusinessActionContextParameter`：用于修饰Try方法中需要传递给`BusinessActivityContext`的TCC参数; 在try方法的参数上添加这个注释，参数将被传递到`BusinessActivityContext` 。在Confirm方法和Cancel方法中可以通过`BusinessActivityContext`获取。

  - `paramName`：参数名称。`BusinessActionContext`存在`actionContext`属性，类型为`Map<String, Object>`，被`@BusinessActionContextParameter`修饰的参数将会存储在这个Map集合中，key为`paramName`指定的参数名称。`paramName`为非必填，不过尽量需要填上，因为它的默认值为空字符串，即key意味着为空字符串。
  - `isShardingParam`：是否是一个分片参数，默认false。
  - `index`：指定该参数在List中的索引。如果指定`index`参数值大于等于0，那么被`@BusinessActionContextParameter`修饰的参数的类型必须为`java.util.List`。而`index`就是指该List的索引。List中该索引的值会被放入`BusinessActivityContext` 中。
  - `isParamInProperty`：是否从对象的属性获取参数，默认`false`。如果为`true`，则意味着被`@BusinessActionContextParameter`修饰的参数必须是一个pojo对象，seata会解析该pojo对象中的所有字段，如果字段被`@BusinessActionContextParameter`修饰，那么该字段的值将会被存入`BusinessActivityContext`中。如果`@BusinessActionContextParameter`声明了`paramName`参数，那么将以该参数作为key，如果没有声明，将以字段名作为key。同样，如果标注在字段上的`@BusinessActionContextParameter`声明了`index`或`isParamInProperty=true`，那么也需要符合相应的规则。

  示例：

  ```java
  @TwoPhaseBusinessAction(name = "deductAmount", commitMethod = "commitDeductAmount", rollbackMethod = "rollbackDeductAmount")
  void deductAmount(BusinessActionContext context,
    @BusinessActionContextParameter(paramName = "name") String name,
    @BusinessActionContextParameter(paramName = "list", index = 2) List<String> list,
    @BusinessActionContextParameter(paramName = "account", isParamInProperty = true) Account account);
  ```

  POJO：Account

  ```java
  public class Account {
  	......
      @ApiModelProperty(value = "余额")
      @BusinessActionContextParameter
      private BigDecimal balance;
      ......
  }
  ```

  >对被修饰`@BusinessActionContextParameter`注解参数的解析函数位于：`io.seata.rm.tcc.interceptor.ActionInterceptorHandler#fetchActionRequestContext`

## BusinessActionContext

`BusinessActionContext`用于传递事务信息，是一个普通的POJO对象，包含4个成员变量，分别是：

- `String xid`：全局事务ID。
- `String branchId`：当前分支事务ID。
- `String actionName`：当前分支事务的名称——`@TwoPhaseBusinessAction`注解name属性所声明的名称。
- `Map<String, Object> actionContext`：Try方法中被`@BusinessActionContextParameter`修饰的参数的。

## 幂等，空回滚，悬挂

TCC模式具有幂等，空回滚，悬挂等问题。Seata-TCC提供了子事务屏障解决了这些问题，使开发人员只需要关注业务即可。



添加子事务屏障[SQL](https://github.com/seata/seata/blob/develop/script/client/tcc/db)：

```mysql
CREATE TABLE IF NOT EXISTS `tcc_fence_log`
(
    `xid`           VARCHAR(128)  NOT NULL COMMENT 'global id',
    `branch_id`     BIGINT        NOT NULL COMMENT 'branch id',
    `action_name`   VARCHAR(64)   NOT NULL COMMENT 'action name',
    `status`        TINYINT       NOT NULL COMMENT 'status(tried:1;committed:2;rollbacked:3;suspended:4)',
    `gmt_create`    DATETIME(3)   NOT NULL COMMENT 'create time',
    `gmt_modified`  DATETIME(3)   NOT NULL COMMENT 'update time',
    PRIMARY KEY (`xid`, `branch_id`),
    KEY `idx_gmt_modified` (`gmt_modified`),
    KEY `idx_status` (`status`)
) ENGINE = InnoDB
DEFAULT CHARSET = utf8mb4;
```







> Seata-TCC从2.0.0版本开始提供子事务屏障机制用于解决幂等，空回滚，悬挂等问题。在2.0.0版本之前，需要开发人员在业务代码中自行控制。



