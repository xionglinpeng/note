# Seata AT

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
    >
    > 如果没有启动Seata Server，将报错：can not register RM,err:can not connect to services-server。
    
5. 启用数据源代理

   在每一个分支事务服务都需要启用数据源代理。

   ```java
   @EnableAutoDataSourceProxy
   ```

6. 创建undo_log

   在每一个分支事务服务都需要创建undo_log表。

   ```sql
   -- for AT mode you must to init this sql for you business database. the seata server not need it.
   CREATE TABLE IF NOT EXISTS `undo_log`
   (
       `branch_id`     BIGINT       NOT NULL COMMENT 'branch transaction id',
       `xid`           VARCHAR(128) NOT NULL COMMENT 'global transaction id',
       `context`       VARCHAR(128) NOT NULL COMMENT 'undo_log context,such as serialization',
       `rollback_info` LONGBLOB     NOT NULL COMMENT 'rollback info',
       `log_status`    INT(11)      NOT NULL COMMENT '0:normal status,1:defense status',
       `log_created`   DATETIME(6)  NOT NULL COMMENT 'create datetime',
       `log_modified`  DATETIME(6)  NOT NULL COMMENT 'modify datetime',
       UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
   ) ENGINE = InnoDB
     AUTO_INCREMENT = 1
     DEFAULT CHARSET = utf8mb4 COMMENT ='AT transaction mode undo table';
   ```


## 实现案例

### 开启全局事务

在调用方添加`@GlobalTransactional`注解开启全局事务。

调用方也是分支事务之一，当前调用方是分支事务Order。

```java
@Override
@Transactional
@GlobalTransactional
public void createOrder(OrderDTO orderDTO) {
    Order order = new Order();
    BeanUtils.copyProperties(orderDTO, order);
    order.insert();
    accountFeign.deductAmount(1L, BigDecimal.valueOf(orderDTO.getValue()));
    integralFeign.addIntegral(1L, orderDTO.getValue());
}
```

### 分支事务Account

分支事务Account属于非调用方，它不需要做任何额外的工作，只需要专注完成业务即可。

```java
@Override
@Transactional
public void deductAmount(Long userId, BigDecimal amount) {
    Account account = lambdaQuery().eq(Account::getUserId, userId).one();
    account.setBalance(account.getBalance().subtract(amount));
    account.updateById();
}
```

### 分支事务Integral

分支事务Account属于非调用方，它不需要做任何额外的工作，只需要专注完成业务即可。

```java
@Override
@Transactional
public void addIntegral(Long userId, Integer grade) {
    Integral integral = lambdaQuery().eq(Integral::getUserId, userId).one();
    integral.setGrade(integral.getGrade() + grade);
    integral.updateById();
}
```

### Feign接口

Feign接口就是普通的Feign接口，不需要做额外的工作。

- 调用账户服务的Feign接口：

    ```java
    @FeignClient("sadness-tcc-account")
    @RequestMapping("/account")
    public interface AccountFeign {
        @PutMapping("/deduct/{userId}/{amount}")
        void deductAmount(@PathVariable("userId") Long userId, @PathVariable("amount") BigDecimal amount);
    }
    ```
- 调用积分服务的Feign接口：

  ```java
  @FeignClient("sadness-tcc-integral")
  @RequestMapping("/integral")
  public interface IntegralFeign {
      @PutMapping("/{userId}/{grade}")
      void addIntegral(@PathVariable("userId") Long userId, @PathVariable("grade") Integer grade);
  }
  ```

  