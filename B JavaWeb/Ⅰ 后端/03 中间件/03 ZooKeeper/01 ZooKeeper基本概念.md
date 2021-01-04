# ZooKeeper基本概念

## 1 Redis回顾

![image-20200801210935306](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801210935306.png)

Redis回顾

1. 优点
   - Redis是单实例的，基于内存的，所以非常快
   - 复制集群，保证高可用
   - 集群模式：分片
2. 弊端：分布式协调、分布式锁

ZooKeeper可以实现此功能

## 2 ZooKeeper：分布式应用程序的分布式协调服务

ZooKeeper 是一个典型的分布式数据一致性解决方案，分布式应用程序可以基于它实现诸如发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。

ZooKeeper是用于分布式应用程序的分布式，开放源代码协调服务。

它公开了一组简单的原语，分布式应用程序可以基于这些原语（如下图）来实现用于同步，配置维护以及组和命名的更高级别的服务。

![image-20201221224052599](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201221224052599.png)

它的设计易于编程，并使用了按照文件系统熟悉的目录树结构样式设置的数据模型。它以Java运行，并且具有Java和C的绑定。

众所周知，协调服务很难做到。它们特别容易出现诸如比赛条件和死锁之类的错误。ZooKeeper背后的动机是减轻分布式应用程序从头开始实施协调服务的责任。

## 3 ZooKeeper设计目标

### 3.1 ZooKeeper很简单

ZooKeeper允许分布式进程通过共享的分层名称空间相互协调，该名称空间的组织方式类似于标准文件系统。名称空间由数据寄存器（在ZooKeeper看来是znode）组成，它们类似于文件和目录。与设计用于存储的典型文件系统不同，ZooKeeper数据保留在**内存**中，这意味着ZooKeeper**可以实现高吞吐量和低延迟数**。

### 3.2 ZooKeeper复制集群

![image-20200728004707992](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200728004707992.png)





![image-20201221225439473](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201221225439473.png)



ZooKeeper是主从模式，主从模式会有一个Leader，Leader肯定会挂，服务不可靠会造成不可靠的集群，事实上，ZooKeeper集群及其高可用的需要一种方式快速的恢复出一个Leader。

Leader挂掉之后会进入无主模型

![image-20201221225236321](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201221225236321.png)

所以ZooKeeper会有两种运行情况：

- 可用状态
- 不可用状态：无主模型

所以不可用状态恢复到可用状态应该越快越好。

官方200ms恢复，截图

![image-20201221225854821](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201221225854821.png)









### 3.3 ZooKeeper已订购

ZooKeeper用一个反映所有ZooKeeper事务顺序的数字标记每个更新。后续操作可以使用该命令来实现更高级别的抽象，例如同步原语。

### 3.4 ZooKeeper很快

在“读取为主”（读写分离）的工作负载中，它特别快。ZooKeeper应用程序可在数千台计算机上运行，并且在读取比写入更为常见的情况下，其性能最佳，比率约为10：1（10读取，1写入）。

## 4 数据模型和分层名称空间

ZooKeeper提供的名称空间与标准文件系统的名称空间非常相似。名称是由斜杠（/）分隔的一系列路径元素。ZooKeeper命名空间中的每个节点均由路径标识。

### 4.1 ZooKeeper的层次命名空间

![image-20201221230805061](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201221230805061.png)

## 5 节点和短暂节点

与标准文件系统不同，ZooKeeper命名空间中的每个节点都可以具有与其关联的数据以及子节点。就像拥有一个文件系统一样，该文件系统也允许文件成为目录。（ZooKeeper旨在存储协调数据：状态信息，配置，位置信息等，因此存储在每个节点上的数据通常很小，在字节到千字节范围内。）我们使用术语*znode*来明确表示在谈论ZooKeeper数据节点。

> ZooKeeper每个节点1M为了减少网络带宽和数据时延，不要把ZooKeeper当数据库用

Znodes维护一个统计信息结构，其中包括用于数据更改，ACL更改和时间戳的版本号，以允许进行缓存验证和协调更新。znode的数据每次更改时，版本号都会增加。例如，每当客户端检索数据时，它也会接收数据的版本。

原子地读取和写入存储在命名空间中每个znode上的数据。读取将获取与znode关联的所有数据字节，而写入将替换所有数据。每个节点都有一个访问控制列表（ACL），用于限制谁可以执行操作。

ZooKeeper还具有短暂节点的概念。只要创建znode的会话处于活动状态，这些znode就存在。会话结束时，将删除znode。

![image-20200801211459742](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801211459742.png)



## 6 有条件的更新和监视

ZooKeeper支持*watches*的概念。客户端可以在znode上设置*watches*。znode更改时，将触发并删除监视。触发监视后，客户端会收到一个数据包，说明znode已更改。如果客户端和其中一个ZooKeeper服务器之间的连接断开，则客户端将收到本地通知。

**3.6.0中的新增功能：**客户端还可以在znode上设置永久的，递归的*watches*，这些*watches*在被触发时不会被删除，并且会以递归方式触发已注册znode以及所有子znode的更改。

## 7 保证

ZooKeeper非常快速且非常简单。但是，由于其目标是作为构建更复杂的服务（例如同步）的基础，因此它提供了一组保证。这些是：

- 顺序一致性：来自客户端的更新将按照发送的顺序应用。
- 原子性：更新成功或失败。没有部分结果。
- 单个系统映像：无论客户端连接到哪个服务器，客户端都将看到相同的服务视图。也就是说，即使客户端故障转移到具有相同会话的其他服务器，客户端也永远不会看到系统的较旧视图。
- 可靠性：应用更新后，此更新将一直持续到客户端覆盖更新为止。
- 及时性：确保系统的客户视图在特定时间范围内是最新的。

zookeeper功能

![image-20200801212049408](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801212049408.png)

- zookeeper功能可以使用临时节点实现分布式锁
- zookeeper功能可以HA



