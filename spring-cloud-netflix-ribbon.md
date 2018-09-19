# spring-cloud-netflix-ribbon



![](C:\Users\HP\Desktop\md\RibbonLoadBalancerClient.png)

## ServiceInstanceChooser

`ServiceInstanceChooser`接口源码如下，它只有一个抽象方法`ServiceInstance choose(String serviceId);`它是传递指定的服务ID，获取对应的服务实例。

```java
package org.springframework.cloud.client.loadbalancer;

import org.springframework.cloud.client.ServiceInstance;

/**
 * 由使用负载均衡器选择要发送请求的服务器的类实现。
 * @author Ryan Baxter
 */
public interface ServiceInstanceChooser {
    /**
     * 为指定的服务从LoadBalancer选择ServiceInstance。
     * @param serviceId 查找负载均衡器的服务ID。
     * @return 匹配serviceId的ServiceInstance。
     */
    ServiceInstance choose(String serviceId);
}
```



## LoadBalancerClient

`LoadBalancerClient`是一个负载均衡客户端，

```java
package org.springframework.cloud.client.loadbalancer;

import org.springframework.cloud.client.ServiceInstance;

import java.io.IOException;
import java.net.URI;

/**
 * 表示客户端负载均衡器。
 * @author Spencer Gibb
 */
public interface LoadBalancerClient extends ServiceInstanceChooser {

   /**
    * 使用LoadBalancer中的ServiceInstance对指定的服务执行请求。
    * @param serviceId 查找负载均衡器的服务ID。
    * @param request 允许实现执行前置和后置操作，例如递增的度量。
    * @return 选择的ServiceInstance上的LoadBalancerRequest回调的结果。
    */
   <T> T execute(String serviceId, LoadBalancerRequest<T> request) throws IOException;

   /**
    * 使用LoadBalancer中的ServiceInstance对指定的服务执行请求。
    * @param serviceId 查找负载均衡器的服务ID。
    * @param serviceInstance 执行请求的服务。
    * @param request 允许实现执行前置和后置操作，例如递增的度量。
    * @return 选择的ServiceInstance上的LoadBalancerRequest回调的结果。
    */
   <T> T execute(String serviceId, ServiceInstance serviceInstance, LoadBalancerRequest<T> request) throws IOException;

   /**
    * 为系统使用的主机和端口创建适当的URI。
    * 有些系统使用逻辑服务名作为主机的URI，比如http://myservice/path/to/service。
    * 这将用ServiceInstance的host:port替换服务名。
    * @param instance
    * @param original 将主机作为逻辑服务名的URI。
    * @return 一个重建的URI。
    */
   URI reconstructURI(ServiceInstance instance, URI original);
}
```