



## 异步执行





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
    kill -9 $$
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

  296007576xlp

  597646251xlp

https://www.cnblogs.com/lslxdx/p/6566465.html

https://www.cnblogs.com/anyehome/p/9068971.html

https://www.cnblogs.com/zxf330301/p/6534643.html

https://movie.douban.com/review/7631253/

https://baijiahao.baidu.com/s?id=1563035253717548&wfr=spider&for=pc





