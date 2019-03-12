



Redis的Sentinel系统用于管理多个Redis服务器（instance），该系统执行以下三个任务：

- **监控（Monitoring）：**Sentinel会不断地检查你的主服务器和从服务器是否运作正常。
- **提醒（Notification）：**当被监控的某个Redis服务器出现问题时，Sentinel可以通过API向管理员或者其他应用程序发送通知。
- **自动故障迁移（Automatic failover）：**当一个主服务器不能正常工作时，Sentinel会开始一次自动故障迁移操作，它会将失效主服务器的其中一个从服务器升级为新的主服务器，并让失效主服务器的其他从服务器改为复制新的主服务器；当客户端试图连接失效的主服务器时，集群也会向客户端返回新主服务器的地址，使得集群可以使用新主服务器带起代替服务器。



## 获取Sentinel



## 启动Sentinel



## 配置Sentinel



## 主观下线和客观下线

前面说过，Redis的Sentinel中关于下线（down）有两个不同的概念：

- 主观下线（Subjectively Down，简称SDOWN）指的是单个Sentinel实例对服务器做出的下线判断。
- 客观下线（Objectively Down，简称ODOWN）指的是多个Sentinel实例在对同一个服务器做出SDOWN判断，并且通过`sentinel is-master-down-by-addr`命令来询问对方是否认为给定的服务器已下线。

如果一个服务器没有在`master-down-after-milliseconds`选项所指定的时间内，对向它发生`PING`命令的Sentinel返回一个有效回复（vlid reply），那么Sentinel就会将这个服务器标记为主观下线。

服务器对PING命令的有效回复可以是一下三种回复的其中一种：

- 返回+PONG。
- 返回-LOADING错误。
- 返回-MASTERDOWN错误。

如果服务器返回除以上三种回复之外的其他回复，又或者在指定时间内没有回复PING命令，那么Sentinel认为服务器返回的回复无效（non-valid）。

注意，一个服务器必须在`master-down-after-millseconds`毫秒内，一直返回无效回复才会被Sentinel标记为主观下线。举个例子，如果`master-down-after-millseconds`选项的值为30000毫秒（30秒），那么只要服务器能在每29秒之内返回至少一次有效回复，这个服务器就仍然会被认为是处于正常状态内。

从主观下线状态切换到客观下线状态并没有使用严格的法定人数算法（strong quorum algorithm），而是使用了流言协议：如果Sentinel在给定的时间范围内从其他Sentinel哪里接收到了足够数量的主服务器下线报告，那么Sentinel就会将主服务器的状态从主观下线改变为客观下线。如果之后其他Sentinel不再报告主服务器已下线，那么客观下线状态就会被移除。

客观下线条件**只适用于主服务器**：对于任何其他的Redis实例，Sentinel在将他们判断为下线前不需要进行协商，所以从服务器或者其他Sentinel永远不会达到客观下线条件。

只要一个Sentinel发现某个主服务器进入了客观下线状态，这个Sentinel就可能会被其他Sentinel推选出，并对失效的主服务器执行自动故障转移操作。

## 每个Sentinel都需要定期执行的任务

- 每个sentinel以每秒钟一次的频率向它所知的主服务器、从服务器以及其他Sentinel实例发送一个PING命令。
- 如果一个实例（instance）距离最后一次有效回复`PING`命令的时间超过`down-after-milliseconds`选项所指定的值，那么这个实例会被Sentinel标记为主观下线。一个有效回复可以是：`+PONG`、`-LOADING`或者`-MASTERDOWN`。
- 如果一个主服务器被标记为主观下线，那么正在监视这个主服务器的所有Sentinel要以每秒一次的频率确认主服务器的确进入了主观下线状态。
- 如果一个主服务器被标记为主观下线，并且有足够数量的Sentinel在指定的时间范围内同意这一判断，那么这个主服务器被标记为客观下线。
- 在一般情况下，每个Sentinel会以每10秒一次的频率向它已知的所有主服务器和从服务器发送`INFO`命令，，当一个主服务器被Sentinel标记为客观下线时，Sentinel向下线主服务器的所有从服务器发送`INFO`命令的频率会从10秒一次改为每秒一次。
- 当没有足够数量的Sentinel同意主服务器已经下线，主服务器的客观下线状态就会被移除。当主服务器重新向Sentinel的PING命令返回有效回复时，主服务器的主观下线状态就会被移除。

## 自动发现Sentinel和从服务器

一个Sentinel可以与其他多个Sentinel进行连接，各个Sentinel之间可以互相检查对方的可用性，并进行信息交换。你无须为运行的每个Sentinel分别设置其他Sentinel的地址，因为Sentinel可以通过发布与订阅功能来自动发现正在监视相同主服务器的其他Sentinel，这一功能是通过向频道`__sentinel__:hello`发送信息来实现的。

与此类似，你也不必手动列出主服务器属下的所有从服务器，因为Sentinel可以通过询问主服务器来获得所有从服务器的信息。

- 每个Sentinel会以每两秒一次的频率，通过发布与订阅功能，向被它监视的所有主服务器和从服务器的`__sentinel__:hello`频道发送一条信息，信息中包含了Sentinel的IP地址、端口号和运行ID（runid）。
- 每个Sentinel都订阅了被它监视的所有主服务器和从服务器的`__sentinel__:hello`频道，查找之前未出现过的sentinel（looking for unknown sentinels）。当一个sentinel发现一个新的sentinel时，他会将新的sentinel添加到一个列表中，这个列表保存sentinel已知的，监视同一个主服务器的所有其他sentinel。
- Sentinel发送的信息中还包括完整的主服务器当前配置（configuration）。如果一个Sentinel包含的主服务器配置比另一个Sentinel发送的配置要旧，那么这个Sentinel会立即升级到新配置上。
- 在将一个新Sentinel添加到监视主服务器的列表上面之前，Sentinel会先检查列表中是否已经包含了和要添加的Sentinel拥有相同运行ID或者相同地址（包括IP地址和端口号）的Sentinel，如果是的话，Sentinel会先移除列表中已有的那些用哟与相同运行ID或者相同地址的Sentinel，然后再添加新的Sentinel。

## Sentinel API

在默认情况下，Sentinel使用TCP端口26379（普通Redis服务器使用的是6379）。

Sentinel接收Redis协议格式的命令请求，所以你可以使用Redis-cli或者任何其他Redis客户端来与Sentinel进行通讯。

有两种方式可以和Sentinel进行通讯：

- 第一种方法是通过直接发送命令来查询被监视Redis服务器的当前状态，以及Sentinel所知道的关于其他Sentinle的信息，诸如此类。
- 另一种方法是使用发布与订阅功能，通过接收Sentinel发送的通知：当执行故障转移操作，或者某个被监视的服务器被判断为主观下线或者客观下线时，Sentinle就会发送相应的信息。

## Sentinel命令

以下列出的是Sentinel接受的命令：

```shell
[root@localhost ~]# redis-cli -p 26379
```

### ping

返回PONG;

```shell
127.0.0.1:26379> ping
PONG
```

### sentinel masters

列出所有被监视的主服务器，以及这些主服务器的当前状态。

```shell
127.0.0.1:26379> sentinel masters
1)  1) "name"
    2) "master"
    3) "ip"
    4) "192.168.1.110"
    5) "port"
    6) "6380"
    7) "runid"
    8) "5b9fa849fa31a853cdc118445535719cc3d51e51"
    9) "flags"
   10) "master"
   11) "link-pending-commands"
   12) "0"
   13) "link-refcount"
   14) "1"
   15) "last-ping-sent"
   16) "0"
   17) "last-ok-ping-reply"
   18) "982"
   19) "last-ping-reply"
   20) "982"
   21) "down-after-milliseconds"
   22) "15000"
   23) "info-refresh"
   24) "9524"
   25) "role-reported"
   26) "master"
   27) "role-reported-time"
   28) "441492"
   29) "config-epoch"
   30) "19"
   31) "num-slaves"
   32) "2"
   33) "num-other-sentinels"
   34) "2"
   35) "quorum"
   36) "2"
   37) "failover-timeout"
   38) "180000"
   39) "parallel-syncs"
   40) "1"
```

### sentinel slaves [master-name]

列出给定主服务器的所有从服务器，以及这些从服务器的当前状态。

```shell
127.0.0.1:26379> sentinel slaves master
1)  1) "name"
    2) "192.168.1.110:6381"
    3) "ip"
    4) "192.168.1.110"
    5) "port"
    6) "6381"
    7) "runid"
    8) "3fb161884b8c29f9ff182b50fdbf2fd9d2bf37d5"
    9) "flags"
   10) "slave"
   11) "link-pending-commands"
   12) "0"
   13) "link-refcount"
   14) "1"
   15) "last-ping-sent"
   16) "0"
   17) "last-ok-ping-reply"
   18) "722"
   19) "last-ping-reply"
   20) "722"
   21) "down-after-milliseconds"
   22) "15000"
   23) "info-refresh"
   24) "8862"
   25) "role-reported"
   26) "slave"
   27) "role-reported-time"
   28) "572611"
   29) "master-link-down-time"
   30) "0"
   31) "master-link-status"
   32) "ok"
   33) "master-host"
   34) "192.168.1.110"
   35) "master-port"
   36) "6380"
   37) "slave-priority"
   38) "100"
   39) "slave-repl-offset"
   40) "117555"
2)  1) "name"
    2) "192.168.1.110:6379"
    3) "ip"
    4) "192.168.1.110"
    5) "port"
    6) "6379"
    7) "runid"
    8) "083bbdfaff01c2ba03a18a4d2eaf2b48fafa65cb"
    9) "flags"
   10) "slave"
   11) "link-pending-commands"
   12) "0"
   13) "link-refcount"
   14) "1"
   15) "last-ping-sent"
   16) "0"
   17) "last-ok-ping-reply"
   18) "722"
   19) "last-ping-reply"
   20) "722"
   21) "down-after-milliseconds"
   22) "15000"
   23) "info-refresh"
   24) "8862"
   25) "role-reported"
   26) "slave"
   27) "role-reported-time"
   28) "572611"
   29) "master-link-down-time"
   30) "0"
   31) "master-link-status"
   32) "ok"
   33) "master-host"
   34) "192.168.1.110"
   35) "master-port"
   36) "6380"
   37) "slave-priority"
   38) "100"
   39) "slave-repl-offset"
   40) "117555"
```

### sentinel get-master-addr-by-name [master-name]

返回给定名字的主服务器的IP地址和端口号。如果这个主服务器正在执行故障转移操作，或者针对这个主服务器的故障转移操作以及完成，那么这个命令返回新的主服务器的IP地址和端口号。

```shell
127.0.0.1:26379> sentinel get-master-addr-by-name master
1) "192.168.1.110"
2) "6380"
```

### sentinel reset [pattern]

重置所有名字和给定模式pattern相匹配的主服务器。pattern参数是一个Glob风格的模式。重置操作清除主服务器目前的所有状态，包括正在执行中的故障转移，并移除目前已经发现和关联的，主服务器的所有从服务器和Sentinel。

```shell
127.0.0.1:26379> sentinel reset mas*
(integer) 1
```

### sentinel failover

当主服务器失效时，在不询问其他Sentinel意见的情况下，强制开始一次自动故障迁移（不过发起故障转移的Sentinel会向其他Sentinel发送一个新的配置，其他Sentinel会根据这个配置进行相应的更新）。

```shell
127.0.0.1:26379> sentinel failover master
OK
127.0.0.1:26379> sentinel get-master-addr-by-name master
1) "192.168.1.110"
2) "6379"
```

## 发布与订阅信息

客户端可以将Sentinel看做是一个只提供了订阅功能的Redis服务器：你不可以使用`publish`命令向这个服务器发送信息，但你可以用`subscribe`或者`psubscribe`命令，通过订阅给定的频道来获取相应的事件提醒。

一个频道能够接收和这个频道的名字相同的事件。比如说，名为`+sdown`的频道就可以接收所有实例进入主观下线（SDOWN）状态的事件。

通过执行`psubscribe *`命令可以接收所有事件信息。

一下列出的是客户端可以通过订阅来获得的频道和信息的格式：第一个英文单词是频道/事件的名字，其余的是数据的格式。

注意：当格式中包含instance details字样时，表示频道所返回的信息中包含了以下用于识别目标实例的内容。

```
<instance-type> <name> <ip> <port> @ <master-name> <master-ip> <master-port>
```

下面是一个订阅全部事件的示例：

```shell
127.0.0.1:26379> psubscribe *
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "*"
3) (integer) 1
1) "pmessage"
2) "*"
3) "+sdown"	//主服务器192.168.1.110 6379进入了主观下线
4) "master master 192.168.1.110 6379"
1) "pmessage"
2) "*"
3) "+odown"	//主服务器192.168.1.110 6379进入了客观下线，法定人数3/2
4) "master master 192.168.1.110 6379 #quorum 3/2"
1) "pmessage"
2) "*"
3) "+new-epoch"	//当前的epoch已被更新
4) "21"
1) "pmessage"
2) "*"
3) "+try-failover"	//故障迁移执行中，等待被大多数Sentinel选中
4) "master master 192.168.1.110 6379"
1) "pmessage"
2) "*"
3) "+vote-for-leader"	//
4) "b181e9255d264e4f766fdf950304b9f1761734b9 21"
1) "pmessage"
2) "*"
3) "+elected-leader"	//赢得指定epoch的选举，可以开始故障迁移了
4) "master master 192.168.1.110 6379"
1) "pmessage"
2) "*"
3) "+failover-state-select-slave"	//故障迁移操作现在处于select-slave状态（Sentinel正在寻找可以升级为主服务器的从服务器）
4) "master master 192.168.1.110 6379"
1) "pmessage"
2) "*"
3) "+selected-slave" //Sentinel顺利找到适合进行升级为主服务器的从服务器，是192.168.1.110 6381。
4) "slave 192.168.1.110:6381 192.168.1.110 6381 @ master 192.168.1.110 6379"
1) "pmessage"
2) "*"
3) "+failover-state-send-slaveof-noone"	 //Sentinel正在将指定从服务器升级为主服务器
4) "slave 192.168.1.110:6381 192.168.1.110 6381 @ master 192.168.1.110 6379"
1) "pmessage"
2) "*"
3) "+failover-state-wait-promotion"				//
4) "slave 192.168.1.110:6381 192.168.1.110 6381 @ master 192.168.1.110 6379"
1) "pmessage"
2) "*"
3) "-role-change"				//
4) "slave 192.168.1.110:6381 192.168.1.110 6381 @ master 192.168.1.110 6379 new reported role is master"
1) "pmessage"
2) "*"
3) "+promoted-slave"				//
4) "slave 192.168.1.110:6381 192.168.1.110 6381 @ master 192.168.1.110 6379"
1) "pmessage"
2) "*"
3) "+failover-state-reconf-slaves" //故障转移状态切换到了reconf-slaves状态
4) "master master 192.168.1.110 6379"
1) "pmessage"
2) "*"
3) "+slave-reconf-sent" //领头（leader）的Sentinel向slave实例发送了slave of命令，设置其新的主服务器。
4) "slave 192.168.1.110:6380 192.168.1.110 6380 @ master 192.168.1.110 6379"
1) "pmessage"
2) "*"
3) "+slave-reconf-inprog" //实例正在将自己设置为指定主服务器的从服务器，但相应的同步过程仍未完成。
4) "slave 192.168.1.110:6380 192.168.1.110 6380 @ master 192.168.1.110 6379"
1) "pmessage"
2) "*"

3) "-odown" //原主服务器192.168.1.110 6379上线了，不再处于客观下线状态
4) "master master 192.168.1.110 6379"
1) "pmessage"
2) "*"
3) "+slave-reconf-done" //从服务器已经成功完成对新主服务器的同步
4) "slave 192.168.1.110:6380 192.168.1.110 6380 @ master 192.168.1.110 6379"
1) "pmessage"
2) "*"
3) "+failover-end" //故障转移操作顺利完成。所有从服务器都开始复制新的主服务器了。
4) "master master 192.168.1.110 6379"
1) "pmessage"
2) "*"
3) "+switch-master" //配置变更，主服务器的IP和地址已经改变。
4) "master 192.168.1.110 6379 192.168.1.110 6381"
1) "pmessage"
2) "*"
3) "+slave" //一个新的从服务器已经被Sentinel识别并关联。
4) "slave 192.168.1.110:6380 192.168.1.110 6380 @ master 192.168.1.110 6381"
1) "pmessage"
2) "*"
3) "+slave" //一个新的从服务器已经被Sentinel识别并关联。
4) "slave 192.168.1.110:6379 192.168.1.110 6379 @ master 192.168.1.110 6381"
1) "pmessage"
2) "*"
3) "+sdown" //实例192.168.1.110 6379正处于主观下线状态
4) "slave 192.168.1.110:6379 192.168.1.110 6379 @ master 192.168.1.110 6381"
1) "pmessage"
2) "*"
3) "-role-change"				//
4) "slave 192.168.1.110:6379 192.168.1.110 6379 @ master 192.168.1.110 6381 new reported role is master"
1) "pmessage"
2) "*"
3) "-sdown"				//
4) "slave 192.168.1.110:6379 192.168.1.110 6379 @ master 192.168.1.110 6381"
1) "pmessage"
2) "*"
3) "+role-change"				//
4) "slave 192.168.1.110:6379 192.168.1.110 6379 @ master 192.168.1.110 6381 new reported role is slave"
```







## 故障转移

一次故障转移操作由一下步骤组成：

- 发现主服务器已经进入客观下线状态。
- 对我们当前纪元进行自增，并尝试在这个纪元中当选。
- 如果当选失败，那么在设定的故障迁移超时时间的两倍之后，重新尝试当选。如果当选成功，那么执行一下步骤。
- 选出一个从服务器，并将它升级为主服务器。
- 向被选中的从服务器发送`slaveof no one`命令，让它转变为主服务器。
- 通过发布与订阅功能，将更新后的配置传播给所有其他Sentinel，其他Sentinel对它们的配置进行更新。
- 向已下线主服务器的从服务器发送`slaveof`命令，让它们去复制新的主服务器。
- 当所有从服务器都已经开始复制新的主服务器时，领头Sentinel终止这次故障迁移操作。

每当一个Redis实例被重新配置（reconfigured）—— 无论是被设置成主服务器、从服务器、又或者被设置成其他主服务器的从服务器 —— Sentinel都会向被重新配置的实例发送一个`config rewrite`命令，从而确保这些配置会持久化在硬盘里。

Sentinel使用以下规则来选择新的主服务器：

- 在失效主服务器属下的从服务器中，那些被标记为主观下线、已断线、或者最后一次回复`ping`命令的时间大于5秒钟的从服务器都会被淘汰。
- 在失效主服务器属下的从服务器中，那些与失效主服务器连接断开的时长朝富哦`down-after`选项指定的时长10倍的从服务器都会被淘汰。
- 在经历了以上两轮淘汰之后剩下来的从服务器中，我们选出复制偏移量（replicarion offset）最大的那个从服务器作为新的主服务器；如果复制偏移量不可用，或者从服务器的复制偏移量相同，那么带有最小运行ID的那个从服务器成为新的主服务器。

## Sentinel自动故障迁移的一致性特质

Sentinel自动故障迁移使用Raft算法来选举领头（leader）Sentinel，从而确保在一个给定的的纪元（epoch）里，只有一个领头产生。

这表示在同一个纪元中，不会有两个Sentinel同时被选中为领头，并且各个Sentinel在同一个纪元中只会对一个领头进行投票

更高的配置纪元总是优于较低的纪元，因此每个Sentinel都会主动使用更新的纪元来代替自己的配置。

简单来说，我们可以将Sentinel配置看作是一个带有版本号的状态。一个状态会以最后写入者胜出（last-write-wins）的方式（也即是，最新的配置总是胜出）传播至所有其他Sentinel。

举个例子，当出现网络分裂（network partitions）时，一个Sentinel可能会包含了叫旧的配置，而当这个Sentinel接到其他Sentinel发来的版本更新的配置时，Sentinel就会对自己的配置进行更新。

如果要在网络分割出现的情况下仍然保持一致性，那么应该使用`min-slaves-to-write`选项，让主服务器在连接的从实例少于给定数量时停止执行写操作，与此同时，应该在每个运行Redis主服务器或从服务器的机器上运行Redis Sentinel进程。

## Sentinel状态的持久化

Sentinel的状态会被持久化在Sentinel配置文件里面。

每当Sentinel接收到一个新的配置，或者当领头Sentinel为主服务器创建一个新的配置时，这个配置就会与配置纪元一起被保存到磁盘里面。

这意味着停止和重启Sentinel进程都是安全的。

## Sentinel在非故障迁移的情况下对实例进行重新配置

即使没有自动故障迁移操作在进行，Sentinel总会尝试将当前的配置设置到被监视的实例上面。特别是：

- 根据当前的配置，如果一个从服务器被宣告为主服务器，那么它会代替原有的主服务器，成为新的主服务器，并且成为原有主服务器的所有从服务器的复制对象。那些连接错误主服务器的从服务器会被重新配置，是的这些从服务器会去复制正确的主服务器。

不过，在以上这些条件满足之后，Sentinel在对实例进行重新配置之前仍然会等待一段足够长的时间，确保可以接收到其他Sentinel发来的配置更新，从而避免自身因为保存了过期的配置而对实例进行了不必要的重新配置。

## TILT模式

Redis Sentinel严重依赖计算机的时间功能，比如说，为了判断一个实例是否可用，Sentinel会记录这个实例随后一次相应PING命令的时间，并将这个时间和当前时间进行对比。从而知道这个实例有多长时间没有和Sentinel进行成功通讯。不过，一旦计算机的时间功能出现故障，或者计算机非常忙碌，又或者进程因为某些原因而被阻塞时，Sentinel可能也会跟着出现故障。

TILT模式是一种特殊的保护模式；当Sentinel发现系统有些不对劲时，Sentinel就会进入TILT模式。

因为Sentinel的时间中断器默认每秒执行10次，所以我们预期时间中断器的两次执行之间的间隔为100毫秒左右。Sentinel的做法是，记录上一次时间中断器执行时的时间，并将它和这一次时间中断器执行的时间进行对比：

- 如果两次条用时间之间的差距为负值，或者非常大（超过2秒钟），那么Sentinel进入TILT模式。
- 如果Sentinel已经进入TILT模式，那么Sentinl延迟退出TILT的模式的时间。

当Sentinel进入TILT模式时，它仍然会继续监视所有目标，但是：

- 它不再执行任何操作，比如故障转移。
- 当有实例向这个Sentinel发送`Sentinel is-master-down-by-addr`命令时，Sentinel返回负值，因为这个Sentinel所进行的下线判断依据不再准确。

如果TILT可以正常维持30秒钟，那么Sentinel退出TILT模式。

## 处理-BUSY状态

当Lua脚本的运行时间超过指定的限制时间，那么Redis就会返回-BUSY错误。

当出现这种情况时，Sentinel在尝试执行故障转移操作之前，会先向服务器发送一个`SCRIPT KILL`命令，如果服务器正在执行的是一个只读脚本的话，那么这个脚本就会被杀死，服务器就会回到正常状态。