# 性能监控

## 1 监控工具一：show profile

MySQL官网：https://dev.MySQL.com/doc/refman/8.0/en/show-profile.html

### 1.1 show profile all

all：显示所有性能信息

```mysql
show profile all for query 3;
```

![image-20200805000102570](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200805000102570.png)



```mysql
MySQL> show profile for query 3;
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 0.000063 |
| checking permissions | 0.000008 |
| Opening tables       | 0.000029 |
| System lock          | 0.000017 |
| init                 | 0.000040 |
| optimizing           | 0.000008 |
| statistics           | 0.000038 |
| preparing            | 0.000020 |
| executing            | 0.000005 |
| Sending data         | 0.000102 |
| end                  | 0.000004 |
| query end            | 0.000004 |
| closing tables       | 0.000018 |
| freeing items        | 0.000059 |
| logging slow query   | 0.000003 |
| cleaning up          | 0.000005 |
+----------------------+----------+
16 rows in set (0.00 sec)
```

## 2 监控工具二：Performance Sche4ma

MySQL官网：https://dev.MySQL.com/doc/refman/8.0/en/performance-schema.html

### 2.1 performance_schema介绍

MySQL的Performance Schema用于监控MySQL server在一个较低级别的运行过程中的资源消耗、资源等待等情况。

1. 提供了一种在数据库运行时实时检查server的内部执行情况的方法。performance_schema 数据库中的表使用performance_schema存储引擎。该数据库主要关注数据库运行过程中的性能相关的数据，与information_schema不同，information_schema主要关注server运行过程中的元数据信息

2. performance_schema通过监视server的事件来实现监视server内部运行情况， “事件”就是server内部活动中所做的任何事情以及对应的时间消耗，利用这些信息来判断server中的相关资源消耗在了哪里？一般来说，事件可以是函数调用、操作系统的等待、SQL语句执行的阶段（如sql语句执行过程中的parsing 或 sorting阶段）或者整个SQL语句与SQL语句集合。事件的采集可以方便的提供server中的相关存储引擎对磁盘文件、表I/O、表锁等资源的同步调用信息。

3. performance_schema中的事件与写入二进制日志中的事件（描述数据修改的events）、事件计划调度程序（这是一种存储程序）的事件不同。performance_schema中的事件记录的是server执行某些活动对某些资源的消耗、耗时、这些活动执行的次数等情况。
4. performance_schema中的事件只记录在本地server的performance_schema中，其下的这些表中数据发生变化时不会被写入binlog中，也不会通过复制机制被复制到其他server中。
5. 当前活跃事件、历史事件和事件摘要相关的表中记录的信息。能提供某个事件的执行次数、使用时长。进而可用于分析某个特定线程、特定对象（如mutex或file）相关联的活动。
6. PERFORMANCE_SCHEMA存储引擎使用server源代码中的“检测点”来实现事件数据的收集。对于performance_schema实现机制本身的代码没有相关的单独线程来检测，这与其他功能（如复制或事件计划程序）不同
7. 收集的事件数据存储在performance_schema数据库的表中。这些表可以使用SELECT语句查询，也可以使用SQL语句更新performance_schema数据库中的表记录（如动态修改performance_schema的setup_*开头的几个配置表，但要注意：配置表的更改会立即生效，这会影响数据收集）
8. performance_schema的表中的数据不会持久化存储在磁盘中，而是保存在内存中，一旦服务器重启，这些数据会丢失（包括配置表在内的整个performance_schema下的所有数据）
9. MySQL支持的所有平台中事件监控功能都可用，但不同平台中用于统计事件时间开销的计时器类型可能会有所差异。 

### 2.2 performance_schema入门

在MySQL5.7版本中，性能模式默认是开启的，如果想关闭的话，需要修改配置文件，不能直接进行修改，会报错Variable 'performance_schema' is a read only variable。

#### 2.2.1 基本属性

1. 查看performance_schema属性

   ```mysql
   show variables like 'performance_schema';
   ```

   ![image-20200808084527652](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200808084527652.png)

2. 查看performance_schema数据库中的所有表

   ```
   use performance_schema;
   show tables;
   ```

   一共有87张表

   87 rows in set (0.01 sec)

#### 2.2.2 两个基本概念

- instrument

  生产者：用于采集MySQL中各种各样的操作产生的时间信息，对应配置表中的配置项可以称为监控采集配置项。

- consumer

  消费者：对应的消费者表用于存储来自instrument采集的数据，对应配置表中的配置项可以称为消费存储配置项。

### 2.3 performance_schema表的分类

performance_schema库中的表可以按照监视的维度进行分类

#### 2.3.1 事件记录相关表

```mysql
show tables like 'events_statement%';
```

- events_statements_current：记录语句事件信息，当前语句时间表
- events_statements_history：历史语句事件表长语句历史事件表
- events_statements_history_long：长语句历史事件表
- events_statements_summary_*：聚合后的摘要表，summary可以根据账号（account）、主机（host）、程序（program）、线程（thread）、用户（user）和全局（global）细分

![image-20200808085752520](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200808085752520.png)

#### 2.3.2 等待事件记录表

与语句事件类型相关的表类似

```mysql
show tables like '%events_wait%';
```

![image-20200808090552680](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200808090552680.png)

#### 2.3.3 阶段事件记录表

记录语句执行的阶段事件表

```mysql
show tables like '%stage%';
```

![image-20200808090957999](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200808090957999.png)

#### 2.3.4 事务事件记录表

记录事务相关事件的表

```mysql
show tables like '%transaction%';
```

![image-20200808091119874](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200808091119874.png)

#### 2.3.5 监控文件系统层调用表

```mysql
show tables like '%file%';
```

![image-20200808091333129](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200808091333129.png)

#### 2.3.6 监控内存使用的表

```mysql
show tables like '%memory%';
```

![image-20200808091433002](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200808091433002.png)

#### 2.3.7 动态对performance_schema进行配置的配置表

```mysql
show tables like '%setup%';
```

![image-20200808091458943](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200808091458943.png)

### 2.4 performance_schema的简单配置与使用

数据库刚刚初始化并启动时，并非所有instruments（事件采集项，在采集项的配置表中每一项都有一个开关字段，或为YES，或为NO）和consumers（与采集项类似，也有一个对应的事件类型保存表配置项，为YES就表示对应的表保存性能数据，为NO就表示对应的表不保存性能数据）都启用了，所以默认不会收集所有的事件，可能你需要检测的事件并没有打开，需要进行设置，可以使用如下两个语句打开对应的instruments和consumers（行计数可能会因MySQL版本而异）。

## 3 性能监控工具三：show processlist

官网：https://dev.MySQL.com/doc/refman/8.0/en/show-processlist.html

![image-20200808100428938](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200808100428938.png)