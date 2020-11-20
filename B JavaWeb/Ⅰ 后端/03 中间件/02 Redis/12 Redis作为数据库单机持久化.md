- # Redis数据库持久化

  ## 1 存储层存储数据前景

  缓存：数据可以丢失，速度快

  数据库：数据不可以丢失，速度快+**持久性**

  ### 1.1 存储层存储数据方式

  - 快照/副本
  - 日志

  ### 1.2 存储数据实现方案

  #### 1.2.1 第一种方案：阻塞

  [![image-20200716234042265](https://camo.githubusercontent.com/cb1ce04497f93cde5b85e508e42238d3e403480fd68a0572c46e1ba5b0b67615/68747470733a2f2f796579616e677368752d706963676f2e6f73732d636e2d7368616e676861692e616c6979756e63732e636f6d2f696d672f696d6167652d32303230303731363233343034323236352e706e67)](https://camo.githubusercontent.com/cb1ce04497f93cde5b85e508e42238d3e403480fd68a0572c46e1ba5b0b67615/68747470733a2f2f796579616e677368752d706963676f2e6f73732d636e2d7368616e676861692e616c6979756e63732e636f6d2f696d672f696d6167652d32303230303731363233343034323236352e706e67)

  #### 1.2.2 第二种方案：非阻塞

  [![image-20200716234138562](https://camo.githubusercontent.com/dc45454ea2e92ae6ce042ad06f0918d349efe9610e74c9d4f3dcfe5324707192/68747470733a2f2f796579616e677368752d706963676f2e6f73732d636e2d7368616e676861692e616c6979756e63732e636f6d2f696d672f696d6167652d32303230303731363233343133383536322e706e67)](https://camo.githubusercontent.com/dc45454ea2e92ae6ce042ad06f0918d349efe9610e74c9d4f3dcfe5324707192/68747470733a2f2f796579616e677368752d706963676f2e6f73732d636e2d7368616e676861692e616c6979756e63732e636f6d2f696d672f696d6167652d32303230303731363233343133383536322e706e67)

  #### 1.2.3 第三种方案：fork()+copy on write

  子进程只会读文件，写文件不会改文件

  - fork()：Linux系统调用

  - copy on write：机制

  [![image-20200716234415135](https://camo.githubusercontent.com/37796e4b8290bbba9c109026778db7e7cd1fb8771c0237b8f02c33bdd80ec9e4/68747470733a2f2f796579616e677368752d706963676f2e6f73732d636e2d7368616e676861692e616c6979756e63732e636f6d2f696d672f696d6167652d32303230303731363233343431353133352e706e67)](https://camo.githubusercontent.com/37796e4b8290bbba9c109026778db7e7cd1fb8771c0237b8f02c33bdd80ec9e4/68747470733a2f2f796579616e677368752d706963676f2e6f73732d636e2d7368616e676861692e616c6979756e63732e636f6d2f696d672f696d6167652d32303230303731363233343431353133352e706e67)

  [![image-20200716234547508](https://camo.githubusercontent.com/ba489b0edf82001d0dba85d67bfa8c9297c96b941074cc10a81b847f3736928e/68747470733a2f2f796579616e677368752d706963676f2e6f73732d636e2d7368616e676861692e616c6979756e63732e636f6d2f696d672f696d6167652d32303230303731363233343534373530382e706e67)](https://camo.githubusercontent.com/ba489b0edf82001d0dba85d67bfa8c9297c96b941074cc10a81b847f3736928e/68747470733a2f2f796579616e677368752d706963676f2e6f73732d636e2d7368616e676861692e616c6979756e63732e636f6d2f696d672f696d6167652d32303230303731363233343534373530382e706e67)

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
  >    [root@node1 ~]# echo $BASHPID | mor	e
  >    6359
  >    ```

  ## 2 Redis持久化方式

  Redis是内存的数据库，掉电易失，所以需要数据需要持久化。

  Redis提供了不同级别的持久化方式：

  - RDB：在指定的时间间隔能对你的数据进行快照存储。
  - AOF：记录每次对服务器的写操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以Redis协议追加保存每次的写操作到文件的末尾，Redis还能对AOF文件进行后台重写，使得AOF的体积不至于过大。
  - 也可以同时开启两种持久化方式，在这种情况下，当Redis重启的时候会有限载入AOF文件来恢复原始的数据，因为通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整

  ### 2.1 RDB（Redis DataBase）

  #### 2.1.1 RDB优点

  - RDB是一个非常紧凑的文件，它保存了某个时间点的数据集，非常适用于数据集的备份，比如可以在每个小时保存一下过去24小时内的数据，同时每天保存过去30天的数据，这样即使出了问题也可以根据需求恢复到不同版本的数据集。
  - RDB是一个紧凑的单一文件，很方便传送到另一个远端数据中心，非常适用于灾难恢复。
  - RDB在保存文件时父进程唯一需要做的就是fork出一个子进程，接下来的工作全部由子进程来做，父进程不需要再做其他IO操作，所以RDB持久化方式可以最大化Redis性能。
  - 与AOF相比，在恢复大的数据集的时候，RDB方式会更快一些。

  #### 2.1.2 RDB缺点

  - 如果你希望在Redis意外停止工作（例如电源中断）的情况下丢失的数据最少的话，那么RDB不适合你。虽然你可以配置不同的save时间点（例如每隔5分钟并且对数据集有100个写的操作），Redis要完整的保存整个数据集是一个比较繁重的工作，你通常会每隔5分钟或者更久做一次完整的保存，万一在Redis意外宕机，你可能会丢失几分钟的数据。
  - RDB需要经常fork子进程来保存数据集到硬盘上，当数据集比较大的时候，fork的过程是非常耗时的，可能会导致Redis在一些毫秒级内不能响应客户端的请求。如果数据集巨大并且CPU性能不是很好的情况下，这种情况会持续1秒，AOF也需要fork，但是你可以调节重写日志文件的频率来提高数据集的耐久度。

  总结：

  - 不支持拉链，只会有一个dump.rdb。
  - 容易丢失数据，时点与时点之间的窗口数据容易丢失。

  #### 2.1.3 RDB实现方式

  ##### 2.1.3.1 RDB配置方式

  1. 手动`SAVE`：适合关机维护时使用

  2. 手动`BGSAVE`：fork()，创建子进程

  3. 配置文件方式`BGSAVE`规则：注意，使用save标识，触发bsave

     ```properties
     ################################ SNAPSHOTTING  ################################
     #
     # Save the DB on disk:
     #
     #   save <seconds> <changes>
     #
     #   Will save the DB if both the given number of seconds and the given
     #   number of write operations against the DB occurred.
     #
     #   In the example below the behaviour will be to save:
     #   after 900 sec (15 min) if at least 1 key changed
     #   after 300 sec (5 min) if at least 10 keys changed
     #   after 60 sec if at least 10000 keys changed
     #
     #   Note: you can disable saving completely by commenting out all "save" lines.
     #
     #   It is also possible to remove all the previously configured save
     #   points by adding a save directive with a single empty string argument
     #   like in the following example:
     #
     #   save ""
     
     save 900 1
     save 300 10
     save 60 10000
     
     # By default Redis will stop accepting writes if RDB snapshots are enabled
     # (at least one save point) and the latest background save failed.
     # This will make the user aware (in a hard way) that data is not persisting
     # on disk properly, otherwise chances are that no one will notice and some
     # disaster will happen.
     #
     # If the background saving process will start working again Redis will
     # automatically allow writes again.
     #
     # However if you have setup your proper monitoring of the Redis server
     # and persistence, you may want to disable this feature so that Redis will
     # continue to work as usual even if there are problems with disk,
     # permissions, and so forth.
     stop-writes-on-bgsave-error yes
     
     # Compress string objects using LZF when dump .rdb databases?
     # For default that's set to 'yes' as it's almost always a win.
     # If you want to save some CPU in the saving child set it to 'no' but
     # the dataset will likely be bigger if you have compressible values or keys.
     rdbcompression yes
     
     # Since version 5 of RDB a CRC64 checksum is placed at the end of the file.
     # This makes the format more resistant to corruption but there is a performance
     # hit to pay (around 10%) when saving and loading RDB files, so you can disable it
     # for maximum performances.
     #
     # RDB files created with checksum disabled have a checksum of zero that will
     # tell the loading code to skip the check.
     rdbchecksum yes
     
     # The filename where to dump the DB
     dbfilename dump.rdb
     
     # The working directory.
     #
     # The DB will be written inside this directory, with the filename specified
     # above using the 'dbfilename' configuration directive.
     #
     # The Append Only File will also be created inside this directory.
     #
     # Note that you must specify a directory here, not a file name.
     dir ./
     ----------------------------------------------------------------------------------
     save 900 1
     save 300 10
     save 60 10000
     
     dbfilename dump.rdb
     dir /var/lib/redis/6379 
     ```

  ##### 2.1.3.2 RDB工作原理

  当 Redis 需要保存 dump.rdb 文件时， 服务器执行以下操作:

  - Redis 调用forks。同时拥有父进程和子进程。
  - 子进程将数据集写入到一个临时 RDB 文件中。
  - 当子进程完成对新 RDB 文件的写入时，Redis 用新 RDB 文件替换原来的 RDB 文件，并删除旧的 RDB 文件。

  这种工作方式使得 Redis 可以从写时复制（copy-on-write）机制中获益。

  ### 2.2 AOF（Append Only File）

  #### 2.2.1 AOF优点

  - 使用AOF会让Redis更加耐久
  - 

  #### 2.2.2 AOF缺点

  - 相同的数据集，AOF的体积通常要大于RDB文件的体积

  #### 2.2.3 持久化使用方式

  Redis配置文件 `APPEND ONLY MODE` 部分介绍了AOF的配置

  ```properties
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

  