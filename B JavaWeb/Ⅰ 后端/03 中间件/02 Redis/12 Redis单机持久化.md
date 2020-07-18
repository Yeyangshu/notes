# 持久化

## 1 前景

缓存：数据可以丢失，速度快

数据库：数据不可以丢失，速度快+持久性

### 1.1 存储层存储数据方式

- 快照/副本
- 日志

#### 1.1.1 快照实现方式

第一种情况：阻塞

![image-20200716234042265](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200716234042265.png)

第二种情况：非阻塞

![image-20200716234138562](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200716234138562.png)

第三种情况：fork()

子进程只会读文件，写文件不会该文件

![image-20200716234415135](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200716234415135.png)



![image-20200716234547508](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200716234547508.png)

> 管道
>
> 1. 衔接，前一个命令的输出作为后一个命令的输入
>
> 2. 管道会触发创建子进程
>
>    ```
>    [root@node1 ~]# num=0
>    [root@node1 ~]# echo $num
>    0
>    [root@node1 ~]# ((num++))
>    [root@node1 ~]# echo $num
>    1
>    [root@node1 ~]# ((num++)) | echo ok
>    ok
>    [root@node1 ~]# echo $$
>    6340
>    [root@node1 ~]# echo $$ | more // $$ 优先级高于 |
>    6340
>    [root@node1 ~]# echo $BASHPID
>    6340
>    [root@node1 ~]# echo $BASHPID | more // more优先级高于$BASHPID
>    6357
>    [root@node1 ~]# echo $BASHPID | more
>    6359
>    ```

## 2 Redis持久化

Redis提供了两种不同的持久化方式

- RDB：在指定的时间间隔能对你的数据进行快照存储
- AOF：记录每次对服务器的写操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以Redis协议追加保存每次的写操作到文件的末尾，Redis还能对AOF文件进行后台重写，使得AOF的体积不至于过大
- 也可以同时开启两种持久化方式，在这种情况下，当Redis重启的时候会有限载入AOF文件来恢复原始的数据，因为通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整

### 2.1 RDB（Redis DataBase）

#### 2.1.1 持久化使用方式

1. save

2. bgsave

3. 配置文件方式bgsave规则：使用save标识（注意）

   ```
   save 900 1
   save 300 10
   save 60 10000
   
   dbfilename dump.rdb
   dir /var/lib/redis/6379 
   ```

#### 2.1.2 优点

类似Java中的数据化，恢复速度相对快

#### 2.1.3 缺点

- 不支持拉链，只会有一个dump.rdb
- 容易丢失数据，时点与时点之间的窗口数据容易丢失



### 2.2 AOF（Append Only File）

#### 2.2.1 持久化使用方式

Redis配置文件 `APPEND ONLY MODE` 部分介绍了AOF的配置

```
# 是否开启AOF，默认关闭（no）
appendonly yes

# 指定 AOF 文件名
appendfilename appendonly.aof

# Redis支持三种不同的刷写模式：
# appendfsync always #每次收到写命令就立即强制写入磁盘，是最有保证的完全的持久化，但速度也是最慢的，一般不推荐使用。
appendfsync everysec #每秒钟强制写入磁盘一次，在性能和持久化方面做了很好的折中，是受推荐的方式。
# appendfsync no     #完全依赖OS的写入，一般为30秒左右一次，性能最好但是持久化最没有保证，不被推荐。

#在日志重写时，不进行命令追加操作，而只是将其放在缓冲区里，避免与命令的追加造成DISK IO上的冲突。
#设置为yes表示rewrite期间对新写操作不fsync,暂时存在内存中,等rewrite完成后再写入，默认为no
no-appendfsync-on-rewrite no 

#当前AOF文件大小是上次日志重写得到AOF文件大小的二倍时，自动启动新的日志重写过程。
auto-aof-rewrite-percentage 100

#当前AOF文件启动新的日志重写过程的最小值，避免刚刚启动Reids时由于文件尺寸较小导致频繁的重写。
auto-aof-rewrite-min-size 64mb
```

#### 2.2.2 优点

- 丢失数据少
- 

#### 2.2.3 缺点

- 相同的数据集，AOF的体积通常要大于RDB文件的体积
- 