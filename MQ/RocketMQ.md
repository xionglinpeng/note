# RocketMQ



## 下载、安装、启动

### 下载

RocketMQ的Binary版是一些编译好的jar和辅助的shell脚本，可以直接从官网找到下载链接（http://rocketmq.apache.org/dowloading/releases/）。也可以下载源码自己编译。

### 安装

下载完成之后直接解压即可，其是免安装的。当然，运行Java程序的相关环境必须是要有的。

```shell
# 解压
$ unzip rocketmq-all-4.9.2-bin-release.zip
# 进入到rocketmq目录
$ cd rocketmq-4.9.2/
```

rocketmq-4.9.2文件夹下包含如下内容：

```
benchmark/
bin/
conf/
lib/
LICENSE
NOTICE
README.md
```

- LICENSE、NOTICE和README.md包含一些版本声明和功能说明信息。

- benchmark文件夹包含运行benchmark程序的shell脚本。

- bin文件夹包含各种使用RocketMQ的shell脚本（Linux）和cmd脚本（Windows）。


  主要是常用的—启动NameServer的脚本`mqnamesrv`；启动Broker的脚本`mqbroker`；集群管理脚本`mqadmin`。
- conf文件夹包含一些示例配置文件。

  包括三种方式的broker配置文件，logback日志配置文件等。用户在写配置文件的时候，一般基于这些示例配置文件，加上自己特殊的需求即可。

- lib文件夹包含RocketMQ各个模块编译成的jar包，以及RocketMQ依赖的一些jar包。

### 启动

#### 启动namesrv

```shell
$ nohup sh bin/mqnamesrv > ~/logs/rocketmqlogs/namesrv.log 2>&1 &
```

启动日志

```shell
$ cat  ~/logs/rocketmqlogs/namesrv.log
nohup: 忽略输入
Java HotSpot(TM) 64-Bit Server VM warning: Using the DefNew young collector with the CMS collector is deprecated and will likely be removed in a future release
Java HotSpot(TM) 64-Bit Server VM warning: UseCMSCompactAtFullCollection is deprecated and will likely be removed in a future release.
The Name Server boot success. serializeType=JSON
t/rocketmq-4.9.2
2021-12-11 16:11:49 INFO main - kvConfigPath=/root/namesrv/kvConfig.json
2021-12-11 16:11:49 INFO main - configStorePath=/root/namesrv/namesrv.properties
2021-12-11 16:11:49 INFO main - productEnvName=center
2021-12-11 16:11:49 INFO main - clusterTest=false
2021-12-11 16:11:49 INFO main - orderMessageEnable=false
2021-12-11 16:11:49 INFO main - listenPort=9876
2021-12-11 16:11:49 INFO main - serverWorkerThreads=8
2021-12-11 16:11:49 INFO main - serverCallbackExecutorThreads=0
2021-12-11 16:11:49 INFO main - serverSelectorThreads=3
2021-12-11 16:11:49 INFO main - serverOnewaySemaphoreValue=256
2021-12-11 16:11:49 INFO main - serverAsyncSemaphoreValue=64
2021-12-11 16:11:49 INFO main - serverChannelMaxIdleTimeSeconds=120
2021-12-11 16:11:49 INFO main - serverSocketSndBufSize=65535
2021-12-11 16:11:49 INFO main - serverSocketRcvBufSize=65535
2021-12-11 16:11:49 INFO main - serverPooledByteBufAllocatorEnable=true
2021-12-11 16:11:49 INFO main - useEpollNativeSelector=false
2021-12-11 16:11:49 INFO main - Server is running in TLS permissive mode
2021-12-11 16:11:49 INFO main - Tls config file doesn't exist, skip it
2021-12-11 16:11:49 INFO main - Log the final used tls related configuration
2021-12-11 16:11:49 INFO main - tls.test.mode.enable = true
2021-12-11 16:11:49 INFO main - tls.server.need.client.auth = none
2021-12-11 16:11:49 INFO main - tls.server.keyPath = null
2021-12-11 16:11:49 INFO main - tls.server.keyPassword = null
2021-12-11 16:11:49 INFO main - tls.server.certPath = null
2021-12-11 16:11:49 INFO main - tls.server.authClient = false
2021-12-11 16:11:49 INFO main - tls.server.trustCertPath = null
2021-12-11 16:11:49 INFO main - tls.client.keyPath = null
2021-12-11 16:11:49 INFO main - tls.client.keyPassword = null
2021-12-11 16:11:49 INFO main - tls.client.certPath = null
2021-12-11 16:11:49 INFO main - tls.client.authServer = false
2021-12-11 16:11:49 INFO main - tls.client.trustCertPath = null
2021-12-11 16:11:49 INFO main - Using JDK SSL provider
2021-12-11 16:11:50 INFO main - SSLContext created for server
2021-12-11 16:11:50 INFO main - Try to start service thread:FileWatchService started:false lastThread:null
2021-12-11 16:11:50 INFO NettyEventExecutor - NettyEventExecutor service started
2021-12-11 16:11:50 INFO main - The Name Server boot success. serializeType=JSON
2021-12-11 16:11:50 INFO FileWatchService - FileWatchService service started
2021-12-11 16:11:50 INFO NettyServerCodecThread_1 - NETTY SERVER PIPELINE: channelRegistered 192.168.56.1:61585
2021-12-11 16:11:50 INFO NettyServerCodecThread_1 - NETTY SERVER PIPELINE: channelActive, the channel[192.168.56.1:61585]
2021-12-11 16:11:51 INFO NettyServerCodecThread_1 - NETTY SERVER PIPELINE: channelInactive, the channel[192.168.56.1:61585]
2021-12-11 16:11:51 INFO NettyServerCodecThread_1 - NETTY SERVER PIPELINE: channelUnregistered, the channel[192.168.56.1:61585]
2021-12-11 16:11:59 INFO NettyServerCodecThread_2 - NETTY SERVER PIPELINE: channelRegistered 127.0.0.1:43704
2021-12-11 16:11:59 INFO NettyServerCodecThread_2 - NETTY SERVER PIPELINE: channelActive, the channel[127.0.0.1:43704]
2021-12-11 16:11:59 INFO RemotingExecutorThread_2 - new topic registered, work1 QueueData [brokerName=work1, readQueueNums=1, writeQueueNums=1, perm=7, topicSysFlag=0]
2021-12-11 16:11:59 INFO RemotingExecutorThread_2 - new topic registered, SCHEDULE_TOPIC_XXXX QueueData [brokerName=work1, readQueueNums=18, writeQueueNums=18, perm=6, topicSysFlag=0]
2021-12-11 16:11:59 INFO RemotingExecutorThread_2 - new topic registered, RMQ_SYS_TRANS_HALF_TOPIC QueueData [brokerName=work1, readQueueNums=1, writeQueueNums=1, perm=6, topicSysFlag=0]
2021-12-11 16:11:59 INFO RemotingExecutorThread_2 - new topic registered, DefaultCluster_REPLY_TOPIC QueueData [brokerName=work1, readQueueNums=1, writeQueueNums=1, perm=6, topicSysFlag=0]
2021-12-11 16:11:59 INFO RemotingExecutorThread_2 - new topic registered, BenchmarkTest QueueData [brokerName=work1, readQueueNums=1024, writeQueueNums=1024, perm=6, topicSysFlag=0]
2021-12-11 16:11:59 INFO RemotingExecutorThread_2 - new topic registered, OFFSET_MOVED_EVENT QueueData [brokerName=work1, readQueueNums=1, writeQueueNums=1, perm=6, topicSysFlag=0]
2021-12-11 16:11:59 INFO RemotingExecutorThread_2 - new topic registered, TopicTest QueueData [brokerName=work1, readQueueNums=4, writeQueueNums=4, perm=6, topicSysFlag=0]
2021-12-11 16:11:59 INFO RemotingExecutorThread_2 - new topic registered, %RETRY%ROCKETMQ_GROUP_DEMO QueueData [brokerName=work1, readQueueNums=1, writeQueueNums=1, perm=6, topicSysFlag=0]
2021-12-11 16:11:59 INFO RemotingExecutorThread_2 - new topic registered, TBW102 QueueData [brokerName=work1, readQueueNums=8, writeQueueNums=8, perm=7, topicSysFlag=0]
2021-12-11 16:11:59 INFO RemotingExecutorThread_2 - new topic registered, SELF_TEST_TOPIC QueueData [brokerName=work1, readQueueNums=1, writeQueueNums=1, perm=6, topicSysFlag=0]
2021-12-11 16:11:59 INFO RemotingExecutorThread_2 - new topic registered, DefaultCluster QueueData [brokerName=work1, readQueueNums=16, writeQueueNums=16, perm=7, topicSysFlag=0]
2021-12-11 16:11:59 INFO RemotingExecutorThread_2 - new broker registered, 192.168.56.3:10911 HAServer: 10.0.2.6:10912
2021-12-11 16:12:00 INFO NettyServerCodecThread_3 - NETTY SERVER PIPELINE: channelRegistered 192.168.56.1:61598
2021-12-11 16:12:00 INFO NettyServerCodecThread_3 - NETTY SERVER PIPELINE: channelActive, the channel[192.168.56.1:61598]
2021-12-11 16:12:50 INFO NSScheduledThread1 - --------------------------------------------------------
2021-12-11 16:12:50 INFO NSScheduledThread1 - configTable SIZE: 0
```

#### 启动broker

```shell
$ nohup sh bin/mqbroker -n 127.0.0.1:9876 > ~/logs/rocketmqlogs/mqbroker.log 2>&1 &
```

启动日志

```shell
$ cat ~/logs/rocketmqlogs/mqbroker.log
nohup: 忽略输入
The broker[work1, 10.0.2.6:10911] boot success. serializeType=JSON and name server is 127.0.0.1:9876
```

> Note: 启动broker的时候需要添加`-n`参数，该参数指定的namesrv的地址。如果不指定该地址，broker和namesrv不能关联起来。
>
> 指定`-n`和不知道`-n`参数的日志差异：
>
> - 指定`-n`参数
>
>   ```shell
>   The broker[work1, 10.0.2.6:10911] boot success. serializeType=JSON
>   ```
>
> - 不指定`-n`参数
>
>   ```shell
>   The broker[work1, 10.0.2.6:10911] boot success. serializeType=JSON and name server is 127.0.0.1:9876
>   ```

#### 关闭broker

```shell
$ sh bin/mqshutdown broker
The mqbroker(4143) is running...
Send shutdown request to mqbroker(4143) OK
```

#### 关闭namesrv

```shell
$ sh bin/mqshutdown namesrv
The mqnamesrv(3910) is running...
Send shutdown request to mqnamesrv(3910) OK
```

#### Error

**There is insufficient memory for the Java Runtime Environment to continue.**

**内存不足，Java运行时环境无法继续。**

报错的原因已经很明确了。由于RocketMQ是基于Java语言开发的，因此其也是运行基于Java虚拟机之上。RocketMQ启动namesrv和broker的时候配置了`-Xms`、`-Xmx`和`-Xmn`参数。默认情况下启动namesrv需要4G堆内存，启动broker需要8G堆内存。其相关配置位于shell脚本`bin/runserver.sh`和b`in/runbroker.sh`。代码如下所示：

bin/runserver.sh

```shell
71       JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
76       JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```

bin/runbroker.sh

```shell
67 JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g"
```

如果机器内存不足，将报错There is insufficient memory for the Java Runtime Environment to continue.，其解决方式就是修改-Xms和-Xmx以及-Xmn参数。例如：

bin/runserver.sh

```shell
71       JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
76       JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx4g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```

bin/runbroker.sh

```shell
67 JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g"
```

## RocketMQ角色





## 示例

### Producer示例程序

```java
public static void main(String[] args) throws Exception {
    //创建一个Producer实例，并指定组名。
    DefaultMQProducer producer = new DefaultMQProducer("ROCKETMQ_GROUP_DEMO");
    producer.setNamesrvAddr("192.168.56.3:9876");
    // Launch the instance
    producer.start();
    for (int i = 0; i < 100; i++) {
        //创建一个Message实例，并指定Topic, Tag和Message.
        Message message = new Message(
            "TopicTest", /*Topic*/
            "TagA", /*Tag*/
            ("Hello RockMqQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET)); /*Message body*/
        //调用send方法发送消息，将消息传递给其中一个broker。
        SendResult sendResult = producer.send(message);
        System.out.println(sendResult);
    }
    //一旦Producer实例不再使用，就关闭它。
    producer.shutdown();
}
```

### Consumer示例程序

```java
public static void main(String[] args) throws MQClientException {
    //创建一个Consumer实例，并指定组名称
    DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("ROCKETMQ_GROUP_DEMO");
    //指定namesrv的地址
    consumer.setNamesrvAddr("192.168.56.3:9876");
    //指定在指定的Consumer组是全新的情况下从哪里开始。
    consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);
    //订阅一个更多的主题消费
    consumer.subscribe("TopicTest","*");
    //将从代理获取的消息注册为在到达时执行回调
    consumer.registerMessageListener(new MessageListenerConcurrently() {
        public ConsumeConcurrentlyStatus consumeMessage(
            List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
            System.out.println(Thread.currentThread().getName() + "Receive new messages"+list+"%n");
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        }
    });
    // Launch the consumer instance
    consumer.start();
}
```

### Error

1. **No route info of this topic**

   启动Producer示例程序，报错`No route info of this topic`。导致这个错误的原因是在启动broker的时候没有指定namesrv的地址，即`bin/mqbroker`的`-n`参数。

2. **org.apache.rocketmq.remoting.exception.RemotingTooMuchRequestException: sendDefaultImpl call timeout**

   导致这个错误的原因是由于RocketMQ实例和示例程序不在同一主机。例如，RocketMQ实例常常部署在虚拟机上。而broker暴露出来的地址是虚拟机的内网地址，对于客户端示例程序而言，这个内网地址是不能访问的。因此解决方式就是使broker暴露出主机地址，使客户端程序可以访问即可。

   具体操作：修改`conf/broker.conf`文件，然后启动broker的时候，指定`conf/broker.conf`文件即可。

   - *修改conf/broker.conf文件*

     ```shell
     $ echo "brokerIP1=192.168.56.3" > conf/broker.conf 
     $ cat conf/broker.conf 
     brokerIP1=192.168.56.3
     ```

     > 192.168.56.3就是broker暴露出主机地址，客户端程序可访问。

   - 指定conf/broker.conf文件*

     ```shell
     $ nohup sh bin/mqbroker -n 127.0.0.1:9876 -c conf/broker.conf > ~/logs/rocketmqlogs/mqbroker.log 2>&1 &
     ```

   > conf/broker.conf 源文件内容：
   >
   > ```shell
   > $ cat conf/broker.conf 
   > # Licensed to the Apache Software Foundation (ASF) under one or more
   > # contributor license agreements.  See the NOTICE file distributed with
   > # this work for additional information regarding copyright ownership.
   > # The ASF licenses this file to You under the Apache License, Version 2.0
   > # (the "License"); you may not use this file except in compliance with
   > # the License.  You may obtain a copy of the License at
   > #
   > #     http://www.apache.org/licenses/LICENSE-2.0
   > #
   > #  Unless required by applicable law or agreed to in writing, software
   > #  distributed under the License is distributed on an "AS IS" BASIS,
   > #  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   > #  See the License for the specific language governing permissions and
   > #  limitations under the License.
   > 
   > brokerClusterName = DefaultCluster
   > brokerName = broker-a
   > brokerId = 0
   > deleteWhen = 04
   > fileReservedTime = 48
   > brokerRole = ASYNC_MASTER
   > flushDiskType = ASYNC_FLUSH
   > ```

## Dashboard

运维服务程序是一个Spring Boot程序，需要从GitHub上下载，源码位于https://github.com/apache/rocketmq-dashboard。

启动方式就是Spring Boot项目的启动方式。只是需要注意的是，启动之前需要在配置文件application.properties中指定namesrv的地址，对应配置属性为`rocketmq.config.namesrvAddr=`。

项目端口默认配置为8080，因此，启动完成之后，直接在浏览器http://localhost:8080/访问即可。

## 总结

- namesrv暴露的端口为9876。
- broker暴露的端口为10909。
- Dashboard暴露的端口为8080。





