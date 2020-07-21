## 1 前景

Redis是单机、单节点、单实例的，会遇到如下问题：

- 单点故障
- 容量有限
- 压力

## 2 AKF扩展立方体

![image-20200719003124588](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200719003124588.png)

- x轴：全量、镜像
- y轴：业务、功能
- z轴：优先级、逻辑

### 2.1 存在问题

### 2.1.1 通过AKF使一台机器变为多台，如何保持数据一致性？

解决方案：

1. 强一致性

   所有节点阻塞直到数据全部一致，所有同步的方式都为阻塞方式

   ![image-20200719222038160](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200719222038160.png)

   存在问题：强一致性会破坏可用性，违背最初实现可用性的初衷

2. 强一致性降级，允许数据丢失一部分

   所有同步方式都是异步方式

   ![image-20200719222108340](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200719222108340.png)

   存在问题：数据丢失

3. 最终数据一致性

   同步阻塞，将消息传递给可靠、集群、响应速度足够快的中间件（比如kafka）

   ![image-20200719222132871](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200719222132871.png)

   存在问题：数据不一致

### 2.1.2 主机单点

主备：读写都是主机上，备机就是备用，同步主机的数据，时刻待命着等待主机挂了之后取而代之。因此在**主机还活着的情况下，备机的唯一使命就是同步主机的数据，不对外提供服务**。

主从：从机和备机的区别在于它得除了同步数据之外还得对外提供读的操作

Redis倾向于主从

问题：主读写，又是一个单点的

解决方案：对主做高可用（HA，High Availability），期望主机发生单点故障时自动做故障转移

![image-20200719225231907](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200719225231907.png)

对Redis进行监控

![image-20200719225042900](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200719225042900.png)



怎样判定Redis服务可用？

经验：使用奇数台监控机器

![image-20200719225152176](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200719225152176.png)

## 3 CAP原则

CAP原则又称CAP定理，指的是在一个分布式系统中，一致性（Consistency）、可用性（Availability）、分区容错性（Partition tolerance）。CAP 原则指的是，这三个要素最多只能同时实现两点，不可能三者兼顾。

## 4 主从复制

Redis中文官网：http://redis.cn/topics/replication.html

Redis使用默认的异步复制，其特点是低延迟和高性能，是绝大多数 Redis 用例的自然复制模式。

![image-20200719222108340](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200719222108340.png)

## 5 主从复制配置

![](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200720230305858.png)



repl-diskless-sync no

![image-20200720225705437](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200720225705437.png)

repl-backlog-size 1mb

![image-20200720230203076](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200720230203076.png)



一台机器开三个实例，新建一个test测试文件夹

```
cp /etc/redis/* ./
```

