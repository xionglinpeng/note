# gRPC



gRPC官网：<https://grpc.io/>

gRPC官方文档中文版：<http://doc.oschina.net/grpc>

Github：<https://github.com/grpc/grpc-java>





其他临时资料<https://my.oschina.net/wangmengjun/blog/909867>



**一、gRPC 简介**

gRPC 是Go实现的：一个高性能，开源，将移动和HTTP/2放在首位通用的RPC框架。使用gRPC可以在客户端调用不同机器上的服务端的方法，而客户端和服务端的开发语言和

运行环境可以有很多种，基本涵盖了主流语言和平台。双方交互的协议可以在proto文件中定义，客户端和服务端可以很方便的通过工具生成协议和代理代码。而消息的编码是采

用`google protocol buffer`，数据量小、速度快。

gRPC具有以下特点：

（1）基于 HTTP/2， 继而提供了连接多路复用、Body 和 Header 压缩等机制。可以节省带宽、降低TCP链接次数、节省CPU使用和延长电池寿命等。

（2）支持主流开发语言（C, C++, Python, PHP, Ruby, NodeJS, C#, Objective-C、Golang、Java）

（3）IDL (Interface Definition Language) 层使用了 Protocol Buffers, 非常适合团队的接口设计