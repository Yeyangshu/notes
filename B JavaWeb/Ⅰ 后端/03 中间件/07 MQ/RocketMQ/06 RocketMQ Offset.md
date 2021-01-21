# RocketMQ源码

## Consumer源码

### start启动流程

DefaultMQPushConsumer

### pullMessage消息拉取

### putMessage



## Offset

每个broker中的queue在收到消息时会记录offset，初始值为0，每记录一条消息offset会递增+1

### **minOffset**

最小值

### maxOffset

最大值

### consumerOffset

消费者消费进度/位置

### diffTotal

消费积压/未被消费的消息数量

## 消费者

### DefaultMQPushConsumer 与 DefaultMQPullConsumer

在消费端，我们可以视情况来控制消费过程

**DefaultMQPushConsumer** 由系统自动控制过程，

**DefaultMQPullConsumer** 大部分功能需要手动控制

### 集群消息的消费负载均衡

在集群消费模式下（clustering）

相同的group中的每个消费者只消费topic中的一部分内容

group中的所有消费者都参与消费过程，每个消费者消费的内容不重复，从而达到负载均衡的效果。

使用DefaultMQPushConsumer，新启动的消费者自动参与负载均衡。

### ProcessQueue

消息处理类 源码解析

### 长轮询

Consumer -> Broker RocketMQ采用的长轮询建立连接

- consumer的处理能力Broker不知道
- 直接推送消息 broker端压力较大
- 采用长连接有可能consumer不能及时处理推送过来的数据
- pull主动权在consumer手里

#### 短轮询

client不断发送请求到server，每次都需要重新连接，一般基于HTTP协议

![image-20210121224711761](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210121224711761.png)

#### 长轮询

client发送请求到server，server有数据返回，没有数据请求挂起不断开连接。

长轮询通过客户端和服务端的配合，达到主动权在客户端，同时也能保证数据的实时性；长轮询本质上也是轮询，只不过对普通的轮询做了优化处理，服务端在没有数据的时候并不是马上返回数据，会hold住请求，等待服务端有数据，或者一直没有数据超时处理，然后一直循环下去；

![image-20210121225548452](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210121225548452.png)

#### 长连接

连接一旦建立，永远不断开，Server主动push方式推送，一般基于WebSocket

![image-20210121224740275](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210121224740275.png)