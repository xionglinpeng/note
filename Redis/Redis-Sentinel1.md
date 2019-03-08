



Redis的Sentinel系统用于管理多个Redis服务器（instance），该系统执行以下三个任务：

- **监控（Monitoring）：**Sentinel会不断地检查你的主服务器和从服务器是否运作正常。
- **提醒（Notification）：**当被监控的某个Redis服务器出现问题时，Sentinel可以通过API向管理员或者其他应用程序发送通知。
- **自动故障迁移（Automatic failover）：**当一个主服务器不能正常工作时，Sentinel会开始一次自动故障迁移操作，它会将失效主服务器的其中一个从服务器升级为新的主服务器，并让失效主服务器的其他从服务器改为复制新的主服务器；当客户端试图连接失效的主服务器时，集群也会向客户端返回新主服务器的地址，使得集群可以使用新主服务器带起代替服务器。



## 获取Sentinel



## 启动Sentinel



## 配置Sentinel



## 主观下线和客观下线



## 每个Sentinel都需要定期执行的任务



## 自动发现Sentinel和从服务器



## Sentinel命令



## Sentinel API



## 发布与订阅信息



## Sentinel自动故障迁移的一致性特质

Sentinel自动故障迁移使用Raft算法来选举领头（leader）Sentinel，从而确保在一个给定的的纪元（epoch）里，只有一个领头产生。

这表示在同一个纪元中，不会有两个Sentinel同时被选中为领头，并且各个Sentinel在同一个纪元中只会对一个领头进行投票

更高的配置纪元总是优于较低的纪元，因此每个Sentinel都会主动使用更新的纪元来代替自己的配置。

简单磊说，我们可以将Sentinel配置看作是一个带有版本号的状态。一个状态会以最后写入者胜出（last-write-wins）的方式（也即是，最新的配置总是胜出）传播至所有其他Sentinel。

举个例子，当出现网络分裂（network partitions）时，一个Sentinel可能会包含了叫旧的配置，而当这个Sentinel接到其他Sentinel发来的版本更新的配置时，Sentinel就会对自己的配置进行更新。

如果要在网络分割出现的情况下仍然保持一致性，那么应该使用min-slaves-to-write选项，让主服务器在连接的从实例少于给定数量时停止执行写操作，与此同时，应该在每个运行Redis主服务器或从服务器的机器上运行Redis Sentinel进程。





## Sentinel状态的持久化

Sentinel的状态会被持久化在Sentinel配置文件里面。

每当Sentinel接收到一个新的配置，或者当领头Sentinel为主服务器创建一个新的配置时，这个配置就会与配置纪元一起被保存到磁盘里面。

这意味着停止和重启Sentinel进程都是安全的。





## Sentinel在非故障迁移的情况下对实例进行重新配置



## TILT模式



## 处理-BUSY状态

