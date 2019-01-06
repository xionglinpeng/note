# Shell实战



## 异步执行

shell脚本是同步执行的，但是我们常常需要多个任务同时异步执行。

在执行命令后加`&`操作符，表示将命令放在子shell中异步执行。可以达到多线程的效果，例如：

```shell
[root@localhost ~]# sleep 1h &
[1] 8415
[root@localhost ~]# sleep 10m &
[2] 8416
[root@localhost ~]# sleep 100 &
[3] 8417
```

但是这里有一个问题，虽然加`&`操作符可以达到异步执行的效果，但是我们需要的是在一个shell脚本中等待多个异步任务都执行完毕之后再退出shell进程。

使用`wait`命令可以完成我们的需求。

例如：

```shell
#!/bin/bash
sleep 10 &
sleep 5 &
wait #等待10秒后退出
```

验证`wait`是否等待了所有子任务执行完毕：

```shell
#!/bin/bash
func() {
    echo "my func$1."
    sleep 10 &
    wait # 只等待前面的sleep
    echo "my func$1 execute compilte."
}
func 10 &
func 20 &
wait # 如果没有这个wait，则整个脚本立即退出，不会等待func函数中的sleep
echo "execute success."
```

输出结果

```
my func10.
my func20.
my func10 execute compilte.
my func20 execute compilte.
execute success.
```

## 退出脚本

执行一个shell脚本，在默认情况下，执行过程中如果有一个命令执行失败，那么后面的命令任然会被执行。但是我们常常希望某个地方执行失败了，就退出整个脚本，不在执行后面的内容。

```shell
#!/bin/bash
echo "hello world1."
pwda
echo "hello world2."
```

输出结果

```text
hello world1.
./te.sh: line 3: pwda: command not found
hello world2.
```

使用shell脚本退出有如下三种方式：

- `exit`

  ```shell
  #!/bin/bash
  echo "hello world1."
  pwda
  if [ $? != 0 ]
  then
    echo "执行失败"
    exit 1
  fi
  echo "hello world2."
  ```

  输出结果

  ```
  hello world1.
  ./te.sh: line 3: pwda: command not found
  执行失败
  ```

- `kill -9`

  ```shell
  #!/bin/bash
  echo "hello world1."
  pwda
  if [ $? != 0 ]
  then
    echo "执行失败"
    kill -9 $$ #$$表示当前shell进行的PID
  fi
  echo "hello world2."
  ```

  输出结果

  ```shell
  hello world1.
  ./te.sh: line 3: pwda: command not found
  执行失败
  Killed
  ```

- `tarp`

  ```shell
  #!/bin/bash
  #监听退出信号
  trap 'exit 1' TERM
  echo "hello 1."
  echo "hello 2."
  echo "hello 3."
  echoA
  if [ $? != 0 ]
  then
      echo "执行错误，退出。"
      #向当前shell进程发送退出信号，触发退出操作
      kill -s TERM $$
  fi
  echo "hello 5."
  ```

  输出结果

  ```
  [root@localhost ~]# ./test.sh 
  hello 1.
  hello 2.
  hello 3.
  ./test.sh:行7: echoA: 未找到命令
  执行错误，退出。
  ```
