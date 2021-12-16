# Jetbrains系列产品2020.1.1最新激活方法

注意：下述激活方式仅限于Jetbrains系列产品2020.1.1版本及以下。

## 前置准备

1. 安装Jetbrains系列产品，例如GoLand 2020.1.2 x64。

2. 下载用于激活的jetbrains-agent-latest.zip。

   百度网盘：https://pan.baidu.com/s/1ct2qlEDyVRbVa31IPqkQ3w，提取码：absm

## 激活

1. 放置jetbrains-agent.jar文件

   解压jetbrains-agent-latest.zip，其`lib`目录下有`jetbrains-agent.jar`文件，将此文件拷贝到安装目录下。

   例如`D:\developer\JetBrains\GoLand 2020.1.2\jetbrains-agent.jar`

2. 编辑goland64.exe.vmoptions

   编辑GoLand安装目录的bin目录下的`goland64.exe.vmoptions`文件。

   例如`D:\developer\JetBrains\GoLand 2020.1.2\bin\goland64.exe.vmoptions`

3. 设置jetbrains-agent

   在最后一行添加如下内容：

   ```shell
   -javaagent:D:\developer\JetBrains\GoLand 2020.1.2\jetbrains-agent.jar
   ```

   即第一步所放置的`jetbrains-agent.jar`文件的位置，然后保存。

4. 启动GoLand。

   会提示jetbrainsAgent加载成功

   ![img](images\active-1.png)

5. 激活

   选择Activation Code，添入激活码，然后点击Activate激活。

   > 激活码位于jetbrains-agent-latest.zip中的`lib/ACTIVATION_CODE.txt`文件中。

   ![1590203897814](images\active-2.png)

6. 查看激活信息

   Help->About，有效期到2089年。

   ![1590204021660](images\active-3.png)

## 其他参考信息

上述方式资料来源

https://zhile.io/2018/08/25/jetbrains-license-server-crack.html