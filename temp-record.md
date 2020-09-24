# Temp-record

手机连接电脑不能被识别：

用手机自带的数据bai线和你的电脑正确连接









1. 首先是启动windows的命令窗口，按键盘上的windows+R，然后在输入框中输入cmd，既可以启动命令窗口

   ![windows系统如何查看端口被占用、杀进程](https://exp-picture.cdn.bcebos.com/cd93a5665159854029fb1351b5a23a42a17ac467.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1)

2. 

   进入windows命令窗口之后，输入命令，输入netstat -ano然后回车，就可以看到系统当前所有的端口使用情况。

   ![windows系统如何查看端口被占用、杀进程](https://exp-picture.cdn.bcebos.com/3fe32442a07aa01083e2fb8bbfbb19efa35f3e64.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1)

3. 

   通过命令查找某一特定端口，在命令窗口中输入命令中输入netstat -ano |findstr "端口号"，然后回车就可以看到这个端口被哪个应用占用。

   ![windows系统如何查看端口被占用、杀进程](https://exp-picture.cdn.bcebos.com/a151a233ec3834bb5b3eb5ec8714c27bd3823d64.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1)

4. 

   查看到对应的进程id之后，就可以通过id查找对应的进程名称，使用命令tasklist |findstr "进程id号"

   ![windows系统如何查看端口被占用、杀进程](https://exp-picture.cdn.bcebos.com/0d55dc7bd2828689e997a00265f97fbd4d7c3764.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1)

5. 

   通过命令杀掉进程，或者是直接根据进程的名称杀掉所有的进程，，在命令框中输入如下命令taskkill /f /t /im "进程id或者进程名称"

   ![windows系统如何查看端口被占用、杀进程](https://exp-picture.cdn.bcebos.com/87c8bf46b7b1eef9346c5bcfbfb33c4132ba3264.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1)

6. 

   杀掉对应的进程id或者是进程名称之后，然后再通过查找命令，查找对应的端口，现在就可以看到这个端口没有被其他应用所占用，

   ![windows系统如何查看端口被占用、杀进程](https://exp-picture.cdn.bcebos.com/7efc527c34b33c417c6bc4f2887de137c8762e64.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1)