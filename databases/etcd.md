# Etcd

官方网站：https://etcd.io

Githup：https://github.com/etcd-io/etcd

安装

下载etcd

```shell
$ wget https://github.com/etcd-io/etcd/releases/download/v3.3.18/etcd-v3.3.18-linux-amd64.tar.gz
```

解压

```shell
$ tar -zxvf etcd-v3.3.18-linux-amd64.tar.gz
```

启动

```shell
$ cd etcd-v3.3.18-linux-amd64
总用量 39484
drwxr-xr-x. 10 1000 1000     4096 11月 27 12:40 Documentation
-rwxr-xr-x.  1 1000 1000 22373024 11月 27 12:40 etcd
-rwxr-xr-x.  1 1000 1000 17991872 11月 27 12:40 etcdctl
-rw-r--r--.  1 1000 1000    38864 11月 27 12:40 README-etcdctl.md
-rw-r--r--.  1 1000 1000     7262 11月 27 12:40 README.md
-rw-r--r--.  1 1000 1000     7855 11月 27 12:40 READMEv2-etcdctl.md
$ ./etcd
2020-02-23 12:16:52.993930 I | etcdmain: etcd Version: 3.3.18
2020-02-23 12:16:52.994049 I | etcdmain: Git SHA: 3c8740a79
2020-02-23 12:16:52.994055 I | etcdmain: Go Version: go1.12.9
2020-02-23 12:16:52.994059 I | etcdmain: Go OS/Arch: linux/amd64
2020-02-23 12:16:52.994064 I | etcdmain: setting maximum number of CPUs to 1, total number of available CPUs is 1
2020-02-23 12:16:52.994073 W | etcdmain: no data-dir provided, using default data-dir ./default.etcd
2020-02-23 12:16:52.996540 I | embed: listening for peers on http://localhost:2380
2020-02-23 12:16:52.996780 I | embed: listening for client requests on localhost:2379
2020-02-23 12:16:53.045152 I | etcdserver: name = default
2020-02-23 12:16:53.045167 I | etcdserver: data dir = default.etcd
2020-02-23 12:16:53.045173 I | etcdserver: member dir = default.etcd/member
2020-02-23 12:16:53.045177 I | etcdserver: heartbeat = 100ms
2020-02-23 12:16:53.045181 I | etcdserver: election = 1000ms
2020-02-23 12:16:53.045185 I | etcdserver: snapshot count = 100000
2020-02-23 12:16:53.045199 I | etcdserver: advertise client URLs = http://localhost:2379
2020-02-23 12:16:53.045204 I | etcdserver: initial advertise peer URLs = http://localhost:2380
2020-02-23 12:16:53.045212 I | etcdserver: initial cluster = default=http://localhost:2380
2020-02-23 12:16:53.083042 I | etcdserver: starting member 8e9e05c52164694d in cluster cdf818194e3a8c32
2020-02-23 12:16:53.083070 I | raft: 8e9e05c52164694d became follower at term 0
2020-02-23 12:16:53.083083 I | raft: newRaft 8e9e05c52164694d [peers: [], term: 0, commit: 0, applied: 0, lastindex: 0, lastterm: 0]
2020-02-23 12:16:53.083088 I | raft: 8e9e05c52164694d became follower at term 1
2020-02-23 12:16:53.092400 W | auth: simple token is not cryptographically signed
2020-02-23 12:16:53.097597 I | etcdserver: starting server... [version: 3.3.18, cluster version: to_be_decided]
2020-02-23 12:16:53.100561 I | etcdserver: 8e9e05c52164694d as single-node; fast-forwarding 9 ticks (election ticks 10)
2020-02-23 12:16:53.101505 I | etcdserver/membership: added member 8e9e05c52164694d [http://localhost:2380] to cluster cdf818194e3a8c32
2020-02-23 12:16:53.684140 I | raft: 8e9e05c52164694d is starting a new election at term 1
2020-02-23 12:16:53.684163 I | raft: 8e9e05c52164694d became candidate at term 2
2020-02-23 12:16:53.684175 I | raft: 8e9e05c52164694d received MsgVoteResp from 8e9e05c52164694d at term 2
2020-02-23 12:16:53.684185 I | raft: 8e9e05c52164694d became leader at term 2
2020-02-23 12:16:53.684191 I | raft: raft.node: 8e9e05c52164694d elected leader 8e9e05c52164694d at term 2
2020-02-23 12:16:53.684689 I | etcdserver: setting up the initial cluster version to 3.3
2020-02-23 12:16:53.684736 I | etcdserver: published {Name:default ClientURLs:[http://localhost:2379]} to cluster cdf818194e3a8c32
2020-02-23 12:16:53.685215 E | etcdmain: forgot to set Type=notify in systemd service file?
2020-02-23 12:16:53.685225 I | embed: ready to serve client requests
2020-02-23 12:16:53.686286 N | embed: serving insecure client requests on 127.0.0.1:2379, this is strongly discouraged!
2020-02-23 12:16:53.687118 N | etcdserver/membership: set the initial cluster version to 3.3
2020-02-23 12:16:53.687183 I | etcdserver/api: enabled capabilities for version 3.3
```







## Etcd UI

e3w是由githup上由一个网名叫soyking的人开发的一个针对Etcd的UI管理界面，可以直观的看到Etcd集群的成员以及key和value，用于平时的开发非常方便。

项目中提供了一个[docker-compose.yml](https://github.com/soyking/e3w/blob/master/docker-compose.yml)文件，直接通过`docker-compose up`命令即可启动一个Etcd伪集群和当前UI实例，通过`http://host:8080`即可访问。

docker-compose.yml内容如下：

```yaml
version: '3'

services:
  etcd:
    image: soyking/etcd-goreman:3.2.7
    environment:
      - CLIENT_ADDR=etcd
  e3w:
    image: soyking/e3w:latest
    volumes:
      - ./conf/config.default.ini:/app/conf/config.default.ini
    ports:
      - "8080:8080"
    depends_on:
      - etcd
```

其中依赖了一个名为`soyking/etcd-goreman:3.2.7`的镜像，这个镜像也是有soyking开发的，这就是那个Etcd伪集群，其源码位于githup：https://github.com/soyking/eg。

Githup地址：https://github.com/soyking/e3w