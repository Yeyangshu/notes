# Zookeeper面试题

去offer来了找答案

### 1 CAP定理

### 2 ZAB协议

ZAB（ZooKeeper Atomic Broadcast）即ZooKeeper原子消息广播协议。该协议主要通过唯一的事务编号Zxid（ZooKeeper Transaction id）保障集群状态的唯一性。Zxid与RDBMS中的事务id类似，用于标识一次提议（Proposal）的id；为了保证顺序性，Zxid必须单调递增。

### Zookeeper如何保证可用性？

2、Zookeeper的原理

3、什么情况下会使用Zookeeper，Zookeeper如何监听生成的节点，zk内部是如何实现的

4、Zookeeper0、Zookeeper1、Zookeeper2，三个节点的集群描述一下从zk启动，到zk对外提供服务的整个过程

5、有一个key，往zk写入，到写入成功它的大体过程是什么样的

6、Zookeeper监听器的原理

Zookeeper实现原理，以及选主算法

Leader选举算法和流程

ZooKeeper怎么解决脑裂问题

ZooKeeper一致性协议（ZAB）

## ZAB

ZAB 协议的核心是定义了对于那些会改变 Zookeeper 服务器数据状态的事务请求的处理方式：

所有事务请求必须由一个全局唯一的服务器来协调处理，这样的服务器被称为 Leader 服务器，而余下的其他服务器则成为 Follower 服务器。Leader 服务器负责将一个客户端事务请求转换成一个事务 Proposal（提议），并将该 Proposal（提议）分发给集群中所有的 Follower 服务器。之后 Leader 服务器需要等待所有 Follower 服务器的反馈，一旦超过半数的 Follower 服务器进行了正确的反馈之后，那么 Leader 就会再次向所有的 Follower 服务器分发 Commit 消息，要求其前一个 Proposal 进行提交。

## Zookeeper 怎么防止脑裂

选举：过半机制

## paxos协议

就是过半提交大于当前的提议的编号