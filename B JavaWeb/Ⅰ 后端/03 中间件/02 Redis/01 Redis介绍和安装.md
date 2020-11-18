## 1 Redis概述

Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。 它支持多种类型的数据结构，如 字符串（strings）， 散列（hashes）， 列表（lists）， 集合（sets）， 有序集合（sorted sets） 与范围查询， bitmaps， hyperloglogs 和 地理空间（geospatial） 索引半径查询。 Redis 内置了 复制（replication），LUA脚本（Lua scripting）， LRU驱动事件（LRU eviction），事务（transactions） 和不同级别的 磁盘持久化（persistence）， 并通过 Redis哨兵（Sentinel）和自动 分区（Cluster）提供高可用性（high availability）。

数据库排名网站：https://db-engines.com/en/：上面有很多数据库的介绍，做技术架构、技术选型的时候使用，将网站的模型看一下，文档存储、图、时序的、关系的、键值对的、列式的必须要看，并且背下来，他们之间的差异在哪里？面试的时候会让面试管眼前一亮，并不是背概念。

Redis官方网站：https://db-engines.com/en/

Redis官方文档：https://redis.io/documentation

Redis命令参考：http://doc.redisfans.com/

## 2 安装

官方文档：http://download.redis.io/releases/redis-5.0.5.tar.gz

```shell
安装wget
yum install wget
wget http://download.redis.io/releases/redis-5.0.5.tar.gz
tar xf redis-5.0.5.tar.gz
先看README.md

make
显示/bin/sh: cc: command not found
yum install gcc
清除报错
make distclear
再
make

make install PREFIX=/opt/soft/redis //安装在和源码不同目录
cd /opt/soft/redis/bin

cd /root/soft/bin
环境变量
vi /etc/profile
最后一行添加
export REDIS_HOME=/opt/soft/redis
export PATH=$PATH:$REDIS_HOME/bin

source /etc/profile
echo $PATH
$:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin:/opt/soft/redis/bin
---

安装
./install_server.sh 
Welcome to the redis service installer
This script will help you easily set up a running redis server

Please select the redis port for this instance: [6379] 
Selecting default: 6379
Please select the redis config file name [/etc/redis/6379.conf] 
Selected default - /etc/redis/6379.conf
Please select the redis log file name [/var/log/redis_6379.log] 
Selected default - /var/log/redis_6379.log
Please select the data directory for this instance [/var/lib/redis/6379] 
Selected default - /var/lib/redis/6379
Please select the redis executable path [/opt/soft/redis/bin/redis-server] 
Selected config:
Port           : 6379
Config file    : /etc/redis/6379.conf
Log file       : /var/log/redis_6379.log
Data dir       : /var/lib/redis/6379
Executable     : /opt/soft/redis/bin/redis-server
Cli Executable : /opt/soft/redis/bin/redis-cli
Is this ok? Then press ENTER to go on or Ctrl-C to abort.
Copied /tmp/6379.conf => /etc/init.d/redis_6379
Installing service...
Successfully added to chkconfig!
Successfully added to runlevels 345!
Starting Redis server...
Installation successful!



cd /etc/init.d
会有一个redis_6379脚本
此时可以在任意位置
service redis_6379 status
Redis is running (6014)

进入redis
redis-cli
```

![image-20200714231217635](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200714231217635.png)

再开一个实例

```shell
cd /root/soft/redis-5.0.5/utils

./install_server.sh

输入6380
```

## 3 基本命令

### 3.1 help

```shell
127.0.0.1:6379> help
redis-cli 5.0.5
To get help about Redis commands type:
      "help @<group>" to get a list of commands in <group>
      "help <command>" for help on <command>
      "help <tab>" to get a list of possible help topics
      "quit" to exit

To set redis-cli preferences:
      ":set hints" enable online hints
      ":set nohints" disable online hints
Set your preferences in ~/.redisclirc
```

### 3.2 help @generic

在客户端输入`help @`，按TAB键，会出现所有的分组

查看通用组命令

```shell
127.0.0.1:6379> help @generic

  DEL key [key ...]
  summary: Delete a key
  since: 1.0.0

  DUMP key
  summary: Return a serialized version of the value stored at the specified key.
  since: 2.6.0

  EXISTS key [key ...]
  summary: Determine if a key exists
  since: 1.0.0

  EXPIRE key seconds
  summary: Set a key's time to live in seconds
  since: 1.0.0

  EXPIREAT key timestamp
  summary: Set the expiration for a key as a UNIX timestamp
  since: 1.2.0

  KEYS pattern
  summary: Find all keys matching the given pattern
  since: 1.0.0

  MIGRATE host port key| destination-db timeout [COPY] [REPLACE] [KEYS key]
  summary: Atomically transfer a key from a Redis instance to another one.
  since: 2.6.0

  MOVE key db
  summary: Move a key to another database
  since: 1.0.0

  OBJECT subcommand [arguments [arguments ...]]
  summary: Inspect the internals of Redis objects
  since: 2.2.3

  PERSIST key
  summary: Remove the expiration from a key
  since: 2.2.0

  PEXPIRE key milliseconds
  summary: Set a key's time to live in milliseconds
  since: 2.6.0

  PEXPIREAT key milliseconds-timestamp
  summary: Set the expiration for a key as a UNIX timestamp specified in milliseconds
  since: 2.6.0

  PTTL key
  summary: Get the time to live for a key in milliseconds
  since: 2.6.0

  RANDOMKEY -
  summary: Return a random key from the keyspace
  since: 1.0.0

  RENAME key newkey
  summary: Rename a key
  since: 1.0.0

  RENAMENX key newkey
  summary: Rename a key, only if the new key does not exist
  since: 1.0.0

  RESTORE key ttl serialized-value [REPLACE]
  summary: Create a key using the provided serialized value, previously obtained using DUMP.
  since: 2.6.0

  SCAN cursor [MATCH pattern] [COUNT count]
  summary: Incrementally iterate the keys space
  since: 2.8.0

  SORT key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC|DESC] [ALPHA] [STORE destination]
  summary: Sort the elements in a list, set or sorted set
  since: 1.0.0

  TOUCH key [key ...]
  summary: Alters the last access time of a key(s). Returns the number of existing keys specified.
  since: 3.2.1

  TTL key
  summary: Get the time to live for a key
  since: 1.0.0

  TYPE key
  summary: Determine the type stored at key
  since: 1.0.0

  UNLINK key [key ...]
  summary: Delete a key asynchronously in another thread. Otherwise it is just as DEL, but non blocking.
  since: 4.0.0

  WAIT numreplicas timeout
  summary: Wait for the synchronous replication of all the write commands sent in the context of the current connection
  since: 3.0.0

  SUBSTR key arg arg 
  summary: Help not available
  since: not known

  LOLWUT ...options...
  summary: Help not available
  since: not known

  PFSELFTEST 
  summary: Help not available
  since: not known

  PSYNC arg arg 
  summary: Help not available
  since: not known

  REPLCONF ...options...
  summary: Help not available
  since: not known

  PFDEBUG arg arg ...options...
  summary: Help not available
  since: not known

  LATENCY arg ...options...
  summary: Help not available
  since: not known

  XSETID key arg 
  summary: Help not available
  since: not known

  GEORADIUS_RO key arg arg arg arg ...options...
  summary: Help not available
  since: not known

  ASKING 
  summary: Help not available
  since: not known

  HOST: ...options...
  summary: Help not available
  since: not known

  POST ...options...
  summary: Help not available
  since: not known

  MODULE arg ...options...
  summary: Help not available
  since: not known

  RESTORE-ASKING key arg arg ...options...
  summary: Help not available
  since: not known

  GEORADIUSBYMEMBER_RO key arg arg arg ...options...
  summary: Help not available
  since: not known
```

### 3.2 generic分组命令详解

#### 3.2.1 EXISTS

####  3.2.2 KEYS

#### 3.2.3 MOVE

#### 3.2.4 TYPE

```
127.0.0.1:6379> help type

  TYPE key
  summary: Determine the type stored at key
  since: 1.0.0
  group: generic
```

返回 key 所储存的值的类型（命令是哪个分组，key的类型就是哪个分组）。 

案例：

```shell
127.0.0.1:6379> set k1 99
OK
127.0.0.1:6379> type k1
string
```

#### 3.2.5 OBJECT

```
127.0.0.1:6379> help object

  OBJECT subcommand [arguments [arguments ...]]
  summary: Inspect the internals of Redis objects
  since: 2.2.3
  group: generic
```

OBJECT 命令允许从内部察看给定 `key` 的 Redis 对象。

它通常用在除错(debugging)或者了解为了节省空间而对 `key` 使用特殊编码的情况。

当将Redis用作缓存程序时，你也可以通过 OBJECT 命令中的信息，决定 `key` 的驱逐策略(eviction policies)。

OBJECT 命令有多个子命令：

- `OBJECT REFCOUNT <key>` 返回给定 `key` 引用所储存的值的次数。此命令主要用于除错。
- `OBJECT ENCODING <key>` 返回给定 `key` 锁储存的值所使用的内部表示(representation)，也就是编码。
- `OBJECT IDLETIME <key>` 返回给定 `key` 自储存以来的空转时间(idle， 没有被读取也没有被写入)，以秒为单位。

案例：

```shell
127.0.0.1:6379> set k1 99
OK
127.0.0.1:6379> object encoding k1
"int"

127.0.0.1:6379> set k2 hello
OK
127.0.0.1:6379> object  encoding k2
"embstr"
```

## 4 二进制安全

编码

```shell
127.0.0.1:6379> set k1 99
OK
127.0.0.1:6379> object encoding k1
"int"
127.0.0.1:6379> set k2 hello
OK
127.0.0.1:6379> object encoding k2
"embstr"
```

二进制安全

```shell
127.0.0.1:6379> set k1 hello
OK
127.0.0.1:6379> STRLEN k1
(integer) 5

127.0.0.1:6379> set k2 9
OK
127.0.0.1:6379> OBJECT encoding k2
"int"
127.0.0.1:6379> STRLEN k2
(integer) 1

127.0.0.1:6379> APPEND k2 999
(integer) 4
127.0.0.1:6379> get k2
"9999"
127.0.0.1:6379> OBJECT encoding k2
"raw"

127.0.0.1:6379> INCR k2
(integer) 10000
127.0.0.1:6379> OBJECT encoding k2
"int"
127.0.0.1:6379> STRLEN k2
(integer) 5

127.0.0.1:6379> set k3 a
OK
127.0.0.1:6379> get k3
"a"
127.0.0.1:6379> STRLEN k3
(integer) 1
127.0.0.1:6379> APPEND k3 中
(integer) 3
127.0.0.1:6379> STRLEN k3
(integer) 3
```

Redis只取字节流，一定要在开发中沟通好数据的编码和解码。



