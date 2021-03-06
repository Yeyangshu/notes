# 分布式ID

雪花算法

2^63



全局唯一，局部唯一，按情况递增

**目标/思想：无锁、缓存、锁细化，保证全局唯一即可**



性能：缓存+无锁



## 1 业界实现方案

1. 基于UUID
2. 基于DB数据库多种模式（自增主键、segment）
3. 基于Redis
4. 基于ZK、ETCD
5. 基于SnowFlake
6. 美团Leaf（DB-Segment、zk+SnowFlake）
7. 百度uuid-generator()

## 2 基于UUID生成唯一ID

### 2.1 简介

UUID：

UUID长度128bit，32个16进制字符，占用存储空间多，且生成的ID是无序的；

对于InnoDB这种局级逐渐类型的引擎来说，数据会按照主键进行排序，由于UUID的无序性，InnoDB会产生巨大的IO压力，此时不适合使用UUID做物理主键，可以把它作为逻辑主键，物理主键依然使用自增ID；

组成部分：

为了保证UUID的唯一性，规范定义了包括网卡MAC地址、时间戳、名字空间、随机或伪随机数、时序等元素

优点：

性能非常高：本地生成，没有网络消耗

缺点：

不易于存储：UUID太长，16字节128位，通常以32长度的字符串表示，很多场景不适用

信息不安全：基于MAC地址生成UUID的算法可能会造成MAC地址泄漏，这个漏洞曾被用于寻找梅丽莎病毒的制作者位置

ID作为主键时在特定的环境会存在一些问题，比如做DB主键的场景下，UUID就非常不适用

### 2.2 UUID生成策略

59.00

#### 2.2.1 UUID Version 1：基于时间的UUID

#### 2.2.2 UUID Version 2：DCE安全的UUID

#### 2.2.3 UUID Version 3：基于名字的UUID（MD5）

#### 2.2.4 UUID Version 4：随机的UUID

#### 2.2.5 UUID Version 5：基于名字的UUID（SHA1）

### 2.3 UUID应用

UUID Version 1：基于时间的UUID

Version 1/2适合应用于分布式计算环境中，具有高度的唯一性

## 3 基于DB自增主键方案

## 4 基于Redis实现分布式ID

因为Redis是单线程的，所以天然没有资源争用问题，可以采用incr命令，实现ID的原子性自增。

但是因为Redis的数据数据备份-RDB，会存在漏掉数据的可能，所以理论上存在已使用的ID再次被使用，所以备份方式可以加上AOF方式，这样的话效果有所损耗。