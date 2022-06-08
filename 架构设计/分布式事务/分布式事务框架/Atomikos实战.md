# Atomikos实战

## 1. 添加依赖

为了运行Atomikos，必须需要添加一些依赖的jar包。如下所示：

```xml
<dependency>
    <groupId>com.atomikos</groupId>
    <artifactId>transactions-jdbc</artifactId>
    <version>5.0.9</version>
</dependency>
<dependency>
    <groupId>com.atomikos</groupId>
    <artifactId>transactions-jta</artifactId>
    <version>5.0.9</version>
</dependency>
<dependency>
    <groupId>javax.transaction</groupId>
    <artifactId>jta</artifactId>
    <version>1.1</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.22</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
    <version>8.0.11</version>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

我们添加了5个jar包，分别是：

- `com.atomikos:transactions-jdbc:5.0.9`
- `com.atomikos:transactions-jta:5.0.9`
- `javax.transaction:jta:1.1`
- `com.alibaba:druid:1.1.22`
- `mysql:mysql-connector-java:8.0.11`

> Note
>
> 1. JTA的API并没有位于JDK中，因此需要添加`javax.transaction:jta:1.1`jar包。
>
> 2. 由于MySQL驱动与Atomikos的版本兼容性问题，因此指定MySQL驱动版本为8.0.11。
>
>    如果使用大于8.0.11版本的MySQL驱动，将会报错：`java.lang.NoSuchMethodException: com.mysql.cj.conf.PropertySet.getBooleanReadableProperty(java.lang.String)`。
>
> 3. 数据源没有选用[HikariCP](https://github.com/brettwooldridge/HikariCP)，虽然HikariCP是Spring Boot推荐的默认数据源，但是它不支持XA数据源。
>
>    > 🔤`dataSourceClassName`
>    >
>    > This is the name of the `DataSource` class provided by the JDBC driver. Consult the documentation for your specific JDBC driver to get this class name, or see the [table](https://github.com/brettwooldridge/HikariCP#popular-datasource-class-names) below. Note XA data sources are not supported. XA requires a real transaction manager like [bitronix](https://github.com/bitronix/btm). Note that you do not need this property if you are using `jdbcUrl` for "old-school" DriverManager-based JDBC driver configuration. *Default: none*

## 2. 准备数据表和数据

下单扣库存业务操作，因此准备了两张表：库存-inventory和订单-orders。

```sql
-- inventory
CREATE DATABASE DEMO_INVENTORY;
USE DEMO_INVENTORY;
CREATE TABLE `inventory` (
  `product_id` BIGINT NOT NULL,
  `balance` INT DEFAULT NULL,
  PRIMARY KEY (`product_id`)
);
INSERT INTO `inventory` VALUES (1000,500);

-- orders
CREATE DATABASE DEMO_ORDERS;
USE DEMO_ORDERS;
CREATE TABLE `orders` (
  `order_id` BIGINT NOT NULL,
  `product_id` BIGINT DEFAULT NULL,
  `amount` INT NOT NULL,
  PRIMARY KEY (`order_id`),
  CONSTRAINT `orders_chk_1` CHECK ((`amount` <= 5))
);
```

注意，我们约束了订单的产品数量amount不能大于5.

## 3. 配置Atomikos多数据源

库存-inventory和订单-orders分别位于两个不同的数据库，因此需要分别为其配置数据源。

配置如下：

```properties
spring.datasource.inventory.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.inventory.name=defaultDataSource
spring.datasource.inventory.url=jdbc:mysql://localhost:3306/DEMO_INVENTORY?serverTimezone=UTC
spring.datasource.inventory.username=root
spring.datasource.inventory.password=123456

spring.datasource.orders.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.orders.name=defaultDataSource
spring.datasource.orders.url=jdbc:mysql://localhost:3306/DEMO_ORDERS?serverTimezone=UTC
spring.datasource.orders.username=root
spring.datasource.orders.password=123456
```

数据源使用Druid的`DruidXADataSource`，然后将其包装为Atomikos的`AtomikosDataSourceBean`。

代码如下：

```java
import com.alibaba.druid.pool.xa.DruidXADataSource;
import com.atomikos.icatch.jta.UserTransactionManager;
import com.atomikos.jdbc.AtomikosDataSourceBean;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import javax.transaction.SystemException;

@Configuration
public class AtomikosConfiguration {

    @Bean(name = "inventoryAds", initMethod = "init", destroyMethod = "close")
    public AtomikosDataSourceBean inventoryAtomikosDataSourceBean() {
        AtomikosDataSourceBean dataSource = new AtomikosDataSourceBean();
        dataSource.setUniqueResourceName("inventory");
        dataSource.setXaDataSource(inventoryDruidXADataSource());
        return dataSource;
    }

    @Bean(name = "ordersAds", initMethod = "init", destroyMethod = "close")
    public AtomikosDataSourceBean ordersAtomikosDataSourceBean() {
        AtomikosDataSourceBean dataSource = new AtomikosDataSourceBean();
        dataSource.setUniqueResourceName("orders");
        dataSource.setXaDataSource(ordersDruidXADataSource());
        return dataSource;
    }

    @Bean
    @ConfigurationProperties("spring.datasource.inventory")
    public DruidXADataSource inventoryDruidXADataSource() {
        return new DruidXADataSource();
    }

    @Bean
    @ConfigurationProperties("spring.datasource.orders")
    public DruidXADataSource ordersDruidXADataSource() {
        return new DruidXADataSource();
    }
}
```

## 4. 添加业务代码

定义`IOrderService`接口，接口中定义了两个抽象方法。分别为下单扣库存（`placeOrder`）和获取指定产生的库存数量（`getInventoryBalance`），其中`getInventoryBalance`用于单元测试。

```java
import javax.transaction.SystemException;
public interface IOrderService {
    //下单扣库存
    void placeOrder(Long productId, int amount) throws Exception;
    //获取指定产生的库存数量
    int getInventoryBalance(Long productId) throws Exception;
}
```

IOrderService实现：

```java
import com.atomikos.demo.service.IOrderService;
import com.atomikos.icatch.jta.UserTransactionImp;
import org.springframework.stereotype.Service;
import javax.annotation.Resource;
import javax.sql.DataSource;
import javax.transaction.SystemException;
import java.sql.Connection;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Random;

@Service
public class OrderServiceImpl implements IOrderService {

    @Resource(name = "inventoryAds")
    private DataSource inventoryAds;

    @Resource(name = "ordersAds")
    private DataSource ordersAds;

    @Override
    public int getInventoryBalance(Long productId) throws Exception {
        try (Connection connection = inventoryAds.getConnection();
             Statement statement = connection.createStatement();) {
            String sql = "select balance from inventory where product_id ='" + productId + "'";
            ResultSet resultSet = statement.executeQuery(sql);
            resultSet.next();
            return resultSet.getInt(1);
        }
    }
    
    @Override
    public void placeOrder(Long productId, int amount) throws Exception {
        UserTransactionImp utx = new UserTransactionImp();
        try {
            utx.begin();
            placeOrderForInventory(productId, amount);
            placeOrderForOrder(productId, amount);
            utx.commit();
        } catch (Exception e) {
            e.printStackTrace();
            utx.rollback();
        }
    }

    public void placeOrderForInventory(Long productId, int amount) throws Exception {
        try (Connection connection = inventoryAds.getConnection();
             Statement statement = connection.createStatement();) {
            String sql = "update inventory set balance = balance - " + amount + " where product_id ='" + productId + "'";
            statement.executeUpdate(sql);
        }
    }

    private void placeOrderForOrder(Long productId, int amount) throws SQLException {
        long orderId = new Random().nextLong();
        try (Connection connection = ordersAds.getConnection();
             Statement statement = connection.createStatement()) {
            String sql = "insert into orders values ( '" + orderId + "', '" + productId + "', " + amount + " )";
            statement.executeUpdate(sql);
        }
    }
}

```

## 5. 测试

定义了两个单元测试方法，一个测试成功，一个测试失败。测试失败时通过订单的产品数量amount不能大于5的约束。

```java
@SpringBootTest
public class IOrderServiceTest {

    @Autowired
    private IOrderService orderService;

    @Test
    void placeOrderByFailure() throws Exception {
        long productId = 1000;
        int oldBalance = orderService.getInventoryBalance(productId);
        orderService.placeOrder(productId, 10);
        int newBalance = orderService.getInventoryBalance(productId);
        assert oldBalance == newBalance;
    }

    @Test
    void placeOrderBySuccess() throws Exception {
        long productId = 1000;
        int oldBalance = orderService.getInventoryBalance(productId);
        orderService.placeOrder(productId, 5);
        int newBalance = orderService.getInventoryBalance(productId);
        assert oldBalance != newBalance;
    }
}

```

## 6. 集成Spring事务管理器

上面的业务代码，在placeOrder方法中，我们是通过Atomikos的`UserTransactionImp`来管理事务。现在我要将其切换为通过Spring的事务管理器来管理事务。

首先在`AtomikosConfiguration`配置类中配置事务管理器Bean，代码如下：

```java
@Configuration
public class AtomikosConfiguration {
    
    ......
    
    @Bean(initMethod = "init", destroyMethod = "close")
    public UserTransactionManager userTransactionManager() throws SystemException {
        UserTransactionManager tm = new UserTransactionManager();
        tm.setTransactionTimeout(300);
        tm.setForceShutdown(true);
        return tm;
    }

    @Bean
    public JtaTransactionManager jtaTransactionManager() throws SystemException {
        JtaTransactionManager tm = new JtaTransactionManager();
        tm.setTransactionManager(userTransactionManager());
        tm.setUserTransaction(userTransactionManager());
        return tm;
    }   
}
```

注意，`JtaTransactionManager`位于spring-jdbc，因此需要保证在classpath下具有此jar包。

然后我们对`placeOrder`方法稍作改动，通过`@Transactional`注解来管理事务。代码如下：

```java
@Override
@Transactional(rollbackFor = Exception.class)
public void placeOrder(Long productId, int amount) throws Exception {
    placeOrderForInventory(productId, amount);
    placeOrderForOrder(productId, amount);
}
```

> `@Transactional`注解必须声明`rollbackFor`属性为`Exception.class`，这是因为`@Transactional`注解默认情况下只会捕获`RuntimeException`异常。

最后，如果使用的是Spring Boot，为了避免JTA自动装配的影响，我们禁用JTA自动装配。配置如下属性：

```properties
spring.jta.enabled=false
```

## 7. 集成Spring Boot

Spring Boot提供了对JTA的自动装配，其中就支持Atommikos。因此使用JTA自动装配，可以简化很多配置。

在前面*集成Spring事务管理器*的基础上，移除以下两部分即可，它就会应用JTA自动装配。

1. 移除配置文件中的`spring.jta.enabled=false`。
2. 移除AtomikosConfiguration配置类中的`JtaTransactionManager`和`UserTransactionManager`。

可以将对Atomikos的依赖替换为spring-boot-starter（这不是必须的）：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jta-atomikos</artifactId>
</dependency>
```

## 8. 集成Mybatis

真实的企业级项目中，我们不太可能去手动管理数据源，一般会使用一个ORM框架，例如Hibernate和Mybatis。下面我们将集成Mybatis和Atomikos。

首先添加Mybatis依赖。

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.2</version>
</dependency>
```

然后创建表实体以及对应的Mapper接口

表实体 - Inventory and Orders：

```java
@Data
public class Inventory {
    private Long productId;
    private Integer balance;
}

@Data
public class Orders {
    private Long orderId;
    private Long productId;
    private Integer amount;
}
```

Mapper接口：InventoryMapper

```java
import com.atomikos.demo.entity.Inventory;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Update;
public interface InventoryMapper {
    @Select("select product_id as productId, balance from inventory where product_id=#{productId}")
    Inventory selectById(Long productId);

    @Update("update inventory set balance=#{balance} where product_id=#{productId}")
    Integer updateById(Inventory inventory);
}
```

Mapper接口：OrdersMapper

```java
import com.atomikos.demo.entity.Orders;
import org.apache.ibatis.annotations.Insert;
public interface OrdersMapper {
    @Insert("insert into orders values (#{orderId},#{productId} ,#{amount})")
    Integer insert(Orders orders);
}
```

注意，InventoryMapper和OrdersMapper需要放置在不同的包中，这是为了在使用@MapperScan进行扫描时应用不同的SqlSession配置。

下面我们将配置InventoryMapper和OrdersMapper接口的数据源和对应的SqlSession配置。

Inventory SQLSession Configuration：

```java
import com.alibaba.druid.pool.xa.DruidXADataSource;
import com.atomikos.jdbc.AtomikosDataSourceBean;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@MapperScan(basePackages = "com.atomikos.demo.mapper.inventory", sqlSessionTemplateRef = "inventorySqlSessionTemplate")
public class InventorySQLSessionConfiguration {

    @Bean
    @ConfigurationProperties("spring.datasource.inventory")
    public DruidXADataSource inventoryDruidXADataSource() {
        return new DruidXADataSource();
    }

    @Bean(name = "inventoryAds", initMethod = "init", destroyMethod = "close")
    public AtomikosDataSourceBean inventoryAtomikosDataSourceBean() {
        AtomikosDataSourceBean dataSource = new AtomikosDataSourceBean();
        dataSource.setUniqueResourceName("inventory");
        dataSource.setXaDataSource(inventoryDruidXADataSource());
        return dataSource;
    }

    @Bean("inventorySqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory() throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(inventoryAtomikosDataSourceBean());
        return sqlSessionFactoryBean.getObject();
    }

    @Bean("inventorySqlSessionTemplate")
    public SqlSessionTemplate sqlSessionTemplate() throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory());
    }
}
```

Orders SQLSession Configuration：

```java
import com.alibaba.druid.pool.xa.DruidXADataSource;
import com.atomikos.jdbc.AtomikosDataSourceBean;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@MapperScan(basePackages = "com.atomikos.demo.mapper.orders",sqlSessionTemplateRef = "ordersSqlSessionTemplate")
public class OrdersSQLSessionConfiguration {

    @Bean
    @ConfigurationProperties("spring.datasource.orders")
    public DruidXADataSource ordersDruidXADataSource() {
        return new DruidXADataSource();
    }

    @Bean(name = "ordersAds", initMethod = "init", destroyMethod = "close")
    public AtomikosDataSourceBean ordersAtomikosDataSourceBean() {
        AtomikosDataSourceBean dataSource = new AtomikosDataSourceBean();
        dataSource.setUniqueResourceName("orders");
        dataSource.setXaDataSource(ordersDruidXADataSource());
        return dataSource;
    }

    @Bean("ordersSqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory() throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(ordersAtomikosDataSourceBean());
        return sqlSessionFactoryBean.getObject();
    }

    @Bean("ordersSqlSessionTemplate")
    public SqlSessionTemplate sqlSessionTemplate() throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory());
    }
}
```

最后一步就是修改业务代码，应用我们定义的Mapper接口。

```java
@Service
public class OrderServiceImpl implements IOrderService {

    @Autowired
    private InventoryMapper inventoryMapper;
    @Autowired
    private OrdersMapper ordersMapper;

    @Override
    public int getInventoryBalance(Long productId) throws SQLException {
        return inventoryMapper.selectById(productId).getBalance();
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void placeOrder(Long productId, int amount) throws Exception {
        Inventory inventory = inventoryMapper.selectById(productId);
        inventory.setBalance(inventory.getBalance() - amount);
        inventoryMapper.updateById(inventory);

        Orders orders = new Orders();
        orders.setOrderId(new Random().nextLong());
        orders.setProductId(productId);
        orders.setAmount(amount);
        ordersMapper.insert(orders);
    }
}
```

## 9. 集成Mybatis-Plus

Mybatis-Plus本身只是对Mybatis的增强，其底层仍然是依赖Mybatis的。因此集成Mybatis-Plus很简单，我们只需要将mybatis的`org.mybatis.spring.SqlSessionFactoryBean`替换成Mybatis-Plus的`com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean`即可，然后就可以使用Mybatis-Plus提供的API了。

首先添加Mybatis-Plus依赖。

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.2</version>
</dependency>
```

然后替换SqlSessionFactoryBean。

```java
@Configuration
@MapperScan(basePackages = "com.atomikos.demo.mapper.orders",sqlSessionTemplateRef = "ordersSqlSessionTemplate")
public class OrdersSQLSessionConfiguration {

	......

    @Bean("ordersSqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory() throws Exception {
        MybatisSqlSessionFactoryBean sqlSessionFactoryBean = new MybatisSqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(ordersAtomikosDataSourceBean());
        return sqlSessionFactoryBean.getObject();
    }

    ......
}


@Configuration
@MapperScan(basePackages = "com.atomikos.demo.mapper.inventory", sqlSessionTemplateRef = "inventorySqlSessionTemplate")
public class InventorySQLSessionConfiguration {

	......

    @Bean("inventorySqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory() throws Exception {
        MybatisSqlSessionFactoryBean sqlSessionFactoryBean = new MybatisSqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(inventoryAtomikosDataSourceBean());
        return sqlSessionFactoryBean.getObject();
    }

	......
}
```

