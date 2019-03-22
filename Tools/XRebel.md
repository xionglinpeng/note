官方网址：https://zeroturnaround.com/

## XRebel Quick Start

**Download**(Getting XRebel)

1. [Download Rebel](https://zeroturnaround.com/software/xrebel/download/).
2. 将存档文件解压到您选择的文件夹中。
3. Next → Startup.

**Startup**(Adding XRebel to you application server)

1. Add XRebel to you server startup parameters: `-javaagent:[/path/to/]xrebel.jar`.

2. Start your server!

3. Next → Activation

   Good to know

   - Don't konw how to configure your server parameters? [Follow these instructions](http://manuals.zeroturnaround.com/xrebel/install/index.html#adding-xrebel-to-your-server).
   - Want to konw whether **XRebel** supports your browser,server or database? Check out all [supported environments](http://manuals.zeroturnaround.com/xrebel/support/index.html#xrebel-support).

   Did this work?
   当XRebel启动成功时，您将看到XRebel工具栏出现在web应用程序的左下角。控制台也显示成功启动:
   通过访问地址`http://<host>:<port>/xrebel`，即可访问XRebel监控界面。

   ```
   XRebel: Starting logging to file: C:\Users\dandelion\.xrebel\xrebel.log
   XRebel: 
   XRebel: ################################################################
   XRebel: 
   XRebel:  XRebel 3.6.0 (201812111048)
   XRebel:  (c) Copyright ZeroTurnaround AS, Estonia, Tallinn.
   XRebel: 
   XRebel:  For questions and support, contact xrebel@zeroturnaround.com
   XRebel: 
   XRebel: ################################################################
   XRebel:
       
   ...........................................................................    
       
   XRebel: Started XRebel for application: http://localhost:9000/
   XRebel: XRebel UI is available at http://localhost:9000/xrebel
   XRebel: 
   XRebel: ################################################################
   XRebel: 
   XRebel:  No valid license found
   XRebel: 
   XRebel:  Please use the in-app license activation wizard to set up your license.
   XRebel: 
   XRebel: ################################################################
   XRebel:
   ```

**Activation**(Getting your license)

1. 激活对话框会在XRebel启动后自动弹出。

2. Fill to the from.

3. 按激活XRebel试用。

   ```
   第一次运行时会要求激活:
   I have a license
   Already have an XRebel license? Press 'I have a license' to activate using an existing license.
   - Activation key :Paste your activation key if you have one already.
   - License file (xrebel.lic) :指向您的许可文件。当您购买XRebel时，可以通过电子邮件获得此文件。
   - License server :Enter your Group URL and email.You get the Group URL from network administrator.
   
   这里采用第三种方式License server，使用http://idea.lanyus.com/授权服务器激活。
   操作如下：
   在弹出的激活界面，选择第三个选项（License server），填写如下：
   1. Group URL → http://idea.lanyus.com/{xxx}
   2. Email → 填写任意一个邮件
   
   不出意外，即可激活。
   ```

   ![](http://manuals.zeroturnaround.com/xrebel/_images/activation-form.png)

4. Next → XRebel experience.

**XRebel experience**(Useing XRebel)

1. Click around in your application.

2. 请注意左下角的XRebel工具栏。

3. 按下工具栏按钮打开相关视图:

   ![](https://zeroturnaround.com/wp-content/uploads/2014/08/xrebel-toolbar-334.png)





