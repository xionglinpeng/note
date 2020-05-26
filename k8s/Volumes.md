# Kubernetes存储





## Volumes



## Volume类型



### emptyDir

### hostPath



支持的type：

| 取值              | 行为                                                         |
| ----------------- | ------------------------------------------------------------ |
|                   | 空字符串（默认），用于向后兼容，意味着在安装hostPath卷之前不会执行任何检查。 |
| DirectoryOrCreate | 如果给定的路径上什么都不存在，将自动创建空目录，并设置权限为0755，具有与kubectl相同的组和所有权。 |
| Directory         | 在给定的路径上必须存在目录。                                 |
| FileOrCreate      | 如果给定的路径上什么都不存在，将自动创建空文件，并设置权限为0644，具有与kubectl相同的组和所有权。 |
| File              | 在给定的路径上必须存在文件。                                 |
| Socket            | 在给定的路径上必须存在UNIX套接字。                           |
| CharDevice        | 在给定的路径上必须存在字符设备。                             |
| BlockDevice       | 在给定的路径上必须存在块设备。                               |









### configMap



### nfs

### persistentVolumeClaim

安装nfs

```shell
$ yum -y install nfs-utils
```



```shell
$ mkdir /data/volumes -pv
$ vim /etc/exports
$ systemctl start nfs
$ ss -tnl
```



节点挂载

```shell
$ yum -y install nfs-utils
$ mount -t nfs [host]:/data/volumes /mnt
```







gitRepo（deprecated）