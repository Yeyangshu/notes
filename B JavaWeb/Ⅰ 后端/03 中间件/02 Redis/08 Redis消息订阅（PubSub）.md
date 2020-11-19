# Redis 发布订阅（Pub/Sub）

中文官网：http://redis.cn/topics/pubsub.html

发布、订阅和取消订阅实现了发布/订阅消息范式，发送者（发布者）不是计划发送消息给特定的接收者（订阅者）。而是发布的消息分到不同的频道，不需要知道什么样的订阅者订阅。订阅者对一个或多个频道感兴趣，只需接收感兴趣的消息，不需要知道什么样的发布者发布的。这种发布者和订阅者的解耦合可以带来更大的扩展性和更加动态的网络拓扑。

## 1 Pub/Sub的基本知识

Redis是一个快速、稳定的发布/订阅的信息系统。

## 2 Pub/Sub使用

### 2.1 help @pubsub

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

#### 2.2.1 推送消息的格式

消息是一个有三个元素的多块响应，有三种情况:

- 第一个元素消息类型是subscribe：表示我们成功订阅到响应的第二个元素提供的频道，第三个参数代表我们现在订阅频道的数量。
- 第一个元素消息类型是unsubscribe：表示我们成功取消订阅响应的第二个元素提供的频道，第三个参数代表我们目前订阅的频道的数量。当最后一个参数是0的时候，我们不再订阅到任何频道。当我们在Pub/Sub以外状态，客户端可以发出任何redis命令。
- 第一个元素消息类型是message：这是另外一个客户端发出的发布命令的结果。第二个元素是来源频道的名称，第三个参数是实际消息的内容。

#### 2.2.2 发布/订阅

订阅foo和bar，客户端发出一个订阅的频道名称

```powershell
# client1 订阅渠道foo和bar
127.0.0.1:6379> SUBSCRIBE foo bar
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "foo"
3) (integer) 1
1) "subscribe"
2) "bar"
3) (integer) 2
```

其他客户端发到这些频道的消息将会被推送到所有订阅的客户端



foo和bar发布消息

```powershell
# client2 渠道发布
127.0.0.1:6379> PUBLISH foo Hello
(integer) 1
127.0.0.1:6379> PUBLISH bar World
(integer) 1

# client1 接收消息
127.0.0.1:6379> SUBSCRIBE foo bar
Reading messages... (press Ctrl-C to quit)
# 消息类型message
1) "message"
# 消息来源频道
2) "foo"
# 实际消息内容
3) "Hello"
1) "message"
2) "bar"
3) "World"
```

取消订阅

```powershell
127.0.0.1:6379> UNSUBSCRIBE foo
1) "unsubscribe"
2) "foo"
3) (integer) 0	
```

#### 2.2.3 模式匹配订阅（正则匹配）

Redis 的Pub/Sub实现支持模式匹配。客户端可以订阅全风格的模式以便接收所有来自能匹配到给定模式的频道的消息。

订阅

```powershell
# client1订阅消息
127.0.0.1:6379> PSUBSCRIBE news.*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "news.*"
3) (integer) 1
```

发布

```powershell
# client2消息发布
127.0.0.1:6379> PUBLISH news.foo Hello
(integer) 1

# client1消息接收
127.0.0.1:6379> PSUBSCRIBE news.*
1) "pmessage"
2) "news.*"
3) "news.foo"
4) "Hello"
```

取消订阅

```powershell
127.0.0.1:6379> PUNSUBSCRIBE news.*
1) "punsubscribe"
2) "news.*"
3) (integer) 0
```

#### 2.2.4 同时匹配模式和频道订阅的消息

订阅

```powershell
SUBSCRIBE foo
PSUBSCRIBE f*
```

上面的例子中，如果一个消息被发送到foo,客户端会接收到两条消息：一条message类型，一条pmessage类型。

## 3 发布订阅使用场景：聊天系统

Peter Noordhuis 提供了一个使用EventMachine 和Redis创建多用户高性能网路聊天的很棒的例子，链接：https://gist.github.com/pietern/348262

![image-20200715224733889](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200715224733889.png)