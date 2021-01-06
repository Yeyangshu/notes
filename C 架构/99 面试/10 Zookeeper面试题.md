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



Leader选举算法和流程

ZooKeeper怎么解决脑裂问题

ZooKeeper一致性协议（ZAB）