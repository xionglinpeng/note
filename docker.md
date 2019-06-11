# Docker



## Docker安装



```shell
[root@localhost ~]# wget -qO- https://get.docker.com/ | sh
# Executing docker install script, commit: 2f4ae48
+ sh -c 'yum install -y -q yum-utils'
+ sh -c 'yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo'
已加载插件：fastestmirror
adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
+ '[' stable '!=' stable ']'
+ sh -c 'yum makecache'
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.cqu.edu.cn
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
base                                                                                        | 3.6 kB  00:00:00     
docker-ce-stable                                                                            | 3.5 kB  00:00:00     
extras                                                                                      | 3.4 kB  00:00:00     
updates                                                                                     | 3.4 kB  00:00:00     
(1/12): docker-ce-stable/x86_64/updateinfo                                                  |   55 B  00:00:01     
(2/12): docker-ce-stable/x86_64/primary_db                                                  |  28 kB  00:00:00     
(3/12): docker-ce-stable/x86_64/other_db                                                    | 111 kB  00:00:01     
(4/12): docker-ce-stable/x86_64/filelists_db                                                |  14 kB  00:00:03     
(5/12): extras/7/x86_64/prestodelta                                                         |  62 kB  00:00:01     
(6/12): extras/7/x86_64/filelists_db                                                        | 243 kB  00:00:02     
(7/12): extras/7/x86_64/other_db                                                            | 126 kB  00:00:00     
(8/12): updates/7/x86_64/prestodelta                                                        | 681 kB  00:00:00     
(9/12): updates/7/x86_64/other_db                                                           | 588 kB  00:00:00     
(10/12): base/7/x86_64/other_db                                                             | 2.6 MB  00:00:11     
(11/12): base/7/x86_64/filelists_db                                                         | 7.1 MB  00:00:23     
(12/12): updates/7/x86_64/filelists_db                                                      | 3.7 MB  00:00:19     
元数据缓存已建立
+ '[' -n '' ']'
+ sh -c 'yum install -y -q docker-ce'
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
warning: /var/cache/yum/x86_64/7/docker-ce-stable/packages/docker-ce-18.09.6-3.el7.x86_64.rpm: Header V4 RSA/SHA512 Signature, key ID 621e9f35: NOKEY
docker-ce-18.09.6-3.el7.x86_64.rpm 的公钥尚未安装
http://mirrors.nwsuaf.edu.cn/centos/7.6.1810/os/x86_64/Packages/libsepol-devel-2.5-10.el7.x86_64.rpm: [Errno 14] HTTP Error 503 - Service Unavailable
正在尝试其它镜像。
导入 GPG key 0x621E9F35:
 用户ID     : "Docker Release (CE rpm) <docker@docker.com>"
 指纹       : 060a 61c5 1b55 8a7f 742b 77aa c52f eb6b 621e 9f35
 来自       : https://download.docker.com/linux/centos/gpg
If you would like to use Docker as a non-root user, you should now consider
adding your user to the "docker" group with something like:

  sudo usermod -aG docker your-user

Remember that you will have to log out and back in for this to take effect!

WARNING: Adding a user to the "docker" group will grant the ability to run
         containers which can be used to obtain root privileges on the
         docker host.
         Refer to https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface
         for more information.
```


# docker
docker文档地址：https://docs.docker.com

## moby、docker-ce和docker-ee的区别
最早的时候docker就是一个开源项目，主要是由docker公司维护。
2017年初，docker公司将原先的docker项目改名为moby，并创建了docker-ce和docker-ee。
- moby是继承了原先的docker的项目，是社区维护的开源项目，谁都可以在moby的基础打造自己的容器产品。
- docker-ce是docker公司维护的开源项目，是一个基础moby项目的免费的容器产品。
- docker-ee是docker公司维护的闭源产品，是docker公司的商业产品（简单的说，就是要付钱的）。





Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
[root@localhost ~]# systemctl start docker



## images



### 获取镜像 acquire images

```shell
# docker pull name[:tag]
```

例如：docker pull mysql



### 查看镜像信息

语法

```
docker images [OPTIONS] [REPOSITORY[:TAG]]
```



选项

```
  -a, --all             Show all images (default hides intermediate images)
      --digests         Show digests
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           Only show numeric IDs
```





参数



实例



```shell
[root@localhost ~]# docker images --help

Usage:	docker images [OPTIONS] [REPOSITORY[:TAG]]

List images

Options:
  -a, --all             Show all images (default hides intermediate images)
      --digests         Show digests
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           Only show numeric IDs
  
  
[root@localhost ~]# man docker-images
NAME
       docker-images - List images

SYNOPSIS
       docker images [OPTIONS] [REPOSITORY[:TAG]]

DESCRIPTION
       Alias for docker image ls.

OPTIONS
       -a, --all[=false]
           Show all images (default hides intermediate images)

       --digests[=false]
           Show digests

       -f, --filter=
           Filter output based on conditions provided

       --format=""
           Pretty-print images using a Go template

       -h, --help[=false]
           help for images

       --no-trunc[=false]
           Don't truncate output

       -q, --quiet[=false]
           Only show numeric IDs

SEE ALSO
       docker(1)
```



### 搜寻镜像

docker search TERM

### 删除镜像

删除镜像的命令是`docker rmi INAGE [IMAGE...]`，其中IMAGE可以为**标签**或**ID**。

1. 使用标签删除镜像

   docker rmi nginx:latest

   Error response from daemon: conflict: unable to remove repository reference "centos:latest" (must force) - container b770c3a63681 is using its referenced image 9f38484d220f

2. 使用镜像ID删除镜像

   ```shell
   [root@localhost ~]# docker rmi fce289e99eb9
   Untagged: hello-world:latest
   Untagged: hello-world@sha256:0e11c388b664df8a27a901dce21eb89f11d8292f7fca1b3e3c4321bf7897bffe
   Deleted: sha256:fce289e99eb9bca977dae136fbe2a82b6b7d4c372474c9235adc1741675f587e
   Deleted: sha256:af0b15c8625bb1938f1d7b17081031f649fd14e6b233688eea3c5483994a66a3
   ```

### 创建镜像

创建镜像有三种方式：

- 基于已有镜像的容器创建。

- 基于本地模板导入。

- 基础Dockerfile创建。

1. 基于已有镜像的容器创建。

   asd

   ```shell
   $ docker commit [OPTION] CONTAINER [REPOSITORY[:TAG]]
   ```

   选项

   阿萨德

   ```shell
   [root@localhost ~]# docker images
   REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
   centos              latest              9f38484d220f        2 months ago        202MB
   [root@localhost ~]# docker run -it centos:latest /bin/bash
   [root@f685d3d144d9 /]# cd /opt/
   [root@f685d3d144d9 opt]# touch test
   [root@f685d3d144d9 opt]# exit
   exit
   [root@localhost ~]# docker commit -m "Added a new file" -a "xlp" f685d3d144d9 test:0.1
   sha256:753617346c1da35f14435df085137731b0126a4893c08aa48af6e6d1dc7e55a8
   [root@localhost ~]# docker images
   REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
   test                0.1                 753617346c1d        10 seconds ago      202MB
   centos              latest              9f38484d220f        2 months ago        202MB
   ```

   

2. 基于本地模板导入。

   

3. 基础Dockerfile创建。

   

### save and load image导出和导入镜像



1. 导出镜像

   ```shell
   $ docker save -o centos-latest.jar centos:latest
   ```

2. 导入镜像

   ```shell
   [root@localhost ~]# docker load --input centos-latest.jar 
   [root@localhost ~]# docker load < centos-latest.jar
   ```



### 上传镜像





## container



删除容器

```shell
$ docker rm [选项] CONTAINER [CONTAINER]
```





docker rm e5143771c24f

docker rm b77

docker rm 6b 58

