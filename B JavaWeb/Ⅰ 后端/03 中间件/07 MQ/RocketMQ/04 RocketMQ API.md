# RocketMQ API

## 1 消息消费模式

消息消费模式由消费者来决定，可以由消费者设置 MessageModel 来决定消息模式。

消息模式默认为集群消费模式

```java
// 广播模式
consumer.setMessageModel(MessageModel.BROADCASTING);
// 集群模式
consumer.setMessageModel(MessageModel.CLUSTERING);
```

### 1.1 集群消息

![img](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/160707_kSpS_1469576.png)

集群消息是指**集群化部署消费者**

当使用集群消费模式时，MQ 认为任意一条消息只需要被集群内的任意一个消费者处理即可。

**特点**

- 每条消息只需要被处理一次，broker只会把消息发送给消费集群中的一个消费者
- 在消息重投时，不能保证路由到同一台机器上
- 消费状态由broker维护

### 1.2 广播消息

![img](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/160902_4AOI_1469576.png)

当使用广播消费模式时，MQ 会将每条消息推送给集群内所有注册过的客户端，保证消息至少被每台机器消费一次。

**特点**

- 消费进度由consumer维护

- 保证每个消费者消费一次消息

- 消费失败的消息不会重投

  

## 2 发送方式

### 2.1 同步消息

消息发送中进入同步等待状态，可以保证消息投递一定到达

```java
Message message = new Message("myTopic01", "first message".getBytes());
// 同步消息发送，需要等待回复确认
SendResult sendResult = producer.send(message);
```

### 2.2 异步消息

想要快速发送消息，又不想丢失的时候可以使用异步消息

```java
producer.send(message, new SendCallback() {
    @Override
    public void onSuccess(SendResult sendResult) {
        System.out.println("消息发送成功");
        System.out.println("sendResult：" + sendResult);
    }

    @Override
    public void onException(Throwable throwable) {
        // 如果发送异常，可以尝试重投
        // 或者调整业务逻辑
        System.out.println("发送异常");
    }
});
```

### 2.3 单向消息

只发送消息，不等待服务器响应，只发送请求不等待应答。此方式发送消息的过程耗时非常短，一般在微秒级别。

```java
producer.sendOneway(message);
```

## 3 批量消息发送

可以多条消息打包一起发送，减少网络传输次数提高效率。

`producer.send(Collection c) ` 方法可以接受一个集合 实现批量发送

```java
public SendResult send(
    Collection<Message> msgs) throws MQClientException, RemotingException, MQBrokerException, InterruptedException {
    return this.defaultMQProducerImpl.send(batch(msgs));
}


Message message1 = new Message("myTopic01", "first message".getBytes());
Message message2 = new Message("myTopic01", "second message".getBytes());
Message message3 = new Message("myTopic01", "third message".getBytes());
List<Message> list = new ArrayList<>();
list.add(message1);
list.add(message2);
list.add(message3);

SendResult sendResult = producer.send(list);
```

- 批量消息要求必要具有同一topic、相同消息配置
- 不支持延时消息
- 建议一个批量消息最好不要超过1MB大小
- 如果不确定是否超过限制，可以手动计算大小分批发送

## 4 TAG

可以使用tag来过滤消费

 在Producer中使用Tag：

```java
Message msg = new Message("TopicTest","TagA" ,("Hello RocketMQ " ).getBytes(RemotingHelper.DEFAULT_CHARSET));
```

  在Consumer中订阅Tag：

```java
// * 代表订阅Topic下的所有消息
consumer.subscribe("TopicTest", "TagA||TagB");
```

## 5 SQL表达式过滤

消费者将收到包含 TAGA 或 TAGB B的消息. 但限制是一条消息只能有一个标签，而这对于复杂的情况可能无效。 在这种情况下，您可以使用SQL表达式筛选出消息.

### 5.1 配置

在 `broker.conf ` 中添加配置

```
enablePropertyFilter=true
```

启动broker 加载指定配置文件

```
../bin/mqbroker -n 192.168.150.113:9876 -c broker.conf 
```

随后在集群配置中可以看到

![image-20200219174859476](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200219174859476.png)

### 5.2 实例

```java
// 发送端
for (int i = 0; i < 100; i++) {
Message message = new Message("myTopic01", "first message".getBytes());
message.putUserProperty("age", String.valueOf(i));
list.add(message);
}
// 接收端
MessageSelector messageSelector = MessageSelector.bySql("age >= 18 and age <= 28");
consumer.subscribe("myTopic01", messageSelector);
```

### 5.3 语法

RocketMQ只定义了一些基本的语法来支持这个功能。 你也可以很容易地扩展它.

1. 数字比较, 像 `>`, `>=`, `<`, `<=`, `BETWEEN`, `=`;
2. 字符比较, 像 `=`, `<>`, `IN`;
3. `IS NULL` 或者 `IS NOT NULL`;
4. 逻辑运算`AND`, `OR`, `NOT`;

常量类型是:

1. 数字, 像123, 3.1415;
2. 字符串, 像‘abc’,必须使用单引号;
3. `NULL`, 特殊常数;
4. 布尔常量, `TRUE` 或`FALSE`;

## 6 延迟消息

RocketMQ使用**messageDelayLevel**可以设置延迟投递

**默认配置为**

```
messageDelayLevel	1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
```

### 6.1 配置

在`broker.conf `中添加配置

```
messageDelayLevel=1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
```

这个配置项配置了从1级开始，各级延时的时间，可以修改这个指定级别的延时时间；

时间单位支持：s、m、h、d，分别表示秒、分、时、天；

### 6.2 使用

发送消息时设置

```
message.setDelayTimeLevel(1); 
```

## 7 顺序消费

队列先天支持FIFO模型，单一生产和消费者下只要保证使用`MessageListenerOrderly`监听器即可

顺序消费表示消息消费的顺序同生产者为每个消息队列发送的顺序一致，所以如果正在处理全局顺序是强制性的场景，需要确保使用的主题只有一个消息队列。

并行消费不再保证消息顺序，消费的最大并行数量受每个消费者客户端指定的线程池限制。

那么只要顺序的发送，再保证一个线程只去消费一个队列上的消息，那么他就是有序的。



跟普通消息相比，顺序消息的使用需要在producer的send()方法中添加MessageQueueSelector接口的实现类，并重写select选择使用的队列，因为顺序消息局部顺序，需要将所有消息指定发送到同一队列中。



**保证有序参与因素**

- FIFO
- 队列内保证有序
- 消费线程

### 7.1 消费端

可以设置线程池的个数

```java
consumer.setConsumeThreadMax(1);
consumer.setConsumeThreadMin(1);
```

registerMessageListener可以选择两个MessageListener

- MessageListenerConcurrently：并发，开启多个线程接收消息
- MessageListenerOrderly：线程内有序，每个Queue只开启一个线程

```java
/**
 * 并发的执行
 * Register a callback to execute on message arrival for concurrent consuming.
 *
 * @param messageListener message handling callback.
 */
@Override
public void registerMessageListener(MessageListenerConcurrently messageListener) {
    this.messageListener = messageListener;
    this.defaultMQPushConsumerImpl.registerMessageListener(messageListener);
}

/**
 * 顺序的执行
 * Register a callback to execute on message arrival for orderly consuming.
 *
 * @param messageListener message handling callback.
 */
@Override
public void registerMessageListener(MessageListenerOrderly messageListener) {
    this.messageListener = messageListener;
    this.defaultMQPushConsumerImpl.registerMessageListener(messageListener);
}
```

### 假如我有一个场景要保证消息的顺序，你们应该如何保证？

- 同一topic

- 同一个QUEUE

- 发消息的时候一个线程去发送消息

- 消费的时候 一个线程 消费一个queue里的消息或者使用MessageListenerOrderly

- 多个queue 只能保证单个queue里的顺序

## 8 重试机制

### 8.1 producer

**默认超时时间**

```java
/**
 * Timeout for sending messages.
 */
private int sendMsgTimeout = 3000;
// 异步发送时 重试次数，默认 2
producer.setRetryTimesWhenSendAsyncFailed(1);
// 同步发送时 重试次数，默认 2
producer.setRetryTimesWhenSendFailed(1);	
// 是否向其他broker发送请求 默认false
producer.setRetryAnotherBrokerWhenNotStoreOK(true);
```

### 8.2 Consumer

消费超时，单位分钟

```
consumer.setConsumeTimeout()
```

发送ack，消费失败

```
RECONSUME_LATER
```

### 8.3 broker投递

只有在消息模式为MessageModel.CLUSTERING集群模式时，Broker才会自动进行重试，广播消息不重试

重投使用`messageDelayLevel`

默认值

```
messageDelayLevel	1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
```

## 9 事务消息

分布式系统中的事务可以使用TCC（Try、Confirm、Cancel）、2pc来解决分布式系统中的消息原子性

RocketMQ 4.3+提供分布事务功能，通过 RocketMQ 事务消息能达到分布式事务的最终一致

### 9.1 RocketMQ实现方式

**Half Message：**预处理消息，当broker收到此类消息后，会存储到 RMQ_SYS_TRANS_HALF_TOPIC 的消息消费队列中

**检查事务状态：**Broker会开启一个定时任务，消费RMQ_SYS_TRANS_HALF_TOPIC队列中的消息，每次执行任务会向消息发送者确认事务执行状态（提交、回滚、未知），如果是未知，等待下一次回调。

**超时：**如果超过回查次数，默认回滚消息

### 9.2 TransactionListener的两个方法

#### 9.2.1 executeLocalTransaction

半消息发送成功触发此方法来执行本地事务

#### 9.2.2 checkLocalTransaction

broker将发送检查消息来检查事务状态，并将调用此方法来获取本地事务状态

#### 9.2.3 本地事务执行状态

- **LocalTransactionState.COMMIT_MESSAGE**

  执行事务成功，确认提交

- **LocalTransactionState.ROLLBACK_MESSAGE**

  回滚消息，broker端会删除半消息

- **LocalTransactionState.UNKNOW**

  暂时为未知状态，等待broker回查

```java
package com.yeyangshu.rocketmq;

import org.apache.rocketmq.client.producer.*;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.common.message.MessageExt;

/**
 * 事务消息发送者
 */
public class TransactionProducer {

    public static void main(String[] args) throws Exception {
        // 事务消息
        TransactionMQProducer producer = new TransactionMQProducer("rocketMQGroup");
        // 设置nameserver地址
        producer.setNamesrvAddr("127.0.0.1:9876");

        producer.setTransactionListener(new TransactionListener() {
            // 当发送事务性prepare（half）消息成功时，将调用此方法以执行本地事务。
            public LocalTransactionState executeLocalTransaction(Message msg, Object arg) {
                // 执行本地事务
                // a()
                // b() -> 发消息
                // c()

                try {
                    // 业务代码
                } catch (Exception e) {
                    return LocalTransactionState.ROLLBACK_MESSAGE;
                }
                System.out.println("execute local transaction");
                System.out.println("msg：" + new String(msg.getBody()));
                System.out.println("msg：" + msg.getTransactionId());
                return LocalTransactionState.COMMIT_MESSAGE;
            }

            // broker端回调检查，检察事务
            // 没有响应准备（半）消息时。broker将发送check message以检查交易状态，并将调用此方法以获取本地交易状态。
            public LocalTransactionState checkLocalTransaction(MessageExt msg) {
                System.out.println("check local transaction");
                System.out.println("msg：" + new String(msg.getBody()));
                System.out.println("msg：" + msg.getTransactionId());

                // LocalTransactionState.UNKNOW，等会
                // LocalTransactionState.ROLLBACK_MESSAGE，回滚消息
                // 事务成功消息
                return LocalTransactionState.COMMIT_MESSAGE;
            }
        });

        // 启动producer
        producer.start();

        Message message = new Message("myTopic01", "transaction message test".getBytes());
        TransactionSendResult sendResult = producer.sendMessageInTransaction(message, null);
        System.out.println(sendResult);

    }
}
```

producer结果

```
// LocalTransactionState.UNKNOW
execute local transaction
msg：transaction message test
msg：1E2DA423000018B4AAC2657AA12F0000
SendResult [sendStatus=SEND_OK, msgId=1E2DA423000018B4AAC2657AA12F0000, offsetMsgId=null, messageQueue=MessageQueue [topic=myTopic01, brokerName=M7099E11, queueId=0], queueOffset=2]
// 回调LocalTransactionState.COMMIT_MESSAGE
check local transaction
msg：transaction message test
msg：1E2DA423000018B4AAC2657AA12F0000
```



