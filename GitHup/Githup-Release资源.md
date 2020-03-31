# Githup-Release资源

在下载Githup上的release资源时，虽然链接地址是`github.com`，但是它会重定向到`github-production-release-asset-2e65be.s3.amazonaws.com`，因为不可描述的原因，这个地址在国内并不能被访问，当然，你可解决这个原因。

下面提供一个解决方法：

1. 修改hosts文件

   首先打开网址http://ping.chinaz.com/进行ping检测，找到响应速度比较小的，编辑hosts文件，添加如下内容

   ```
   52.216.146.179  github-production-release-asset-2e65be.s3.amazonaws.com
   52.216.97.171  github-production-release-asset-2e65be.s3.amazonaws.com
   52.216.176.115  github-production-release-asset-2e65be.s3.amazonaws.com
   52.216.130.131  github-production-release-asset-2e65be.s3.amazonaws.com
   52.216.108.11  github-production-release-asset-2e65be.s3.amazonaws.com
   ```

   添加完成之后在进行下载。虽然下载速度较慢，只有10K+的速度，但是只要耐心等待，都可以下载完，总比下载不了好。

2. 通过IDM下载

   [IDM](https://chromecj.com/genuine/2019-05/2586.html) 全称 Internet Download Manager，是一款非常优秀的[多线程](https://chromecj.com/tag/多线程)下载和视频嗅探工具，不仅可以显著提高文件下载速度，配合IDM[浏览器扩展插件](https://chromecj.com/)，还可以嗅探并下载YouTube、知乎等网页视频。截止2020年2月23日最新版本是v6.36。

   官方地址：http://www.internetdownloadmanager.com/

   使用教程地址：

   ①. https://chromecj.com/software/2019-10/2954.html

   ②. https://www.52pojie.cn/thread-947484-1-1.html?tdsourcetag=s_pctim_aiomsg

