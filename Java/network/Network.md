# Network





## IP



## InetAddress

全类名：`java.net.InetAddress`



- `public static InetAddress getByName(String host)`：
- `public static InetAddress getByAddress(byte[] addr)`：
- `public static InetAddress getByAddress(String host, byte[] addr)`：
- `public static InetAddress[] getAllByName(String host)`：
- `public static InetAddress getLocalHost()`：
- `public static InetAddress getLoopbackAddress()`：



## InetSocketAddress

全类名：`java.net.InetSocketAddress`

- `public InetSocketAddress(int port)` : 

- `public InetSocketAddress(String hostname, int port)` : 

- `public InetSocketAddress(InetAddress addr, int port)` : 

  





- `public final InetAddress getAddress()` :  
- `public final String getHostName()` :  
- `public final String getHostString()` :  
- `public final int getPort()` :  
- `public final boolean isUnresolved()` :  检查地址是否已被解析。









## Socket

全类名：`java.net.Socket`

- `public Socket()` :
- `public Socket(Proxy proxy)` :
- `public Socket(String host, int port)` :
- `public Socket(InetAddress address, int port)` :
- `public Socket(String host, int port, InetAddress localAddr, int localPort)` :
- `public Socket(InetAddress address, int port, InetAddress localAddr, int localPort)` : 

@Deprecated

- `public Socket(String host, int port, boolean stream)` : 
- `public Socket(InetAddress host, int port, boolean stream)` : 









```java
@SuppressWarnings("all")
public static void communicationServer() throws IOException {
    //创建一个Socket服务，监听端口8080端口
    ServerSocket server = new ServerSocket(8080);
    while (true) {
        //阻塞式监听请求
        Socket socket = server.accept();
        //获取请求
        byte[] requestData = request(socket);
        //处理请求
        byte[] responseData = process(requestData);
        //响应请求
        response(socket, responseData);
    }
}

/**
 * 读取请求数据。
 * 注意，此处不能关闭流，因为后续还要向客户端写。
 */
private static byte[] request(Socket socket) throws IOException {
    InputStream is = socket.getInputStream();
    byte[] b = new byte[1024];
    int l;
    ByteArrayOutputStream bos = new ByteArrayOutputStream();
    while ((l = is.read(b)) != -1) {
        bos.write(b, 0, l);
    }
    return bos.toByteArray();
}

private static void response(Socket socket, byte[] data) throws IOException {
    try (OutputStream os = socket.getOutputStream()) {
        os.write(data);
        os.flush();
    }
}

private static byte[] process(byte[] data) {
    System.out.println("请求数据 : " + new String(data));
    System.out.println("逻辑处理...");
    System.out.println("处理完毕，开始响应...");
    return "Hello client.".getBytes();
}
```



```java
public static void communicationClient() throws IOException {
    //IP地址
    InetAddress address = InetAddress.getByName("127.0.0.1");
    //创建一个socket连接
    Socket socket = new Socket(address,8080);
    //向服务器发送数据
    send(socket,"Hello server.".getBytes());
    //告诉服务器已经发送完毕
    socket.shutdownOutput();
    //接收服务器响应的数据
    byte[] data = accept(socket);
    System.out.println(new String(data));
    //通信完毕，关闭连接
    socket.close();
}

/**
 * 发送请求数据。
 * 注意，此处不能关闭流，因为后续还要向服务端端发送。
 */
private static void send(Socket socket, byte[] data) throws IOException {
    OutputStream os = socket.getOutputStream();
    os.write(data);
    os.flush();
}

private static byte[] accept(Socket socket) throws IOException {
    try (InputStream is = socket.getInputStream()){
        byte[] b = new byte[1024];
        int l;
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        while ((l = is.read(b)) != -1) {
            bos.write(b, 0, l);
        }
        return bos.toByteArray();
    }
}
```









## UDP







```JAVA
public static void updServer() throws IOException {
    // 创建一个UDP服务端
    DatagramSocket socket = new DatagramSocket(8080);
    //创建一个数据包，用于接收数据
    byte[] b = new byte[1024];
    DatagramPacket packet = new DatagramPacket(b, b.length);
    //接收数据
    socket.receive(packet);
    System.out.println(new String(packet.getData(), 0, packet.getLength()));
    //关闭socket
    socket.close();
}
```



```java
public static void udpClient() throws IOException {
    //创建一个UDP客户端
    DatagramSocket socket = new DatagramSocket();
    //创建一个数据包
    byte[] data = "Hello UDP server.".getBytes();
    int offset = 0, length = data.length;
    InetAddress address = InetAddress.getByName("127.0.0.1");
    int port = 8080;
    DatagramPacket packet = new DatagramPacket(data, offset, length, address, port);
    //发送数据包
    socket.send(packet);
    //关闭socket
    socket.close();
}
```



