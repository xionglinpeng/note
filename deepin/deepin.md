# Deepin





## deepin全屏

使用VirtualBox安装的deepin，默认情况下只有很小的一个窗口，下面将描述如何设置全屏。

1. 选择“设备”，然后选择“安装增强功能...”。

   当执行完这一步之后会没有任何变化，当却是必不可少的，将在第三步说明。

   ![1587202774355](images\1587202774355.png)

2. 选择“文件管理器”

   ![1587202477214](images\1587202477214.png)

3. 选择VBox_GAs_6.0.16，然后鼠标右键，选择“在终端打开”。

   注意这里的VBox_GAs_6.0.16目录，必须在第一步中心之后才会有这个目录，如果不执行第一步，仍然打开“文件管理器”，看到的将不是VBox_GAs_6.0.16。

   ![1587202536522](images\1587202536522.png)

5. 执行脚本`VBoxLinuxAdditions.run`。

   ```shell
   $ ./VBoxLinuxAdditions.run
   ```

6. 重启系统

   通过方向键`-> + ctrl + f`控制是否全屏。



## 镜像源

官方相关资料：https://wiki.deepin.org/wiki/%E8%BD%AF%E4%BB%B6%E6%BA%90

Deepin的包管理功能是apt-get，关于apt-get这里不过多描述，只说明如何修改镜像源。

### 修改镜像源

如果你需要修改软件源，方法有两种：

一、运行控制中心——更新——更新设置——关闭智能镜像源，切换镜像，选择你喜欢的软件源。

1. 运行控制中心

   ![1587202326181](images\1587202326181.png)

2. 更新

   ![1587203216638](images\1587203216638.png)

   3. 更新设置

      ![1587203262042](images\1587203262042.png)

   4. 关闭智能镜像源，选择切换镜像

      ![1587203306923](images\1587203306923.png)

二、手工修改源配置文件（如果你不清楚其中的危险性，请不要修改），终端执行（二选一）：

1. ```shell
   sudo edit /etc/apt/sources.list
   ```

2. ```shell
   sudo deepin-editor /etc/apt/sources.list
   ```

修改完成保存后需要刷新软件源列表，终端执行：

```shell
$ sudo apt-get update
```



## 开启SSH服务

默认情况下Deepin没有开启SSH服务，故而也就不能在外部通过xshell等终端工具连接。

### 安装SSH服务

```shell
$ sudo apt install openssh-server
```

### 启动SSH服务

三种启动方式：

1. ```shell
   $ sudo /etc/init.d/ssh start
   ```

2. ```shell
   $ service ssh start
   ```

3. ```shell
   $ systemctl restart sshd
   ```

### 配置端口

SSH服务的默认端口为`22`，一般情况下不需要修改。

ssh-server配置文件位于`/etc/ssh/sshd_config`，其配置中有`Port 22`配置，默认被注释，打开并设置自定义端口，然后重启即可。

```shell
$ vim /etc/ssh/sshd_config
......
#Port 22
......
```

### 其他操作

| command                       | description                          |
| ----------------------------- | ------------------------------------ |
| `sodo ssh -V`                 | 查看openssh-server版本               |
| `sudo /etc/init.d/ssh status` | 查看openssh-server状态               |
| `service sshd status`         | 查看openssh-server状态               |
| `service sshd start`          | 启动openssh-server服务               |
| `service sshd stop`           | 停止openssh-server服务               |
| `service sshd restart`        | 重启openssh-server状态(服务会中断)   |
| `service sshd reload`         | 重载openssh-server状态(服务不会中断) |

