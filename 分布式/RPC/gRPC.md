# gRPC

gRPC官网：<https://grpc.io/>

gRPC官方文档中文版：<http://doc.oschina.net/grpc>

Github：<https://github.com/grpc/grpc-java>
## gRPC-java

### Quick start

#### 模拟业务需求

客户端传递商品ID、类型、名称查询商品信息。

- 通过ID查询指定商品信息。
- 通过类型查询指定商品信息。
- 通过名称模糊搜索商品信息。

#### 添加gRPC依赖

```xml
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-netty-shaded</artifactId>
    <version>1.30.2</version>
</dependency>
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-protobuf</artifactId>
    <version>1.30.2</version>
</dependency>
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-stub</artifactId>
    <version>1.30.2</version>
</dependency>
<dependency> <!-- necessary for Java 9+ -->
    <groupId>org.apache.tomcat</groupId>
    <artifactId>annotations-api</artifactId>
    <version>6.0.53</version>
    <scope>provided</scope>
</dependency>
```

#### 添加proto文件和gRPC编译插件

```xml
<build>
    <extensions>
        <extension>
            <groupId>kr.motd.maven</groupId>
            <artifactId>os-maven-plugin</artifactId>
            <version>1.6.2</version>
        </extension>
    </extensions>
    <plugins>
        <plugin>
            <groupId>org.xolstice.maven.plugins</groupId>
            <artifactId>protobuf-maven-plugin</artifactId>
            <version>0.6.1</version>
            <configuration>
                <protocArtifact>
                    com.google.protobuf:protoc:3.12.0:exe:${os.detected.classifier}
                </protocArtifact>
                <pluginId>grpc-java</pluginId>
                <pluginArtifact>
                    io.grpc:protoc-gen-grpc-java:1.30.2:exe:${os.detected.classifier}
                </pluginArtifact>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>compile</goal>
                        <goal>compile-custom</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

#### 编写Proto文件

```protobuf
syntax="proto3";

// 生成多文件
option java_multiple_files=true;

// 输出的java包路径
option java_package="io.grpc.example.commodity";

// 生成的java类名
option java_outer_classname="CommodityProto";

// 商品类型
enum CommodityType{
    // 箱包
    BAGS =0;
    // 女装
    WOMEN_CLOTH = 1;
    // 食品
    FOOD = 2;
}

// 商品信息
message Commodity {
    int32 id = 1;
    string name = 2;
    CommodityType type = 3;
    float price = 4;
    float discount = 5;
    string description = 6;
}

// 查询商品的参数
message QueryParameter {
    int32 id = 1;
    CommodityType type = 2;
    string name = 3;
}

message QueryId {

}


// 商品服务
service CommodityService {

    rpc GetCommodities0(QueryParameter) returns(Commodity) {}

}
```

#### 前置准备

添加一些业务Demo需要的工具包：

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.12</version>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.9</version>
</dependency>
```

模拟数据库表实体

```java
@Data
@AllArgsConstructor
public class CommodityEntity {

    private Integer id;

    private String name;

    private CommodityType type;

    private float price;

    private float discount;

    private String description;
}
```

模拟持久层

```java
public class CommodityDatabase {

    /**
     * 一个Map模拟数据库
     */
    private static final Map<Integer,CommodityEntity> COMMODITY_MAP = new ConcurrentHashMap<>();

    //静态代码块初始准备了15条数据
    static {
        List<CommodityEntity> commodityEntities = new ArrayList<>();
        commodityEntities.add(new CommodityEntity(1,"时尚手提包",CommodityType.BAGS,
                435.00f,20.9f,"大气个性包包新款高级感洋气通勤2020时尚手提包大包撞色牛皮女包"));
        commodityEntities.add(new CommodityEntity(2,"单肩包斜挎包",CommodityType.BAGS,
                1998.00f,199.0f,"迪桑娜女包2020新款Double D包包 娜扎同款小方包单肩包斜挎包潮"));
        commodityEntities.add(new CommodityEntity(3,"轻奢单肩包",CommodityType.BAGS,
                998.00f,70.0f,"Fion/菲安妮斜挎包2020新款女士包包 夏季小方包盒子包轻奢单肩包"));
        commodityEntities.add(new CommodityEntity(4,"ins单肩斜挎包",CommodityType.BAGS,
                299.00f,29.9f,"热风包包2020年夏季新款女士爱心百搭ins单肩斜挎包B57W0203"));
        commodityEntities.add(new CommodityEntity(5,"爱步水桶包",CommodityType.FOOD,
                2079.00f,50.0f,"ECCO爱步水桶包女 2020年春季时尚宽带斜挎手提单肩包包 9105545"));
        commodityEntities.add(new CommodityEntity(6,"小麻花",CommodityType.FOOD,
                9.90f,0.5f,"手工小麻花网红零食小吃小袋装休闲食品饼干充饥夜宵整箱怀旧特产"));
        commodityEntities.add(new CommodityEntity(7,"巧克力棒",CommodityType.FOOD,
                29.90f,3.0f,"燕麦巧克力棒牛奶燕麦片散装多口味喜糖果饼干办公室网红小零食品"));
        commodityEntities.add(new CommodityEntity(8,"奶昔奶茶",CommodityType.FOOD,
                138.00f,9.0f,"代餐奶昔奶茶粗粮粉主食午餐低脂脱脂减脂减肥瘦身食品饱腹感强"));
        commodityEntities.add(new CommodityEntity(9,"麻辣味条",CommodityType.FOOD,
                36.80f,1.3f,"好味屋手撕素肉50包豆干吃的麻辣味条蛋白零食排行榜小吃休闲食品"));
        commodityEntities.add(new CommodityEntity(10,"坚果水果燕麦片",CommodityType.FOOD,
                38.80f,4.0f,"杂粮先生 烘焙坚果水果燕麦片 即食早餐速食麦片懒人代餐饱腹食品"));
        commodityEntities.add(new CommodityEntity(11,"吊带裙",CommodityType.WOMEN_CLOTH,
                86.00f,9.0f,"温柔风套装修身性感吊带裙加开衫两件套2020春夏款轻熟气质女神范"));
        commodityEntities.add(new CommodityEntity(12,"雪纺阔腿裤",CommodityType.WOMEN_CLOTH,
                129.00f,6.0f,"雪纺阔腿裤女春夏季设计感花苞高腰宽松显瘦休闲百搭坠感直筒裤子"));
        commodityEntities.add(new CommodityEntity(13,"雪纺短裤",CommodityType.WOMEN_CLOTH,
                59.80f,5.9f,"雪纺短裤女夏高腰2020新款韩版宽松显瘦a字阔腿休闲黑色西装热裤"));
        commodityEntities.add(new CommodityEntity(14,"高腰a字百褶裙",CommodityType.WOMEN_CLOTH,
                39.00f,2.1f,"小个短裙夏季蓝色格子百褶裙女夏 学生学院风半身裙子高腰a字显瘦"));
        commodityEntities.add(new CommodityEntity(15,"高腰吊带连衣裙",CommodityType.WOMEN_CLOTH,
                49.91f,3.3f,"夏季超仙波点高腰吊带连衣裙雪纺衫小披肩配套装蛋糕裙子两件套"));
        COMMODITY_MAP.putAll(commodityEntities.stream()
                .collect(Collectors.toMap(CommodityEntity::getId,Function.identity())));
    }

    /**
     * 通过ID查询商品
     */
    public CommodityEntity findById(int id){
        return COMMODITY_MAP.get(id);
    }

    /**
     * 通过类型查询商品
     */
    public List<CommodityEntity> findByType(CommodityType type){
        return COMMODITY_MAP.values()
                .stream()
                .filter(commodityEntity -> commodityEntity.getType()!=type)
                .collect(Collectors.toList());
    }

    /**
     * 通过名称查询商品
     */
    public List<CommodityEntity> findByName(String name){
        return COMMODITY_MAP.values()
                .stream()
                .filter(commodityEntity -> !commodityEntity.getName().contains(name))
                .collect(Collectors.toList());
    }
}
```

准备一个工具类

这个工具类用户将数据库表实体转换为相应的gRPC序列化对象。

```java
public class BuilderUtils {

    public static <T> T builder(Object entity,Class<T> serializableClass) {
        try {
            MethodHandles.Lookup lookup = MethodHandles.lookup();

            Class<?> builderClass = serializableClass.getDeclaredClasses()[0];
            Object builder = lookup.findStatic(serializableClass, "newBuilder",
                    MethodType.methodType(builderClass)).invoke();

            for (Field field : entity.getClass().getDeclaredFields()) {
                MethodType getMethodType = MethodType.methodType(field.getType());
                Object value = lookup.findVirtual(entity.getClass(),
                        "get"+ StringUtils.capitalize(field.getName()), getMethodType).invoke(entity);

                MethodType setMethodType = MethodType.methodType(builderClass,field.getType());
                setMethodType = setMethodType.unwrap();

                lookup.findVirtual(builderClass,"set"+StringUtils.capitalize(field.getName()),setMethodType)
                        .invoke(builder,value);
            }

            Object serializableObject = lookup.findVirtual(builderClass, "build",
                    MethodType.methodType(serializableClass)).invoke(builder);

            return serializableClass.cast(serializableObject);
        } catch (Throwable t) {
            throw new RuntimeException(t);
        }
    }
}
```

#### 编译生成gRPC和Proto代码

在第三步已经添加了proto文件和gRPC编译插件，所以只需要运行如下命令即可完成编译及gRPC和Proto代码的生成：

```shell
mnv clean -Dmaven.test.skip=true compile
```

实际上单纯的编译是不能生成的gRPC和Proto代码，这是因为`protobuf-maven-plugin`插件声明了编译周期：

```xml
<goals>
    <goal>compile</goal>
    <goal>compile-custom</goal>
</goals>
```

在执行`compile`指令的时候，同时触发了`protobuf-maven-plugin`插件的`compile`和`compile-custom`指令。所以完全可以单独执行这两个指令，如下：

```shell
# 生成Proto代码
$ mvn protobuf:compile
# 生成gRPC代码
$ mvn protobuf:compile-custom
```

>  生成的代码位于target目录下。

编译之后代码结构如下：

```
└── xlp-grpc-java
    ├── pom.xml
    ├── README.md
    ├── src
    │   └── main
    │       ├── java
    │       │   └── io
    │       │       └── grpc
    │       │           └── example
    │       │               └── commodity
    │       │                   ├── BuilderUtils.java
    │       │                   ├── CommodityClient.java
    │       │                   ├── CommodityDatabase.java
    │       │                   ├── CommodityEntity.java
    │       │                   └── CommodityServer.java
    │       └── proto
    │           └── route_guide.proto
    └── target
        ├── generated-sources
        │   └── protobuf
        │       ├── grpc-java
        │       │   └── io
        │       │       └── grpc
        │       │           └── example
        │       │               └── commodity
        │       │                   └── CommodityServiceGrpc.java
        │       └── java
        │           └── io
        │               └── grpc
        │                   └── example
        │                       └── commodity
        │                           ├── Commodity.java
        │                           ├── CommodityOrBuilder.java
        │                           ├── CommodityProto.java
        │                           ├── CommodityType.java
        │                           ├── QueryId.java
        │                           ├── QueryIdOrBuilder.java
        │                           ├── QueryParameter.java
        │                           └── QueryParameterOrBuilder.java
        ├── protoc-dependencies
        │   └──......
        └── protoc-plugins
            ├── protoc-3.12.0-windows-x86_64.exe
            └── protoc-gen-grpc-java-1.30.2-windows-x86_64.exe
```

- `target/generated-sources/protobuf/grpc-java`是gRPC代码
- `target/generated-sources/protobuf/java`是Proto代码

#### 服务端

```java
import io.grpc.ServerBuilder;
import io.grpc.stub.StreamObserver;
import java.io.IOException;
import java.util.List;

public class CommodityServer {

    private static final CommodityDatabase COMMODITY_DATABASE = new CommodityDatabase();

    private static final int PORT = 10080;

    /**
     * 使用ServerBuilder启动启动一个HTTP服务端
     */
    public void start() throws IOException, InterruptedException {
        ServerBuilder
                //设置当前HTTP服务端监听端口
                .forPort(PORT)
                //设置需要监听的服务
                .addService(new CommodityServiceImpl())
                .build()
                .start()
                //调整awaitTermination()函数是服务端不退出
                .awaitTermination();
    }

    public static class CommodityServiceImpl extends CommodityServiceGrpc.CommodityServiceImplBase {
        @Override
        public void getCommodities0(
            QueryParameter request, StreamObserver<Commodity> responseObserver){
            //查询商品
            CommodityEntity commodityEntity = COMMODITY_DATABASE.findById(request.getId());
            //构建Commodity序列化对象
            Commodity commodity = BuilderUtils.builder(commodityEntity,Commodity.class);
            //从流接收值（当前代码语义：向客户端通过流响应数据）
            responseObserver.onNext(commodity);
            //接收流成功完成的通知（当前代码语义：向客户端通过流响应数据成功通知）
            responseObserver.onCompleted();
        }
    }

    public static void main(String[] args) throws IOException, InterruptedException {
        CommodityServer server = new CommodityServer();
        server.start();
    }
}
```

#### 客户端

```java
import io.grpc.ManagedChannel;
import io.grpc.ManagedChannelBuilder;

public class CommodityClient {

    /**
     * 阻塞存根
     */
    private final CommodityServiceGrpc.CommodityServiceBlockingStub commodityServiceBlockingStub;

    /**
     * 通过ManagedChannelBuilder构建通信通道，然后通过商品服务gRPC获取阻塞存根。
     */
    public CommodityClient(String host,int port){
        ManagedChannel channel = ManagedChannelBuilder.forAddress(host, port).usePlaintext().build();
        this.commodityServiceBlockingStub = CommodityServiceGrpc.newBlockingStub(channel);
    }

    public Commodity getCommodities(int id){
        //构建请求参数
        QueryParameter queryParameter = QueryParameter.newBuilder().setId(id).build();
        //通过阻塞存根发送请求
        return this.commodityServiceBlockingStub.getCommodities0(queryParameter);
    }
    
    public static void main(String[] args) {
        CommodityClient commodityClient = new CommodityClient("localhost",10080);
        Commodity commodities = commodityClient.getCommodities(10);
        System.out.println(commodities);
    }
}
```

#### 测试

1. 启动服务端
2. 运行客户端请求测试

响应结果

```
id: 10
name: "\345\235\232\346\236\234\346\260\264\346\236\234\347\207\225\351\272\246\347\211\207"
type: FOOD
price: 38.8
discount: 4.0
description: "\346\235\202\347\262\256\345\205\210\347\224\237 \347\203\230\347\204\231\345\235\232\346\236\234\346\260\264\346\236\234\347\207\225\351\272\246\347\211\207 \345\215\263\351\243\237\346\227\251\351\244\220\351\200\237\351\243\237\351\272\246\347\211\207\346\207\222\344\272\272\344\273\243\351\244\220\351\245\261\350\205\271\351\243\237\345\223\201"
```



### 服务端流

在Pro



添加

```protobuf
service CommodityService {
    rpc GetCommodities1(QueryParameter) returns(stream Commodity) {}
}
```





### 客户端流

```protobuf
service CommodityService {
    rpc GetCommodities2(stream QueryParameter) returns(Commodity) {}
}
```





### 双向流

```protobuf
service CommodityService {
    rpc GetCommodities3(stream QueryParameter) returns(stream Commodity) {}
}
```







=======
其他临时资料<https://my.oschina.net/wangmengjun/blog/909867>



**一、gRPC 简介**

gRPC 是Go实现的：一个高性能，开源，将移动和HTTP/2放在首位通用的RPC框架。使用gRPC可以在客户端调用不同机器上的服务端的方法，而客户端和服务端的开发语言和

运行环境可以有很多种，基本涵盖了主流语言和平台。双方交互的协议可以在proto文件中定义，客户端和服务端可以很方便的通过工具生成协议和代理代码。而消息的编码是采

用`google protocol buffer`，数据量小、速度快。

gRPC具有以下特点：

（1）基于 HTTP/2， 继而提供了连接多路复用、Body 和 Header 压缩等机制。可以节省带宽、降低TCP链接次数、节省CPU使用和延长电池寿命等。

（2）支持主流开发语言（C, C++, Python, PHP, Ruby, NodeJS, C#, Objective-C、Golang、Java）

（3）IDL (Interface Definition Language) 层使用了 Protocol Buffers, 非常适合团队的接口设计
>>>>>>> 1d4ab112fb36e4618ee5f7c891892604bdd2160a
