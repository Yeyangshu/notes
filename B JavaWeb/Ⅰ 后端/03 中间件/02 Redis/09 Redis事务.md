# 事务

Redis中文官网：http://redis.cn/topics/transactions.html

将一组命令放在同一个事务中进行处理

## 1 事务简介

MULTI、EXEC、DISCARD和WATCH是Redis食物相关的命令。事务可以一次执行多个命令，并且带有以下两个重要保证：

- 事务是一个单独的隔离操作

  事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。

- 事务是一个原子操作

  事务中的所有命令要么全部被执行，要么全部都不执行。

EXEC命令负责触发并执行事务中的所有的命令

- 如果客户端在使用 MULTI 开启了一个事物，却因为断线而没有成功执行 EXEC，那么事务中的所有命令都不会被执行。
- 如果客户端成功在开启事务之后执行 EXEC，那么事务中的所有命令都会被执行。

## 2 事务使用

### 2.1 help @transactions

```
127.0.0.1:6379> help @transactions

  DISCARD -
  summary: Discard all commands issued after MULTI
  since: 2.0.0

  EXEC -
  summary: Execute all commands issued after MULTI
  since: 1.2.0

  MULTI -
  summary: Mark the start of a transaction block
  since: 1.2.0

  UNWATCH -
  summary: Forget about all watched keys
  since: 2.2.0

  WATCH key [key ...]
  summary: Watch the given keys to determine execution of the MULTI/EXEC block
  since: 2.2.0
```

MULTI：标记一个事务块的开始。 随后的指令将在执行 EXEC 时作为一个原子执行。

- 返回值：始终为OK

EXEC：执行事务中所有在排队等待的指令并将链接状态恢复到正常，当使用 WATCH 时，只有当被监视的键没有被修改，且允许检查设定机制时，EXEC会被执行。

- 返回值：每个元素与原子事务中的指令一一对应 当使用 WATCH 时，如果被终止，EXEC 则返回一个空的应答集合

DISCARD：刷新一个事务中所有在排队等待的指令，并且将连接状态恢复到正常。如果已使用WATCH，DISCARD将释放所有被WATCH的key。

- 返回值：所有返回都是 OK

WATCH：标记所有指定的key 被监视起来，在事务中有条件的执行（乐观锁）。

- 返回值：始终为OK

### 2.2 事务用法

#### 2.2.1 开启并执行事务

MULTI 用于开启一个事务，它总是返回 `OK`，MULTI 执行之后，客户端可以继续向服务器发送任意多条命令，这些命令不会立即执行，而是被放在一个队列中，当 EXEC 命令被调用时，所有队列中的命令才会被执行。

一个事务

```powershell
127.0.0.1:6379> MULTI 
OK
127.0.0.1:6379> SET k1 aaa
QUEUED
127.0.0.1:6379> set k2 bbb
QUEUED
127.0.0.1:6379> EXEC
1) OK
2) OK
```

两个客户端两个事物不同顺序

```powershell
# client1:
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> get k1
QUEUED

# client2:
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> DEL k1
QUEUED
127.0.0.1:6379> EXEC
1) (integer) 1

# client1:
127.0.0.1:6379> EXEC
1) (nil)
```

#### 2.2.2 放弃事务

当执行 DISCARD 命令时， 事务会被放弃， 事务队列会被清空， 并且客户端会从事务状态中退出

#### 2.2.3 WATCH，使用 check-and-set 操作实现乐观锁

WATCH 命令可以为 Redis 事务提供 check-and-set （CAS）行为。

被 WATCH  的键会被监视，并会发觉这些键是否被改动过了。 如果有至少一个被监视的键在 EXEC 执行之前被修改了， 那么整个事务都会被取消， EXEC 返回 nil 来表示事务已经失败。

测试：

同时开启两个Redis客户端，client1和client2，数据库中存在k1:abc，client1 WATCH k1，先开始事务后执行，client2后开启事务先执行

```powershell
# client1开启事务不执行
127.0.0.1:6379> KEYS *
1) "k1"
127.0.0.1:6379> WATCH k1
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> GET k1
QUEUED
127.0.0.1:6379> KEYS *
QUEUED

# client2开启事务并执行，此时k1值被更改
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> KEYS *
QUEUED
127.0.0.1:6379> SET k1 def
QUEUED
127.0.0.1:6379> EXEC
1) 1) "k1"
2) OK
127.0.0.1:6379> GET k1
"def"

# client1执行事务返回nil，证明k1事务执行失败
127.0.0.1:6379> EXEC
(nil)
```
