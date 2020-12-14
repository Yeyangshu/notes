# Mysql日志

官网：https://dev.mysql.com/doc/refman/5.7/en/server-logs.html

## 1 Mysql日志分类

- 1.redo 重做日志

  作用：确保日志的**持久性**，防止在发生故障，脏页未写入磁盘。重启数据库会进行redo log执行重做，达到事务一致性

- undo 回滚日志

  作用：保证数据的原子性，记录事务发生之前的一个版本，用于回滚，innodb事务可重复读和读取已提交 隔离级别就是通过mvcc+undo实现

- bin log 二进制日志

  作用：用于主从复制，实现主从同步

- errlog 错误日志

  作用：Mysql本身启动，停止，运行期间发生的错误信息

- relay log 中继日志

  作用：用于数据库主从同步，将主库发来的bin log保存在本地，然后从库进行回放

- slow query log 慢查询日志

  作用：记录执行时间过长的sql，时间阈值可以配置，只记录执行成功

- general log 普通日志

  作用：记录数据库的操作明细，默认关闭，开启后会降低数据库性能

## 2 Mysql日志作用

### 2.1 Redo日志—innodb存储引擎的日志文件

- 当发生数据修改的时候，innodb引擎会先将记录写到redo log中，并更新内存，此时更新就算是完成了，同时innodb引擎会在合适的时机将记录操作到磁盘中

- Redo log 是**固定大小**的，是循环写的过程

- 有了Redo log之后，innodb就可以保证即使数据库发生异常重启，之前的记录也不会丢失，叫做crash-safe

  ![](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200711165713.png)

  为什么写redo log的时候不会造成IO的问题？

  WAL：write ahead log
  
  用户空间 -> 内核 -> 磁盘
  
  ![image-20201214224118795](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201214224118795.png)
  
  写磁盘三种方式：
  
  ![image-20201214224057127](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201214224057127.png)

### 2.2 Undo log

- undo log是为了实现事务的原子性，在Mysql数据库innodb**存储引擎**中，还用undo log来实现多版本并发控制（简称MVCC）
- 在操作任何数据之前，首先将数据备份到一个地方（这给存储数据备份的地方称为undo log），然后进行数据的修改。如果出现了错误或者用户执行了rollback语句，系统可以用undo log中的备份数据将数据恢复到事务开始之前的状态
- 注意：undo log是逻辑日志，可以理解为：
  - 当delete一条数据时，undo log中会记录一条对应的insert语句
  - 当insert一条记录时，undo log中会记录一条对应的delete记录
  - 当update一条记录时，undo log中会记录一条对应的update记录

### 2.3 Binlog

- Binlog中会记录所有的逻辑，并且采用追加写的方式

- 一般在企业中数据库会有备份系统，可以定期执行备份，备份的周期可以自己设置

  恢复数据的过程：

  1. 找到最近的一次全量备份数据
  2. 从备份的时间点开始，将备份的Binlog取出来，重放到要恢复的那个时刻

- Binlog是server层的日志，主要做mysql功能层面的事情

- 与redo日志区别
  1. redo是innodb独有的，Binlog是所有引擎都可以使用的
  2. redo是物理日志，记录的是某个数据页上做了什么修改，Binlog是逻辑日志，记录的是这个语句的原始逻辑
  3. redo是循环写的，空间会用完，Binlog是可以追加写的，不会覆盖之前的日志信息
  
- Binlog中会记录所有的逻辑，并且采用追加写的方式

- 一般在企业中数据库会有备份系统，可以定期执行备份，备份的周期可以自己设置

- 恢复数据的过程：

  1. 找到最近一次的全量备份数据
  2. 从备份的时间点开始，将备份的Binlog取出来，重放到要恢复的那个时刻。

Binlog是默认关闭的

![image-20200711182852037](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200711182852.png)

### 2.4 分清楚日志作用范围

![image-20201214225530242](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201214225530242.png)

## 3 数据更新流程

![image-20200711185422893](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200711185422.png)

1. 执行器先从引擎中找到数据，如果在内存中直接返回，如果不在内存中，查询后返回
2. 执行器拿到数据之后会先修改数据，然后调用引擎接口重新吸入数据
3. 引擎将数据更新到内存，同时写数据到redo中，此时处于prepare阶段，并通知执行器执行完成，随时可以操作
4. 执行器生成这个操作的Binlog
5. 执行器调用引擎的事务提交接口，引擎把刚刚写完的redo改成commit状态



面试：为什么redo log两阶段提交?

答：**为了保证数据的一致性**，redo log和Binlog数据一致，不会发生数据错乱。

![](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200711190534.png)

