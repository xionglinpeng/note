AIO Server

```JAVA
public void start(int port) throws IOException, InterruptedException {
    ExecutorService executor = Executors.newCachedThreadPool();
    AsynchronousChannelGroup.withCachedThreadPool(executor, 1);
    AsynchronousServerSocketChannel server = AsynchronousServerSocketChannel.open();
    server.bind(new InetSocketAddress(port));
    System.out.println("服务启动，监听端口：" + port);
    server.accept(null, new CompletionHandler<AsynchronousSocketChannel, Object>() {

        final ByteBuffer buffer = ByteBuffer.allocate(1024);

        @SneakyThrows
        @Override
        public void completed(AsynchronousSocketChannel channel, Object attachment) {
            System.out.println("I/O操作成功，开始获取数据");
            try {
                buffer.clear();
                channel.read(buffer);
                buffer.flip();
                System.out.println(new String(buffer.array()));
                channel.write(ByteBuffer.wrap("我收到啦".getBytes()));
            } finally {
                server.accept(null, this);
            }
            System.out.println("操作完成");
        }

        @Override
        public void failed(Throwable exc, Object attachment) {
            System.out.println("I/O操作失败：" + exc.getMessage());
        }
    });
    Thread.sleep(Integer.MAX_VALUE);
}
```

AIO Client

```java
public static void main(String[] args) throws IOException, InterruptedException {
    AsynchronousSocketChannel socketChannel = AsynchronousSocketChannel.open();

    socketChannel.connect(new InetSocketAddress("127.0.0.1", 8080), null,
                          new CompletionHandler<Void, Object>() {
                              @Override
                              public void completed(Void result, Object attachment) {
                                  socketChannel.write(ByteBuffer.wrap("Hello AIO.".getBytes()));
                                  System.out.println("已发送数据到服务端");
                              }

                              @Override
                              public void failed(Throwable exc, Object attachment) {
                                  exc.printStackTrace();
                              }
                          });

    final ByteBuffer buffer = ByteBuffer.allocate(1024);
    socketChannel.read(buffer, null, new CompletionHandler<Integer, Object>() {
        @Override
        public void completed(Integer result, Object attachment) {
            System.out.println("I/O操作完成："+result);
            System.out.println("反馈结果："+new String(buffer.array()));
        }

        @Override
        public void failed(Throwable exc, Object attachment) {
            exc.printStackTrace();
        }
    });
    Thread.sleep(Integer.MAX_VALUE);
}
```