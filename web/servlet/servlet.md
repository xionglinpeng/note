# Setvlet









## 异步处理支持

在Servlet3.0之前，Servlet采用Thread-Per-Request的方式处理请求，即每一次HTTP请求都由某一个线程从头到尾负责处理。如果一个请求需要进行IO操作，比如访问数据库、调用第三方服务接口等，那么其所对应的线程将同步的等待IO操作完成，而IO操作是非常慢的，所以此时的线程将不能及时放回到线程池以供后续使用。在并发量越来越大的情况下，这将带来严重的性能问题。即便是像Spring这样的高层框架也脱离不了这样的桎梏。为了解决这样的问题，Setvlet3.0引入了异步处理支持，然后又在Setvlet3.1引入了非阻塞IO来进一步增强异步的处理性能。











```java

```



AsyncContext API：

- `public ServletRequest getRequest();`
- `public ServletResponse getResponse();`
- `public boolean hasOriginalRequestAndResponse();`
- `public void dispatch();`
- `public void dispatch(String path);`
- `public void dispatch(ServletContext context, String path);`
- `public void start(Runnable run);`
- `public void addListener(AsyncListener listener);`
- `public void addListener(AsyncListener listener,ServletRequest servletRequest,ServletResponse servletResponse);`
- `public <T extends AsyncListener> T createListener(Class<T> clazz)throws ServletException; `
- `public void setTimeout(long timeout);`
- `public long getTimeout();`











资料：

- [Servlet 3.0 新特性详解]: https://www.ibm.com/developerworks/cn/java/j-lo-servlet30/#major3

- [Servlet 4.0 入门]:https://www.ibm.com/developerworks/cn/java/j-javaee8-servlet4/index.html

  

  