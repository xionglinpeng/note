# 111



```shell
[root@localhost ~]# ping baidu.com
ping: baidu.com: 未知的名称或服务
```







```shell
[root@localhost ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.56.1    0.0.0.0         UG    0      0        0 enp0s8
10.0.2.0        0.0.0.0         255.255.255.0   U     0      0        0 enp0s3
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 enp0s3
169.254.0.0     0.0.0.0         255.255.0.0     U     1003   0        0 enp0s8
192.168.56.0    0.0.0.0         255.255.255.0   U     0      0        0 enp0s8
```







```shell
[root@localhost ~]# route del default
[root@localhost ~]# route add default gw 10.0.2.1 dev enp0s3
[root@localhost ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.2.1        0.0.0.0         UG    0      0        0 enp0s3
10.0.2.0        0.0.0.0         255.255.255.0   U     0      0        0 enp0s3
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 enp0s3
169.254.0.0     0.0.0.0         255.255.0.0     U     1003   0        0 enp0s8
192.168.56.0    0.0.0.0         255.255.255.0   U     0      0        0 enp0s8
[root@localhost ~]# ping baidu.com
PING baidu.com (39.156.69.79) 56(84) bytes of data.
64 bytes from 39.156.69.79 (39.156.69.79): icmp_seq=1 ttl=45 time=67.1 ms
64 bytes from 39.156.69.79 (39.156.69.79): icmp_seq=2 ttl=45 time=56.5 ms
......
```

rc.local