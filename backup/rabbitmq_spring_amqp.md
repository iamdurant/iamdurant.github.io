##### 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
    <version>3.2.10</version>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

##### 如何声明队列、如何创建消费者？

**由配置类完成：**

- 声明队列，，注入容器
- 创建consumer类，@Bean注入容器

对于consumer，也可直接@component注入容器

**consumer涉及注解：**

- @RabbitListener
- @RabbitHandler

```java
@Configuration
@Profile({"tut2", "work-queues"})
public class Tut2Config {
    @Bean
    public Queue queue() {
        return new Queue("hello_a");
    }

    @Profile({"receiver"})
    private static class ReceiverConfig {
        @Bean
        public Tut2Receiver receiver1() {
            return new Tut2Receiver(1);
        }

        @Bean
        public Tut2Receiver receiver2() {
            return new Tut2Receiver(2);
        }
    }

    @Profile({"sender"})
    @Bean
    public Tut2Sender sender() {
        return new Tut2Sender();
    }
}
```

##### 如何发送消息？

- RabbitTemplate对底层代码做了封装，可直接注入使用

- 对RabbiTemplate再做一层封装

##### fanout使用示例

- 配置类

  ```java
  @Configuration
  @Profile({"tut3", "publish-subscribe"})
  public class Tut3Config {
      @Bean
      public FanoutExchange fanoutExchange() {
          return new FanoutExchange("tut.fanout");
      }
  
      @Profile({"receiver"})
      private static class Receiver {
          @Bean
          public Queue autoDeleteQueue1() {
              return new AnonymousQueue();
          }
  
          @Bean
          public Queue autoDeleteQueue2() {
              return new AnonymousQueue();
          }
  
          @Bean
          public Binding binding1(@Autowired @Qualifier("autoDeleteQueue1") Queue autoDeleteQueue1, FanoutExchange fanoutExchange) {
              return BindingBuilder.bind(autoDeleteQueue1).to(fanoutExchange);
          }
  
          @Bean
          public Binding binding2(@Autowired @Qualifier("autoDeleteQueue2") Queue autoDeleteQueue2, FanoutExchange fanoutExchange) {
              return BindingBuilder.bind(autoDeleteQueue2).to(fanoutExchange);
          }
  
          @Bean
          public Tut3Receiver tut3Receiver() {
              return new Tut3Receiver();
          }
      }
  
      @Profile({"sender"})
      @Bean
      public Tut3Sender tut3Sender() {
          return new Tut3Sender();
      }
  }
  ```

- producer

  ```java
  public class Tut3Sender {
      @Autowired
      private RabbitTemplate rabbit;
  
      @Autowired
      private FanoutExchange fanoutExchange;
  
      @Scheduled(fixedDelay = 1000, initialDelay = 500)
      public void send() {
          Random random = new Random();
          String message = "广播消息" + random.nextInt(Integer.MAX_VALUE);
          rabbit.convertAndSend(fanoutExchange.getName(), "", message);
      }
  }
  ```

- consumer

  ```java
  public class Tut3Receiver {
      @RabbitListener(queues = {"#{autoDeleteQueue1.name}"})
      public void receive1(String body) {
          System.out.println("worker-1消费消息：" + body);
      }
  
      @RabbitListener(queues = {"#{autoDeleteQueue2.name}"})
      public void receive2(String body) {
          System.out.println("worker-2消费消息：" + body);
      }
  }
  ```

##### direct使用示例

- 配置类

  ```java
  @Configuration
  @Profile({"tut4"})
  public class Tut4Config {
      @Bean
      public DirectExchange directExchange() {
          return new DirectExchange("tut.direct");
      }
  
      @Profile({"receiver"})
      private static class Receiver {
          @Bean
          public Queue q1() {
              return new AnonymousQueue();
          }
  
          @Bean
          public Queue q2() {
              return new AnonymousQueue();
          }
  
          @Bean
          public Binding b1(@Autowired @Qualifier("q1") AnonymousQueue q1, DirectExchange exchange) {
              return BindingBuilder.bind(q1).to(exchange).with("1");
          }
  
          @Bean
          public Binding b2(@Autowired @Qualifier("q1")AnonymousQueue q1, DirectExchange exchange) {
              return BindingBuilder.bind(q1).to(exchange).with("2");
          }
  
          @Bean
          public Binding b3(@Autowired @Qualifier("q2")AnonymousQueue q2, DirectExchange exchange) {
              return BindingBuilder.bind(q2).to(exchange).with("3");
          }
  
          @Bean
          public Binding b4(@Autowired @Qualifier("q2")AnonymousQueue q2, DirectExchange exchange) {
              return BindingBuilder.bind(q2).to(exchange).with("4");
          }
  
          @Bean
          public Tut4Receiver receiver() {
              return new Tut4Receiver();
          }
      }
  
      @Profile({"sender"})
      @Bean
      public Tut4Sender sender() {
          return new Tut4Sender();
      }
  }
  ```

- producer

  ```java
  public class Tut4Sender {
      @Autowired
      private RabbitTemplate rabbit;
  
      @Autowired
      private DirectExchange exchange;
  
      @Scheduled(fixedDelay = 1000, initialDelay = 500)
      public void send() {
          Random random = new Random();
          int v = random.nextInt(5);
          rabbit.convertAndSend(exchange.getName(),
                                String.valueOf(v),
                                v);
      }
  }
  ```

- consumer

  ```java
  public class Tut4Receiver {
      @RabbitListener(queues = {"#{q1.name}"})
      public void receive1(String body) {
          System.out.println("1-消费了消息：" + body);
      }
  
      @RabbitListener(queues = {"#{q2.name}"})
      public void receive2(String body) {
          System.out.println("2-消费了消息：" + body);
      }
  }
  ```

##### topic使用示例

- 配置类

  ```java
  @Configuration
  @Profile({"tut5"})
  public class Tut5Config {
      @Bean
      public TopicExchange topicExchange() {
          return new TopicExchange("tut.topic_exchange");
      }
  
      @Profile({"receiver"})
      private static class Receiver {
          @Bean
          public Queue q1() {
              return new AnonymousQueue();
          }
  
          @Bean
          public Queue q2() {
              return new AnonymousQueue();
          }
  
          @Bean
          public Binding b1(@Autowired @Qualifier("q1") Queue q1, TopicExchange exchange) {
              return BindingBuilder.bind(q1).to(exchange).with("*.orange.*");
          }
  
          @Bean
          public Binding b2(@Autowired @Qualifier("q2") Queue q2, TopicExchange exchange) {
              return BindingBuilder.bind(q2).to(exchange).with("*.*.rabbit");
          }
  
          @Bean
          public Binding b3(@Autowired @Qualifier("q2") Queue q2, TopicExchange exchange) {
              return BindingBuilder.bind(q2).to(exchange).with("lazy.#");
          }
  
          @Bean
          public Tut5Receiver receiver() {
              return  new Tut5Receiver();
          }
      }
  
      @Profile({"sender"})
      @Bean
      public Tut5Sender sender() {
          return new Tut5Sender();
      }
  }
  ```

- producer

  ```java
  public class Tut5Sender {
      @Autowired
      private RabbitTemplate rabbit;
  
      @Autowired
      private TopicExchange exchange;
  
      private final String[] routingKeys = {"quick.orange.rabbit", "lazy.orange.elephant", "quick.orange.fox",
              "lazy.brown.fox", "lazy.pink.rabbit", "quick.brown.fox"};
  
      @Scheduled(fixedDelay = 2000, initialDelay = 500)
      public void send() {
          Random random = new Random();
          String routingKey = routingKeys[random.nextInt(routingKeys.length)];
          rabbit.convertAndSend(exchange.getName(), routingKey, routingKey);
      }
  }
  ```

- consumer

  ```java
  public class Tut5Receiver {
      @RabbitListener(queues = {"#{q1.name}"})
      public void receive1(String body)  {
          System.out.println("*.orange.* 消费消息：" + body);
      }
  
      @RabbitListener(queues = {"#{q2.name}"})
      public void receive2(String body)  {
          System.out.println("*.*.rabbit | lazy.# 消费消息：" + body);
      }
  }
  ```

  

