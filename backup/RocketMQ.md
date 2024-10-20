# RocketMQ api实战

### Consumer

- 推模式：broker推送消息给consumer
- 拉模式：consumer主动从broker拉取消息

> 推模式消费者

```java
package com.mq.consumer;

import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;


public class SyncConsumer {
    public static void main(String[] args) throws Exception{
        // 推模式(boroker(主动推送) -> consumer)
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("SyncConsumer");

        // set name server addr
        consumer.setNamesrvAddr("172.24.192.134:9876");

        // subscribe
        consumer.subscribe("wwb", "*");

        // consume message
        consumer.setMessageListener((MessageListenerConcurrently) (list, consumeConcurrentlyContext) -> {
            for(int i = 0;i < list.size();i++) {
                System.out.println("消费一条消息");
            }

            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        });

        consumer.start();
        System.out.println("消费者启动成功...");
    }
}
```

> 拉模式消费者（@deprecated）

```java
package com.mq.consumer;

import org.apache.rocketmq.client.consumer.DefaultMQPullConsumer;
import org.apache.rocketmq.client.consumer.PullResult;
import org.apache.rocketmq.client.consumer.PullStatus;
import org.apache.rocketmq.client.consumer.store.ReadOffsetType;
import org.apache.rocketmq.client.exception.MQBrokerException;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.message.MessageQueue;
import org.apache.rocketmq.remoting.exception.RemotingException;

import java.util.HashSet;
import java.util.List;
import java.util.Set;

public class PullConsumer {
    @Deprecated
    public static void main(String[] args) throws Exception{
        DefaultMQPullConsumer consumer = new DefaultMQPullConsumer("PullConsumer");

        // set name server addr
        consumer.setNamesrvAddr("172.24.192.134:9876");

        // set register topic
        consumer.setRegisterTopics(new HashSet<>(List.of("wwb", "async_wwb", "TopicTest")));

        consumer.start();

        while(true) {
            consumer.getRegisterTopics().forEach(t -> {
                try {
                    Set<MessageQueue> queues = consumer.fetchSubscribeMessageQueues(t);
                    queues.forEach(q -> {
                        try {
                            long offset = consumer.getOffsetStore().readOffset(q, ReadOffsetType.READ_FROM_MEMORY);
                            if(offset < 0) offset = consumer.getOffsetStore().readOffset(q, ReadOffsetType.READ_FROM_STORE);
                            if(offset < 0) offset = consumer.maxOffset(q);
                            if(offset < 0) offset = 0;

                            PullResult result = consumer.pull(q, "*", offset, 32);
                            if(result.getPullStatus() == PullStatus.FOUND) {
                                System.out.println("消息消费成功");
                                consumer.updateConsumeOffset(q, result.getNextBeginOffset());
                            }
                        } catch (MQClientException | MQBrokerException | RemotingException | InterruptedException e) {
                            throw new RuntimeException(e);
                        }
                    });
                } catch (MQClientException e) {
                    throw new RuntimeException(e);
                }
            });
        }
    }
}

```

> 拉模式消费者

```java
package com.mq.consumer;

import org.apache.rocketmq.client.consumer.DefaultLitePullConsumer;
import org.apache.rocketmq.common.message.MessageExt;
import org.apache.rocketmq.common.message.MessageQueue;

import java.nio.charset.StandardCharsets;
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;

public class LitePullConsumer {
    public static void main(String[] args) throws Exception{
        pullAssignQueue();
    }

    /**
     * 随机拉取某个队列的消息
     * @throws Exception
     */
    static void pullRandomQueue() throws Exception{
        DefaultLitePullConsumer consumer = new DefaultLitePullConsumer("LitePullConsumer");
        consumer.setNamesrvAddr("172.24.192.134:9876");
        consumer.subscribe("wwb", "*");

        consumer.start();

        while(true) {
            List<MessageExt> list = consumer.poll();
            list.forEach(m -> {
                byte[] body = m.getBody();
                System.out.println("消息消费成功: " + new String(body, StandardCharsets.UTF_8));
            });
        }
    }

    /**
     * 指定拉取某个队列的消息
     * @throws Exception
     */
    static void pullAssignQueue() throws Exception{
        DefaultLitePullConsumer consumer = new DefaultLitePullConsumer("LitePullConsumer2");
        consumer.setNamesrvAddr("172.24.192.134:9876");
        consumer.start();

        Collection<MessageQueue> queues = consumer.fetchMessageQueues("wwb");
        ArrayList<MessageQueue> li = new ArrayList<>(queues);
        consumer.assign(li);
        consumer.seek(li.get(1), 0);

        while(true) {
            List<MessageExt> list = consumer.poll();
            list.forEach(m -> {
                System.out.println(m);
                System.out.println("消息消费成功: " + new String(m.getBody(), StandardCharsets.UTF_8));
            });
        }
    }
}
```



### Producer

**消息发送的三种方式**

- 同步发送：等待消息返回后再继续进行下面的操作，适用于并发较低以及可靠性较高的场景
- 异步发送：不等待消息返回直接进行后续代码流程，适用于并发高的场景
- 单向发送：只负责发送，不管消息是否发送成功，适用于日志等特殊场景



> 同步生产者：

```java
package com.mq.producer;

import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;

import java.nio.charset.StandardCharsets;
import java.util.Random;

public class SyncProducer {
    public static void main(String[] args) throws Exception{
        DefaultMQProducer producer = new DefaultMQProducer("SyncProducer");

        // set name server addr
        producer.setNamesrvAddr("172.24.192.134:9876");

        producer.start();

        for(int i = 0;i < 10;i++) {
            Message mess = new Message(
                    "wwb",
                    String.valueOf(i),
                    String.valueOf(new Random().nextInt(10)).getBytes(StandardCharsets.UTF_8));
            SendResult result = producer.send(mess);
            System.out.println(result);
        }

        producer.shutdown();
    }
}
```


> 异步生产者

```java
package com.mq.producer;

import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendCallback;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;

import java.nio.charset.StandardCharsets;
import java.util.Arrays;
import java.util.Random;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

public class AsyncProducer {
    public static void main(String[] args) throws Exception{
        DefaultMQProducer producer = new DefaultMQProducer("AsyncProducer");

        // set name server addr
        producer.setNamesrvAddr("172.24.192.134:9876");

        producer.start();
        CountDownLatch cd = new CountDownLatch(10);
        for(int i = 0;i < 10;i++) {
            Message mess = new Message(
                    "async_wwb",
                    String.valueOf(i),
                    String.valueOf(new Random().nextInt(10)).getBytes(StandardCharsets.UTF_8));
            producer.send(mess, new SendCallback() {
                @Override
                public void onSuccess(SendResult sendResult) {
                    System.out.println("消息发送成功: " + sendResult);
                    cd.countDown();
                }

                @Override
                public void onException(Throwable throwable) {
                    System.out.println("消息发送失败" + Arrays.toString(throwable.getStackTrace()));
                    cd.countDown();
                }
            });
        }

        boolean await = cd.await(5, TimeUnit.SECONDS);
        if(!await) {
            System.out.println("部分消息丢失");
        }
        producer.shutdown();
    }
}
```

> 单向生产者

```java
package com.mq.producer;

import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.common.message.Message;

import java.nio.charset.StandardCharsets;
import java.util.Random;

public class OneWayProducer {
    public static void main(String[] args) throws Exception{
        DefaultMQProducer producer = new DefaultMQProducer("OneWayProducer");

        // set name server addr
        producer.setNamesrvAddr("172.24.192.134:9876");

        producer.start();

        for(int i = 0;i < 10;i++) {
            Message mess = new Message(
                    "wwb",
                    String.valueOf(i),
                    String.valueOf(new Random().nextInt(10)).getBytes(StandardCharsets.UTF_8));
            producer.sendOneway(mess);
        }

        producer.shutdown();
    }
}
```



### 消息类型

- **Normal（普通消息）**
- **FIFO（顺序消息）**
- **Delay（延迟消息）**

延迟消息具有18个级别：1s到2个小时

几个有关的方法：

1. setDelayTimeLevel() ：根据延迟级别延迟投递

2. setDelayTimeMs() ：自定义延迟毫秒数

3. setDelayTimeSec() ：自定义延迟秒数

4. setDeliverTimeMs() ：在某个时间戳投递

|延迟级别|延迟时间|
|--- |--- |
|1|1s|
|2|5s|
|3|10s|
|4|30s|
|5|1m|
|6|2m|
|7|3m|
|8|4m|
|9|5m|
|10|6m|
|11|7m|
|12|8m|
|13|9m|
|14|10m|
|15|20m|
|16|30m|
|17|1h|
|18|2h|

- **Transaction（事务消息）**

RocketMQ 的事务消息用于保证消息和本地事务操作的最终一致性。通常用于需要将消息发送与数据库或其他外部系统的事务操作结合在一起的场景。事务消息可以保证消息只会在事务成功时被消费，如果事务失败，消息也不会被消费。这非常类似于分布式事务的一种实现。

**常见应用场景**

- 电商系统中下单与扣库存操作
- 支付系统中扣款与生成支付通知操作
- 金融系统中资金转账和账务更新操作

**事务消息的执行流程**

1. 发送半事务消息，生产者先发送一条**半事务消息**到 RocketMQ Broker，消息暂时不会被消费者消费。此时消息的状态是“未决状态”
2. 执行本地事务，生产者执行本地事务（如数据库更新操作）。如果本地事务成功，提交消息；如果失败，回滚消息
3. Broker回查，当 Broker 没有收到生产者的事务提交或回滚的确认时，它会向生产者发起回查（Check），询问事务的最终状态。生产者需要根据本地事务的状态返回**提交**或**回滚**的结果

**核心方法**

- sendMessageInTransaction：生产者发送事物消息
- executeLocalTransaction：生产者执行本地事务
- checkLocalTransaction：Broker回查事务状态

**事务消息原理**

RocketMQ事物消息通过”两阶段提交“来实现最终一致性：

1. 第一阶段：生产者发送半事务消息到 Broker，消息处于**暂时不可消费**状态。
2. 第二阶段：生产者执行本地事务，事务成功则提交消息使其可消费，事务失败则回滚消息。

当 RocketMQ Broker 没有收到生产者的确认消息时，会定时询问生产者“这条消息对应的事务到底成功还是失败”，生产者根据本地事务的结果做出相应的响应。

**事务消息的状态**

- COMMIT_MESSAGE：本地事务执行成功，提交消息，允许消费
- ROLLBACK_MESSAGE：本地事务执行失败，回滚消息，不允许消费
- UNKNOWN：暂时无法确定事务状态，Broker回定时回查



从RocketMQ5.x版本中，默认开启了消息强制校验，若定义为normal，尝试发送FIFO会被broker拒绝并返回类型不匹配异常，5.x以下的版本需要程序员自己管理消息类型的正确性



> 顺序消息producer

```java
package com.mq.order;

import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;

import java.nio.charset.StandardCharsets;

public class OrderProducer {
    public static void main(String[] args) throws Exception{
        DefaultMQProducer producer = new DefaultMQProducer("god");

        producer.setNamesrvAddr("172.24.192.134:9876");
        producer.start();

        for (int i = 0;i < 10;i++) {
            for(int j = 0;j < 10;j++) {
                Message message = new Message("kevin", "a", "key", (i + "_" + j).getBytes(StandardCharsets.UTF_8));
                SendResult result = producer.send(message, (list, message1, o) -> {
                    Integer index = (Integer) o;
                    return list.get(index % list.size());
                }, i);
                System.out.println(result);
            }
        }

        producer.shutdown();
    }
}
```

> 顺序消息consumer

```java
package com.mq.order;

import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeOrderlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerOrderly;


public class OrderConsumer {
    public static void main(String[] args) throws Exception{
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("kevin");
        consumer.setNamesrvAddr("172.24.192.134:9876");
        consumer.subscribe("kevin", "*");

        consumer.setMessageListener((MessageListenerOrderly) (list, consumeOrderlyContext) -> {
            list.forEach(m -> {
                System.out.println(new String(m.getBody()));
                System.out.println(m);
            });
            return ConsumeOrderlyStatus.SUCCESS;
        });

        consumer.start();
    }
}
```

> 延迟消息

```java
package com.mq.delay;

import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;

import java.nio.charset.StandardCharsets;

public class DelayProducer {
    public static void main(String[] args) throws Exception{
        DefaultMQProducer producer = new DefaultMQProducer("delay_message");
        producer.setNamesrvAddr("172.24.192.134:9876");
        producer.start();

        for(int i = 0;i < 10;i++) {
            Message message = new Message("delay_message", (i + "delay").getBytes(StandardCharsets.UTF_8));
            message.setDelayTimeLevel(4);
            SendResult result = producer.send(message);
            System.out.println(result);
        }

        producer.shutdown();
    }
}
```

> 过滤消息producer

```java
package com.mq.filter;

import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.common.message.Message;

import java.nio.charset.StandardCharsets;

public class FilterProducer {
    public static void main(String[] args) throws Exception{
        DefaultMQProducer producer = new DefaultMQProducer("filtered_message");
        producer.setNamesrvAddr("172.24.192.134:9876");
        producer.start();

        String[] arr = {"a", "b", "c"};

        for(int i = 0;i < 10;i++) {
            Message message = new Message("filtered_topic", arr[i % 3], "message".getBytes(StandardCharsets.UTF_8));
            System.out.println(producer.send(message));
        }

        producer.shutdown();
    }
}
```

> 过滤消息consumer

```java
package com.mq.filter;

import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.common.message.MessageExt;

import java.util.List;

public class FilterConsumer {
    public static void main(String[] args) throws Exception {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("filter_consumer");
        consumer.setNamesrvAddr("172.24.192.134:9876");
        consumer.subscribe("filtered_topic", "a");

        consumer.setMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                System.out.println("消息消费成功...");

                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });

        consumer.start();
    }
}
```

> 事务消息

```java
package com.mq.transaction;

import org.apache.rocketmq.client.producer.LocalTransactionState;
import org.apache.rocketmq.client.producer.TransactionListener;
import org.apache.rocketmq.client.producer.TransactionMQProducer;
import org.apache.rocketmq.client.producer.TransactionSendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.common.message.MessageExt;

import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeUnit;

public class TransactionProducer {
    public static void main(String[] args) throws Exception{
        TransactionMQProducer producer = new TransactionMQProducer("transaction_producer");
        producer.setNamesrvAddr("172.24.192.134:9876");
        producer.start();

        producer.setTransactionListener(new TransactionListener() {
            @Override
            public LocalTransactionState executeLocalTransaction(Message message, Object o) {
                try {
                    // 耗时操作 模拟local transaction
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }

                // 假设local transaction success
                return LocalTransactionState.COMMIT_MESSAGE;
            }

            @Override
            public LocalTransactionState checkLocalTransaction(MessageExt messageExt) {
                return LocalTransactionState.UNKNOW;
            }
        });

        Message m = new Message("tran_topic", "message".getBytes(StandardCharsets.UTF_8));
        TransactionSendResult re = producer.sendMessageInTransaction(m, null);
        System.out.println(re);
    }
}
```



### 常见问题

**如何保证消息不丢失**

1. 在producer端保证message的重新发送
2. 在broker中使用同步刷盘
3. 在消费者端保证消息的成功消费，有消费失败重试机制

**存储结构**

1. commit_log：所有message顺序存储的文件
2. consumer_queue：与tags有关的索引
3. index_file：与keys有关的索引







