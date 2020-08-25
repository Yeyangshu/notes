# Mysql查询优化

## 1 查询慢的原因

1. 网络

2. CPU
3. IO
4. 上下文切换
5. 系统调用
6. 生成统计时间
7. 锁等待时间

## 2 优化数据访问

### 2.1 大量数据优化

查询性能低下的主要原因是访问的数据量太多，某些查询不可避免的需要筛选大量的数据，我们可以进行减少访问数据量的方式进行优化

1. 确认应用程序是否在检索大量超过需要的数据
2. 确认MySQL服务器是否在分析大量超过需要的数据行

### 2.2 多余数据优化

是否向数据库请求了不需要的数据？

1. 查询不需要的记录

   我们常常会误以为mysql会只返回需要的数据，实际上mysql却是先返回全部结果再进行计算，在日常的开发习惯中，经常是先用select语句查询大量的结果，然后获取前面的N行后关闭结果集。

   **优化方式**：在查询后面添加limit

2. 多表关联时返回全部列

   ```sql
   select * from actor inner join film_actor using(actor_id) inner join film using(film_id) where film.title='Academy Dinosaur';
   ```

   ![image-20200815112843464](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200815112843464.png)

   优化方式：

   ```sql
   select actor.* from actor inner join film_actor using(actor_id) inner join film using(film_id) where film.title='Academy Dinosaur';
   ```

   ![image-20200815113006237](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200815113006237.png)

3. 总是取出全部列

   在公司的企业需求中，禁止使用select *,虽然这种方式能够简化开发，但是会影响查询的性能，所以尽量不要使用

4. 重复查询相同的数据

   如果需要不断的重复执行相同的查询，且每次返回完全相同的数据，因此，基于这样的应用场景，我们可以将这部分数据缓存起来，这样的话能够提高查询效率

## 3 执行过程优化

### 3.1 查询缓存

在解析一个查询语句之前，如果查询缓存是打开的，那么mysql会优先检查这个查询是否命中查询缓存中的数据，如果查询恰好命中了查询缓存，那么会在返回结果之前会检查用户权限，如果权限没有问题，那么mysql会跳过所有的阶段，就直接从缓存中拿到结果并返回给客户端（新版本已经移除）

### 3.2 查询优化处理

MySQL查询完缓存之后会经过以下几个步骤：解析SQL、预处理、优化SQL执行计划，这几个步骤出现任何错误，都可能会终止查询

#### 3.2.1 语法解析器和预处理

MySQL通过关键字将SQL语句进行解析，并生成一颗解析书，MySQL解析器将使用MySQL语法规则验证和解析查询，例如验证使用了错误的关键字或者顺序是否正确等等，预处理会进一步检查解析书是否合法，例如表名和列名是否存在，是否存在歧义，还会验证权限等等

#### 3.2.2 查询优化器

- CBU：基于成本的优化器
- RBU：基于规则的

当语法树没有问题之后，相应的要由优化器将其转成执行计划，一条查询语句可以使用非常多的执行计划，最后都可以得到对应的结果，但是不同的执行方式带来的执行效率是不同的，优化器的最主要目的就是要选择最有效的执行计划。

MySQL使用的是基于成本的优化器，在优化的时候回尝试预测一个查询使用查询计划时候的成本，并选择其中成本最小的一个

##### 3.2.2.1 信息统计

```sql
select count(*) from film_actor;
show status like 'last_query_cost';
```

![image-20200816091440915](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200816091440915.png)

可以看到这条查询语句大概需要做804个数据页才能找到对应的数据，这时经过一系列的统计信息计算来的。

统计信息如下：

- 每个表或索引的页面个数
- 索引的基数
- 索引和数据行的长度
- 索引的分布情况

##### 3.2.2.2 选择错误的执行计划

在很多情况下，MySQL会选择错误的执行计划，原因如下：

- 统计信息不准确

  InnoDB因为其mvcc的架构，并不能维护一个数据表的行数的精确统计信息

- 执行计划的成本估算不等同于实际执行的成本

  有时候某个执行计划虽然需要读取更多的页面，但是他的成本却更小，因为如果这些页面都是顺序读或者这些页面都已经在内存中的话，那么它的访问成本将很小，mysql层面并不知道哪些页面在内存中，哪些在磁盘，所以查询之际执行过程中到底需要多少次IO是无法得知的

- mysql的最优可能跟你想的不一样

  mysql的优化是基于成本模型的优化，但是有可能不是最快的优化

- mysql不考虑其他并发执行的查询

- mysql不会考虑不受其控制的操作成本

  不考虑执行存储过程或者用户自定义函数的成本

##### 3.2.2.3 优化器的优化策略

1. 静态优化

   直接对解析树进行分析，并完成优化

2. 动态优化

   动态优化与查询的上下文有关，也可能跟取值、索引对应的行数有关

mysql对查询的静态优化只需要一次，但对动态优化在每次执行时都需要重新评估

##### 3.2.2.4 优化器的优化类型

1. 重新定义关联表的顺序

   数据表的关联并不总是按照在查询中指定的顺序进行，决定关联顺序时优化器很重要的功能

2. 将外连接转化成内连接，内连接的效率要高于外连接

3. 使用等价变换规则，mysql可以使用一些等价变化来简化并规划表达式

   a != 4 好于 a > 4 and a < 4

4. 优化count()，min()，max()

   索引和列是否可以为空通常可以帮助mysql优化这类表达式：例如，要找到某一列的最小值，只需要查询**索引**的最左端的记录即可，不需要全文扫描比较

5. 预估并转化为常数表达式，当mysql检测到一个表达式可以转化为常数的时候，就会一直把该表达式作为常数进行处理

   ![image-20200816095725978](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200816095725978.png)

6. 索引覆盖扫描，当索引中的列包含所有查询中需要使用的列的时候，可以使用覆盖索引

7. 子查询优化

   mysql在某些情况下可以将子查询转换一种效率更高的形式，从而减少多个查询多次对数据进行访问，例如将经常查询的数据放入到缓存中

8. 等值传播

   如果两个列的值通过等式关联，那么mysql能够把其中一个列的where条件传递到另一个上：

   ```sql
   explain select film.film_id from film inner join film_actor using(film_id) where film.film_id > 500;
   ```

   这里使用film_id字段进行等值关联，film_id这个列不仅适用于film表而且适用于film_actor表

   ```sql
   explain select film.film_id from film inner join film_actor using(film_id) where film.film_id > 500 and film_actor.film_id > 500;
   ```

   ![image-20200816101730698](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200816101730698.png)

##### 3.2.2.5 关联查询

1. join的实现原理

   - Block Nested-Loop Join

     使用如下命令可以查看参数`block_nested_loop=on`

     ```txt
     optimizer_switch | index_merge=on,index_merge_union=on,index_merge_sort_union=on,index_merge_intersection=on,engine_condition_pushdown=on,index_condition_pushdown=on,mrr=on,mrr_cost_based=on,block_nested_loop=on,batched_key_access=off,materialization=on,semijoin=on,loosescan=on,firstmatch=on,duplicateweedout=on,subquery_materialization_cost_based=on,use_index_extensions=on,condition_fanout_filter=on,derived_merge=on
     ```
     
     
     
     ```sql
     mysql> show variables like '%optimizer_switch%';
     +------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
     | Variable_name    | Value                                                                                                                                         |
     +------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
     | optimizer_switch | index_merge=on,index_merge_union=on,index_merge_sort_union=on,index_merge_intersection=on,engine_condition_pushdown=on,index_condition_pushdown=on,mrr=on,mrr_cost_based=on,block_nested_loop=on,batched_key_access=off,materialization=on,semijoin=on,loosescan=on,firstmatch=on,duplicateweedout=on,subquery_materialization_cost_based=on,use_index_extensions=on,condition_fanout_filter=on,derived_merge=on |
     +------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
     1 row in set, 10 warnings (0.03 sec)
     ```

2. 案例演示

   查看不同的顺序执行方式对查询性能的影响：

   MySQL优化器执行顺序：actor -> film_actor -> film

   ```sql
   mysql> explain select film.film_id,film.title,film.release_year,actor.actor_id,actor.first_name,actor.last_name from film inner join film_actor using(film_id) inner join actor using(actor_id);
   +----+-------------+------------+------------+--------+------------------------+---------+---------+---------------------------+------+----------+-------------+
   | id | select_type | table      | partitions | type   | possible_keys          | key     | key_len | ref                       | rows | filtered | Extra       |
   +----+-------------+------------+------------+--------+------------------------+---------+---------+---------------------------+------+----------+-------------+
   |  1 | SIMPLE      | actor      | NULL       | ALL    | PRIMARY                | NULL    | NULL    | NULL                      |  200 |   100.00 | NULL        |
   |  1 | SIMPLE      | film_actor | NULL       | ref    | PRIMARY,idx_fk_film_id | PRIMARY | 2       | sakila.actor.actor_id     |   27 |   100.00 | Using index |
   |  1 | SIMPLE      | film       | NULL       | eq_ref | PRIMARY                | PRIMARY | 2       | sakila.film_actor.film_id |    1 |   100.00 | NULL        |
   +----+-------------+------------+------------+--------+------------------------+---------+---------+---------------------------+------+----------+-------------+
   3 rows in set, 1 warning (0.00 sec)
   ```

   不使用，固定顺序：film -> film_actor -> actor，效率低一些

   ```sql
   explain select straight_join film.film_id,film.title,film.release_year,actor.actor_id,actor.first_name,actor.last_name from film inner join film_actor using(film_id) inner join actor using(actor_id);
   ```
   
   ![](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200822235358.png)

##### 3.2.2.6 排序优化

排序的算法

- 两次传输算法

  第一次数据读取是将需要排序的字段读取出来，然后进行排序，第二次是将排好序的结果按照需要去读取数据行。

  这种方式效率比较低，原因是读取数据的时候因为已经排好序，需要去读取所有记录而此时更多是随机IO，读取数据成本会比较高。

  两次传输的优势，在顺序的时候存储尽可能少的数据，让排序缓冲区可以尽可能多的容纳行数来进行排序操作

- 单次传输算法

  先读取查询所需要的的所有列，然后再根据定列进行排序，最后直接返回排序结果，此方式只需要一次顺序IO，问题在于查询的列特别多的时候，会占用大量的存储空间，无法存储大量的数据

当需要排序的列的总大小超过`max_length_for_sort_data`定义的字节，MySQL会选择双次排序，反之使用单次排序，用户可以设置此参数值来选择排序的方式。

```sql
mysql> show variables like '%max_length_for_sort_data%';
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| max_length_for_sort_data | 1024  |
+--------------------------+-------+
1 row in set, 10 warnings (0.00 sec)
```

## 4 优化特定类型的查询

### 4.1 优化count()查询

- count(*)
- count(rental_id)
- count(1)

```sql
mysql> select count(*) from rental;
+----------+
| count(*) |
+----------+
|    16044 |
+----------+
1 row in set (0.04 sec)

mysql> select count(rental_id) from rental;
+------------------+
| count(rental_id) |
+------------------+
|            16044 |
+------------------+
1 row in set (0.01 sec)

mysql> select count(1) from rental;
+----------+
| count(1) |
+----------+
|    16044 |
+----------+
1 row in set (0.00 sec)

mysql> explain select count(*) from rental;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                        |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | Select tables optimized away |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
1 row in set, 1 warning (0.00 sec)

mysql> explain select count(rental_id) from rental;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                        |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | Select tables optimized away |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
1 row in set, 1 warning (0.00 sec)

mysql> explain select count(1) from rental;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                        |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | Select tables optimized away |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
1 row in set, 1 warning (0.00 sec)
```

#### 4.1.1 count(*)效率

**执行效果上**

- count(\*)：包括了所有的列，相当于行数，在统计结果的时候，不会忽略列值为NULL 
- count(1)：包括了忽略所有列，用1代表代码行，在统计结果的时候，不会忽略列值为NULL 
- count(列名)：只包括列名那一列，在统计结果的时候，会忽略列值为空（这里的空不是只空字符串或者0，而是表示null）的计数，即某个字段值为NULL时，不统计。

以上结果看来，**三者效率是一样的**。总有人认为myisam的count函数比较快，这是有前提条件的，只有没有任何where条件的count(*)才是比较快的。

#### 4.1.2 使用近似值

在某些应用场景中，不需要完全精确的值，可以参考使用近似值来代替，比如可以使用explain来获取近似的值
其实在很多OLAP的应用中，需要计算某一个列值的基数，有一个计算近似值的算法叫hyperloglog。

#### 4.1.3 更复杂的优化

一般情况下，count()需要扫描大量的行才能获取精确的数据，其实很难优化，在实际操作的时候可以考虑使用索引覆盖扫描，或者增加汇总表，或者增加外部缓存系统。

### 4.2 优化关联查询

#### 4.2.1 on、using使用索引

确保on或者using子句中的列上有索引，在创建索引的时候就要考虑到关联的顺序

当表A和表B使用列C关联的时候，如果优化器的关联顺序是B、A，那么就不需要再B表的对应列上建上索引，没有用到的索引只会带来额外的负担，一般情况下来说，只需要在关联顺序中的第二个表的相应列上创建索引

#### 4.2.2 group by、order by使用索引

确保任何的group by和order by中的表达式只涉及到一个表中的列，这样mysql才有可能使用索引来优化这个过程

###  4.3 优化子查询

子查询的优化最重要的优化建议是尽可能使用关联查询代替（不是完全代替）

原因：子查询会产生临时表

### 4.4 优化limit分页

在很多应用场景中我们需要将数据进行分页，一般会使用limit加上偏移量的方法实现，同时加上合适的order by 的子句，如果这种方式有索引的帮助，效率通常不错，否则的话需要进行大量的文件排序操作，还有一种情况，当偏移量非常大的时候，前面的大部分数据都会被抛弃，这样的代价太高。要优化这种查询的话，要么是在页面中限制分页的数量，要么优化大偏移量的性能

优化此类查询最简单的办法就是尽可能的使用覆盖索引，而不是查询所有的列。

```sql
mysql> select film_id,description from film order by title limit 50,5;
+---------+---------------------------------------------------------------------------------------------------------------------------------+
| film_id | description                                                                                                                     |
+---------+---------------------------------------------------------------------------------------------------------------------------------+
|      51 | A Insightful Panorama of a Forensic Psychologist And a Mad Cow who must Build a Mad Scientist in The First Manned Space Station |
|      52 | A Thrilling Documentary of a Composer And a Monkey who must Find a Feminist in California                                       |
|      53 | A Epic Drama of a Madman And a Cat who must Face a A Shark in An Abandoned Amusement Park                                       |
|      54 | A Awe-Inspiring Drama of a Car And a Pastry Chef who must Chase a Crocodile in The First Manned Space Station                   |
|      55 | A Awe-Inspiring Story of a Feminist And a Cat who must Conquer a Dog in A Monastery                                             |
+---------+---------------------------------------------------------------------------------------------------------------------------------+
5 rows in set (0.03 sec)

mysql> select film.film_id,film.description from film inner join (select film_id from film order by title limit 50,5) as lim using(film_id);
+---------+---------------------------------------------------------------------------------------------------------------------------------+
| film_id | description                                                                                                                     |
+---------+---------------------------------------------------------------------------------------------------------------------------------+
|      51 | A Insightful Panorama of a Forensic Psychologist And a Mad Cow who must Build a Mad Scientist in The First Manned Space Station |
|      52 | A Thrilling Documentary of a Composer And a Monkey who must Find a Feminist in California                                       |
|      53 | A Epic Drama of a Madman And a Cat who must Face a A Shark in An Abandoned Amusement Park                                       |
|      54 | A Awe-Inspiring Drama of a Car And a Pastry Chef who must Chase a Crocodile in The First Manned Space Station                   |
|      55 | A Awe-Inspiring Story of a Feminist And a Cat who must Conquer a Dog in A Monastery                                             |
+---------+---------------------------------------------------------------------------------------------------------------------------------+
5 rows in set (0.02 sec)

mysql> explain select film_id,description from film order by title limit 50,5;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra          |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
|  1 | SIMPLE      | film  | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 1000 |   100.00 | Using filesort |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
1 row in set, 1 warning (0.01 sec)

mysql> explain select film.film_id,film.description from film inner join (select film_id from film order by title limit 50,5) as lim using(film_id);
+----+-------------+------------+------------+--------+---------------+-----------+---------+-------------+------+----------+-------------+
| id | select_type | table      | partitions | type   | possible_keys | key       | key_len | ref         | rows | filtered | Extra       |
+----+-------------+------------+------------+--------+---------------+-----------+---------+-------------+------+----------+-------------+
|  1 | PRIMARY     | <derived2> | NULL       | ALL    | NULL          | NULL      | NULL    | NULL        |   55 |   100.00 | NULL        |
|  1 | PRIMARY     | film       | NULL       | eq_ref | PRIMARY       | PRIMARY   | 2       | lim.film_id |    1 |   100.00 | NULL        |
|  2 | DERIVED     | film       | NULL       | index  | NULL          | idx_title | 514     | NULL        |   55 |   100.00 | Using index |
+----+-------------+------------+------------+--------+---------------+-----------+---------+-------------+------+----------+-------------+
3 rows in set, 1 warning (0.00 sec)
```

### 4.5 优化union查询

mysql总是通过创建并填充临时表的方式来执行union查询，因此很多优化策略在union查询中都没法很好的使用。经常需要手工的将where、limit、order by等子句下推到各个子查询中，以便优化器可以充分利用这些条件进行优化。

除非确实需要服务器消除重复的行，否则一定要使用union all，因为没有all关键字，mysql会在查询的时候给临时表加上distinct的关键字，这个操作的代价很高

### 4.6 使用自定义变量

#### 4.6.1 自定义变量的使用

#### 4.6.2 自定义变量的限制

1. 无法使用查询缓存
2. 不能在使用常量或者标识符的地方使用自定义变量，例如表名、列名或者limit子句
3. 用户自定义变量的生命周期是在一个连接中有效，所以不能用它们来做连接间的通信
4. 不能显式地声明自定义变量的类型
5. MySQL优化器在某些场景可能会将这些变量优化掉，这可能导致代码不按预想的方式运行
6. 赋值符号：=的优先级非常低，所以在使用赋值表达式的时候应该明确的使用括号
7. 使用未定义的变量不会产生任何语法错误

#### 4.6.3 自定义变量的案例

##### 4.6.3.1 优化排名查询

1. 在给一个变量赋值的同时使用这个变量

   ```sql
   mysql> select actor_id, @rownum:=@rownum+1 as rownum from actor limit 10;
   +----------+--------+
   | actor_id | rownum |
   +----------+--------+
   |       58 |   NULL |
   |       92 |   NULL |
   |      182 |   NULL |
   |      118 |   NULL |
   |      145 |   NULL |
   |      194 |   NULL |
   |       76 |   NULL |
   |      112 |   NULL |
   |       67 |   NULL |
   |      190 |   NULL |
   +----------+--------+
   10 rows in set (0.01 sec)
   
   再进行set
   set @rownum:=0;
   ```

2. 查询获取演过最多电影的前十名演员，然后根据出演电影次数做一个排名

   ```sql
   mysql> set @actor_number:=0;
   Query OK, 0 rows affected (0.00 sec)
   
   mysql> select actor_id, cnt, @actor_number:=@actor_number+1 from (select actor_id, count(*) as cnt from film_actor group by actor_id order by cnt desc limit 10) t;
   +----------+-----+--------------------------------+
   | actor_id | cnt | @actor_number:=@actor_number+1 |
   +----------+-----+--------------------------------+
   |      107 |  42 |                              1 |
   |      102 |  41 |                              2 |
   |      198 |  40 |                              3 |
   |      181 |  39 |                              4 |
   |       23 |  37 |                              5 |
   |       81 |  36 |                              6 |
   |       37 |  35 |                              7 |
   |      106 |  35 |                              8 |
   |       60 |  35 |                              9 |
   |       13 |  35 |                             10 |
   +----------+-----+--------------------------------+
   10 rows in set (0.00 sec)
   ```

##### 4.6.3.2 避免查询刚刚更新的数据

当需要高效的更新一条记录的时间戳，同时希望查询当前记录中存放的时间戳是什么

```sql
mysql> create table t1(id int, t_date datetime);
Query OK, 0 rows affected (0.06 sec)

mysql> insert into t1 values(1, now());
Query OK, 1 row affected (0.02 sec)

mysql> select * from t1;
+------+---------------------+
| id   | t_date              |
+------+---------------------+
|    1 | 2020-08-20 20:33:10 |
+------+---------------------+
1 row in set (0.01 sec)
```

1. 重新查询时间

   ```sql
   update t1 set t_date = now() where id = 1;
   ```

   ![image-20200823091255244](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200823091255244.png)

2. 使用自定义变量存储时间，不用使用sql二次查询

   ```sql
   update t1 set t_date = now() where id = 1 and @now:=now();
   ```

   ![image-20200823091425693](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200823091425693.png)

##### 4.6.3.3 确定取值的顺序

在赋值和读取变量的时候可能是在查询的不同阶段

1. 因为where和select在查询的不同阶段执行，所以查询到两条记录，这不符合预期

   ```sql
   set @rownum = 0;
   select actor_id, @rownum := @rownum + 1 as cnt from actor where @rownum <= 1;
   ```

   ![image-20200823092308433](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200823092308433.png)

2. 尝试添加order by解决问题

   ```sql
   select actor_id, @rownum := @rownum + 1 as cnt from actor where @rownum <= 1 order by first_name;
   ```

   当引入了order by之后，发现打印出了全部结果，这是因为order by引入了文件排序，而where条件是在文件排序操作之前取值的  

   ![image-20200823092749056](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200823092749056.png)

3. 解决这个问题的关键在于让变量的赋值和取值发生在执行查询的同一阶段

   ```sql
   select actor_id, @rownum as cnt from actor where (@rownum := @rownum + 1) <= 1;
   ```

   ![image-20200823093145879](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200823093145879.png)

