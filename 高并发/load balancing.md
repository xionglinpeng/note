# Load Balancing

- 硬件负载均衡
- 服务端负载均衡
- 客户端负载均衡

## load balance algorithm
- 随机算法
  - 完全随机
  - 加权随机
- 轮询
  - 完全轮询
  ```java
  package com.resourcces.loadBalance;

  import java.util.ArrayList;
  import java.util.List;

  /**
   * 完全轮询
   */
  public class FullPolling {

      public static class Server{
          private String ip;

          public Server(String ip) {
              this.ip = ip;
          }

          public String getIp() {
              return ip;
          }

          public void setIp(String ip) {
              this.ip = ip;
          }
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
  - 加权轮询
  ```java
  package com.resourcces.loadBalance;

  import java.util.ArrayList;
  import java.util.List;

  /**
   * 加权轮询
   */
  public class WeightPolling {

      public static class Server{
          private int weight;
          private String ip;

          public Server(int weight, String ip) {
              this.weight = weight;
              this.ip = ip;
          }

          public int getWeight() {
              return weight;
          }

          public void setWeight(int weight) {
              this.weight = weight;
          }

          public String getIp() {
              return ip;
          }

          public void setIp(String ip) {
              this.ip = ip;
          }
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
  - 平滑加权轮询
  ```java
  package com.resourcces.loadBalance;

  import java.util.ArrayList;
  import java.util.Arrays;
  import java.util.List;

  public class SmoothWeightedPolling {

      public static class Server{
          private int fixedWeight;
          private int notFixedWeight;
          private String ip;

          public Server(int fixedWeight, String ip) {
              this.fixedWeight = fixedWeight;
              this.notFixedWeight = fixedWeight;
              this.ip = ip;
          }

          public int getFixedWeight() {
              return fixedWeight;
          }

          public void setFixedWeight(int fixedWeight) {
              this.fixedWeight = fixedWeight;
          }

          public int getNotFixedWeight() {
              return notFixedWeight;
          }

          public void setNotFixedWeight(int notFixedWeight) {
              this.notFixedWeight = notFixedWeight;
          }

          public String getIp() {
              return ip;
          }

          public void setIp(String ip) {
              this.ip = ip;
          }

          @Override
          public String toString() {
              return "Server{" +
                      "fixedWeight=" + fixedWeight +
                      ", notFixedWeight=" + notFixedWeight +
                      ", ip='" + ip + '\'' +
                      '}';
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
                          weightSum = servers.stream().mapToInt(Server::getFixedWeight).sum();
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
              server.setNotFixedWeight(server.getNotFixedWeight()+server.getFixedWeight());
          }

          maxWeightServer.setNotFixedWeight(maxWeightServer.getNotFixedWeight() - servers.getWeightSum());

          //设置有所服务新的非固定权重
          for (Server server : servers.getServers()) {
  //            if (maxWeightServer == server) {
  //                server.setNotFixedWeight(server.getNotFixedWeight()+server.getFixedWeight() - servers.getWeightSum());
  //            } else {
  //                server.setNotFixedWeight(server.getNotFixedWeight()+server.getFixedWeight());
  //            }
  //
              System.out.print("("+server.getNotFixedWeight()+") - ");
          }
          System.out.println();

          return maxWeightServer.getIp();
      }


      public static void main(String[] args) {
          for (int i = 0; i < 20; i++) {
              System.out.print((i+1)+" => ");
              String ip = obtain();
  //            System.out.println(ip);
          }
      }

  }

  ```
- 哈希
- 最小压力
