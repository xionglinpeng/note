# [linux 禁用 swap](https://www.cnblogs.com/whm-blog/p/10920881.html)



一、不重启电脑，禁用启用swap，立刻生效

\# 禁用命令

sudo swapoff -a

 

\# 启用命令

sudo swapon -a

 

\# 查看交换分区的状态

sudo free -m

 

二、重新启动电脑，永久禁用Swap

\# 把根目录文件系统设为可读写

sudo mount -n -o remount,rw /

 

\# 用vi修改/etc/fstab文件，在swap分区这行前加 # 禁用掉，保存退出

vi /etc/fstab

i      #进入insert 插入模式

:wq   #保存退出

 

\# 重新启动电脑，使用free -m查看分区状态

reboot

sudo free -m





# [/proc/sys/net/ipv4/ip_forward](https://www.cnblogs.com/ailx10/p/5535943.html)



```
ip地址分公有地址和私有地址，public address是由INIC(internet network information center)负责，这些ip地址分配给注册并向INIC提出申请的组织机构。通过它访问internet.private address是属于非注册地址，专门为组织内部使用，private ip address是不可能直接用来跟WAN通信的，要么利用帧来通信（FRE帧中继，HDLC,PPP）,要么需要路由的NAT功能把私有地址转换为一个公有ip!
选择一台电脑（有两个网卡或者用单网卡然后用软件虚拟多一个网卡）充当网关，一个网卡(eth0)连接外网ISP，另一网卡(eth1)连接内网(即局域网)。局域网内的ip地址都是私用地址，只能在内部使用，在公网上是不可见的，所以局域网电脑要上网必须修改ip，这就是网关的工作。
工作原理：
内网主机向公网发送数据包时，由于目的主机跟源主机不在同一网段，所以数据包暂时发往内网默认网关处理，而本网段的主机对此数据包不做任何回应。由于源主机ip是私有的，禁止在公网使用，所以必须将数据包的源发送地址修改成公网上的可用ip，这就是网关收到数据包之后首先要做的工作--ip转换。然后网关再把数据包发往目的主机。目的主机收到数据包之后，只认为这是网关发送的请求，并不知道内网主机的存在，也没必要知道，目的主机处理完请求，把回应信息发还给网关。网关收到后，将目的主机发还的数据包的目的ip地址修改为发出请求的内网主机的ip地址，并将其发给内网主机。这就是网关的第二个工作--数据包的路由转发。内网的主机只要查看数据包的目的ip与发送请求的源主机ip地址相同，就会回应，这就完成了一次请求。
 
出于安全考虑，Linux系统默认是禁止数据包转发的。所谓转发即当主机拥有多于一块的网卡时，其中一块收到数据包，根据数据包的目的ip地址将包发往本机另一网卡，该网卡根据路由表继续发送数据包。这通常就是路由器所要实现的功能。
配置Linux系统的ip转发功能，首先保证硬件连通，然后打开系统的转发功能
less /proc/sys/net/ipv4/ip_forward，该文件内容为0，表示禁止数据包转发，1表示允许，将其修改为1。可使用命令echo "1" > /proc/sys/net/ipv4/ip_forward 修改文件内容，重启网络服务或主机后效果不再。若要其自动执行，可将命令echo "1" > /proc/sys/net/ipv4/ip_forward 写入脚本/etc/rc.d/rc.local 或者 在/etc/sysconfig/network脚本中添加 FORWARD_IPV4="YES"
```





因谷歌网络限制问题，国内的K8ser大多数在学习Kubernetes过程中因为镜像下载失败问题间接地产生些许失落感，笔者也因此脑壳疼，故翻阅资料得到以下解决方式：

　　在应用yaml文件创建资源时，将文件中镜像地址进行内容替换即可：

\### k8s.gcr.io 地址替换

　　将k8s.gcr.io替换为

　　registry.cn-hangzhou.aliyuncs.com/google_containers

　　或者

　　registry.aliyuncs.com/google_containers

　　或者

　　mirrorgooglecontainers

\### quay.io 地址替换

　　将 quay.io 替换为 quay-mirror.qiniu.com

\### gcr.io 地址替换

　　将 gcr.io 替换为 registry.aliyuncs.com