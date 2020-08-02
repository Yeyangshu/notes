# 原理知识

## 1 ZooKeeper特点

ZooKeeper是干嘛用的？

具有分布式协调、可扩展性、可靠性、时序性特点

### 1.1 扩展性

#### 1.1.1 架构

分布式架构有三种角色

- Leader
- Follower
- Observer

#### 1.1.2 读写分离

observer放大查询能力

#### 1.1.3 Observer设置

```
zoo.cfg
server.1=node01:2888:3888
server.2=node02:2888:3888
server.3=node03:2888:3888
server.4=node04:2888:3888:observer
```

### 1.2 可靠性

ZooKeeper必须要记住一句话：**攘外必先安内**

攘外可以理解为对外服务

安内可以理解为快速选主

#### 1.2.1 快速恢复

ZooKeeper也会挂，ZooKeeper的可靠性来自于它的快速恢复，选出Leader

#### 1.2.2 数据

可靠性还包括数据的可靠性、可用性、一致性，这就相当于攘其外。

分布式一致性属于什么一致性呢？使用的最终一致性。

最终一致性过程中节点是否对外提供服务？ZooKeeper组件没有过半数的，没有连接到Leader的，需要shutdown自己对外服务的，这是ZooKeeper一大特点

##### 分布式一致性算法-Paxos

以上数据都是基于分布式的，需要了解分布式一下协议。

文章：https://www.douban.com/note/208430424/

过半通过，两阶段提交

##### ZooKeeper一致性算法-ZAB

ZooKeeper有自己的一个ZAB协议，作用在有Leader、可用的状态

Zab协议 的全称是 **Zookeeper Atomic Broadcast** （Zookeeper原子广播）

1. Client向Follower发起一个写操作，create ooxx

2. Follower转发给Leader

3. Leader创建了一个事务id，假设是8

4. 使用队列向两个Follower发起写日志（4-1）

   zk的数据状态在内存，用磁盘保存日志

5. Follower1会回复ok，此时已经过半（4-1 ok）

6. Leader向两个Follower发起写，写到内存，写完回复Leaderok（4-2）

7. Leader向两个Follower1回复ok（5）

8. Follower1向客户端回复ok（6）

   ![image-20200802103915714](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200802103915714.png)

## 2 ZooKeeper Leader选举

### 2.1 场景

两个场景：

1. 第一次启动集群
2. 重启集群

每个机器会有自己的mid，会有事务的Zxid

### 2.2 选取机制

怎么选取新的Leader？

1. 第一步，判断经验最丰富Zxid
2. 第二步，myid，选年龄最大的

对于所有Zxid，过半通过的才是真数据，你见到的Zxid都是可用的

### 2.3 案例

四台机器，过半：4/2+1=3

#### 2.3.1 第一次启动集群

假设依次启动node01，node02，node03

- node01：myid=1，事务id=Z0
- node02：myid=2，事务id=Z0
- node03：myid=3，事务id=Z0

此时事务id都是0，node03 myid增最大，node03作为连接

![image-20200802232623921](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200802232623921.png)

#### 2.3.2 重启集群

假设node04是Leader，node04开启队列向其他节点同步

![image-20200802233012317](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200802233012317.png)

- node01：事务id=8
- node02：事务id=8
- node03：事务id=7

node04挂掉，与其他节点断开连接

![image-20200802233059191](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200802233059191.png)

最极端的情景，node03使用事务Zxid=7先发起投票，node01和node02比较之后淘汰了Zxid=7，node01和node02会发送自己的事务id给node03，给自己加一票并且会广播给node01



最终node02 3票

![image-20200803001829588](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200803001829588.png)



### 2.4 ZooKeeper选举过程

1. 3888造成两两通信！

2. 只要任何人投票，都会触发那个准leader发起自己的投票

3. 推选制：先比较zxid，如果zxid相同，再比较myid

