# docker-machine



官网地址：https://docs.docker.com/machine/

GitHup：https://github.com/docker/machine



## 安装

1. 下载并安装

   ```shell
   $ base=https://github.com/docker/machine/releases/download/v0.16.0 &&
     curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
     sudo mv /tmp/docker-machine /usr/local/bin/docker-machine &&
     chmod +x /usr/local/bin/docker-machine
   ```

2. 检查是否安装成功

   ```shell
   $ docker-machine version
   docker-machine version 0.16.2, build bd45ab13
   ```

docker-machine其实就是一个可执行文件，所以这里所谓的安装就是下载这个可执行文件到`/usr/local/bin/`目录，如果windows环境，配置其环境变量即可。

## Quick Start

首先使用`docker-machine.exe`创建一个docker主机，命令及打印的日志如下：

```shell
$ docker-machine.exe create --driver virtualbox default
Creating CA: C:\Users\xlp\.docker\machine\certs\ca.pem
Creating client certificate: C:\Users\xlp\.docker\machine\certs\cert.pem
Running pre-create checks...
(default) Image cache directory does not exist, creating it at C:\Users\xlp\.docker\machine\cache...
(default) No default Boot2Docker ISO found locally, downloading the latest release...
(default) Latest release for github.com/boot2docker/boot2docker is v19.03.5
(default) Downloading C:\Users\xlp\.docker\machine\cache\boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v19.03.5/boot2docker.iso...
```

从日志可以看出：

1. 在目录`%USERPROFILE%\.docker\machine\certs\`创建了一个`ca.pem`文件。
2. 在目录`%USERPROFILE%\.docker\machine\certs\`创建了一个`cert.pem`文件。
3. 执行创建检检查。
4. 检查到镜像缓存目录不存在，在目录`%USERPROFILE%\.docker\machine\`创建名为`cache`的目录。
5. 在本地没有找到默认的`Boot2Docker`ISO，联网下载。
6. 在github地址`github.com/boot2docker/boot2docker`查找到最新版本是`v19.03.5`。
7. 从https://github.com/boot2docker/boot2docker/releases/download/v19.03.5/boot2docker.iso下载`boot2docker.iso`文件到`%USERPROFILE%\.docker\machine\cache\`目录。

这个命令会下载 boot2docker，基于 boot2docker 创建一个虚拟主机。boot2docker 是一个轻量级的 linux 发行版，基于专门为运行 docker 容器而设计的 Tiny Core Linux 系统，完全从 RAM 运行，45Mb左右，启动时间约5s。

`boot2docker.iso`文件被托管在？？？？？？？？？上，因为网络的原因，不能被直接下载，所以使用其他方式（例如IDM）下载下载将其拷贝到`%USERPROFILE%\.docker\machine\cache\`目录，再次执行docker主机安装：

```shell
$ docker-machine.exe create --driver virtualbox default
Running pre-create checks...
Creating machine...
(default) Copying C:\Users\xlp\.docker\machine\cache\boot2docker.iso to C:\Users\xlp\.docker\machine\machines\default\boot2docker.iso...
(default) Creating VirtualBox VM...
(default) Creating SSH key...
(default) Starting the VM...
(default) Check network to re-create if needed...
(default) Windows might ask for the permission to create a network adapter. Sometimes, such confirmation window is minimized in the taskbar.
(default) Found a new host-only adapter: "VirtualBox Host-Only Ethernet Adapter"
(default) Windows might ask for the permission to configure a network adapter. Sometimes, such confirmation window is minimized in the taskbar.
(default) Windows might ask for the permission to configure a dhcp server. Sometimes, such confirmation window is minimized in the taskbar.
Error creating machine: Error in driver during machine creation: Unable to start the VM: D:\developer\Oracle\VirtualBox\VBoxManage.exe startvm default --type headless failed:
VBoxManage.exe: error: Failed to open/create the internal network 'HostInterfaceNetworking-VirtualBox Host-Only Ethernet Adapter' (VERR_INTNET_FLT_IF_NOT_FOUND).
VBoxManage.exe: error: Failed to attach the network LUN (VERR_INTNET_FLT_IF_NOT_FOUND)
VBoxManage.exe: error: Details: code E_FAIL (0x80004005), component ConsoleWrap, interface IConsole

Details: 00:00:02.093837 Power up failed (vrc=VERR_INTNET_FLT_IF_NOT_FOUND, rc=E_FAIL (0X80004005))
```

1. 首先还是执行创建检检查。
2. 因为boot2docker.iso文件已经存在了，所以省去了下载步骤；将缓存目录中的boot2docker.iso文件拷贝到`%USERPROFILE%\.docker\machine\machines\default\boot2docker.iso`。
3. 开始创建虚拟机。
4. 创建SSH key。
5. 启动虚拟机。
6. 在这之后就开始配置网络。

根据上面的日志可以看到，最终启动失败了，报错`Failed to attach the network LUN (VERR_INTNET_FLT_IF_NOT_FOUND)`，这个原因是？？？？？？？？？

解决方式有两种：

1. 添加`--virtualbox-hostonly-cidr`参数
2. ？？？？？？？

虽然docker主机启动失败了，但是虚拟机却是创建成功了。打开Virtualbox的管理界面，可以看到一个名为default的虚拟机

？？？？？

执行以下命令删除这个docker主机：

```shell
$ docker-machine rm default
```

本地的????是192.168.56.1/24，添加`--virtualbox-hostonly-cidr`创建重新创建，如下：

```shell
$ docker-machine create --driver virtualbox --virtualbox-hostonly-cidr "192.168.56.1/24" default
Running pre-create checks...
Creating machine...
(default) Copying C:\Users\xlp\.docker\machine\cache\boot2docker.iso to C:\Users\xlp\.docker\machine\machines\default\boot2docker.iso...
(default) Creating VirtualBox VM...
(default) Creating SSH key...
(default) Starting the VM...
(default) Check network to re-create if needed...
(default) Windows might ask for the permission to configure a dhcp server. Sometimes, such confirmation window is minimized in the taskbar.
(default) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env default
```

创建并启动成功，可以看到日志输出了Docker is up and running!。



现在来通过Virtualbox的管理界面来看看这个虚拟机的一些基本信息。









```shell
$ docker-machine create \
	-d virtualbox \
	--virtualbox-boot2docker-url=http://xlp.resource.com/linux/boot2docker/v19.03.5/boot2docker.iso \
	--virtualbox-hostonly-cidr 192.168.56.1/24 \
	--engine-registry-mirror=https://wk4vahax.mirror.aliyuncs.com default
```





```shell
$ docker-machine create -d virtualbox --virtualbox-boot2docker-url=http://xlp.resource.com/linux/boot2docker/v19.03.5/boot2docker.iso --virtualbox-hostonly-cidr 192.168.56.1/24 --engine-registry-mirror=https://wk4vahax.mirror.aliyuncs.com default
Running pre-create checks...
Creating machine...
(default) Downloading C:\Users\xlp\.docker\machine\cache\boot2docker.iso from http://localhost:8081/boot2docker.iso...
(default) 0%....10%....20%....30%....40%....50%....60%....70%....80%....90%....100%
(default) Creating VirtualBox VM...
(default) Creating SSH key...
(default) Starting the VM...
(default) Check network to re-create if needed...
(default) Windows might ask for the permission to configure a dhcp server. Sometimes, such confirmation window is minimized in the taskbar.
(default) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env default
```







默认用户名密码

```
docker tcuser
```





*docker-machine help*

```shell
C:\Users\xlp>docker-machine help
Usage: docker-machine [OPTIONS] COMMAND [arg...]

Create and manage machines running Docker.

Version: 0.16.2, build bd45ab13

Author:
  Docker Machine Contributors - <https://github.com/docker/machine>

Options:
  --debug, -D                                           Enable debug mode
  --storage-path, -s "C:\Users\xlp\.docker\machine"     Configures storage path [$MACHINE_STORAGE_PATH]
  --tls-ca-cert                                         CA to verify remotes against [$MACHINE_TLS_CA_CERT]
  --tls-ca-key                                          Private key to generate certificates [$MACHINE_TLS_CA_KEY]
  --tls-client-cert                                     Client cert to use for TLS [$MACHINE_TLS_CLIENT_CERT]
  --tls-client-key                                      Private key used in client TLS auth [$MACHINE_TLS_CLIENT_KEY]
  --github-api-token                                    Token to use for requests to the Github API [$MACHINE_GITHUB_API_TOKEN]
  --native-ssh                                          Use the native (Go-based) SSH implementation. [$MACHINE_NATIVE_SSH]
  --bugsnag-api-token                                   BugSnag API token for crash reporting [$MACHINE_BUGSNAG_API_TOKEN]
  --help, -h                                            show help
  --version, -v                                         print the version

Commands:
  active                Print which machine is active
  config                Print the connection config for machine
  create                Create a machine
  env                   Display the commands to set up the environment for the Docker client
  inspect               Inspect information about a machine
  ip                    Get the IP address of a machine
  kill                  Kill a machine
  ls                    List machines
  provision             Re-provision existing machines
  regenerate-certs      Regenerate TLS Certificates for a machine
  restart               Restart a machine
  rm                    Remove a machine
  ssh                   Log into or run a command on a machine with SSH.
  scp                   Copy files between machines
  mount                 Mount or unmount a directory from a machine with SSHFS.
  start                 Start a machine
  status                Get the status of a machine
  stop                  Stop a machine
  upgrade               Upgrade a machine to the latest version of Docker
  url                   Get the URL of a machine
  version               Show the Docker Machine version or a machine docker version
  help                  Shows a list of commands or help for one command

Run 'docker-machine COMMAND --help' for more information on a command.
```



## Options

|                                                      |      |      |
| ---------------------------------------------------- | ---- | ---- |
| `--debug, -D`                                        |      |      |
| `--storage-path, -s "%USERPROFILE%\.docker\machine"` |      |      |
| `--tls-ca-cert`                                      |      |      |
| `--tls-ca-key`                                       |      |      |
| `--tls-client-cert`                                  |      |      |
| `--github-api-token`                                 |      |      |
| `--native-ssh`                                       |      |      |
| `--bugsnag-api-token`                                |      |      |
| `--bugsnag-api-token`                                |      |      |



## Commands



```shell
$ docker-machine ls
NAME      ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER     ERRORS
default   -        virtualbox   Running   tcp://192.168.56.102:2376           v19.03.5

$ docker-machine active
No active host found
```





### create







```shell
$ docker-machine create --help
Usage: docker-machine create [OPTIONS] [arg...]

Create a machine

Description:
   Run 'docker-machine create --driver name --help' to include the create flags for that driver in the help text.

Options:

   --driver, -d "virtualbox"                                                                            Driver to create machine with. [$MACHINE_DRIVER]
   --engine-env [--engine-env option --engine-env option]                                               Specify environment variables to set in the engine
   --engine-insecure-registry [--engine-insecure-registry option --engine-insecure-registry option]     Specify insecure registries to allow with the created engine
   --engine-install-url "https://get.docker.com"                                                        Custom URL to use for engine installation [$MACHINE_DOCKER_INSTALL_URL]
   --engine-label [--engine-label option --engine-label option]                                         Specify labels for the created engine
   --engine-opt [--engine-opt option --engine-opt option]                                               Specify arbitrary flags to include with the created engine in the form flag=value
   --engine-registry-mirror [--engine-registry-mirror option --engine-registry-mirror option]           Specify registry mirrors to use [$ENGINE_REGISTRY_MIRROR]
   --engine-storage-driver                                                                              Specify a storage driver to use with the engine
   --swarm                                                                                              Configure Machine to join a Swarm cluster
   --swarm-addr                                                                                         addr to advertise for Swarm (default: detect and use the machine IP)
   --swarm-discovery                                                                                    Discovery service to use with Swarm
   --swarm-experimental                                                                                 Enable Swarm experimental features
   --swarm-host "tcp://0.0.0.0:3376"                                                                    ip/socket to listen on for Swarm master
   --swarm-image "swarm:latest"                                                                         Specify Docker image to use for Swarm [$MACHINE_SWARM_IMAGE]
   --swarm-join-opt [--swarm-join-opt option --swarm-join-opt option]                                   Define arbitrary flags for Swarm join
   --swarm-master                                                                                       Configure Machine to be a Swarm master
   --swarm-opt [--swarm-opt option --swarm-opt option]                                                  Define arbitrary flags for Swarm master
   --swarm-strategy "spread"                                                                            Define a default scheduling strategy for Swarm
   --tls-san [--tls-san option --tls-san option]                                                        Support extra SANs for TLS certs
   --virtualbox-boot2docker-url                                                                         The URL of the boot2docker image. Defaults to the latest available version [$VIRTUALBOX_BOOT2DOCKER_URL]
   --virtualbox-cpu-count "1"                                                                           number of CPUs for the machine (-1 to use the number of CPUs available) [$VIRTUALBOX_CPU_COUNT]
   --virtualbox-disk-size "20000"                                                                       Size of disk for host in MB [$VIRTUALBOX_DISK_SIZE]
   --virtualbox-host-dns-resolver                                                                       Use the host DNS resolver [$VIRTUALBOX_HOST_DNS_RESOLVER]
   --virtualbox-hostonly-cidr "192.168.99.1/24"                                                         Specify the Host Only CIDR [$VIRTUALBOX_HOSTONLY_CIDR]
   --virtualbox-hostonly-nicpromisc "deny"                                                              Specify the Host Only Network Adapter Promiscuous Mode [$VIRTUALBOX_HOSTONLY_NIC_PROMISC]
   --virtualbox-hostonly-nictype "82540EM"                                                              Specify the Host Only Network Adapter Type [$VIRTUALBOX_HOSTONLY_NIC_TYPE]
   --virtualbox-hostonly-no-dhcp                                                                        Disable the Host Only DHCP Server [$VIRTUALBOX_HOSTONLY_NO_DHCP]
   --virtualbox-import-boot2docker-vm                                                                   The name of a Boot2Docker VM to import [$VIRTUALBOX_BOOT2DOCKER_IMPORT_VM]
   --virtualbox-memory "1024"                                                                           Size of memory for host in MB [$VIRTUALBOX_MEMORY_SIZE]
   --virtualbox-nat-nictype "82540EM"                                                                   Specify the Network Adapter Type [$VIRTUALBOX_NAT_NICTYPE]
   --virtualbox-no-dns-proxy                                                                            Disable proxying all DNS requests to the host [$VIRTUALBOX_NO_DNS_PROXY]
   --virtualbox-no-share                                                                                Disable the mount of your home directory [$VIRTUALBOX_NO_SHARE]
   --virtualbox-no-vtx-check                                                                            Disable checking for the availability of hardware virtualization before the vm is started [$VIRTUALBOX_NO_VTX_CHECK]
   --virtualbox-share-folder                                                                            Mount the specified directory instead of the default home location. Format: dir:name [$VIRTUALBOX_SHARE_FOLDER]
   --virtualbox-ui-type "headless"                                                                      Specify the UI Type: (gui|sdl|headless|separate) [$VIRTUALBOX_UI_TYPE]
```



`--engine-registry-mirror [--engine-registry-mirror option --engine-registry-mirror option]`

指定使用注册仓库镜像

`--virtualbox-boot2docker-url `

boot2docker映像的URL。默认为最新可用的版本

`--virtualbox-hostonly-cidr "192.168.99.1/24`

指定Host Only CIDR

`--virtualbox-disk-size "20000"`

主机的磁盘大小（MB）

`--virtualbox-memory "1024"`

主机的内存大小（MB）

`--virtualbox-import-boot2docker-vm`

要导入的Boot2Docker VM的名称









### rm

删除虚拟机

```shell
$ docker-machine rm default
About to remove default
WARNING: This action will delete both local reference and remote instance.
Are you sure? (y/n): y
Successfully removed default
```

### env

查询虚拟机环境变量

```shell
$ docker-machine env default
SET DOCKER_TLS_VERIFY=1
SET DOCKER_HOST=tcp://192.168.56.102:2376
SET DOCKER_CERT_PATH=C:\Users\xlp\.docker\machine\machines\default
SET DOCKER_MACHINE_NAME=default
SET COMPOSE_CONVERT_WINDOWS_PATHS=true
REM Run this command to configure your shell:
REM     @FOR /f "tokens=*" %i IN ('docker-machine env default') DO @%i
```

### ssh

通过ssh连接到指定的docker主机。

```shell
$ docker-machine ssh default
exit status 255

C:\Users\xlp>docker-machine ssh default
   ( '>')   ( '>')
  /) TC (\   Core is distributed with ABSOLUTELY NO WARRANTY.
 (/-_--_-\)           www.tinycorelinux.net

docker@default:~$ exit
logout
```

### ip

获取指定docker主机的IP地址。

```shell
$ docker-machine ip default
192.168.56.102
```

### url

获取指定docker主机监听的url。

```shell
$ docker-machine url default
tcp://192.168.56.102:2376
```

### inspect

以json格式输出指定Docker主机的详细信息。

```shell
C:\Users\xlp>docker-machine inspect default
{
    "ConfigVersion": 3,
    "Driver": {
        "IPAddress": "192.168.56.102",
        "MachineName": "default",
        "SSHUser": "docker",
        "SSHPort": 50569,
        "SSHKeyPath": "C:\\Users\\xlp\\.docker\\machine\\machines\\default\\id_rsa",
        "StorePath": "C:\\Users\\xlp\\.docker\\machine",
        "SwarmMaster": false,
        "SwarmHost": "tcp://0.0.0.0:3376",
        "SwarmDiscovery": "",
        "VBoxManager": {},
        "HostInterfaces": {},
        "CPU": 1,
        "Memory": 1024,
        "DiskSize": 20000,
        "NatNicType": "82540EM",
        "Boot2DockerURL": "",
        "Boot2DockerImportVM": "",
        "HostDNSResolver": false,
        "HostOnlyCIDR": "192.168.56.1/24",
        "HostOnlyNicType": "82540EM",
        "HostOnlyPromiscMode": "deny",
        "UIType": "headless",
        "HostOnlyNoDHCP": false,
        "NoShare": false,
        "DNSProxy": true,
        "NoVTXCheck": false,
        "ShareFolder": ""
    },
    "DriverName": "virtualbox",
    "HostOptions": {
        "Driver": "",
        "Memory": 0,
        "Disk": 0,
        "EngineOptions": {
            "ArbitraryFlags": [],
            "Dns": null,
            "GraphDir": "",
            "Env": [],
            "Ipv6": false,
            "InsecureRegistry": [],
            "Labels": [],
            "LogLevel": "",
            "StorageDriver": "",
            "SelinuxEnabled": false,
            "TlsVerify": true,
            "RegistryMirror": [],
            "InstallURL": "https://get.docker.com"
        },
        "SwarmOptions": {
            "IsSwarm": false,
            "Address": "",
            "Discovery": "",
            "Agent": false,
            "Master": false,
            "Host": "tcp://0.0.0.0:3376",
            "Image": "swarm:latest",
            "Strategy": "spread",
            "Heartbeat": 0,
            "Overcommit": 0,
            "ArbitraryFlags": [],
            "ArbitraryJoinFlags": [],
            "Env": null,
            "IsExperimental": false
        },
        "AuthOptions": {
            "CertDir": "C:\\Users\\xlp\\.docker\\machine\\certs",
            "CaCertPath": "C:\\Users\\xlp\\.docker\\machine\\certs\\ca.pem",
            "CaPrivateKeyPath": "C:\\Users\\xlp\\.docker\\machine\\certs\\ca-key.pem",
            "CaCertRemotePath": "",
            "ServerCertPath": "C:\\Users\\xlp\\.docker\\machine\\machines\\default\\server.pem",
            "ServerKeyPath": "C:\\Users\\xlp\\.docker\\machine\\machines\\default\\server-key.pem",
            "ClientKeyPath": "C:\\Users\\xlp\\.docker\\machine\\certs\\key.pem",
            "ServerCertRemotePath": "",
            "ServerKeyRemotePath": "",
            "ClientCertPath": "C:\\Users\\xlp\\.docker\\machine\\certs\\cert.pem",
            "ServerCertSANs": [],
            "StorePath": "C:\\Users\\xlp\\.docker\\machine\\machines\\default"
        }
    },
    "Name": "default"
}
```

### status

获取指定docker主机的状态。？？？？？？？？？？

```shell
$ docker-machine status default
Running
```

### start 

启动一个指定的docker主机。

```shell
$ docker-machine start default
Starting "default"...
(default) Check network to re-create if needed...
(default) Windows might ask for the permission to configure a dhcp server. Sometimes, such confirmation window is minimized in the taskbar.
(default) Waiting for an IP...
Machine "default" was started.
Waiting for SSH to be available...
Detecting the provisioner...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.
```

### stop

停止一个Docker主机。

```shell
$ docker-machine stop default
Stopping "default"...
Machine "default" was stopped.
```

### kill

直接杀死指定的Docker主机。

```shell
$ docker-machine kill default
Killing "default"...
Machine "default" was killed.
```



### config

查看到激活的Docker主机的连接信息。

```shell
$ docker-machine config default
--tlsverify
--tlscacert="C:\\Users\\xlp\\.docker\\machine\\machines\\default\\ca.pem"
--tlscert="C:\\Users\\xlp\\.docker\\machine\\machines\\default\\cert.pem"
--tlskey="C:\\Users\\xlp\\.docker\\machine\\machines\\default\\key.pem"
-H=tcp://192.168.56.102:2376
```

### scp



```shell
$ docker-machine scp help
Usage: docker-machine scp [OPTIONS] [arg...]

Copy files between machines

Description:
   Arguments are [[user@]machine:][path] [[user@]machine:][path].

Options:

   --recursive, -r      Copy files recursively (required to copy directories)
   --delta, -d          Reduce amount of data sent over network by sending only the differences (uses rsync)
   --quiet, -q          Disables the progress meter as well as warning and diagnostic messages from ssh
Improper number of arguments
```



```shell
$ docker-machine scp C:\Users\xlp\Desktop\boot2docker.iso docker0://opt/docker/files/resources
```









## docker-machine重启主机数据丢失

### 问题描述

通过docker-machine创建docker主机，并运行容器映射数据卷到目录`/home/docker/`，发现在重启docker主机之后，数据卷中的文件都是丢失。

### 错误原因

这是因为`/home/docker/`挂载在`tmpfs`，即使对Linux系统不了解，看到这个名字，就应该知道为什么重启之后，文件丢失了。

```shell
docker@docker0:~$ df /home/docker/
Filesystem           1K-blocks      Used Available Use% Mounted on
tmpfs                   911900    290496    621404  32% /
```

再查看其他常用目录`/opt/`、`/etc/`、`/root/`、`/usr/`、`/var/`，都是如此。

```shell
docker@docker0:~$ df /opt/     
Filesystem           1K-blocks      Used Available Use% Mounted on
tmpfs                   911900    290496    621404  32% /
docker@docker0:~$ df /etc/
Filesystem           1K-blocks      Used Available Use% Mounted on
tmpfs                   911900    290496    621404  32% /
docker@docker0:~$ df /root/
Filesystem           1K-blocks      Used Available Use% Mounted on
tmpfs                   911900    290496    621404  32% /
docker@docker0:~$ df /usr/                                                    
Filesystem           1K-blocks      Used Available Use% Mounted on
tmpfs                   911900    290496    621404  32% /
docker@docker0:~$ df /var/
Filesystem           1K-blocks      Used Available Use% Mounted on
tmpfs                   911900    290496    621404  32% /
```

> tmpfs是基于内存的临时文件系统，machine关闭后，数据不会保存。

### 解决方案

#### 方式一（不可用）

通过前面的分析可知，`/home/docker/`挂载在`tmpfs`下，所以更换`/home/docker/`的挂载，将其挂载在设备文件下即可持久化。

通过`df -h`命令查看所有信息：

```shell
docker@docker0:~$ df -h                        
Filesystem                Size      Used Available Use% Mounted on
tmpfs                   890.5M    283.7M    606.8M  32% /
tmpfs                   494.7M         0    494.7M   0% /dev/shm
/dev/sda1                17.8G    178.3M     16.7G   1% /mnt/sda1
cgroup                  494.7M         0    494.7M   0% /sys/fs/cgroup
/c/Users                118.0G     98.5G     19.5G  83% /c/Users
/dev/sda1                17.8G    178.3M     16.7G   1% /mnt/sda1/var/lib/docker
```

设备文件`/dev/sda1`分别挂载了目录`/mnt/sda1`和`/mnt/sda1/var/lib/docker`，所以也只需要将对应目录挂载在设备文件下即可。

```shell
docker@docker0:~$ sudo mount /dev/sda1 /home/docker/
```

再次查看挂载信息

```shell
docker@docker0:~$ df /home/docker/                                                      
Filesystem           1K-blocks      Used Available Use% Mounted on
/dev/sda1             18714000    182528  17542344   1% /mnt/sda1
docker@docker0:~$ df -h
Filesystem                Size      Used Available Use% Mounted on
tmpfs                   890.5M    283.7M    606.8M  32% /
tmpfs                   494.7M         0    494.7M   0% /dev/shm
/dev/sda1                17.8G    178.3M     16.7G   1% /mnt/sda1
cgroup                  494.7M         0    494.7M   0% /sys/fs/cgroup
/c/Users                118.0G     98.5G     19.5G  83% /c/Users
/dev/sda1                17.8G    178.3M     16.7G   1% /mnt/sda1/var/lib/docker
/dev/sda1                17.8G    178.3M     16.7G   1% /home/docker
```

到此，就可以将文件放在`/home/docker/`目录下了。

> 设备文件存在在/dev/目录下，在boot2docker中，其对应的设备文件有0-9共10个，但是只有sda1可以进行挂载。
>
> ```shell
> docker@docker0:~$ ls -l /dev/ | grep sda
> lrwxrwxrwx    1 root     root             4 Mar  6 15:19 flash -> sda1
> brw-rw----    1 root     staff       8,   0 Mar  6 15:19 sda
> brw-rw----    1 root     staff       8,   1 Mar  6 15:19 sda1
> brw-rw----    1 root     staff       8,   2 Mar  6 15:19 sda2
> brw-rw-r--    1 root     staff       8,   3 Oct 19 21:54 sda3
> brw-rw-r--    1 root     staff       8,   4 Oct 19 21:54 sda4
> brw-rw-r--    1 root     staff       8,   5 Oct 19 21:54 sda5
> brw-rw-r--    1 root     staff       8,   6 Oct 19 21:54 sda6
> brw-rw-r--    1 root     staff       8,   7 Oct 19 21:54 sda7
> brw-rw-r--    1 root     staff       8,   8 Oct 19 21:54 sda8
> brw-rw-r--    1 root     staff       8,   9 Oct 19 21:54 sda9
> ```
>
> 现在尝试通过非sda1进行挂载：
>
> ```shell
> docker@docker0:~$ sudo mount /dev/sda /home/docker/
> mount: /home/docker: /dev/sda already mounted or mount point busy.
> docker@docker0:~$ sudo mount /dev/sda2 /home/docker/
> mount: /home/docker: unknown filesystem type 'swap'.
> docker@docker0:~$ sudo mount /dev/sda3 /home/docker/
> mount: /home/docker: /dev/sda3 is not a valid block device.
> docker@docker0:~$ sudo mount /dev/sda4 /home/docker/
> mount: /home/docker: /dev/sda4 is not a valid block device.
> docker@docker0:~$ sudo mount /dev/sda5 /home/docker/
> mount: /home/docker: /dev/sda5 is not a valid block device.
> docker@docker0:~$ sudo mount /dev/sda6 /home/docker/
> mount: /home/docker: /dev/sda6 is not a valid block device.
> docker@docker0:~$ sudo mount /dev/sda7 /home/docker/
> mount: /home/docker: /dev/sda7 is not a valid block device.
> docker@docker0:~$ sudo mount /dev/sda8 /home/docker/
> mount: /home/docker: /dev/sda8 is not a valid block device.
> docker@docker0:~$ sudo mount /dev/sda9 /home/docker/ 
> mount: /home/docker: /dev/sda9 is not a valid block device.
> ```
>
> /dev/sda提示：已经挂载或挂载点忙。
>
> /dev/sda2提示：未知的文件系统类型“swap”。
>
> 其他sda提示：不是有效的块设备。
>
> 再次回顾一下设备挂载情况
>
> ```shell
> docker@docker0:~$ df -h
> Filesystem                Size      Used Available Use% Mounted on
> tmpfs                   890.5M    283.7M    606.8M  32% /
> tmpfs                   494.7M         0    494.7M   0% /dev/shm
> /dev/sda1                17.8G    178.3M     16.7G   1% /mnt/sda1
> cgroup                  494.7M         0    494.7M   0% /sys/fs/cgroup
> /c/Users                118.0G     98.5G     19.5G  83% /c/Users
> /dev/sda1                17.8G    178.3M     16.7G   1% /mnt/sda1/var/lib/docker
> /dev/sda1                17.8G    178.3M     16.7G   1% /home/docker
> ```
>
> `/dev/sda1`设备磁盘空间的大小为17.8G，而docker主机在未指定磁盘容量的情况下默认大小为20G，其`/dev/sda1`设备已经被分配了几乎全部的磁盘空间，自然其他设备也就没有空间可以分配了。

#### 方式二

上面我们挂载了指定目录，当然也可以不用操作挂载指定目录，直接将需要长久保存的文件存放到`/mnt/sda1/`目录下即可。

#### 方式三

另一个解决方式就是将文件存在`/c/Users`目录下，注意`df -h`命令所输出的磁盘挂载信息。

docker主机的`/c/Users`目录自动挂载到了宿主机（Windows或Mac）的`/c/Users`目录下，即Windows的`C:\Users`目录下，所以也可以直接将文件方存在`/c/Users`目录即可。

> 只说明了Window的具体情况，因为我是在windows上安装的virtualBox。

### 验证

现在尝试验证上述三种方式是否可以解决问题，

在`/mnt/sda1/`、`/home/docker/`、`/c/Users`目录下分别创建文件`test1`、`test2`、`test3`文件

```shell
docker@docker0:~$ sudo touch /mnt/sda1/test1
docker@docker0:~$ ls -l /mnt/sda1/
total 24
drwx------    2 root     root         16384 Mar  1 04:23 lost+found
-rw-r--r--    1 root     root             0 Mar  7 02:30 test1
drwxrwxrwt    4 root     staff         4096 Mar  7 02:30 tmp
drwxr-xr-x    3 root     root          4096 Mar  1 04:23 var
docker@docker0:~$ sudo touch /home/docker/test2
docker@docker0:~$ ls -l /home/docker/                                             
total 24
drwx------    2 root     root         16384 Mar  1 04:23 lost+found
-rw-r--r--    1 root     root             0 Mar  7 02:30 test1
-rw-r--r--    1 root     root             0 Mar  7 02:31 test2
drwxrwxrwt    4 root     staff         4096 Mar  7 02:32 tmp
drwxr-xr-x    3 root     root          4096 Mar  1 04:23 var
docker@docker0:~$ sudo touch /c/Users/test3                                
touch: /c/Users/test3: Protocol error
docker@docker0:~$ sudo touch /c/Users/xlp/test3                               
docker@docker0:~$ ls -l /c/Users/xlp/ | grep test                         
-rwxrwxrwx    1 docker   staff            0 Mar  7 02:33 test3
```

> 1. 在`/home/docker/`目录创建`test2`的时候，查看创建情况，会发现`/home/docker/`目录下同时也存在了刚在创建在`/mnt/sda1/`目录下的`test1`文件，这是因为`/home/docker/`与`/mnt/sda1/`挂载在同一个设备文件下，`/home/docker/`挂载到`/dev/sda1`，其实就是对应到`/mnt/sda1/`。
>
> 2. 在`/c/Users`创建`test3`的时候提示协议错误，这个Windows系统的限制，所以创建`/c/Users/%USERNAME%/`下，创建之后在Windows系统打开`C:\Users\%USERNAME%`目录，可以确实多了一个`test3`文件。
>
>    ```shell
>    C:\Users\xlp>echo %USERNAME%
>    xlp
>    ```

重启docker主机

```batch
C:\Users\xlp>docker-machine stop docker0
Stopping "docker0"...
Machine "docker0" was stopped.

C:\Users\xlp>docker-machine start docker0
Starting "docker0"...
(docker0) Check network to re-create if needed...
(docker0) Windows might ask for the permission to configure a dhcp server. Sometimes, such confirmation window is minimized in the taskbar.
(docker0) Waiting for an IP...
Machine "docker0" was started.
Waiting for SSH to be available...
Detecting the provisioner...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.
```

> 1. 可以直接使用`docker-machine restart docker0`命令重启，不过为了更加严谨，可以在stop之后，以确认它确实关闭了。
>
> 2. 这里的docker0是我的docker主机名称。

查询文件存在情况

```shell
docker@docker0:~$ ls -l /c/Users/xlp/ | grep test                  
-rwxrwxrwx    1 docker   staff            0 Mar  7 02:33 test3

docker@docker0:~$ ls -l /mnt/sda1/                     
total 24
drwx------    2 root     root         16384 Mar  1 04:23 lost+found
-rw-r--r--    1 root     root             0 Mar  7 02:30 test1
-rw-r--r--    1 root     root             0 Mar  7 02:31 test2
drwxrwxrwt    4 root     staff         4096 Mar  7 02:44 tmp
drwxr-xr-x    3 root     root          4096 Mar  1 04:23 var

docker@docker0:~$ ls -l /home/docker/                  
total 0
drwxr-sr-x    3 root     root            60 Mar  7 02:37 docker-nginx

docker@docker0:~$ df /home/docker/                         
Filesystem           1K-blocks      Used Available Use% Mounted on
tmpfs                   911900    290496    621404  32% /
```

结果出乎意料，`/c/Users/xlp/`和`/mnt/sda1/`都验证成功，而`/home/docker/`却验证失败，并且其挂载设备又恢复成`tmpfs`了。

到这了这里就算讲完了，虽然没有解决`/home/docker/`挂载重启失效的问题。如果有解决的小伙伴，欢迎留言。

因个人水平有限，如有错漏，敬请指正，QQ：296007576





