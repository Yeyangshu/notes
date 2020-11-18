# Redis 发布订阅（Pub/Sub）

中文官网：http://redis.cn/topics/pubsub.html

发布、订阅和取消订阅实现了发布/订阅消息范式。

发送者（发布者）不是计划发送消息给特定的接收者（订阅者）。而是发布的消息分到不同的频道，不需要知道什么样的订阅者订阅。

订阅者对一个或多个频道感兴趣，只需接收感兴趣的消息，不需要知道什么样的发布者发布的。这种发布者和订阅者的解耦合可以带来更大的扩展性和更加动态的网络拓扑。

## 1 Pub/Sub的基本知识

redis是一个快速、稳定的发布/订阅的信息系统。



## 2 使用

### 2.1 help

```
127.0.0.1:6379> help @pubsub

  PSUBSCRIBE pattern [pattern ...]
  summary: Listen for messages published to channels matching the given patterns
  since: 2.0.0

  PUBLISH channel message
  summary: Post a message to a channel
  since: 2.0.0

  PUBSUB subcommand [argument [argument ...]]
  summary: Inspect the state of the Pub/Sub subsystem
  since: 2.8.0

  PUNSUBSCRIBE [pattern [pattern ...]]
  summary: Stop listening for messages posted to channels matching the given patterns
  since: 2.0.0

  SUBSCRIBE channel [channel ...]
  summary: Listen for messages published to the given channels
  since: 2.0.0

  UNSUBSCRIBE [channel [channel ...]]
  summary: Stop listening for messages posted to the given channels
  since: 2.0.0
```

### 2.2 发布订阅

```
// session2订阅渠道
127.0.0.1:6379> SUBSCRIBE ooxx
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "ooxx"
3) (integer) 1

// session1渠道发布
127.0.0.1:6379> PUBLISH ooxx 11
(integer) 1

// session2监听
1) "message"
2) "ooxx"
3) "11

```

### 2.3 聊天系统

![image-20200715224733889](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200715224733889.png)