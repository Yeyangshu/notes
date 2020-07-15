# 事务

Redis中文官网：http://redis.cn/topics/transactions.html

将一组命令放在同一个事务中进行处理

# 1 基本知识

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

## 2 使用

- MULTI：命令用于开启一个事务，它总是返回 `OK`
- EXEC：负责触发并执行事务中的所有命令
- DISCARD：事务会被放弃， 事务队列会被清空， 并且客户端会从事务状态中退出
- WATCH：使得 EXEC 命令需要有条件地执行： 事务只能在所有被监视键都没有被修改的前提下执行， 如果这个前提不能满足的话，事务就不会被执行。



一个事务

```
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

```
client1:
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> get k1
QUEUED
client2:
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> DEL k1
QUEUED
127.0.0.1:6379> EXEC
1) (integer) 1
client1:
127.0.0.1:6379> EXEC
1) (nil)
```



