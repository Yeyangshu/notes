# ZooKeeper安装和命令

## 1 安装并设置环境变量

ZooKeeper官网：https://zookeeper.apache.org/ -> Getting Started -> Download 



下载zookeeper-3.4.10.tar.gz并解压，移动解压后的文件夹至/opt/soft

zookeeper目录结构

![](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200729223552532.png)

进入conf，复制一份

修改临时文件目录

tmp/zookeeper -> var/zookeeper

```properties
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# 修改文件夹
dataDir=/var/zookeeper
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

# 新增配置，一定要先配置服务器hosts
server.1=node01:2888:3888
server.2=node02:2888:3888
server.3=node03:2888:3888
server.4=node04:2888:3888
```



>一定要先配置服务器hosts
>
>vi /etc/hosts
>
>192.168.163.111	   node01
>192.168.163.112	   node02
>192.168.163.113	   node03
>192.168.163.114	   node04

新建并进入文件夹/var/zookeeper，创建myid

```
[root@node1 zookeeper]# cat myid 
1
```

> 关闭其他虚拟机防火墙
>
> service iptables stop

复制zookeeper文件夹所有文件至其他虚拟机

```
scp -r ./zookeeper-3.4.10/ root@192.168.163.112:/opt/soft/zookeeper-3.4.10
```

新建文件夹并设置myid

```
mkdir -p /var/zookeeper
echo 2 > /var/zookeeper/myid
cat zookeeper/myid
2
```

设置环境变量

```properties
#set ZooKeeper environment
export ZOOKEEPER_HOME = /opt/soft/zookeeper-3.4.10
export PATH = $PATH:$ZOOKEEPER_HOME/bin
```

## 2 启动

zkServer.sh help可以查看启动方式

![image-20200801163835482](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801163835482.png)

```
zkServer.sh start
zkServer.sh start-foreground // 前台启动
```

### 2.1 前台启动测试

先使用前台启动方式依次启动node01~node04

使用

```
zkServer.sh status
```

可以查看leader，follower

本次测试node03是leader

![image-20200801171602787](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801171602787.png)

其余是追随者

![image-20200801171631540](X:\Users\11077\AppData\Roaming\Typora\typora-user-images\image-20200801171631540.png)

此时挂掉node01，选出node04作为leader

![image-20200801172018605](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801172018605.png)

![image-20200801172039973](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801172039973.png)

### 2.2 客户端命令

```
zkCli.sh
   help
   ls /   //根目录结构
    create  /ooxx  ""  //创建目录结构
    create -s /abc/aaa
    create -e /ooxx/xxoo //创建临时节点
    create -s -e /ooxx/xoxo
    get /ooxx //查看数据
```

#### 2.2.1 help命令

打印所有命令

![image-20200801174815508](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801174815508.png)

#### 2.2.2 ls /

![image-20200801181155987](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801181155987.png)

#### 2.2.3 create

![image-20200801175304765](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801175304765.png)

```
cZxid：该数据节点被创建时的事务id
0x：16进制
2:代表leader纪元
00000002：事务id

mZxid：该数据节点被修改时最新的事物id
PZxid：当前节点的父级节点事务id
```

##### create -e 临时节点

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

##### create -s

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

## 3 主从连接

查找2888/3888进程

```
netstat -natp   |   egrep  '(2888|3888)' 
```

此时node04是主

![image-20200801214706626](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801214706626.png)

