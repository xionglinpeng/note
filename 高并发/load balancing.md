# Load Balancing

- 硬件负载均衡
- 服务端负载均衡
- 客户端负载均衡

## load balance algorithm
### 随机算法

#### 完全随机



```java
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

public class FullRandom {
    private List<String> servers = new ArrayList<String>() {
        {
            add("192.168.1.1");
            add("192.168.1.2");
            add("192.168.1.3");
        }
    };

    private Random random = new Random();

    public String loadBalanceIP(){
        int randomNumber = random.nextInt(servers.size());
        return servers.get(randomNumber);
    }

    public static void main(String[] args) {
        FullRandom fullRandom = new FullRandom();
        for (int i = 0; i < 5; i++) {
            System.out.println(fullRandom.loadBalanceIP());
        }
    }
}
```



#### 加权随机



```java
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

public class WeightRandom {

    @Data
    @AllArgsConstructor
    private static class Server{
        private int weight;
        private String ip;
    }

    @Getter
    private static class Servers{
        private List<Server> serverList = new ArrayList<>();
        private int weightNum;

        Servers(){
            this.serverList.add(new Server(2,"192.168.1.100"));
            this.serverList.add(new Server(7,"192.168.1.101"));
            this.serverList.add(new Server(3,"192.168.1.102"));
            this.weightNum = this.serverList.stream().mapToInt(Server::getWeight).sum();
        }
    }

    private Servers servers = new Servers();
    private Random random = new Random();

    public String loadBalanceIP(){
        int randomWeight = random.nextInt(servers.getWeightNum());
        for (Server server : servers.getServerList()) {
            if (server.getWeight() >= randomWeight)
                return server.getIp();
            randomWeight -= server.getWeight();
        }
        return null;
    }

    public static void main(String[] args) {
        WeightRandom weightRandom = new WeightRandom();
        for (int i = 0; i < 5; i++) {
            System.out.println(weightRandom.loadBalanceIP());
        }
    }
}
```



### 轮询

#### 完全轮询

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import java.util.ArrayList;
import java.util.List;

public class FullPolling {

    @Data
    @AllArgsConstructor
    public static class Server{
        private String ip;
    }

    public static class Servers{
        private List<Server> servers = new ArrayList<Server>(){
            {
                add(new Server("192.168.100.1"));
                add(new Server("192.168.100.2"));
                add(new Server("192.168.100.3"));
            }
        };

        public List<Server> getServers() {
            return servers;
        }
    }
    static Servers servers = new Servers();
    static int index;

    public static String obtain(){
        List<Server> serverList = servers.getServers();
        if (index == serverList.size())
            index = 0;
        return serverList.get(index++).getIp();
    }

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            System.out.println(obtain());
        }
    }
}
```
#### 加权轮询

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import java.util.ArrayList;
import java.util.List;

public class WeightPolling {

    @Data
    @AllArgsConstructor
    public static class Server{
        private int weight;
        private String ip;
    }

    public static class Servers{
        private List<Server> servers = new ArrayList<Server>(){
            {
                add(new Server(5,"192.168.100.1"));
                add(new Server(1,"192.168.100.2"));
                add(new Server(2,"192.168.100.3"));
            }
        };

        private int weightSum;

        public List<Server> getServers() {
            return servers;
        }

        public int getWeightSum() {
            if (weightSum == 0) {
                synchronized (this){
                    if (weightSum == 0) {
                        weightSum = servers.stream().mapToInt(Server::getWeight).sum();
                    }
                }
            }
            return weightSum;
        }
    }
    static Servers servers = new Servers();
    static int index;

    public static String obtain(){
        int number = (index++) % servers.getWeightSum();
        for (Server server : servers.getServers()) {
            if (server.getWeight() > number) {
                return server.getIp();
            }
            number -= server.getWeight();
        }
        return null;
    }

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            System.out.println(obtain());
        }
    }
}
```
#### 平滑加权轮询

```java
import lombok.Data;

import java.util.ArrayList;
import java.util.List;

public class SmoothWeightedPolling {

    @Data
    public static class Server{
        private int fixedWeight;
        private int notFixedWeight;
        private String ip;

        public Server(int fixedWeight, String ip) {
            this.fixedWeight = fixedWeight;
            this.notFixedWeight = fixedWeight;
            this.ip = ip;
        }
    }

    public static class Servers{
        private List<Server> servers = new ArrayList<Server>(){
            {
                add(new Server(5,"192.168.100.1"));
                add(new Server(10,"192.168.100.2"));
                add(new Server(1,"192.168.100.3"));
            }
        };

        private int weightSum;

        public List<Server> getServers() {
            return servers;
        }

        public int getWeightSum() {
            if (weightSum == 0) {
                synchronized (this){
                    if (weightSum == 0) {
                        weightSum = 
                            servers.stream().mapToInt(Server::getFixedWeight).sum();
                    }
                }
            }
            return weightSum;
        }
    }

    private static Servers servers = new Servers();

    public static String obtain(){
        //获取最大权重服务
        Server maxWeightServer = null;
        for (Server server : servers.getServers()) {
            if (maxWeightServer == null ||
                    server.getNotFixedWeight() > maxWeightServer.getNotFixedWeight()) {
                maxWeightServer = server;
            }
            //重置非固定权重（上一次非固定权重+固定权重）
            server.setNotFixedWeight(server.getNotFixedWeight()+server.getFixedWeight());
        }
        //最大权重服务的非固定权重（上一次非固定权重+固定权重-固定权重和）
        assert maxWeightServer != null;
        maxWeightServer.setNotFixedWeight(
            maxWeightServer.getNotFixedWeight() - servers.getWeightSum());
        //日志输出
        for (Server server : servers.getServers()) {
            System.out.print("("+server.getNotFixedWeight()+") - ");
        }
        return maxWeightServer.getIp();
    }

    public static void main(String[] args) {
        for (int i = 1; i <= 20; i++) {
            System.out.print(i+" => ");
            String ip = obtain();
            System.out.println(ip);
        }
    }
}
```

### 哈希

### 最小压力