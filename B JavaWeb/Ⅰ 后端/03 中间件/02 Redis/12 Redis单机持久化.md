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

### 2.1 RDB

#### 2.1.1 持久化方式

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

#### 2.1.2 弊端

- 不支持拉链，只会有一个dump.rdb
- 容易丢失数据，时点与时点之间的窗口数据容易丢失

#### 2.1.3 优点

类似Java中的数据化，恢复速度相对快

### 2.2 AOF

