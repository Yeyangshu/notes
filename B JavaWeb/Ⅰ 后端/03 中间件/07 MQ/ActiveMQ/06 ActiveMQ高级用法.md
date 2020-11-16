# 高级使用

## 1 JMS消息结构（Message）

Message主要由三部分组成，分别是Header，Properties，Body， 详细如下：

| 属性       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| Header     | 消息头，所有类型的这部分格式都是一样的                       |
| Properties | 属性，按类型可以分为应用设置的属性，标准属性和消息中间件定义的属性 |
| Body       | 消息正文，指我们具体需要消息传输的内容。                     |

### 1.1 Header

JMS消息头使用的所有方法：

```java
public interface Message {
    public Destination getJMSDestination() throws JMSException;
    public void setJMSDestination(Destination destination) throws JMSException;
    public int getJMSDeliveryMode() throws JMSException
    public void setJMSDeliveryMode(int deliveryMode) throws JMSException;
    public String getJMSMessageID() throws JMSException;
    public void setJMSMessageID(String id) throws JMSException;
    public long getJMSTimestamp() throws JMSException'
    public void setJMSTimestamp(long timestamp) throws JMSException;
    public long getJMSExpiration() throws JMSException;
    public void setJMSExpiration(long expiration) throws JMSException;
    public boolean getJMSRedelivered() throws JMSException;
    public void setJMSRedelivered(boolean redelivered) throws JMSException;
    public int getJMSPriority() throws JMSException;
    public void setJMSPriority(int priority) throws JMSException;
    public Destination getJMSReplyTo() throws JMSException;
    public void setJMSReplyTo(Destination replyTo) throws JMSException;
    public String getJMScorrelationID() throws JMSException;
    public void setJMSCorrelationID(String correlationID) throws JMSException;
    public byte[] getJMSCorrelationIDAsBytes() throws JMSException;
    public void setJMSCorrelationIDAsBytes(byte[] correlationID) throws JMSException;
    public String getJMSType() throws JMSException;
    public void setJMSType(String type) throws JMSException;
}
```

**消息头分为自动设置和手动设置的内容**

#### 1.1.1 自动头信息

有一部分可以在创建Session和MessageProducer时设置

| 属性名称        | 说明                                                         | 设置者   |
| --------------- | ------------------------------------------------------------ | -------- |
| JMSDeliveryMode | 消息的发送模式，分为**NON_PERSISTENT**和**PERSISTENT**，即非持久性模式的和持久性模式。默认设置为**PERSISTENT（持久性）。**一条**持久性消息**应该被传送一次（就一次），这就意味着如果JMS提供者出现故障，该消息并不会丢失； 它会在服务器恢复正常之后再次传送。一条**非持久性消息**最多只会传送一次，这意味着如果JMS提供者出现故障，该消息可能会永久丢失。在持久性和非持久性这两种传送模式中，消息服务器都不会将一条消息向同一消息者发送一次以上（成功算一次）。 | send     |
| JMSMessageID    | 消息ID，需要以ID:开头，用于唯一地标识了一条消息              | send     |
| JMSTimestamp    | 消息发送时的时间。这条消息头用于确定发送消息和它被消费者实际接收的时间间隔。时间戳是一个以毫秒来计算的Long类型时间值（自1970年1月1日算起）。 | send     |
| JMSExpiration   | 消息的过期时间，以毫秒为单位，用来防止把过期的消息传送给消费者。任何直接通过编程方式来调用setJMSExpiration()方法都会被忽略。 | send     |
| JMSRedelivered  | 消息是否重复发送过，如果该消息之前发送过，那么这个属性的值需要被设置为true, 客户端可以根据这个属性的值来确认这个消息是否重复发送过，以避免重复处理。 | Provider |
| JMSPriority     | 消息的优先级,0-4为普通的优化级，而5-9为高优先级，通常情况下，高优化级的消息需要优先发送。任何直接通过编程方式调用setJMSPriority()方法都将被忽略。 | send     |
| JMSDestination  | 消息发送的目的地，主要是指Topic或Queue                       | send     |



**JMSDeliveryMode**

```java
MessageProducer producer = session.createProducer(topic);
producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
```

**JMSExpiration**

```java
//将过期时间设置为1小时（1000毫秒 ＊60 ＊60）
producer.setTimeToLive(1000 * 60 * 60);
```

**JMSPriority**

```java
producer.setPriority(9);
```

#### 1.1.2 手动头信息

| 属性名称         | 说明                                                         | 设置者 |
| ---------------- | ------------------------------------------------------------ | ------ |
| JMSCorrelationID | 关联的消息ID，这个通常用在需要回传消息的时候                 | client |
| JMSReplyTo       | 消息回复的目的地，其值为一个Topic或Queue, 这个由发送者设置，但是接收者可以决定是否响应 | client |
| JMSType          | 由消息发送者设置的消息类型，代表消息的结构，有的消息中间件可能会用到这个，但这个并不是是批消息的种类，比如TextMessage之类的 | client |

从上表中我们可以看到，系统提供的标准头信息一共有10个属性，其中有6个是由send方法在调用时设置的，有三个是由客户端（client）设置的，还有一个是由消息中间件（Provider）设置的。

需要注意的是，这里

**JMSReplyTo**

```java
textMessage.setJMSReplyTo(new ActiveMQQueue("replyTo"));
```

**JMSCorrelationID**

粒度细于栓选





## 2 QueueBrowser

可以查看队列中的消息而不消费，没有订阅的功能

```java
public class BrowserQueue {

    public static void main(String[] args) throws Exception {

        // 1.获取连接工厂
        ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory(
                "admin",
                "admin",
                "tcp://localhost:61616"
        );
        // 1.1 添加信任的 持久化类型

        ArrayList<String> list = new ArrayList<String>();
        list.add(Girl.class.getPackage().getName());

        connectionFactory.setTrustedPackages(list);

        // 2.获取一个向ActiveMQ的连接
        Connection connection = connectionFactory.createConnection();
        System.out.println("ReceiverQueue-1 2  Started");
        connection.start();
        // 3.获取session
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4.找目的地，获取destination，消费端，也会从这个目的地取消息
        MessageConsumer consumer = session.createConsumer(new ActiveMQQueue("xxoo"));

        QueueBrowser browser = session.createBrowser(new ActiveMQQueue("xxoo"));
        // 返回队列剩余消息
        Enumeration enumeration = browser.getEnumeration();
        // 将剩余消息遍历出来，但是不消费，消息还在queue中
        while (enumeration.hasMoreElements()) {
            TextMessage textMessage = (TextMessage) enumeration.nextElement();
            System.out.println("testMessage " + textMessage);
        }
    }
    
}
```

打印

```json
ReceiverQueue-1 2  Started
testMessage ActiveMQTextMessage {commandId = 5, responseRequired = true, messageId = ID:M7099E11-64600-1604908716275-1:1:1:1:1, originalDestination = null, originalTransactionId = null, producerId = ID:M7099E11-64600-1604908716275-1:1:1:1, destination = queue://xxoo, transactionId = null, expiration = 0, timestamp = 1604908716662, arrival = 0, brokerInTime = 1604908716663, brokerOutTime = 1604908761807, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@2ef9b8bc, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = null, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Message from ServerA xxx}
```



# 集群
