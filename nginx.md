# Nginx



## Nginx搭建文件下载服务器

**背景**

有时临时需要搭建一个文件服务器、提供文件目录浏览和文件下载功能，有一种比较简单的方法是使用nginx的[目录列表](https://nginxlibrary.com/enable-directory-listing/)功能，有[ngx_http_autoindex_module](http://nginx.org/en/docs/http/ngx_http_autoindex_module.html)提供。

搭建方式非常简单，只需要在nginx的配置文件上添加上`ngx_http_autoindex_module`模块的配置，并定义好一个存放下载文件的目录即可。

**nginx配置文件**

```nginx
http {
	server {
		listen       9090;
		server_name  localhost;
		charset utf-8;
		location / {
			root html/download;
			autoindex on;
			autoindex_exact_size off;
			autoindex_format html;
			autoindex_localtime on;
	   }
	}
}
```

配置说明：

- `root`

  文件存储的目录

- `autoindex`

  ```
  Syntax: 	autoindex on | off;
  Default: 	autoindex off;
  Context: 	http, server, location
  ```

  启用或禁用目录列表输出。

- `autoindex_exact_size`

  ```
  Syntax: 	autoindex_exact_size on | off;
  Default: 	autoindex_exact_size on;
  Context: 	http, server, location
  ```

  对于HTML格式，指定应该在目录清单中输出确切的文件大小，还是四舍五入为千字节、兆字节和千兆字节。

- `autoindex_format`

  ```
  Syntax: 	autoindex_format html | xml | json | jsonp;
  Default: 	autoindex_format html;
  Context: 	http, server, location
  这个指令出现在版本1.7.9中。
  ```

  设置目录列表的格式。

  当使用JSONP格式时，使用回调请求参数设置回调函数的名称。如果参数缺失或值为空，则使用JSON格式。

  可以使用[ngx_http_xslt_module](http://nginx.org/en/docs/http/ngx_http_xslt_module.html)模块转换XML输出。

- `autoindex_localtime`

  ```
  Syntax: 	autoindex_localtime on | off;
  Default: 	autoindex_localtime off;
  Context: 	http, server, location
  ```

  对于HTML格式，指定目录列表中的时间应该在本地时区还是UTC中输出。

在上面的例子中，将文件存储在`/opt/file/download`目录下，通过`http://localhost:9090`即可访问文件目录列表。

*文件目录列表示例：*

- 目录列表格式：HTML

  ![1583044247494](C:\Users\xlp\AppData\Roaming\Typora\typora-user-images\1583044247494.png)

- 目录列表格式：JSON or JSONP

  ![1583044095086](C:\Users\xlp\AppData\Roaming\Typora\typora-user-images\1583044095086.png)

- 目录列表格式：XML

  ![1583044161407](C:\Users\xlp\AppData\Roaming\Typora\typora-user-images\1583044161407.png)



**注意事项：**

1. 只能处理与斜杠字符(‘/’) 结尾的请求，即`location / {...}`（关于这一点已经在ngx_http_autoindex_module模块的文档中有所描述）。
2. 文件目录下（`root /...`）不能包含`index.html`文件，如果包含此文件，将会直接加载`index.html`，而不会是文件目录列表。

**续:修改页面的外观**

上面最终效果过于平淡，不仅白底黑字，而且访问过的链接还会变色。 nginx 第三方的模块 [Fancy Index](https://www.nginx.com/resources/wiki/modules/fancy_index/) , 可以给下载页面增添一些样式。

