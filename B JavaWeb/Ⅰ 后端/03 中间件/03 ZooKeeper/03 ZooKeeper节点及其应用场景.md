# ZooKeeper节点及其应用场景

千万不要把ZooKeeper做数据库用

## 1 节点数据结构

![image-20200801175304765](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801175304765.png)

```
cZxid：该数据节点被创建时的事务id
0x：16进制
2:代表leader纪元
00000002：事务id

mZxid：该数据节点被修改时最新的事物id
PZxid：当前节点的父级节点事务id
```

Java代码

```java
@InterfaceAudience.Public
    public class Stat implements Record {
        // 该数据节点被创建时的事务id
        private long czxid;
        // 该数据节点被修改时最新的事物id
        private long mzxid;
        // 该数据节点创建时间
        private long ctime; 
        // 该数据节点最后修改时间
        private long mtime; 
        // 当前节点版本号（可以理解为修改次数，每修改一次值+1）
        private int version; 
        // 子节点版本号（子节点修改次数，每修改一次值+1）
        private int cversion;
        // 当前节点acl版本号（acl节点被修改次数，每修改一次值+1）
        private int aversion; 
        // 临时节点标示，当前节点如果是临时节点，则存储的创建者的会话id（sessionId），如果不是，那么值=0
        private long ephemeralOwner; 
        // 当前节点数据长度
        private int dataLength;
        // 当前节点子节点个数
        private int numChildren; 
        // 当前节点的父级节点事务id
        private long pzxid; 
        public Stat() {
        }
    }
```

## 2 持久节点 create xx

## 3 持久顺序节点  create -s xx

### create -s

情景：分布式下两个客户端想在同一目录下创建同一名称的目录，会不会覆盖？

客户端1创建没有问题

![image-20200801185806776](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801185806776.png)

客户端2创建时提示已创建

![image-20200801185834493](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801185834493.png)

ZooKeeper可以使用create -s来保证节点不重复 ，每个节点都会有一个id

客户端1，此时是009

![image-20200801190539256](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801190539256.png)

客户端2，此时是010

![image-20200801190617482](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801190617482.png)

-s 可以做以下

- 分布式统一命名规则，分布式ID

## 4  临时节点  create -e xx

### create -e 

客户端连接时会创建session id（session共享视图）

![image-20200801183028386](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801183028386.png)

创建临时节点

```
create -e /ooxx "123"
```

![image-20200801183232944](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801183232944.png)

session创建会消耗一个事务id

验证：

先连接一个客户端1，创建一个key，此时cZxid是002

![image-20200801184351073](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801184351073.png)

再连接一个客户端2，什么都不做，此时也会创建一个session id

![image-20200801184445883](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801184445883.png)

客户端1再创建一个key，此时cZxid变成了004

![image-20200801184541775](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801184541775.png)

客户端断开的时候也会同步session，为了统一视图，所有节点都删掉，会消耗一个事务id，验证

客户端3退出

![image-20200801184842158](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801184842158.png)

客户端1再新建一个key，此时cZxid变成了006

![image-20200801184927152](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801184927152.png)

## 5 临时顺序节点 create -e -s

## 6 节点应用场景

### 6.1 统一配置管理

1M数据实现

### 6.2 分组管理

path结构实现

### 6.3 统一命名

sequential实现

### 6.4 同步

临时节点实现

#### 6.4.1 分布式锁

临时节点实现

#### 6.4.2 事务锁

![image-20201224002319965](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201224002319965.png)

#### 6.4.3 HA选主