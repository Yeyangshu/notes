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

### 2.1.2 一变多，主机单点

- 主备：读写都是主机上，备机就是备用，同步主机的数据，时刻待命着等待主机挂了之后取而代之。因此在**主机还活着的情况下，备机的唯一使命就是同步主机的数据，不对外提供服务**。

- 主从：从机和备机的区别在于它得除了同步数据之外还得对外提供读的操作

Redis倾向于主从

存在问题：主机负责读写，又是一个单点的

解决方案：对主做高可用（HA，High Availability），期望主机发生单点故障时自动做故障转移

![image-20200719225231907](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200719225231907.png)

对Redis进行监控

![image-20200719225042900](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200719225042900.png)



怎样判定Redis服务可用？

经验：使用奇数台监控机器

![image-20200719225152176](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200719225152176.png)

## 3 CAP原则

CAP原则又称CAP定理，指的是在一个分布式系统中，一致性（Consistency）、可用性（Availability）、分区容错性（Partition tolerance）。CAP 原则指的是，这三个要素最多只能同时实现两点，不可能三者兼顾。

## 4 Redis主从复制

Redis中文官网：http://redis.cn/topics/replication.html

Redis使用默认的异步复制，其特点是低延迟和高性能，是绝大多数 Redis 用例的自然复制模式。

![image-20200719222108340](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200719222108340.png)

## 5 主从复制配置和演示

### 5.1 手动演示

#### 5.1.1 准备工作

一台机器开三个实例，新建一个test测试文件夹，进入test文件夹，拷贝所有redis配置文件至test文件夹

```
cp /etc/redis/* ./

主机和从机都设置一下日志
// 关闭后台运行
daemonize no
// 关闭日志文件，实时打印日志
#logfile /var/log/redis_6379.log
// 关掉AOF日志
appendonly no
效果：Redis实例前台阻塞运行，没有AOF日志
```

进入Redis持久化目录，删除文件夹里之前持久化的数据

```
cd /var/lib/redis/

ls
6379  6380  6381
// 保留目录，删除文件夹里面的数据
rm -fr ./*
```

进入test文件夹，启动redis

```
redis-server ~/test/6379.conf
redis-server ./6380.conf
redis-server ./6381.conf
```

![image-20200721224518202](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200721224518202.png)

进入redis程序

```
redis-cli -p 6379
redis-cli -p 6380
redis-cli -p 6381
```

现在已经有了三台实例，希望6379作为主机

#### 5.1.2 手动复制

从机复制命令

- 5.0以前：SLAVEOF
- 5.0以后：REPLICAIF

```
help SLAVEOF
```

![image-20200721225251413](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200721225251413.png)

6380机器连接6379

```
slaveof 192.168.1.1 6379
```

6379机器日志

```
6629:M 13 Jul 2020 15:37:10.256 * Replica 127.0.0.1:6380 asks for synchronization
6629:M 13 Jul 2020 15:37:10.256 * Partial resynchronization not accepted: Replication ID mismatch (Replica asked for '753f1f7659bb7ff017cc7c9e1c1ef0bb23eee769', my replication IDs are 'ce4485f2cc7f71534865b54ca649a7833b7c71a5' and '0000000000000000000000000000000000000000')
6629:M 13 Jul 2020 15:37:10.256 * Starting BGSAVE for SYNC with target: disk
6629:M 13 Jul 2020 15:37:10.292 * Background saving started by pid 6748
6748:C 13 Jul 2020 15:37:10.302 * DB saved on disk
6748:C 13 Jul 2020 15:37:10.303 * RDB: 4 MB of memory used by copy-on-write
6629:M 13 Jul 2020 15:37:10.355 * Background saving terminated with success
6629:M 13 Jul 2020 15:37:10.355 * Synchronization with replica 127.0.0.1:6380 succeeded
```

![image-20200721231737542](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200721231737542.png)

6380机器日志

```
6669:S 13 Jul 2020 15:37:09.547 * Before turning into a replica, using my master parameters to synthesize a cached master: I may be able to synchronize with the new master with just a partial transfer.
6669:S 13 Jul 2020 15:37:09.547 * REPLICAOF 127.0.0.1:6379 enabled (user request from 'id=3 addr=127.0.0.1:52883 fd=7 name= age=1580 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=44 qbuf-free=32724 obl=0 oll=0 omem=0 events=r cmd=replicaof')
6669:S 13 Jul 2020 15:37:10.254 * Connecting to MASTER 127.0.0.1:6379
6669:S 13 Jul 2020 15:37:10.256 * MASTER <-> REPLICA sync started
6669:S 13 Jul 2020 15:37:10.256 * Non blocking connect for SYNC fired the event.
6669:S 13 Jul 2020 15:37:10.256 * Master replied to PING, replication can continue...
6669:S 13 Jul 2020 15:37:10.256 * Trying a partial resynchronization (request 753f1f7659bb7ff017cc7c9e1c1ef0bb23eee769:1).
6669:S 13 Jul 2020 15:37:10.317 * Full resync from master: 6f76be684c7924f28999c4a24b6f212002285eff:0
6669:S 13 Jul 2020 15:37:10.317 * Discarding previously cached master state.
6669:S 13 Jul 2020 15:37:10.355 * MASTER <-> REPLICA sync: receiving 175 bytes from master
// 清除老数据
6669:S 13 Jul 2020 15:37:10.355 * MASTER <-> REPLICA sync: Flushing old data
6669:S 13 Jul 2020 15:37:10.355 * MASTER <-> REPLICA sync: Loading DB in memory
6669:S 13 Jul 2020 15:37:10.355 * MASTER <-> REPLICA sync: Finished with success
```

![image-20200721231941240](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200721231941240.png)

6379插入数据k1

```
set k1 123
```

6380可以查询此数据，但是6380不可以新增

```
27.0.0.1:6380> get k1
"123"
127.0.0.1:6380> set k2 123
(error) READONLY You can't write against a read only replica.
```

如果6380挂了一段时间后恢复，数据怎么恢复呢？全部copy一遍还是复制增量？

6380重新启动，继续追随6379

```
./6380.conf  --replicaof 127.0.0.1 6379
```



![image-20200721233029017](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200721233029017.png)



也可以使用下面命令，会产生RDB

```
./6380.conf  --replicaof 127.0.0.1 6379 -- appendonly yes
```

![image-20200721233433980](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200721233433980.png)



主机可以知道有那些机器连接到它

如果主机挂了，slave就会断开连接

![image-20200721233825534](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200721233825534.png)

如果slave不想追随主机

```
REPLICAOF no one
```

![image-20200721233951199](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200721233951199.png)

### 5.2 配置文件

查看6379.conf

![image-20200721235844532](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200721235844532.png)

```
// 机器
replicaof <masterip> <masterport>
// 密码
masterauth <master-password>

// 设置成yes，主从复制中，从服务器可以响应客户端请求;设置成no，主从复制中，从服务器将阻塞所有请求，有客户端请求时返回“SYNC with master in progress”
replica-serve-stale-data yes

replica-read-only yes
repl-diskless-sync no
```

![](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200720230305858.png)



repl-diskless-sync no

![image-20200720225705437](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200720225705437.png)

repl-backlog-size 1mb

![image-20200720230203076](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200720230203076.png)

### 5.3 存在问题

以上都是手动去配置，需要人工维护主的故障问题，怎样自动去维护故障问题呢？

哨兵！