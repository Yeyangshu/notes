## 1 ZooKeeper Leader选举

### 1.1 场景

两个场景：

1. 第一次启动集群
2. 重启集群

每个机器会有自己的mid，会有事务的Zxid

### 1.2 选取机制

怎么选取新的Leader？

1. 第一步，判断经验最丰富Zxid
2. 第二步，myid，选年龄最大的

对于所有Zxid，过半通过的才是真数据，你见到的Zxid都是可用的

### 1.3 案例

四台机器，过半：4/2+1=3

#### 1.3.1 第一次启动集群

假设依次启动node01，node02，node03

- node01：myid=1，事务id=Z0
- node02：myid=2，事务id=Z0
- node03：myid=3，事务id=Z0

此时事务id都是0，node03 myid增最大，node03作为连接

![image-20200802232623921](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200802232623921.png)

#### 1.3.2 重启集群

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



### 1.4 ZooKeeper选举过程

1. 3888造成两两通信！

2. 只要任何人投票，都会触发那个准leader发起自己的投票

3. 推选制：先比较zxid，如果zxid相同，再比较myid



如果四台机器4是leader，stop掉node04，node03会变成leader，stop掉node03，node和node02会报以下错误

![image-20200803220044756](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200803220044756.png)