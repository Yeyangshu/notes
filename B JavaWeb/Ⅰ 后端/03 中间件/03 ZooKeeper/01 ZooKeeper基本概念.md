# ZooKeeper基本概念

![image-20200801210935306](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801210935306.png)

Redis的弊端分布式协调、分布式锁

ZooKeeper可以实现此功能

## 1 设计目标



![image-20200728004707992](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200728004707992.png)

两种情况

zookeeper每个节点1M为了减少网络带宽和数据时延



![image-20200801211400382](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801211400382.png)





![image-20200801211459742](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801211459742.png)

session



zookeeper功能

![image-20200801212049408](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801212049408.png)

- zookeeper功能可以使用临时节点实现分布式锁
- zookeeper功能可以HA

