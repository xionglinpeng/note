浏览器在某些请求中，在正式通信前会增加一次HTTP查询请求，称为“预检”请求（preflight）。

浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用那些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的XMLHttpRequest请求，否则这就报错。



```
Request URL: http://127.0.0.1:8080/data/brands/list
Request Method: OPTIONS
Status Code: 403 
Remote Address: 127.0.0.1:8080
Referrer Policy: no-referrer-when-downgrade
```





```
OPTIONS http://127.0.0.1:8080/index 403 ()
Failed to load http://127.0.0.1:8080/data/brands/list: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:8080' is therefore not allowed access. The response had HTTP status code 403.
```

