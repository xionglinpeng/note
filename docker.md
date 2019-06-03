<<<<<<< HEAD
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

=======
# docker
docker文档地址：https://docs.docker.com

## moby、docker-ce和docker-ee的区别
最早的时候docker就是一个开源项目，主要是由docker公司维护。
2017年初，docker公司将原先的docker项目改名为moby，并创建了docker-ce和docker-ee。
- moby是继承了原先的docker的项目，是社区维护的开源项目，谁都可以在moby的基础打造自己的容器产品。
- docker-ce是docker公司维护的开源项目，是一个基础moby项目的免费的容器产品。
- docker-ee是docker公司维护的闭源产品，是docker公司的商业产品（简单的说，就是要付钱的）。
>>>>>>> 2a07e6ce0269c13e6a2f420b105ff9e5fddf674b
