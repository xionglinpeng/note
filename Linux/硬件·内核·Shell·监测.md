# [硬件·内核·Shell·监测](http://man.linuxde.net/par/3)



## E

### exit

exit命令同于退出shell，并返回给定值。在shell脚本中可以终止当前脚本执行。执行exit可使shell以指定的状态值退出。若不设置状态值参数，则shell以预设值退出。状态值0代表执行成功，其他值代表执行失败。

#### 语法

```shell
exit (参数)
```

#### 参数

返回值：执行shell返回值。

#### 实例

**退出当前shell**

```shell
[root@localhost ~]# exit
登出
Connection closing...Socket close.
```

**检查上一个命令是否执行成功**

```shell
#!/bin/bash
echo "helo world"
EXCODE=$?
if [ $EXCODE == 0 ]
then
    echo "O.K"
fi
```

**如果执行失败则退出**

```shell
#!/bin/bash
echo1
EXCODE=$?
if [ $EXCODE != 0 ]
then
    echo "execute failed."
    exit 1
fi
echo "execute success?"
```



## W

### wait

wait命令用来等待指令的指令，直到其执行完毕后返回终端。该指令常用于shell脚本编程中，待指定的指令执行完成后，才会继续执行后面的任务。该指定等待作业时，在作业标识号前必须添加备份号“%”。

#### 语法

```
wait (参数)
```

#### 参数

进程或作业标示：指定进程号或者作业号

使用命令wait等待作业号为1的作业完成后再返回，输入如下命令：

```shell
wait %1
```

#### 实例

执行上面的指令后，将输出指定作业好的指令，如下所示：

```shell
[root@localhost ~]# sleep 10 &
[1] 8091
[root@localhost ~]# wait %1
[1]+  完成                  sleep 10
```

