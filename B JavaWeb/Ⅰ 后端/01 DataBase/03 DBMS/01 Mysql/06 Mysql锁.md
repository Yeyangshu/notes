# Mysql锁机制

## 1 Mysql锁的介绍

**锁是计算机协调多个进程或线程并发访问某一资源的机制。**在数据库中，除传统的 计算资源（如CPU、RAM、I/O等）的争用以外，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一 个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂。

相对其他数据库而言，MySQL的锁机制比较简单，其最 显著的特点是不同的**存储引擎**支持不同的锁机制。比如，MyISAM和MEMORY存储引擎采用的是表级锁（table-level locking）；InnoDB存储引擎既支持行级锁（row-level locking），也支持表级锁，但默认情况下是采用行级锁。 

**表级锁：**开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。 
**行级锁：**开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。  

从上述特点可见，很难笼统地说哪种锁更好，只能就具体应用的特点来说哪种锁更合适！仅从锁的角度 来说：表级锁更适合于以查询为主，只有少量按索引条件更新数据的应用，如Web应用；而行级锁则更适合于有大量按索引条件并发更新少量不同数据，同时又有 并发查询的应用，如一些在线事务处理（OLTP）系统。

> OLTP：联机事务处理，侧重时效性
>
> OLAP：联机分析处理，侧重历史数据整体分析

## 2 MylSAM表锁

Mysql的表级锁有两种模式：**表共享读锁（Table Read Lock）**和**表独占写锁（Table Write Lock）**。  

对MyISAM表的读操作，不会阻塞其他用户对同一表的读请求，但会阻塞对同一表的写请求；对 MyISAM表的写操作，则会阻塞其他用户对同一表的读和写操作；MyISAM表的读操作与写操作之间，以及写操作之间是串行的！

建表语句

```mysql
CREATE TABLE `mylock` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `NAME` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

INSERT INTO `mylock` (`id`, `NAME`) VALUES ('1', 'a');
INSERT INTO `mylock` (`id`, `NAME`) VALUES ('2', 'b');
INSERT INTO `mylock` (`id`, `NAME`) VALUES ('3', 'c');
INSERT INTO `mylock` (`id`, `NAME`) VALUES ('4', 'd');
```

### 2.1 MyISAM写锁阻塞读的案例

当一个线程获得对一个表的写锁之后，只有持有锁的线程可以对表进行更新操作。其他线程的写操作都会等待，知道锁释放为止。

| 序号 |                           session1                           |                         session2                          |
| :--: | :----------------------------------------------------------: | :-------------------------------------------------------: |
|  1   |       获取表的write锁定<br />lock table mylock write;        |                                                           |
|  2   | 当前session对表的查询，插入，更新操作都可以执行<br />select * from mylock;<br />insert into mylock values(5,'e'); | 当前session对表的查询会被阻塞<br />select * from mylock； |
|  3   |                释放锁：<br />unlock tables；                 |          当前session能够立刻执行，并返回对应结果          |

1. 序号1

   session1：

   ```mysql
   mysql> lock table mylock write;
   Query OK, 0 rows affected (0.00 sec)
   ```

2. 序号2

   session1：

   ```mysql
   mysql> select * from mylock;
   +----+------+
   | id | NAME |
   +----+------+
   |  1 | a    |
   |  2 | b    |
   |  3 | c    |
   |  4 | d    |
   +----+------+
   4 rows in set (0.00 sec)
   ```

   session2被阻塞：

   ```mysql
   mysql> select * from mylock;
   _
   ```

   ![image-20200712095154263](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200712095154.png)

   session1插入数据：

   ```mysql
   mysql> insert into mylock values (5, 'e');
   Query OK, 1 row affected (0.00 sec)
   
   mysql> select * from mylock;
   +----+------+
   | id | NAME |
   +----+------+
   |  1 | a    |
   |  2 | b    |
   |  3 | c    |
   |  4 | d    |
   |  5 | e    |
   +----+------+
   5 rows in set (0.00 sec)
   ```

3. session1释放锁
   session1：

   ```mysql
   mysql> unlock tables;
   Query OK, 0 rows affected (0.00 sec)
   ```

   session2立即执行：

   ```mysql
   mysql> select * from mylock;
   +----+------+
   | id | NAME |
   +----+------+
   |  1 | a    |
   |  2 | b    |
   |  3 | c    |
   |  4 | d    |
   |  5 | e    |
   +----+------+
   5 rows in set (4 min 37.20 sec)
   ```

### 2.1 MyISAM读阻塞写的案例

一个session使用lock table给表加读锁，这个session可以锁定表中的记录，但更新和访问其他表都会提示错误，同时，另一个session可以查询表中的记录，但更新就会出现锁等待。

| 序号 |                           session1                           |                           session2                           |
| :--: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|  1   |        获得表的read锁定<br />lock table mylock read;         |                                                              |
|  2   |   当前session可以查询该表记录：<br />select * from mylock;   |   当前session可以查询该表记录：<br />select * from mylock;   |
|  3   | 当前session不能查询没有锁定的表<br />select * from person<br />Table 'person' was not locked with LOCK TABLES | 当前session可以查询或者更新未锁定的表<br />select * from mylock<br />insert into person values(1,'zhangsan'); |
|  4   | 当前session插入或者更新表会提示错误<br />insert into mylock values(6,'f')<br />Table 'mylock' was locked with a READ lock and can't be updated<br />update mylock set name='aa' where id = 1;<br />Table 'mylock' was locked with a READ lock and can't be updated | 当前session插入数据会等待获得锁<br />insert into mylock values(6,'f'); |
|  5   |                  释放锁<br />unlock tables;                  |                       获得锁，更新成功                       |

1. 序号1

   session1获得read锁

   ```mysql
   mysql> lock table mylock read;
   Query OK, 0 rows affected (0.00 sec)
   ```

2. 序号2

   session1可以读取表中数据

   ```mysql
   mysql> select * from mylock;
   +----+------+
   | id | NAME |
   +----+------+
   |  1 | a    |
   |  2 | b    |
   |  3 | c    |
   |  4 | d    |
   |  5 | e    |
   +----+------+
   5 rows in set (0.00 sec)
   ```

   session2也可以读取表中数据

3. 序号3

   session1不能查询没有锁定的表

   ```mysql
   mysql> select * from psn;
   ERROR 1100 (HY000): Table 'psn' was not locked with LOCK TABLES
   ```

   session2可以查找

   ```mysql
   mysql> select * from psn;
   Empty set (0.00 sec)
   ```

4. 序号4

   session1插入或更新会提示错误

   ```mysql
   mysql> update mylock set name = 'bb' where id = 2;
   ERROR 1099 (HY000): Table 'mylock' was locked with a READ lock and can't be updated
   ```

   session2更新会被阻塞

   ```mysql
   mysql> update mylock set name = 'aa' where id = 1;
   _
   ```

   ![image-20200712100415158](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200712100415.png)

5. 序号5

   session1释放锁

   ```mysql
   mysql> unlock tables;
   Query OK, 0 rows affected (0.00 sec)
   ```

   session2立即执行

   ```mysql
   mysql> update mylock set name = 'aa' where id = 1;
   Query OK, 1 row affected (42.13 sec)
   Rows matched: 1  Changed: 1  Warnings: 0
   ```

   此时查看表中数据，已被更改

   ```mysql
   mysql> select * from mylock;
   +----+------+
   | id | NAME |
   +----+------+
   |  1 | aa   |
   |  2 | b    |
   |  3 | c    |
   |  4 | d    |
   |  5 | e    |
   +----+------+
   5 rows in set (0.00 sec)
   ```

   