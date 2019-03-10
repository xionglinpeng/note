# Redis-Sentinel

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

## 自动发送Sentinel和从服务器

一个Sentinel可以与其他多个Sentinel进行连接，各个Sentinel之间可以互相检查对方的可用性，并进行信息交换。你无须为运行的每个Sentinel分别设置其他Sentinel的地址，因为Sentinel可以通过发布与订阅功能来自动发现正在监视相同主服务器的其他Sentinel，这一功能是通过向频道`__sentinel__:hello`发送信息来实现的。

与此类似，你也不必手动列出主服务器属下的所有从服务器，因为Sentinel可以通过询问主服务器来获得所有从服务器的信息。

- 每个Sentinel会以每两秒一次的频率，通过发布与订阅功能，向被它监视的所有主服务器和从服务器的`__sentinel__:hello`频道发送一条信息，信息中包含了Sentinel的IP地址、端口号和运行ID（runid）。
- 每个Sentinel都订阅了被它监视的所有主服务器和从服务器的`__sentinel__:hello`频道，查找之前未出现过的sentinel（looking for unknown sentinels）。当一个sentinel发现一个新的sentinel时，他会将新的sentinel添加到一个列表中，这个列表保存sentinel已知的，监视同一个主服务器的所有其他sentinel。
- Sentinel发送的信息中还包括完整的主服务器当前配置（configuration）。如果一个Sentinel包含的主服务器配置比另一个Sentinel发送的配置要旧，那么这个Sentinel会立即升级到新配置上。
- 在将一个新Sentinel添加到监视主服务器的列表上面之前，Sentinel会先检查列表中是否已经包含了和要添加的Sentinel拥有相同运行ID或者相同地址（包括IP地址和端口号）的Sentinel，如果是的话，Sentinel会先移除列表中已有的那些用哟与相同运行ID或者相同地址的Sentinel，然后再添加新的Sentinel。

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