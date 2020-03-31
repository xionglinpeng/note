# docker-swarm





## 3、集群模式入门

### 3.1、

### 3.2、创建swarm

在完成前置准备工作之后，就可以创建一个集群了（确保在主机上安装并启动了Docker Engine守护进程）。

1. 打开一个终端，并连接到要运行swarm manager节点的机器上。本教程使用的计算机名是`docker0`。如果使用Docker machine，则可以使用以下命令通过SSH连接manager：

   ```shell
   $ docker-machine ssh docker0
   ```

2. 运行以下命令以创建新的集群：

   ```shell
   $ docker swarm init --advertise-addr <MANAGER-IP>
   ```
   
   执行示例如下：
   
   ```shell
   docker@docker0:~$ docker swarm init --advertise-addr 192.168.56.106
   
   Swarm initialized: current node (tjly64v3wt3iorvxabqd5h4pt) is now a manager.
   
   To add a worker to this swarm, run the following command:
   
       docker swarm join --token SWMTKN-1-3dicxldmq4wthw7owjaliz2jrgpgp04s7uqtv9112wd1n6qto0-b6wqq5phjavcwg1oht2646rod 192.168.56.106:2377
   
   To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
   ```
   
   > - `--advertise-addr`标签配置将地址`192.168.56.106`发布为manager节点。
   >
   > - 输出信息包含了**将新节点加入到集群中**的命令。节点将根据`--token`选项声明的值作为manager或worker加入到集群。

3. 运行`docker info`以查看集群当前的状态。

   为了对比集群创建前后的差异，在集群创建前后分别查看其状态。

   *集群创建前状态：*

   ```shell
   docker@docker0:~$ docker info
    ......
    Swarm: inactive
    ......
   ```

   *集群创建后状态：*

   ```shell
   docker@docker0:~$ docker info
    ......
    Swarm: active
     NodeID: tjly64v3wt3iorvxabqd5h4pt
     Is Manager: true
     ClusterID: 91ucu9xlx87vew18iq1hvpqb4
     Managers: 1
     Nodes: 1
     Default Address Pool: 10.0.0.0/8  
     SubnetSize: 24
     Data Path Port: 4789
     Orchestration:
      Task History Retention Limit: 5
     Raft:
      Snapshot Interval: 10000
      Number of Old Snapshots to Retain: 0
      Heartbeat Tick: 1
      Election Tick: 10
     Dispatcher:
      Heartbeat Period: 5 seconds
     CA Configuration:
      Expiry Duration: 3 months
      Force Rotate: 0
     Autolock Managers: false
     Root Rotation In Progress: false
     Node Address: 192.168.56.106
     Manager Addresses:
      192.168.56.106:2377
     ......
   ```

4. 使用`docker node ls`命令查看集群有关节点信息。

   *集群创建前：*
   
   ```shell
docker@docker0:~$ docker node ls                                      
   Error response from daemon: This node is not a swarm manager. Use "docker swarm init" or "docker swarm join" to connect this node to swarm and try again.
   ```
   
   *集群创建后：*
   
   ```shell
   docker@docker0:~$ docker node ls
   ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
   tjly64v3wt3iorvxabqd5h4pt *   docker0             Ready               Active              Leader              19.03.5
   ```
   
   

### 3.3、添加节点到swarm

一旦创建了一个swarm集群管理节点，就可以向集群中添加工作节点了。

1. 打开终端，连接到工作节点所在的计算机。本教程使用的名称分别是docker1和docker2。

2. 运行`docker swarm join`命令以将当前工作节点加入到集群中。这个命令在`创建swarm`步骤中的`docker searm init`命令的输出结果中已经输出。

   ```shell
   docker@docker1:~$ docker swarm join --token SWMTKN-1-3dicxldmq4wthw7owjaliz2jrgpgp04s7uqtv9112wd1n6qto0-b6wqq5phjavcwg1oht2646rod 192.168.56.106:2377                            
   This node joined a swarm as a worker.
   ```

   如果不能在`docker swarm init`命令的输出结果中找到，则可以通过在manager节点上运行以下命令查找工作节点的连接命令：

   ```shell
   docker@docker0:~$ docker swarm join-token worker
   To add a worker to this swarm, run the following command:
   
       docker swarm join --token SWMTKN-1-3dicxldmq4wthw7owjaliz2jrgpgp04s7uqtv9112wd1n6qto0-b6wqq5phjavcwg1oht2646rod 192.168.56.106:2377
   ```

3. 以此类推，将docker2也加入集群。

4. 打开终端，连接到manager节点所在的计算机，执行`docker node ls`命令查看工作节点的信息。

   ```shell
   docker@docker0:~$ docker node ls
   ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
   tjly64v3wt3iorvxabqd5h4pt *   docker0             Ready               Active              Leader              19.03.5
   54yn6gzsyehr0l8ni8hznmz2c     docker1             Ready               Active                                  19.03.5
   ykf1rwgmjwc6okrcozoirnxyj     docker2             Ready               Active                                  19.03.5
   ```

   MANAGER STATUS列表标识了manager节点和worker节点。

   > 集群管理命令`docker node ls`仅在manager节点生效。























