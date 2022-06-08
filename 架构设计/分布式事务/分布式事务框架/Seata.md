# Seata

[TOC]

> Seata version 1.5.0

## 简介

<span alt="emp"></span>

alt="wavy"

alt="underline"

alt="hide"

<del>

<u>

## 部署

Seata Server支持多种方式部署：直接部署, Docker, Docker-Compose, Kubernetes, Helm。

### 直接部署

1. 下载[启动包](https://github.com/seata/seata/releases)并解压。

2. 命令行启动

   进入到seata\bin目录，

   Linux环境
   ```shell
   $ seata-server.sh -h 127.0.0.1 -p 8091 -m file -n 1 -e dev
   ```
   windows环境

   ```shell
   $ seata-server.bat -h 127.0.0.1 -p 8091 -m file -n 1 -e dev
   ```

3. 如果启动模式为db，则创建表并修改存储db参数。

4. 如果启动模式为redis，则创建表并修改存储redis参数。

Seata Server启动参数

| 简写 | 全写         | 作用                           | 描述                                                         |
| ---- | ------------ | ------------------------------ | ------------------------------------------------------------ |
| -h   | --host       | 指定注册到注册中心的IP。       | 注册到注册中心的IP。                                         |
| -p   | --port       | 指定Server RPC监听的端口。     | Server RPC监听的端口，默认为8091。                           |
| -m   | --storeMode  | 指定全局事务会话信息存储模式。 | 全局事务会话信息存储模式，支持<br/>file，db和redis，默认为file。 |
| -n   | --serverNode | 指定Seata-server节点编号。     | Seata-server节点，多个Server时，<br>需要区分各自节点，用于生成不同<br/>区间的transactionId，以免冲突。<br/>如 `1`,`2`,`3`..., 默认为 `1`。 |
| -e   | --seataEnv   | 指定Seata-server运行环境。     | 如 `dev`, `test` 等, 服务启动时会使用<br/> `registry-{env}.conf` 这样的配置。 |

### Docker部署

#### 快速启动

```shell
$ docker run --name seata-server -p 8091:8091 -p 7091:7091 seataio/seata-server
```

容器命令行

```shell
$ docker exec -it seata-server sh
```

查看日志

```shell
$ docker logs -f seata-server
```

#### 自定义配置文件

seata-server的配置文件位于容器`/seata-server/resources/application.yml`，只需要自定义application.yml配置文件，并配置文件映射即可。

命令如下：

```shell
		$ docker run --name seata-server \
		-p 8091:8091 \
		-p 7091:7091 \
		--restart always  \
		--memory 300M \
		--privileged \
		-d \
		-v /PATH/TO/application.yml:/seata-server/resources/application.yml \
		seataio/seata-server:latest
```

#### 环境变量	

- **SEATA_IP**

  可选，指定seata-server启动的IP，该IP用于向注册中心注册时使用，如eureka等。

- **SEATA_PORT**

  可选，指定seata-server启动的端口，默认为8091。

- **STORE_MODE**

  可选，指定seata-server的事务日志存储方式，支持db、file、redis。

- **SERVER_NODE**

  可选，用于指定seata-server节点ID，如1、2、3、......，默认为 - 根据IP生成。

- **SEATA_ENV**

  可选，指定seata-server运行环境，如dev、test、prod等。

- **SEATA_CONFIG_NAME**

  

### Docker compose部署

创建docker-compose.yaml文件：

```yaml
version: "3"
services:
  seata-server:
    image: seataio/seata-server
    hostname: seata-server
    container_name: seata-server
    ports:
      - "8091:8091"
      - "7091:7091"
    restart: always
    environment:
      - SEATA_PORT=8091
      - STORE_MODE=file
    privileged: true
    volumes:
      - /PATH/TO/application.yml:/seata-server/resources/application.yml
```

docker-compose命令启动：

```shell
$ docker-compose -f seata/docker-compose.yaml up -d
```

### Kubernates部署

### Helm部署

### 高可用部署

## 配置中心

如果采用了配置中心进行Seata-Server的配置，那么将只有配置中心的配置生效，如果配置中心没有配置，则将采用默认值（即使进行了本地配置）。

### Nacos配置中心

```yaml
seata:
  config:
    # support: nacos, consul, apollo, zk, etcd3
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8848
      namespace:
      group: SEATA_GROUP
      username: nacos
      password: nacos
      ##if use MSE Nacos with auth, mutex with username/password attribute
      #access-key: ""
      #secret-key: ""
      data-id: seataServer.properties
```



#### 通过脚本上传配置到Nacos

Seata将所有的配置放置在一个名为[config.txt](https://github.com/seata/seata/blob/develop/script/config-center/config.txt)的文本文件中。有两种方式可以将其上传到nacos配置中心。

1. 方式一：在nacos配置中心创建名为`seataServer.properties`的**Data ID**，然后将需要的配置从config.txt文本文件中拷贝到**Data ID**中，并修改为自己所需要的配置值。

2. 方式二：Seata提供了shell脚本可以将config.txt文本文件中的所有配置上传到nacos配置中心。

   通过shell脚本上传的方式有两种，分别是交互模式和命令模式。

   - 交互模式：交互模式使用脚本[nacos-confg-interactive.sh](https://github.com/seata/seata/blob/develop/script/config-center/nacos/nacos-confg-interactive.sh)。

     命令如下：
   
     ```shell
      $ sh nacos-confg-interactive.sh
     Please enter the host of nacos.
     请输入nacos的host [localhost]:
     >>> 
     Please enter the port of nacos.
     请输入nacos的port [8848]:
     >>> 
     Please enter the group of nacos.
     请输入nacos的group [SEATA_GROUP]:
     >>> 
     Please enter the tenant of nacos.
     请输入nacos的tenant:
     >>> 
     Please enter the username of nacos.
     请输入nacos的username:
     >>> nacos
     Please enter the password of nacos.
     请输入nacos的password:
     >>> nacos
     Are you sure to continue? [y/n]y
     ```

   - 命令模式：命令模式使用脚本[nacos-config.sh](https://github.com/seata/seata/blob/develop/script/config-center/nacos/nacos-config.sh)。

      命令如下：

      ```shell
$ nacos-config.sh -h localhost -p 8848 -g SEATA_GROUP -t 5a3c7d6c-f497-4d68-a71a-2e5e3340b3ca -u username -w password
      ```
   
      参数：
   
      - **-h**：主机，默认值是localhost。
      - **-p**：端口，默认值是8848。
      - **-g**：配置组，默认值是SEATA_GROUP。
      - **-t**：租户信息，对应Nacos的namespace ID字段，默认值为''。
      - **-u**：用户名，在nacos 1.2.0+上的权限控制，默认值是''。
      - **-w**：密码，在nacos 1.2.0+上的权限控制，默认值是''。
   
   > <font>IMPORTENT</font>
   >
   > 1、config.txt文件必须放在脚本的上级目录。例如，脚本所在目录为/root/seata/script/nacos-config.sh，那么config.txt文件必须位于/root/seata/config.txt。
   >
   > 
   >
   > 2、如果提示错误==$'\r': 未找到命令==，则表示shell脚本的文件格式不正确。例如：
   >
   > ```SHELL
   > $ sh nacos-config.sh -h 192.168.56.3 -p 8848 -g SEATA_GROUP -t 5a3c7d6c-f497-4d68-a71a-2e5e3340b3ca -u nacos -w nacos
   > nacos-config.sh:行15: $'\r': 未找到命令
   > nacos-config.sh:行18: 未预期的符号 `$'in\r'' 附近有语法错误
   > 'acos-config.sh:行18: `  case $opt in
   > ```
   >
   > 通过vim命令编辑shell脚本，然后通过`:set ff`指令查看编码格式，如果`ff`的值为`dos`，则表示文件格式为windows系统上的文件格式，通过指令`:set ff=unix`设置为unix系统的上的文件格式，最后`:wq`保存退出。
   >
   > 
   >
   > 3、使用脚本上传config,txt文件配置时，其配置的值必须不为空，例如：
   >
   > ```properties
   > store.mode=
   > ```
   >
   > 属性store.mode没有值，是不合法的。如果存在属性没有值，则将会报。例如：
   >
   > ```
   > nacos-config.sh: 第 88 行:[: 参数太多
   > Set store.publicKey= failure 
   > ...
   > nacos-config.sh: 第 88 行:[: 参数太多
   > Set store.redis.sentinel.masterName= failure 
   > nacos-config.sh: 第 88 行:[: 参数太多
   > Set store.redis.sentinel.sentinelHosts= failure 
   > ...
   > nacos-config.sh: 第 88 行:[: 参数太多
   > Set store.redis.password= failure
   > ```
   >
   > 解决方式是为其设置值或者将其移除即可。





### Apollo配置中心

### Etcd3配置中心

### Consul配置中心

### Zookeeper配置中心

## 注册中心

### Nacos注册中心



```yaml
seata:
  registry:
    # support: nacos, eureka, redis, zk, consul, etcd3, sofa
    type: nacos
    nacos:
      application: seata-server
      server-addr: 127.0.0.1:8848
      group: SEATA_GROUP
      namespace:
      cluster: default
      username: nacos
      password: nacos
      ##if use MSE Nacos with auth, mutex with username/password attribute
      #access-key: ""
      #secret-key: ""
```







### Eureka注册中心

```yaml
seata:
  registry:
    # support: nacos, eureka, redis, zk, consul, etcd3, sofa
    type: eureka
    eureka:
      service-url: http://localhost:8761/eureka
      application: seata-server
      weight: 1
```







### Etcd3注册中心

### Consul注册中心

### Zookeeper注册中心



## 存储模式

Seata Server需要存储全局事务的会话信息。提供了三种存储模式，分别为file，db和redis，默认(不更改配置)的情况下，存储模式为file。后续将引入raft和mongodb。

### file

file模式为单机模式，无需改动任何配置，直接启动即可。file模式下，全局事务的会话信息将在内存中读写，并持久化到本地文件root.data，性能较高。

> root.data文件位于${path}\seata\bin\sessionStore\root.data。

file模式 - Server存储配置

```yaml
serta:
  store:
    # support: file 、 db 、 redis
    mode: file
    session:
      mode: file
    lock:
      mode: file
    file:
      dir: sessionStore
      max-branch-session-size: 16384
      max-global-session-size: 512
      file-write-buffer-cache-size: 16384
      session-reload-read-size: 100
      flush-disk-mode: async
```

### db

db模式为高可用模式，全局事务会话信息通过db共享，相对file模式性能差些。

SQL脚本位于https://github.com/seata/seata/tree/develop/script/server/db，提供了MySQL，Oracle和PostgreSQL三种不同关系型数据库的SQL脚本。

MySQL Table：

```sql
CREATE DATABASE seata;
USE seata;

-- The script used when storeMode is 'db' the table to store GlobalSession data
CREATE TABLE IF NOT EXISTS `global_table`
(
    `xid`                       VARCHAR(128) NOT NULL,
    `transaction_id`            BIGINT,
    `status`                    TINYINT      NOT NULL,
    `application_id`            VARCHAR(32),
    `transaction_service_group` VARCHAR(32),
    `transaction_name`          VARCHAR(128),
    `timeout`                   INT,
    `begin_time`                BIGINT,
    `application_data`          VARCHAR(2000),
    `gmt_create`                DATETIME,
    `gmt_modified`              DATETIME,
    PRIMARY KEY (`xid`),
    KEY `idx_status_gmt_modified` (`status` , `gmt_modified`),
    KEY `idx_transaction_id` (`transaction_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

-- the table to store BranchSession data
CREATE TABLE IF NOT EXISTS `branch_table`
(
    `branch_id`         BIGINT       NOT NULL,
    `xid`               VARCHAR(128) NOT NULL,
    `transaction_id`    BIGINT,
    `resource_group_id` VARCHAR(32),
    `resource_id`       VARCHAR(256),
    `branch_type`       VARCHAR(8),
    `status`            TINYINT,
    `client_id`         VARCHAR(64),
    `application_data`  VARCHAR(2000),
    `gmt_create`        DATETIME(6),
    `gmt_modified`      DATETIME(6),
    PRIMARY KEY (`branch_id`),
    KEY `idx_xid` (`xid`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

-- the table to store lock data
CREATE TABLE IF NOT EXISTS `lock_table`
(
    `row_key`        VARCHAR(128) NOT NULL,
    `xid`            VARCHAR(128),
    `transaction_id` BIGINT,
    `branch_id`      BIGINT       NOT NULL,
    `resource_id`    VARCHAR(256),
    `table_name`     VARCHAR(32),
    `pk`             VARCHAR(36),
    `status`         TINYINT      NOT NULL DEFAULT '0' COMMENT '0:locked ,1:rollbacking',
    `gmt_create`     DATETIME,
    `gmt_modified`   DATETIME,
    PRIMARY KEY (`row_key`),
    KEY `idx_status` (`status`),
    KEY `idx_branch_id` (`branch_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

CREATE TABLE IF NOT EXISTS `distributed_lock`
(
    `lock_key`       CHAR(20) NOT NULL,
    `lock_value`     VARCHAR(20) NOT NULL,
    `expire`         BIGINT,
    primary key (`lock_key`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('HandleAllSession', ' ', 0);
```

db模式 - Server配置

```yaml
serta:
  store:
    # support: file 、 db 、 redis
    mode: db
    session:
      mode: db
    lock:
      mode: db
    db:
      datasource: druid
      db-type: mysql
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://127.0.0.1:3306/seata?rewriteBatchedStatements=true
      user: mysql
      password: mysql
      min-conn: 5
      max-conn: 100
      global-table: global_table
      branch-table: branch_table
      lock-table: lock_table
      distributed-lock-table: distributed_lock
      query-limit: 100
      max-wait: 5000
```

### redis

redis模式在seata-1.3及以上版本支持，性能较高，不过存在事务信息丢失的风险，需要配置适合当前场景的redis持久化配置。



redis模式 - Server配置

```yaml
serta:
  store:
    # support: file 、 db 、 redis
    mode: redis
    session:
      mode: redis
    lock:
      mode: redis
    redis:
      mode: single
      database: 0
      min-conn: 1
      max-conn: 10
      password:
      max-total: 100
      query-limit: 100
      single:
        host: 127.0.0.1
        port: 6379
      sentinel:
        master-name:
        sentinel-hosts:
```



## 控制台

http://localhost:7091/#/login

默认账号密码：seata，seata



## APM

### Skywalking

### Prometheus





## Server配置



```properties
#For details about configuration items, see https://seata.io/zh-cn/docs/user/configurations.html
#传输配置，用于客户端和服务器  
transport.type=TCP
transport.server=NIO
transport.heartbeat=true
transport.enableTmClientBatchSendRequest=false
transport.enableRmClientBatchSendRequest=true
transport.enableTcServerBatchSendResponse=false
transport.rpcRmRequestTimeout=30000
transport.rpcTmRequestTimeout=30000
transport.rpcTcRequestTimeout=30000
transport.threadFactory.bossThreadPrefix=NettyBoss
transport.threadFactory.workerThreadPrefix=NettyServerNIOWorker
transport.threadFactory.serverExecutorThreadPrefix=NettyServerBizHandler
transport.threadFactory.shareBossWorker=false
transport.threadFactory.clientSelectorThreadPrefix=NettyClientSelector
transport.threadFactory.clientSelectorThreadSize=1
transport.threadFactory.clientWorkerThreadPrefix=NettyClientWorkerThread
transport.threadFactory.bossThreadSize=1
transport.threadFactory.workerThreadSize=default
transport.shutdown.wait=3
transport.serialization=seata
transport.compressor=none

#事务路由规则配置，仅针对客户端
service.vgroupMapping.default_tx_group=default
#如果使用注册表，可以忽略它
service.default.grouplist=127.0.0.1:8091
service.enableDegrade=false
service.disableGlobalTransaction=false

#事务规则配置，仅针对客户端  
client.rm.asyncCommitBufferLimit=10000
client.rm.lock.retryInterval=10
client.rm.lock.retryTimes=30
client.rm.lock.retryPolicyBranchRollbackOnConflict=true
client.rm.reportRetryCount=5
client.rm.tableMetaCheckEnable=false
client.rm.tableMetaCheckerInterval=60000
client.rm.sqlParserType=druid
client.rm.reportSuccessEnable=false
client.rm.sagaBranchRegisterEnable=false
client.rm.sagaJsonParser=fastjson
client.rm.tccActionInterceptorOrder=-2147482648
client.tm.commitRetryCount=5
client.tm.rollbackRetryCount=5
client.tm.defaultGlobalTransactionTimeout=60000
client.tm.degradeCheck=false
client.tm.degradeCheckAllowTimes=10
client.tm.degradeCheckPeriod=2000
client.tm.interceptorOrder=-2147482648
client.undo.dataValidation=true
client.undo.logSerialization=jackson
client.undo.onlyCareUpdateColumns=true
server.undo.logSaveDays=7
server.undo.logDeletePeriod=86400000
client.undo.logTable=undo_log
client.undo.compress.enable=true
client.undo.compress.type=zip
client.undo.compress.threshold=64k
#TCC事务模式
tcc.fence.logTableName=tcc_fence_log
tcc.fence.cleanPeriod=1h

#日志规则配置，用于客户端和服务器
log.exceptionRate=100

#事务存储配置，仅供服务器使用。 file、DB和redis的配置值是可选的。  
store.mode=file
store.lock.mode=file
store.session.mode=file
#用于密码加密。
store.publicKey=

#如果store.mode,store.lock.mode,store.session.mode不等于file，你可以删除这些配置块。
store.file.dir=file_store/data
store.file.maxBranchSessionSize=16384
store.file.maxGlobalSessionSize=512
store.file.fileWriteBufferCacheSize=16384
store.file.flushDiskMode=async
store.file.sessionReloadReadSize=100

#如果store modes是db，则这些配置是必须的。如果store.mode,store.lock.mode,store.session.mode不等于db，你可以删除这些配置块。
store.db.datasource=druid
store.db.dbType=mysql
store.db.driverClassName=com.mysql.jdbc.Driver
store.db.url=jdbc:mysql://127.0.0.1:3306/seata?useUnicode=true&rewriteBatchedStatements=true
store.db.user=username
store.db.password=password
store.db.minConn=5
store.db.maxConn=30
store.db.globalTable=global_table
store.db.branchTable=branch_table
store.db.distributedLockTable=distributed_lock
store.db.queryLimit=100
store.db.lockTable=lock_table
store.db.maxWait=5000

#如果store modes是redis，则这些配置是必须的。如果store.mode,store.lock.mode,store.session.mode不等于redis，你可以删除这些配置块。
store.redis.mode=single
store.redis.single.host=127.0.0.1
store.redis.single.port=6379
store.redis.sentinel.masterName=
store.redis.sentinel.sentinelHosts=
store.redis.maxConn=10
store.redis.minConn=1
store.redis.maxTotal=100
store.redis.database=0
store.redis.password=
store.redis.queryLimit=100

#事务规则配置，仅针对服务端
server.recovery.committingRetryPeriod=1000
server.recovery.asynCommittingRetryPeriod=1000
server.recovery.rollbackingRetryPeriod=1000
server.recovery.timeoutRetryPeriod=1000
server.maxCommitRetryTimeout=-1
server.maxRollbackRetryTimeout=-1
server.rollbackRetryTimeoutUnlockEnable=false
server.distributedLockExpireTime=10000
server.xaerNotaRetryTimeout=60000
server.session.branchAsyncQueueSize=5000
server.session.enableBranchAsyncRemove=true

#监控配置，仅针对服务端
metrics.enabled=false
metrics.registryType=compact
metrics.exporterList=prometheus
metrics.exporterPrometheusPort=9898
```







```yaml
server:
  port: 7091

spring:
  application:
    name: seata-server

logging:
  config: classpath:logback-spring.xml
  file:
    path: ${user.home}/logs/seata
  extend:
    logstash-appender:
      destination: 127.0.0.1:4560
    kafka-appender:
      bootstrap-servers: 127.0.0.1:9092
      topic: logback_to_logstash

console:
  user:
    username: seata
    password: seata

seata:
  config:
    # support: nacos, consul, apollo, zk, etcd3
    type: file
  registry:
    # support: nacos, eureka, redis, zk, consul, etcd3, sofa
    type: file
  store:
    # support: file 、 db 、 redis
    mode: file
#  server:
#    service-port: 8091 #If not configured, the default is '${server.port} + 1000'
  security:
    secretKey: SeataSecretKey0c382ef121d778043159209298fd40bf3850a017
    tokenValidityInMilliseconds: 1800000
    ignore:
      urls: /,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/public/**,/api/v1/auth/login
```





## Client配置