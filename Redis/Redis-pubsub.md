# PUBSUB

订阅，取消订阅和发布实现了发布/订阅消息范式（引自wikipedia），发送者（发布者）不是计划发送消息给特定的接收者（订阅者）。而是发布的消息分到不同的频道，不需要知道什么样的订阅者订阅。订阅者对一个或多个频道感兴趣，只需接收感兴趣的消息，不需要知道什么样的发布者发布的。这种发布者和订阅者的解耦合可以带来更大的扩展性和更加动态的网络拓扑。

为了订阅foo和bar，客户端发布一个订阅的频道名称。

```shell
SUBSCRIBE foo bar
```

其他客户端发到这些频道的消息将会被推送到所有订阅者的客户端。

客户端订阅到一个或多个频道不必发出命令，尽管他能订阅和取消订阅其他频道。订阅和取消订阅的响应被封装在发送的消息中，以便客户端只需要读一个连续的消息流。其中第一个元素标识消息类型。



Spring Data Redis

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.MessageListener;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.listener.ChannelTopic;
import org.springframework.data.redis.listener.RedisMessageListenerContainer;
import org.springframework.data.redis.listener.Topic;

import java.util.*;

@Configuration
public class PubSubConfiguration {

    @Bean
    public MessageListener listener(){
        return new DefaultMessageListener();
    }

    @Bean
    public RedisMessageListenerContainer listenerContainer(RedisConnectionFactory collectionFactory){
        RedisMessageListenerContainer listenerContainer = new RedisMessageListenerContainer();
        listenerContainer.setConnectionFactory(collectionFactory);

        List<Topic> topics = new ArrayList<>();
        topics.add(new ChannelTopic("hello"));

        Map<MessageListener, Collection<? extends Topic>> listeners = new HashMap<>();
        listeners.put(listener(),topics);
        listenerContainer.setMessageListeners(listeners);
        return listenerContainer;
    }
}
```

