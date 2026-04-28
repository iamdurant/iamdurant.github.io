##### rabbitmq安装教程

**环境：ubuntu**

1. 直接使用[官网](https://www.rabbitmq.com/docs/install-debian#apt-quick-start-cloudsmith)提供的一键脚本，会安装较新的erlang以及rabbit-server

2. 启用插件管理，开启web端

   ```shell
   rabbitmq-plugins enable rabbitmq_management
   ```

3. 默认web端只支持localhost访问，额外配置：

   ```shell
   # 添加用户
   rabbitmqctl add_user jasper pubgM666
   # 设置为管理员
   rabbitmqctl set_user_tags jasper administrator
   # 授予权限
   rabbitmqctl set_permissions -p / jasper ".*" ".*" ".*"
   # restart
   systemctl restart rabbitmq-server
   ```



##### tabbitmq简单使用

###### 依赖

```xml
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.19.0</version>
</dependency>

<!-- slf4j日志门面 -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>2.0.11</version>
</dependency>

<!-- slf4j内部实现 -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>2.0.11</version>
</dependency>
```



###### hello world

1. producer

   ```java
   package wwb.rabbitmq.producer;
   
   import com.rabbitmq.client.Channel;
   import com.rabbitmq.client.Connection;
   import com.rabbitmq.client.ConnectionFactory;
   
   import java.nio.charset.StandardCharsets;
   
   public class HelloProducer {
       public static void main(String[] args) {
           ConnectionFactory factory = new ConnectionFactory();
           factory.setHost("172.24.192.134");
           factory.setPort(5672);
           factory.setUsername("jasper");
           factory.setPassword("pubgM666");
           factory.setVirtualHost("/");
   
           try (Connection connection = factory.newConnection();
                Channel channel = connection.createChannel()) {
               channel.queueDeclare("hello", false, false, false, null);
               channel.basicPublish("", "hello", null, "你好".getBytes(StandardCharsets.UTF_8));
           } catch (Exception e) {
   
           }
       }
   }
   ```

   

2. consumer

   ```java
   package wwb.rabbitmq.consumer;
   
   import com.rabbitmq.client.Channel;
   import com.rabbitmq.client.Connection;
   import com.rabbitmq.client.ConnectionFactory;
   import com.rabbitmq.client.DeliverCallback;
   
   public class HelloConsumer {
       public static void main(String[] args) throws Exception{
           ConnectionFactory factory = new ConnectionFactory();
           factory.setHost("172.24.192.134");
           factory.setPort(5672);
           factory.setUsername("jasper");
           factory.setPassword("pubgM666");
           factory.setVirtualHost("/");
   
           Connection connection = factory.newConnection();
           Channel channel = connection.createChannel();
   
           channel.queueDeclare("hello", false, false, false, null);
           DeliverCallback deliverCallback = (consumerTag, mess) -> {
               String message = new String(mess.getBody(), "UTF-8");
               System.out.println(message);
           };
           channel.basicConsume("hello", true, deliverCallback, tag -> {});
       }
   }
   ```

###### acknowlegement

```java
public static void main(String[] args) throws Exception{
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("172.24.192.134");
    factory.setPort(5672);
    factory.setUsername("jasper");
    factory.setPassword("pubgM666");

    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.queueDeclare("workqueues", false, false, false, null);
    // 设置一次只接受一个未消费消息
    channel.basicQos(1); 

    Random random = new Random();
    StringBuilder sb = new StringBuilder();
    for(int i = 0;i < 4;i++) {
        sb.append((char) (random.nextInt(26) + 97));
    }
    String workerName = sb.toString();

    DeliverCallback callback = (tag, entity) -> {
        String body = new String(entity.getBody(), StandardCharsets.UTF_8);
        for(int i = 0;i < body.length();++i) {
            if(body.charAt(i) == '.') {
                try {
                    TimeUnit.MILLISECONDS.sleep(300);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }
        // 手动返回ack
        channel.basicAck(entity.getEnvelope().getDeliveryTag(), false);
        System.out.println(workerName + "消费了：" + body);
    };

    channel.basicConsume("workqueues", false, callback, tag -> {});
}
```

###### message durability

如何确保消息不会丢失：

1. 队列持久化，声明队列时，第二个参数

   ```java
   channel.queueDeclare("workqueues", true, false, false, null);
   ```

2. 消息持久化，发送消息时声明

   ```java
   channel.basicPublish("", "workqueues", MessageProperties.PERSISTENT_TEXT_PLAIN, body.getBytes(StandardCharsets.UTF_8));
   ```

如此过后消息可靠性大幅提升，已经可以满足大部分情况，但是还是存在一个小的窗口导致消息丢失，因为rabbitmq收到消息时，并不会保证将每条消息都刷盘，而是存在缓冲区。若需要保证百分百不丢失，则还需要**publisher confirm**。

##### exchanges

交换机类型：

- direct
- topic
- headers
- fanout

如何声明交换机

```java
channel.exchangeDeclare("exchangeName", "fanout");
```

###### temporary queue

如何声明一个临时队列，只要没有consumer，那么此队列会被删除

```java
String queueName = channel.declareQueue().getQueue();
```

###### queue bind

```java
channel.queueBind(queueName, "exchangeName", "");
```

###### fanout广播示例

1. producer

   ```java
   public static void main(String[] args) throws Exception{
       ConnectionFactory factory = new ConnectionFactory();
       factory.setHost("172.24.192.134");
       factory.setPort(5672);
       factory.setUsername("jasper");
       factory.setPassword("pubgM666");
       factory.setVirtualHost("/");
   
       Connection connection = factory.newConnection();
       Channel channel = connection.createChannel();
   
       // 声明exchange
       channel.exchangeDeclare("logs", "fanout");
       for(int i = 0;i < 10;++i) {
           channel.basicPublish("logs", "", null, ("logs-" + i).getBytes(StandardCharsets.UTF_8));
       }
   
       connection.close();
   }
   ```

2. consumer

   ```java
   public static void main(String[] args) throws Exception{
       ConnectionFactory factory = new ConnectionFactory();
       factory.setHost("172.24.192.134");
       factory.setPort(5672);
       factory.setUsername("jasper");
       factory.setPassword("pubgM666");
       factory.setVirtualHost("/");
   
       Connection connection = factory.newConnection();
       Channel channel = connection.createChannel();
   
       channel.basicQos(1);
       channel.exchangeDeclare("logs", "fanout");
   
       String queueName = channel.queueDeclare().getQueue();
       channel.queueBind(queueName, "logs", "");
   
       DeliverCallback callback = (tagName, entity) -> {
           String body = new String(entity.getBody(), StandardCharsets.UTF_8);
           channel.basicAck(entity.getEnvelope().getDeliveryTag(), false);
           System.out.println("消费了：" + body);
       };
   
       channel.basicConsume(queueName, false, callback, tag -> {});
   }
   ```

###### direct exchange

> direct交换机通过routing key与binding_key进行exchange与queue的匹配，如果message的routing_key不存在匹配的message，则exchange将会discard这条消息。direct可以实现fanout的广播效果，只需多个queue使用相同的binding_key。
>
> exchange与queue可以绑定多个binding_key

使用示例：

simple logginh system：

- producer生产的日志消息将会带上日志级别的routing_key
- console consumer将消费所有日志级别
- file consumer只消费error日志级别

1. producer

   ```java
   public static void main(String[] args) throws Exception{
       ConnectionFactory factory = new ConnectionFactory();
       factory.setHost("172.24.192.134");
       factory.setPort(5672);
       factory.setUsername("jasper");
       factory.setPassword("pubgM666");
   
       Connection connection = factory.newConnection();
       Channel channel = connection.createChannel();
   
       channel.exchangeDeclare("direct_logs", "direct");
   
       channel.basicPublish("direct_logs", args[0], null, (args[0] + "消息").getBytes(StandardCharsets.UTF_8));
       connection.close();
   }
   ```

2. consumer

   ```java
   public static void main(String[] args) throws Exception {
       ConnectionFactory factory = new ConnectionFactory();
       factory.setHost("172.24.192.134");
       factory.setPort(5672);
       factory.setUsername("jasper");
       factory.setPassword("pubgM666");
   
       Connection connection = factory.newConnection();
       Channel channel = connection.createChannel();
   
       channel.basicQos(1);
       channel.exchangeDeclare("direct_logs", "direct");
       String queueName = channel.queueDeclare().getQueue();
   
       for (String bindingKey : args) {
           channel.queueBind(queueName, "direct_logs", bindingKey);
       }
   
       DeliverCallback callback = (tag, entity) -> {
           String body = new String(entity.getBody(), StandardCharsets.UTF_8);
           System.out.println("消费消息：" + body);
       };
   
       channel.basicConsume(queueName, true, callback, tag -> {});
   }
   ```

   

###### topic exchange

> topic exchange对比direct更强大，direct能够关注某一属性的不同值，topic能够关注多个不同属性的多个不同值，topic既可以当direct（binding_key只使用一个word）使用，也可以当fanout使用（binding_key使用#）。

**key结构：**

- key由word以及dot组成
- *代表一个word
- #代表零个或多个word

>  routing_key与binding_key匹配时，才会deliver到相应的queue，否则将会discard掉此message

**使用示例**

1. producer

   ```java
   // 与direct_producer一致，只需修改发送message时的routing_key
   ```

2. consumer

   ```java
   // 与direct_consumer一致，只需修改queueBinding时的binding_key
   ```


##### RPC

> rabbitmq可以可以做为RPC的中间层
>
> 对于被调用的函数，只需声明一个任务队列，从队列中获取调用请求
>
> 对于主动调用方，需要声明call_back_queue，通过BasicProperties的reply_to设置
>
> 如何区分call_back_message？？设置correlation_id与rquest一一对应

1. 被调用函数

   ```java
   public static void main(String[] args) throws Exception{
       ConnectionFactory factory = new ConnectionFactory();
       factory.setHost("172.24.192.134");
       factory.setPort(5672);
       factory.setUsername("jasper");
       factory.setPassword("pubgM666");
   
       Connection connection = factory.newConnection();
       Channel channel = connection.createChannel();
   
       channel.queueDeclare("fib", false, false, false, null);
       channel.basicQos(1);
   
       DeliverCallback callback = (tag, entity) -> {
           String replyToQueue = entity.getProperties().getReplyTo();
           int param = Integer.parseInt(new String(entity.getBody(), StandardCharsets.UTF_8));
           int result = fib(param);
           AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
               .correlationId(entity.getProperties().getCorrelationId())
               .build();
           channel.basicPublish("", replyToQueue, properties,
                                String.valueOf(result).getBytes(StandardCharsets.UTF_8));
       };
   
       channel.basicConsume("fib", true, callback, tag -> {});
   }
   
   private static int fib(int param) {
       if(param <= 1) return param;
       int[] d = new int[param + 1];
       d[1] = 1;
       for(int i = 2;i < d.length;i++) {
           d[i] = d[i - 1] + d[i - 2];
       }
       return d[param];
   }
   ```

2. 主动调用方

   ```java
   public static void main(String[] args) throws Exception{
       ConnectionFactory factory = new ConnectionFactory();
       factory.setHost("172.24.192.134");
       factory.setPort(5672);
       factory.setUsername("jasper");
       factory.setPassword("pubgM666");
   
       Connection connection = factory.newConnection();
       Channel channel = connection.createChannel();
       channel.basicQos(1);
   
       String callBackQueue = channel.queueDeclare().getQueue();
       DeliverCallback callBack = (tag, entity) -> {
           System.out.println("fib函数调用结果：" +
                              new String(entity.getBody(), StandardCharsets.UTF_8));
       };
       channel.basicConsume(callBackQueue, true, callBack, tag -> {});
   
       AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
           .correlationId("")
           .replyTo(callBackQueue)
           .build();
       channel.basicPublish("", "fib", properties, args[0].getBytes(StandardCharsets.UTF_8));
   }
   ```

##### publisher confirm

> publisher confirm是channel_level的，只需开启一次
>
> ```java
> channel.confirmSelect();
> ```

**三种confirm strategy：**

1. 同步发送单条message（性能低）
2. 同步发送多条message（性能较高）
3. 异步（the best）最佳性能和资源使用，在出现错误时具有良好的控制力

**异步confirm步骤**

1. 提供一种将消息与序列号绑定的方法

   ```java
   // 通常
   channel.getNextPublishSeqNo();
   ```

2. 在channel上注册confirm_listener，包括ack以及nacked两个方法

   > 这两个方法参数一致：int seqNo，boolean multiple
   >
   > multiple 为true，则代表单个ack/nacked
   >
   > multiple为false，则代表小于等于seq的多个ack/nacked

**注意：**

> 通常不在nacked异步方法中republish message
>
> 推荐做法：维护一个nacked队列，由定时方法republish

