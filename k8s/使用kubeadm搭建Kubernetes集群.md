# 使用kubeadm搭建Kubernetes集群



k8s官方已经提供了详细的kubeadm安装教程，只需要顺着它给出的教程安装即可完成。如果您看了k8s官方提供的kubeadm安装教程，那么你就会知道，kubeadm的安装本身是很简单的，只需要添加相应的yum源，然后在install一下就可以了，但是这里有一个问题是k8s官方提供的kubeadm安装教程中在实际安装kubeadm之前做了很多前置的准备工作，如果我们仅仅是顺着给出的教程安装，就不能知道当缺失某些准备工作时，kubeadm会有什么样的表现，所以为了更好的学习，我们将略过准备工作，直接安装kubeadm，然后一步步的去解决缺失的前置准备，最后会将其汇总为一个完整的流程。



## 1、添加kubeadm的安装源

```shell
$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

注意到上述添加的k8s的yum源的地址是`packages.cloud.google.com`，因为网络的原因，不能直接访问。幸好，阿里云提供了针对k8s的镜像源，命令如下：

```shell
$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

> 如果你对上述阿里云的kubernetes镜像源有疑虑，可以访问[阿里云镜像源](https://developer.aliyun.com/mirror/)，选择`容器`—>`kubernetes`即可看到相应的镜像配置，如下：
>
> ![1586579183856](images\1586579183856.png)





## 2、安装kubelet kubeadm kubectl

```shell
$ yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```





## 3、初始化kubernetes集群

### 第一次初始化（错误）

执行`kubeadm init`命令初始化kubernetes集群



```shell
[root@localhost ~]# kubeadm init
W0411 11:31:32.914387   12263 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.18.1
[preflight] Running pre-flight checks
[preflight] WARNING: Couldn't create the interface used for talking to the container runtime: docker is required for container runtime: exec: "docker": executable file not found in $PATH
	[WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly
	[WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR NumCPU]: the number of available CPUs 1 is less than the required 2
	[ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist
	[ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
	[ERROR Swap]: running with swap on is not supported. Please disable swap
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```

安装失败，提示了很多`WARNING`和`ERROR`信息



1. WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
2. WARNING: Couldn't create the interface used for talking to the container runtime: docker is required for container runtime: exec: "docker": executable file not found in $PATH
3. [WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly
4. [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
5. [ERROR NumCPU]: the number of available CPUs 1 is less than the required 2
6. [ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist
7. [ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
8. [ERROR Swap]: running with swap on is not supported. Please disable swap



总共是4个警告信息和4个ERROR信息，其中WARNING信息并不影响kubeadm初始化集群的，而ERROR信息是一些致命的错误，也就是说必须解决他们，才能初始化kubernetes集群。现在依次来解决所有的WARNING信息和ERROR信息。

1. aa

2. 这个警告信息是告诉你没有在$PATH路径中找到docker的可执行文件，也就是说环境中必须安装docker。

   关于docker的安装在里

3. 这个警告信息是告诉你firewalld（防火墙）是活动的，在防火墙开启的状态下需要确保`6443`和`10250`这两个端口是可以访问的，不然集群可能无法正常工作，所以这里直接关闭防火墙，如下：

   ```shell
   # 查看防火墙状态（关闭后显示not running，开启后显示running）
   $ firewall-cmd --state
   running
   
   # 停止防火墙
   $ systemctl stop firewalld.service
   
   # 再次查看防火墙状态
   $ firewall-cmd --state
   not running
   
   #禁止防火墙开机启动
   $ systemctl disable firewalld.service
   Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
   Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
   ```

4. kubelet服务未启用，运行如下命令启用。

   ```shell
   $ systemctl enable kubelet.service && systemctl start kubelet
   Created symlink from /etc/systemd/system/multi-user.target.wants/kubelet.service to /usr/lib/systemd/system/kubelet.service.
   ```

5. 在kubernetes的官网，已经很明确的告诉我们，运行kubernetes集群的每个节点至少需要2核CPU，2G内存。

6. 提示`/proc/sys/net/bridge/bridge-nf-call-iptables`不存在，执行如下命令创建

   ```shell
   # 执行此命令创建/proc/sys/net/bridge/bridge-nf-call-iptables
   $ modprobe br_netfilter 
   [root@localhost ~]# cd /proc/sys/net/
   [root@localhost net]# ll
   总用量 0
   dr-xr-xr-x. 1 root root 0 4月  11 12:57 bridge
   dr-xr-xr-x. 1 root root 0 4月  11 12:22 core
   dr-xr-xr-x. 1 root root 0 4月  11 12:13 ipv4
   dr-xr-xr-x. 1 root root 0 4月  11 12:13 ipv6
   dr-xr-xr-x. 1 root root 0 4月  11 12:13 netfilter
   dr-xr-xr-x. 1 root root 0 4月  11 12:13 unix
   [root@localhost net]# cd bridge/
   [root@localhost bridge]# ll
   总用量 0
   -rw-r--r--. 1 root root 0 4月  11 12:58 bridge-nf-call-arptables
   -rw-r--r--. 1 root root 0 4月  11 12:58 bridge-nf-call-ip6tables
   -rw-r--r--. 1 root root 0 4月  11 12:58 bridge-nf-call-iptables
   -rw-r--r--. 1 root root 0 4月  11 12:58 bridge-nf-filter-pppoe-tagged
   -rw-r--r--. 1 root root 0 4月  11 12:58 bridge-nf-filter-vlan-tagged
   -rw-r--r--. 1 root root 0 4月  11 12:58 bridge-nf-pass-vlan-input-dev
   [root@localhost bridge]# echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
   [root@localhost bridge]# cat /proc/sys/net/bridge/bridge-nf-call-iptables
   1
   ```

7. 提示`/proc/sys/net/ipv4/ip_forward`内容没有设置为`1`，所以将其设置为`1`即可，

   ```shell
   calico# 先查看/proc/sys/net/ipv4/ip_forward
   $ cat /proc/sys/net/ipv4/ip_forward
   0
   
   # 修改为1
   $ echo "1" > /proc/sys/net/ipv4/ip_forward
   
   # 再次查看
   $ cat /proc/sys/net/ipv4/ip_forward
   1
   ```

8. 不支持运行swap，请禁用swap，执行如下命令禁用。

   使用`free -m`查看交换分区状态

   ```shell
   $ free -m
                 total        used        free      shared  buff/cache   available
   Mem:           2845         117        2254           8         473        2576
   Swap:          3071           0        3071
   ```

   可以看到Swap行的total列总大小是3071KB。

   执行命令`swapoff -a`禁用Swap，然后再次查看

   ```shell
   $ swapoff -a
   $ free -m
                 total        used        free      shared  buff/cache   available
   Mem:           2845         116        2255           8         473        2576
   Swap:             0           0           0
   ```

   Swap行的total列总大小已经变为0了。

   `swapoff -a`仅仅是关闭了交换分区，当时在重启主机的时候Swap还是会启动，所以直接永久关闭Swap。编辑`/etc/fstab`文件，将swap一行注释掉即可。

   ```shell
   # 查看/etc/fstab
   $ cat /etc/fstab
   # ......
   /dev/mapper/centos-root /                       xfs     defaults        0 0
   UUID=76f0f48e-67e8-43b3-b96a-cbad8d1956c5 /boot                   xfs     defaults        0 0
   /dev/mapper/centos-swap swap                    swap    defaults        0 0
   
   # 编辑
   $ vim /etc/fstab
   
   # 再次查看
   $ cat /etc/fstab
   # ......
   /dev/mapper/centos-root /                       xfs     defaults        0 0
   UUID=76f0f48e-67e8-43b3-b96a-cbad8d1956c5 /boot                   xfs     defaults        0 0
   #/dev/mapper/centos-swap swap                    swap    defaults        0 0
   ```

### 第二次初始化（错误）

现在已经解决了所有的警告个错误，再次初始化kubernetes集群：

```shell
[root@localhost bridge]# kubeadm init
W0411 13:04:27.730336   11108 version.go:102] could not fetch a Kubernetes version from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.txt": Get https://dl.k8s.io/release/stable-1.txt: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
W0411 13:04:27.730392   11108 version.go:103] falling back to the local client version: v1.18.1
W0411 13:04:27.730507   11108 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.18.1
[preflight] Running pre-flight checks
	[WARNING Service-Docker]: docker service is not enabled, please run 'systemctl enable docker.service'
[preflight] The system verification failed. Printing the output from the verification:
KERNEL_VERSION: 3.10.0-1062.18.1.el7.x86_64
CONFIG_NAMESPACES: enabled
CONFIG_NET_NS: enabled
CONFIG_PID_NS: enabled
CONFIG_IPC_NS: enabled
CONFIG_UTS_NS: enabled
CONFIG_CGROUPS: enabled
CONFIG_CGROUP_CPUACCT: enabled
CONFIG_CGROUP_DEVICE: enabled
CONFIG_CGROUP_FREEZER: enabled
CONFIG_CGROUP_SCHED: enabled
CONFIG_CPUSETS: enabled
CONFIG_MEMCG: enabled
CONFIG_INET: enabled
CONFIG_EXT4_FS: enabled (as module)
CONFIG_PROC_FS: enabled
CONFIG_NETFILTER_XT_TARGET_REDIRECT: enabled (as module)
CONFIG_NETFILTER_XT_MATCH_COMMENT: enabled (as module)
CONFIG_OVERLAY_FS: enabled (as module)
CONFIG_AUFS_FS: not set - Required for aufs.
CONFIG_BLK_DEV_DM: enabled (as module)
OS: Linux
CGROUPS_CPU: enabled
CGROUPS_CPUACCT: enabled
CGROUPS_CPUSET: enabled
CGROUPS_DEVICES: enabled
CGROUPS_FREEZER: enabled
CGROUPS_MEMORY: enabled
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR CRI]: container runtime is not running: output: Client:
 Debug Mode: false

Server:
ERROR: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
errors pretty printing info
, error: exit status 1
	[ERROR Service-Docker]: docker service is not active, please run 'systemctl start docker.service'
	[ERROR IsDockerSystemdCheck]: cannot execute 'docker info -f {{.CgroupDriver}}': exit status 2
	[ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1
	[ERROR SystemVerification]: error verifying Docker info: "Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?"
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```



还是失败，出现了新的错误：

1. [WARNING Service-Docker]: docker service is not enabled, please run 'systemctl enable docker.service'
2. ERROR: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
3. [ERROR Service-Docker]: docker service is not active, please run 'systemctl start docker.service'
4. [ERROR IsDockerSystemdCheck]: cannot execute 'docker info -f {{.CgroupDriver}}': exit status 2
5. [ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1
6. [ERROR SystemVerification]: error verifying Docker info: "Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?"

错误又是好些个，仔细查看没有错误信息，可以发现除了第5个之外，其他所描述的都是同一件事：docker服务（自）启动。至于第5个，再看到这个相比应该很熟悉，就是我们之前使用`modprobe br_netfilter `命令创建的。

解决方式如下：

1. 启动docker

   ```shell
   # 启动docker
   $ systemctl start docker
   
   # 设置docker自启动
   [root@localhost bridge]# systemctl enable docker.service
   Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
   ```

3. 设置`bridge-nf-call-iptables`内容为`1`

   ```shell
   $ echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
   ```


### 第三次初始化（错误）

再次初始化

```shell
[root@localhost bridge]# kubeadm init
W0411 13:07:41.802019   11589 version.go:102] could not fetch a Kubernetes version from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.txt": Get https://storage.googleapis.com/kubernetes-release/release/stable-1.txt: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
W0411 13:07:41.802106   11589 version.go:103] falling back to the local client version: v1.18.1
W0411 13:07:41.802247   11589 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.18.1
[preflight] Running pre-flight checks
	[WARNING Service-Docker]: docker service is not enabled, please run 'systemctl enable docker.service'
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/kube-apiserver:v1.18.1: output: Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
, error: exit status 1
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/kube-controller-manager:v1.18.1: output: Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
, error: exit status 1
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/kube-scheduler:v1.18.1: output: Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
, error: exit status 1
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/kube-proxy:v1.18.1: output: Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
, error: exit status 1
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/pause:3.2: output: Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
, error: exit status 1
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/etcd:3.4.3-0: output: Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
, error: exit status 1
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/coredns:1.6.7: output: Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
, error: exit status 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```



还有错误信息，继续解决

1. [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
2. [ERROR ImagePull]: failed to pull image k8s.gcr.io/kube-apiserver:v1.18.1: output: Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)



1. 第一个错误信息与cgroupfs相关，官方给出了一个地址，这里描述了解决方案

   编辑/etc/docker/daemon.json

   ```shell
   [root@localhost ~]# vim /etc/docker/daemon.json 
   [root@localhost ~]# systemctl restart docker
   ```

   添加内容`native.cgroupdriver=systemd`，添加之后内容如下：

   ```shell
   {
     "exec-opts": ["native.cgroupdriver=systemd"],
     "registry-mirrors": ["https://wk4vahax.mirror.aliyuncs.com"]
   }
   ```

   重启docker

   ```shell
   $ systemctl restart docker
   ```

   

2. 这个错误就是拉取容器镜像`k8s.gcr.io/...`拉不下来，不用多说，还是网络问题，针对这个问题需要切换到国内的镜像源，还好的是阿里云已经为广大开发者提供了k8s的容器镜像，所以我们只需要在使用kubeadm初始化kunernetes容器集群的时候，使用`--image-repository `选项指定容器镜像源即可。

   ```shell
   $ kubeadm init --image-repository registry.aliyuncs.com/google_containers
   ```

   > 说明一下国内的容器镜像源

### 第四次初始化（成功）

```shell
[root@localhost bridge]# kubeadm init --image-repository registry.aliyuncs.com/google_containers
W0411 13:56:54.960948   14966 version.go:102] could not fetch a Kubernetes version from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.txt": Get https://dl.k8s.io/release/stable-1.txt: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
W0411 13:56:54.961293   14966 version.go:103] falling back to the local client version: v1.18.1
W0411 13:56:54.961445   14966 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.18.1
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [localhost.localdomain kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.0.2.7]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [localhost.localdomain localhost] and IPs [10.0.2.7 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [localhost.localdomain localhost] and IPs [10.0.2.7 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
W0411 13:59:21.534596   14966 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
W0411 13:59:21.535699   14966 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 28.003058 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.18" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node localhost.localdomain as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node localhost.localdomain as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: 7gd349.kwh3ehctgh9eyvks
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.0.2.7:6443 --token 7gd349.kwh3ehctgh9eyvks \
    --discovery-token-ca-cert-hash sha256:d8cd38193124955199443851777de51b47902ede1f7b1e13890922871519e23d
```



## 4、集群初始化成功日志分析

第四次初始化的时候，kubernetes集群初始化成功了，分析一下上面日志信息。



1. 运行初始化前的检查。

   preflight中文意思是`adj. 起飞前的；为起飞作准备的`，在这里就类似于飞机起飞前的检查。

2. 获取建立Kubernetes集群所需的镜像。

3. 这可能需要一到两分钟，取决于你的互联网连接速度

4. 您还可以在使用'kubeadm config images pull'之前执行此操作

5. 



[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [localhost.localdomain kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.0.2.7]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [localhost.localdomain localhost] and IPs [10.0.2.7 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [localhost.localdomain localhost] and IPs [10.0.2.7 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file





Your Kubernetes control-plane has initialized successfully!



要开始使用您的集群，您需要作为一个普通用户运行以下程序:

```shell
[root@localhost ~]# docker images
REPOSITORY                                                        TAG                 IMAGE ID            CREATED             SIZE
registry.aliyuncs.com/google_containers/kube-proxy                v1.18.1             4e68534e24f6        2 days ago          117MB
registry.aliyuncs.com/google_containers/kube-apiserver            v1.18.1             a595af0107f9        2 days ago          173MB
registry.aliyuncs.com/google_containers/kube-controller-manager   v1.18.1             d1ccdd18e6ed        2 days ago          162MB
registry.aliyuncs.com/google_containers/kube-scheduler            v1.18.1             6c9320041a7b        2 days ago          95.3MB
registry.aliyuncs.com/google_containers/pause                     3.2                 80d28bedfe5d        8 weeks ago         683kB
registry.aliyuncs.com/google_containers/coredns                   1.6.7               67da37a9a360        2 months ago        43.8MB
registry.aliyuncs.com/google_containers/etcd                      3.4.3-0             303ce5db0e90        5 months ago        288MB
hello-world                                                       latest              fce289e99eb9        15 months ago       1.84kB
```





```shell
CONTAINER ID        IMAGE                                               COMMAND                  CREATED             STATUS              PORTS               NAMES
5f684df675c7        4e68534e24f6                                        "/usr/local/bin/kube…"   3 minutes ago       Up 3 minutes                            k8s_kube-proxy_kube-proxy-rff9n_kube-system_746a04e5-3ae6-46eb-a770-7048ec788282_0
62d404c309db        registry.aliyuncs.com/google_containers/pause:3.2   "/pause"                 3 minutes ago       Up 3 minutes                            k8s_POD_kube-proxy-rff9n_kube-system_746a04e5-3ae6-46eb-a770-7048ec788282_0
153a586fe31b        d1ccdd18e6ed                                        "kube-controller-man…"   3 minutes ago       Up 3 minutes                            k8s_kube-controller-manager_kube-controller-manager-localhost.localdomain_kube-system_52ec17d08746eba23e8296d616664d49_0
20d33bc10e97        6c9320041a7b                                        "kube-scheduler --au…"   3 minutes ago       Up 3 minutes                            k8s_kube-scheduler_kube-scheduler-localhost.localdomain_kube-system_2c04fc5e4761bd2ada4d5c31bd4317ad_0
6bb23482cb79        303ce5db0e90                                        "etcd --advertise-cl…"   3 minutes ago       Up 3 minutes                            k8s_etcd_etcd-localhost.localdomain_kube-system_52070e21722e46e21e7a2dae43aedd7f_0
1845c430fd7a        a595af0107f9                                        "kube-apiserver --ad…"   3 minutes ago       Up 3 minutes                            k8s_kube-apiserver_kube-apiserver-localhost.localdomain_kube-system_ee0b96247ba5a4f1db7a76ae291bdb74_0
d45a8e8ada8c        registry.aliyuncs.com/google_containers/pause:3.2   "/pause"                 3 minutes ago       Up 3 minutes                            k8s_POD_etcd-localhost.localdomain_kube-system_52070e21722e46e21e7a2dae43aedd7f_0
669dbc5d4935        registry.aliyuncs.com/google_containers/pause:3.2   "/pause"                 3 minutes ago       Up 3 minutes                            k8s_POD_kube-scheduler-localhost.localdomain_kube-system_2c04fc5e4761bd2ada4d5c31bd4317ad_0
7897b9b803e4        registry.aliyuncs.com/google_containers/pause:3.2   "/pause"                 3 minutes ago       Up 3 minutes                            k8s_POD_kube-controller-manager-localhost.localdomain_kube-system_52ec17d08746eba23e8296d616664d49_0
1765826a9521        registry.aliyuncs.com/google_containers/pause:3.2   "/pause"                 3 minutes ago       Up 3 minutes                            k8s_POD_kube-apiserver-localhost.localdomain_kube-system_ee0b96247ba5a4f1db7a76ae291bdb74_0
```



查看集群中有哪些节点

```shell
[root@localhost bridge]# kubectl get node
error: no configuration has been provided, try setting KUBERNETES_MASTER environment variable
```

报错，这个错误的原因是。。。。。

执行初始化日志中的命令

```shell
[root@localhost bridge]# mkdir -p $HOME/.kube
[root@localhost bridge]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@localhost bridge]# sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

再次查看集群中有哪些节点

```shell
[root@localhost bridge]# kubectl get node
NAME                    STATUS     ROLES    AGE     VERSION
localhost.localdomain   NotReady   master   8m56s   v1.18.1
```

可以看到，当前只有一个master节点



## 5、添加工作节点

官网`xxx`明确告诉我们所有节点都需要安装kubelet kubeadm kubectl，这是因为。。。。。



安装`kubelet`、`kubeadm`、 `kubectl`，与第一、二步是一样，需要先添加镜像源，然后安装。

```shell
$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

$ yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```



### 第一次添加（错误）

进入到work节点主机，执行kunernetes初始化时所打印的添加节点到集群的命令。

```shell
[root@localhost ~]# kubeadm join 10.0.2.7:6443 --token 7gd349.kwh3ehctgh9eyvks \
>     --discovery-token-ca-cert-hash sha256:d8cd38193124955199443851777de51b47902ede1f7b1e13890922871519e23d
W0411 16:32:43.905915    2173 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR Swap]: running with swap on is not supported. Please disable swap
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher

```

报错，同样需要设置native.cgroupdriver=systemd和关闭Swap分区，解决方式跟master节点初始化时一致。

### 第二次添加（错误）

解决之后再次添加节点到集群

```shell
[root@localhost ~]# kubeadm join 10.0.2.7:6443 --token 7gd349.kwh3ehctgh9eyvks     --discovery-token-ca-cert-hash sha256:d8cd38193124955199443851777de51b47902ede1f7b1e13890922871519e23d
W0411 16:33:58.490328    2330 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
error execution phase kubelet-start: a Node with name "localhost.localdomain" and status "Ready" already exists in the cluster. You must delete the existing Node or change the name of this new joining Node
To see the stack trace of this error execute with --v=5 or higher

```

还是报错，不过这次提示error execution phase kubelet-start: a Node with name "localhost.localdomain" and status "Ready" already exists in the cluster. You must delete the existing Node or change the name of this new joining Node

集群中已经存在名为“xxx”和状态为“Ready”的节点。您必须删除现有节点或更改此新连接节点的名称

所以我们更改当前工作节点的名称为work1，执行命令如下：


```shell
$ hostnamectl set-hostname work1
```

### 第三次添加（错误）

解决之后再次添加节点到集群：

```shell
[root@localhost ~]# kubeadm join 10.0.2.7:6443 --token 7gd349.kwh3ehctgh9eyvks     --discovery-token-ca-cert-hash sha256:d8cd38193124955199443851777de51b47902ede1f7b1e13890922871519e23d
W0411 16:35:32.808484    2665 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```

报错：error execution phase preflight: [preflight] Some fatal errors occurred:
[ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists

这个原因是因为第二次添加节点到集群的时候，虽然因为主机名称的原因添加失败，但是已经有一部分相关配置已经创建成功，所以我们需要将其重置，操作如下：

```shell
$ kubeadm reset
[reset] WARNING: Changes made to this host by 'kubeadm init' or 'kubeadm join' will be reverted.
[reset] Are you sure you want to proceed? [y/N]: y
[preflight] Running pre-flight checks
W0411 16:46:11.343381    3324 removeetcdmember.go:79] [reset] No kubeadm config, using etcd pod spec to get data directory
[reset] No etcd config found. Assuming external etcd
[reset] Please, manually reset etcd to prevent further issues
[reset] Stopping the kubelet service
[reset] Unmounting mounted directories in "/var/lib/kubelet"
W0411 16:46:11.352167    3324 cleanupnode.go:99] [reset] Failed to evaluate the "/var/lib/kubelet" directory. Skipping its unmount and cleanup: lstat /var/lib/kubelet: no such file or directory
[reset] Deleting contents of config directories: [/etc/kubernetes/manifests /etc/kubernetes/pki]
[reset] Deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]
[reset] Deleting contents of stateful directories: [/var/lib/dockershim /var/run/kubernetes /var/lib/cni]

The reset process does not clean CNI configuration. To do so, you must remove /etc/cni/net.d

The reset process does not reset or clean up iptables rules or IPVS tables.
If you wish to reset iptables, you must do so manually by using the "iptables" command.

If your cluster was setup to utilize IPVS, run ipvsadm --clear (or similar)
to reset your system's IPVS tables.

The reset process does not clean your kubeconfig files and you must remove them manually.
Please, check the contents of the $HOME/.kube/config file.
```

### 第四次添加（成功）

重置之后添加节点到集群：

```shell
[root@localhost ~]# kubeadm join 10.0.2.7:6443 --token 7gd349.kwh3ehctgh9eyvks     --discovery-token-ca-cert-hash sha256:d8cd38193124955199443851777de51b47902ede1f7b1e13890922871519e23d
W0411 16:53:37.705668    3484 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

添加成功，查看集群节点：

```shell
[root@localhost ~]# kubectl get node
NAME                    STATUS     ROLES    AGE    VERSION
localhost.localdomain   Ready      master   174m   v1.18.1
work1                   NotReady   <none>   28s    v1.18.1
```

可以看到已经添加了一个工作节点了。

```shell
[root@localhost ~]# kubectl get node
NAME                    STATUS   ROLES    AGE     VERSION
localhost.localdomain   Ready    master   176m    v1.18.1
work1                   Ready    <none>   2m15s   v1.18.1
```

依次类推，在将另一个节点也添加到集群中。

## 6、安装calico网络

```shell
$ kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers create
```



**calico网络插件报错：**

Number of node(s) with BGP peering established = 1

查看到日志报错：

bird: BGP: Unexpected connect from unknown address 10.0.0.7

解决方案：

在calico.yaml文件中添加如下内容

```yaml
# Specify interface
- name: IP_AUTODETECTION_METHOD
value: "interface=enp0s8"
```

添加位置参考如下上下文：

```yaml
......

# Cluster type to identify the deployment type
- name: CLUSTER_TYPE
value: "k8s,bgp"
# Specify interface
- name: IP_AUTODETECTION_METHOD
value: "interface=enp0s8"
# Auto-detect the BGP IP address.
- name: IP
value: "autodetect"

......
```





## 6、集群测试

kuberentes集群虽然搭建成功了，但是还是不能确定是否能够成功的部署服务，所以测试一下：

创建nginx服务如下：

```shell
$ cat > pod_nginx-rs.yaml << EOF
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx
  labels:
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      name: nginx
      labels:
        tier: frontend
    spec:
      container:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
EOF
```

应用yaml文件，创建服务

```shell
$ kubectl apply -f pod_nginx-rs.yaml
```

查看pod信息

```shell
$ kubectl get pods -p wide
```



## 7、 安装仪表盘

按理说到了这一步，应该向集群中添加工作节点了，不过为了能有更直观的体验，我们先安装kubernetes的仪表盘。kubernetes仪表盘的安装简单，查看官网，只需要执行如下命令即可安装，但实际上不能使用此命令安装。

*官方提供的安装命令如下：*

```shell
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml
```

不能安装的原因是NodePost。。。。





如下操作

1. 下载recommended.yaml文件

   ```shell
   $ wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml
   ```

2. 编辑recommended.yaml文件，将其修改为NodePost

3. 应用recommended.yaml文件

   ```shell
   $ kubectl apply -f recommended.yaml
   ```

   

到此，kubernetes仪表盘安装成功，在浏览器使用`https://ip-host:30000`访问kubernetes仪表盘。

界面如下：







执行如下命令获取token

```shell
$ kubectl -n kube-system describe $(kubectl -n kube-system get secret -n kube-system -o name | grep namespace) | grep token
Name:         namespace-controller-token-jfr6v
Type:  kubernetes.io/service-account-token
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkFHclhiME9JV0ZwVDVTZDdnNUExczRLdFFyVGNwQUZsTHBDTUduMlh1Wm8ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlci10b2tlbi1qZnI2diIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjI4ZmFkYjE2LWI2ODEtNDg0ZS04M2E5LWZiNWY1YzIyMjE2MCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpuYW1lc3BhY2UtY29udHJvbGxlciJ9.qqzf0ddeOGqWcFRibnhq3OM4h0qq_gHmhXMjTzLB-S05rhEFplYfEaqbtW17g9id06gU7UbKP_qVGWT6EUAWOw0XzvCH_p8Vdcyv2Ot6lCYOdzHYgItluh-PhD7vsvOuq7_tVT43Rt4Gtq4IARdb7lm_mwxqr7mTzqRthUsCCzs6AIeGtxGvnjhmGLtQUdcN-_jyzG5k1e_9tuR6F4l68LbL_FdUickLqc_wpcGLwpeSUIm9cg_FWL8q5bybv7Cp-6-xwsdwsGIK9J8NhWaN0q9tVEyEMY5iY4CKHkgilzxe8uyhljzyB--4ruGUEYAkWPT5ojSZ2s69Q1KcA1TjhA
```









```shell

namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```





## 8、总结

master节点

1. 安装docker
2. 设置docker daemon
3. 设置docker自启动
4. 启动docker
5. 设置主机名
6. 关闭防火墙
7. 关闭swap分区
8. 创建/proc/sys/net/bridge/bridge-nf-call-iptables，并设置为1
9. 设置/proc/sys/net/ipv4/ip_forward为1
10. 添加镜像源
11. 安装kubelet kubeadm kubectl
12. 设置kubelet自启动
13. 启动kubelet
14. 初始化集群

工作节点

1. 安装docker
2. 设置docker daemon
3. 设置docker自启动
4. 启动docker
5. 设置主机名
6. 编辑/etc/hosts，添加主机名映射
7. 关闭swap分区
8. 添加镜像源
9. 安装kubelet kubeadm kubectl
10. 设置kubelet自启动
11. 启动kubelet
12. /proc/sys/net/bridge/bridge-nf-call-iptables设置为1
13. 添加节点到集群



没有编辑/etc/hosts，添加主机名映射的情况

```shell
[root@localhost ~]# kubeadm join 10.0.2.7:6443 --token 7gd349.kwh3ehctgh9eyvks     --discovery-token-ca-cert-hash sha256:d8cd38193124955199443851777de51b47902ede1f7b1e13890922871519e23d
W0419 12:16:21.984081    2767 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
	[WARNING Hostname]: hostname "work2" could not be reached
	[WARNING Hostname]: hostname "work2": lookup work2 on 211.162.130.33:53: no such host
```





token是有有效期的，如果已经过期，情况如下

```shell
[root@localhost ~]# kubeadm join 10.0.2.7:6443 --token 7gd349.kwh3ehctgh9eyvks     --discovery-token-ca-cert-hash sha256:d8cd38193124955199443851777de51b47902ede1f7b1e13890922871519e23d
W0419 12:21:21.126486    3208 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
error execution phase preflight: couldn't validate the identity of the API Server: could not find a JWS signature in the cluster-info ConfigMap for token ID "7gd349"
To see the stack trace of this error execute with --v=5 or higher
```

如果使节点加入集群的命令失效，执行如下命令重新获取

```shell
$ kubeadm token create --print-join-command
```











```shell
[root@localhost ~]# sysctl --system
* Applying /usr/lib/sysctl.d/00-system.conf ...
* Applying /usr/lib/sysctl.d/10-default-yama-scope.conf ...
kernel.yama.ptrace_scope = 0
* Applying /usr/lib/sysctl.d/50-default.conf ...
kernel.sysrq = 16
kernel.core_uses_pid = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.promote_secondaries = 1
net.ipv4.conf.all.promote_secondaries = 1
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
* Applying /etc/sysctl.d/99-sysctl.conf ...
* Applying /etc/sysctl.d/k8s.conf ...
* Applying /etc/sysctl.conf ...
```



