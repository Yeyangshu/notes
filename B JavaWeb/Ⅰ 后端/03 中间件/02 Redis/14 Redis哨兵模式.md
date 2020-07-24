# 哨兵（Sentinel）模式

Redis中文文档：http://redis.cn/topics/sentinel.html

## 1 Redis哨兵简介

Redis的哨兵系统用于管理多个Redis服务器，该系统执行以下三个任务：

- 监控
- 提醒
- 自动故障转移

## 2 Redis哨兵使用

### 2.1 获取 Sentinel

### 2.2 启动 Sentinel

### 2.3 配置 Sentinel

新建三个sentinel配置文件，例26379.conf、26380.conf、26381.conf监听主机，内容为

```
port 26379/26380/26381
sentinel monitor mymaster 127.0.0.1 6379 2
```

运行主机

```
redis-server ./6379.conf
```

运行两个slave

```
redis-server ./6380.conf --replicaof 127.0.0.1 6379
redis-server ./6381.conf --replicaof 127.0.0.1 6379
```

启动哨兵

```
redis-server ./26379.conf --sentinel
redis-server ./26380.conf --sentinel
redis-server ./26381.conf --sentinel
```

![image-20200722211540287](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200722211540287.png)

### 2.4 测试哨兵

此时断开6379，经过一段时间，会投票选出新的主机，本次测试2票选出6381

```
6909:X 13 Jul 2020 17:06:41.887 # +sdown master mymaster 127.0.0.1 6379
6909:X 13 Jul 2020 17:06:41.946 # +odown master mymaster 127.0.0.1 6379 #quorum 3/2
6909:X 13 Jul 2020 17:06:41.946 # +new-epoch 1
6909:X 13 Jul 2020 17:06:41.946 # +try-failover master mymaster 127.0.0.1 6379
6909:X 13 Jul 2020 17:06:41.964 # +vote-for-leader f0d0f7c12cc881a04d88ad7831100d0eea82cec4 1
6909:X 13 Jul 2020 17:06:41.969 # 3dd3212a440f415e989718634222ae89fc2fd219 voted for 3dd3212a440f415e989718634222ae89fc2fd219 1
6909:X 13 Jul 2020 17:06:41.989 # 44c8a525d5b8390571e2621fa3dfa416eafa8c0b voted for f0d0f7c12cc881a04d88ad7831100d0eea82cec4 1
6909:X 13 Jul 2020 17:06:42.027 # +elected-leader master mymaster 127.0.0.1 6379
6909:X 13 Jul 2020 17:06:42.027 # +failover-state-select-slave master mymaster 127.0.0.1 6379
6909:X 13 Jul 2020 17:06:42.100 # +selected-slave slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
6909:X 13 Jul 2020 17:06:42.100 * +failover-state-send-slaveof-noone slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
6909:X 13 Jul 2020 17:06:42.159 * +failover-state-wait-promotion slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
6909:X 13 Jul 2020 17:06:42.445 # +promoted-slave slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
6909:X 13 Jul 2020 17:06:42.445 # +failover-state-reconf-slaves master mymaster 127.0.0.1 6379
6909:X 13 Jul 2020 17:06:42.528 * +slave-reconf-sent slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
6909:X 13 Jul 2020 17:06:43.113 # -odown master mymaster 127.0.0.1 6379
6909:X 13 Jul 2020 17:06:43.482 * +slave-reconf-inprog slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
6909:X 13 Jul 2020 17:06:43.482 * +slave-reconf-done slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
6909:X 13 Jul 2020 17:06:43.554 # +failover-end master mymaster 127.0.0.1 6379
6909:X 13 Jul 2020 17:06:43.554 # +switch-master mymaster 127.0.0.1 6379 127.0.0.1 6381
6909:X 13 Jul 2020 17:06:43.554 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6381
6909:X 13 Jul 2020 17:06:43.554 * +slave slave 127.0.0.1:6379 127.0.0.1 6379 @ mymaster 127.0.0.1 6381
6909:X 13 Jul 2020 17:07:13.568 # +sdown slave 127.0.0.1:6379 127.0.0.1 6379 @ mymaster 127.0.0.1 6381
```

![image-20200722212104628](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200722212104628.png)

哨兵会自动更改配置文件

![image-20200722212459813](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200722212459813.png)

![image-20200722212525730](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200722212525730.png)

### 2.5 哨兵通信

哨兵之间是怎么通信的呢？是通过发布订阅保持支持通信的

```
redis-cli -p 6380
```

![image-20200722213101229](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200722213101229.png)

![image-20200722213224185](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200722213224185.png)