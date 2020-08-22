[TOC]

# 索引优化

## 1 索引的基本知识

### 1.1 索引的优点

1. 大大减少了服务器需要扫描的数据量
2. 帮助服务器避免排序和临时表
3. 将随机io变成顺序io

### 1.2 索引的用处

1. 快速查找匹配WHERE子句的行
2. 从consideration中消除行，如果可以在索引之间进行选择，MySQL通常会使用找到最少行的索引
3. 如果表具有多列索引，则优化器可以使用索引的任何最左前缀查找行
4. 当有表连接的时候，则其他表索引行数据
5. 查找特定索引列的min和max值

### 1.3 索引的分类

### 1.4 索引的数据结构

### 1.5 索引匹配方式

![image-20200810224703468](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200810224703468.png)

#### 1.5.1 全值匹配

全值匹配指的是和索引中的所有的列进行匹配

```sql
explain select * from staffs where name = 'July' and age = '23' and pos = 'dev';
```

![image-20200810230322818](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200810230322818.png)

#### 1.5.2 匹配最左前缀

只匹配前面几个列

```sql
explain select * from staffs where name = 'July' and age = '23';
```

![image-20200810230617106](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200810230617106.png)

```sql
explain select * from staffs where name = 'July';
```

![image-20200810230638970](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200810230638970.png)

#### 1.5.3 匹配列前缀

可以匹配某一列的值的开头部分

```sql
explain select * from staffs where name like 'J%';
```

![image-20200810231037127](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200810231037127.png)

编写sql时一定不要将通配符放在前面，即使有索引也不会使用

![image-20200810231058996](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200810231058996.png)

#### 1.5.4 匹配范围值

```sql
explain select * from staffs where name > 'Mary';
```

![image-20200810231707357](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200810231707357.png)

#### 1.5.5 精确匹配某一列并范围匹配另外一列

可以查询第一列的全部和第二列的部分

```sql
explain select * from staffs where name = 'July' and age > 25;
```

![image-20200810232018094](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200810232018094.png)

```sql
explain select * from staffs where name = 'July' and pos > 25;
```

与上面对比，正确顺序是name，age，pos，此时用不到pos，只匹配name，所以pos > 25没有用，ref = constant。

![image-20200810232248497](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200810232248497.png)

#### 1.5.6 只访问索引的查询

查询的时候只需要访问索引，不需要访问数据行，本质上就是覆盖索引

```sql
explain select name,age,pos from staffs where name = 'July' and age = 25 and pos = 'dev';
```

![image-20200810233017183](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200810233017183.png)

### 1.6  面试

## 2 哈希索引

### 2.1 基本知识

1. 什么是哈希索引？

   基于哈希表的实现，只有精确匹配索引所有列的查询才有效。（不能使用范围查找）

2. 在MySQL中，只有memory的存储引擎显式支持哈希索引

3. 哈希索引自身只需要存储对应的hash值，所以索引的结构十分紧凑，这让哈希索引的查找的速度非常快

### 2.2 哈希索引的限制

1. 哈希索引只包含哈希值和行指针，而不存储字段值，索引不能使用索引中的值避免读取行

   执行顺序：哈希值 -> 行指针 -> 数据

2. 哈希索引数据并不是按照索引值顺序存储的，所以无法进行排序

3. 哈希索引不支持部分列匹配查找，哈希索引是使用索引列的全部内容来计算哈希值

4. 哈希值支持等值比较查询，不支持任何范围查询

5. 访问哈希索引的数据非常快，除非有很多哈希冲突，当出现哈希冲突时，存储引擎必须遍历链表中的所有行指针，逐行进行比较，只到找到所有符合条件的行

6. 哈希冲突比较多的时候，维护代价也会很高

### 2.3 哈希案例

当需要存储大量的URL，并且根据URL进行搜索查找，如果使用B+树，存储的内容就会很大

```sql
select id from url where url = '';
```

也可以利用将URL使用CRC32做哈希，可以使用以下查询方式：

```sql
select id from url where url = '' and url crc = CRC32('');
```

此查询性能较高原因是使用体积很小的索引来完成查找

## 3 组合索引

当包含多个列作为索引，需要注意的是正确的顺序依赖于该索引的查询，需要同时考虑如何更好的满足排序和分组的需要

### 3.1 组合索引案例

建立组合索引a, b, c

```sql
CREATE TABLE IF NOT EXISTS `test_index`(
  `id` int(4) NOT NULL AUTO_INCREMENT,
  `a` int(4) NOT NULL DEFAULT '0',
  `b` int(4) NOT NULL DEFAULT '0',
  `c` int(4) NOT NULL DEFAULT '0',
  `data` int(4) NOT NULL DEFAULT '0',
  PRIMARY KEY  (`id`),
  KEY `union_index` (`a`,`b`,`c`)
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC  DEFAULT CHARSET=binary;
```

**如果b是范围查找，不管c是不是索引列，都会忽略掉，不会参与运算**

1. 使用列a，`type:ref`表示引用查找，`key_len:4`表示索引长度为4

   ```sql
   explain select data from test_index where a = 1;
   ```

   ![image-20200811205124184](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200811205124184.png)

2. 使用列b，`type:ALL`表示全表查找，`key_len:NULL`表示没有索引，不能使用索引来加快查找速度

   ```sql
   explain select data from test_index where b = 1;
   ```

   ![image-20200811205152963](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200811205152963.png)

3. 使用列c，`type:ALL`表示全表查找，`key_len:NULL`表示没有索引，不能使用索引来加快查找速度

   ```sql
   explain select data from test_index where c = 1;
   ```

   ![image-20200811205211418](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200811205211418.png)

4. 使用列a和b，`type:ref`表示引用查找，`key_len:8`表示索引长度为8，`ref:const,const`，利用了a、b联合索引进行查找

   ```sql
   explain select data from test_index where a = 1 and b = 1;
   ```

   ![image-20200811205239715](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200811205239715.png)

   颠倒a和b顺序，使用列b和a，结果同上

   ```sql
   explain select data from test_index where b = 1 and a = 1;
   ```

   ![image-20200811205258860](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200811205258860.png)

5. 使用列a、b和c，`type:ref`表示引用查找，`key_len:12`表示索引长度为12，`ref:const,const,const`，利用了a、b、c联合索引进行查找

   ```sql
   explain select data from test_index where a = 1 and b = 1 and c = 1;
   ```

   ![image-20200811205321252](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200811205321252.png)

6. 使用列a和c，`type:ref`表示引用查找，`key_len:4`表示索引长度为4，`ref:const`，只使用了a索引进行查找

   ```sql
   explain select data from test_index where a = 1 and c = 1;
   ```

   ![image-20200811205337870](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200811205337870.png)

7. 使用列b和c，`type:ALL`表示全表查找，`key_len:NULL`表示没有索引，不能使用索引来加快查找速度

   ```sql
   explain select data from test_index where b = 1 and c = 1;
   ```

   ![image-20200811205351955](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200811205351955.png)

8. a=1 and b > 10 and c = 1，`type:range`表示范围查找，`key_len:8`表示索引长度为8，`ref:null`，利用了a、b联合索引进行查找

   ```sql
   explain select data from test_index where a = 1 and b > 1 and c = 1;
   ```

   ![image-20200811205410350](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200811205410350.png)

9. a=1 and b > 10 and c = 1，`type:ref`表示引用查找，`key_len:4`表示索引长度为4，`ref:const`，只使用了a索引进行查找

   ```sql
   explain select data from test_index where a = 1 and b like '%1' and c = 1;
   ```

   ![image-20200811205429030](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200811205429030.png)

## 4 聚簇索引与非聚簇索引

### 4.1 聚簇索引

不是单独的索引类型，而是一种数据存储方式，指的是数据行跟相邻的键值紧凑的存储在一起

#### 4.1.1 优点

1. 可以把相关数据保存在一起
2. 数据访问更快，因为索引和数据保存在同一个树中
3. 使用覆盖索引扫描的查询可以直接使用页节点中的主键值

#### 4.1.2 缺点

1. 
2. 插入速度严重依赖于插入顺序，按照主键的顺序插入是最快的方式（页分裂、页合并）
3. 更新聚簇索引列的代价很高，因为会强制将每个被更新的行移动到新的位置
4. 基于聚簇索引的表插入新行，或者主键被更新导致需要移动行的时候，可能面临也分裂的问题
5. 聚簇索引可能导致全表扫描变慢，尤其是行比较稀疏，或者由于页分裂导致数据存储不连续的时候

### 4.2 非聚簇索引

数据文件和索引文件分开存放

## 5 覆盖索引

### 5.1 基本介绍

1. 如果一个索引包含所有需要查询的字段，称之为覆盖索引
2. 不是所有类型的索引都可以称为覆盖索引，覆盖索引必须要存储索引列的值
3. 不同的存储实现覆盖索引的方式不同，不是所有的引擎都支持覆盖索引，memory不支持覆盖索引



## 6  优化细节

### 6.1 查询不使用表达式

当使用索引列进行查询的时候尽量不要使用表达式，把计算放到业务层而不是数据库层

不使用表达式

![image-20200811220529545](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200811220529545.png)

使用表达式

![image-20200811220609835](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200811220609835.png)

### 6.2 使用主键查询

尽量使用主键查询，而不是其他索引，因为主键查询不会触发回表查询

### 6.3 使用前缀索引

有时候需要索引很长的字符串，这会让索引变的大且慢，通常情况下可以使用某个列开始的部分字符串，这样大大的节约索引空间，从而提高索引效率，但这会降低索引的选择性，索引的选择性是指不重复的索引值和数据表记录总数的比值，范围从1/#T到1之间。索引的选择性越高则查询效率越高，因为选择性更高的索引可以让mysql在查找的时候过滤掉更多的行。

一般情况下某个列前缀的选择性也是足够高的，足以满足查询的性能，但是对应BLOB，TEXT，VARCHAR类型的列，必须要使用前缀索引，因为mysql不允许索引这些列的完整长度，使用该方法的诀窍在于要选择足够长的前缀以保证较高的选择性，通过又不能太长。

#### 6.3.1 前缀索引案例演示

MySQL官方数据库sakila

1. 创建数据库表

   ```sql
   create table citydemo(city varchar(50) not null);
   insert into citydemo(city) select city from city;
   ```

2. 重复执行5次下面的sql语句

   ```sql
   insert into citydemo(city) select city from citydemo;
   ```

3. 更新城市表的名称

   ```sql
   update citydemo set city = (select city from city order by rand() limit 1);
   ```

4. 查找最常见的城市列表，每个值重复40-60次

   ```sql
   mysql> select count(*) as cnt, city from citydemo group by city order by cnt desc limit 10;
   +-----+-------------------------+
   | cnt | city                    |
   +-----+-------------------------+
   |  60 | Allappuzha (Alleppey)   |
   |  52 | Berhampore (Baharampur) |
   |  48 | Molodetno               |
   |  48 | London                  |
   |  48 | Balurghat               |
   |  47 | Cape Coral              |
   |  47 | Basel                   |
   |  47 | Kowloon and New Kowloon |
   |  47 | Kirovo-Tepetsk          |
   |  46 | Tianjin                 |
   +-----+-------------------------+
   10 rows in set (0.03 sec)
   ```

5. 查找最频繁出现的城市前缀

   - 3个前缀字母

     ```sql
     mysql> select count(*) as cnt, left(city, 3) as pref from citydemo group by pref order by cnt desc limit 10;
     +-----+------+
     | cnt | pref |
     +-----+------+
     | 444 | San  |
     | 191 | Cha  |
     | 186 | al-  |
     | 159 | Tan  |
     | 145 | Sha  |
     | 140 | Sal  |
     | 138 | Sou  |
     | 128 | Hal  |
     | 128 | Shi  |
     | 124 | Bat  |
     +-----+------+
     10 rows in set (0.01 sec)
     ```

   - 4个前缀字母

     ```sql
     mysql> select count(*) as cnt, left(city, 4) as pref from citydemo group by pref order by cnt desc limit 10;
     +-----+------+
     | cnt | pref |
     +-----+------+
     | 207 | Sant |
     | 175 | San  |
     | 107 | Sout |
     | 105 | Chan |
     |  92 | Toul |
     |  81 | al-Q |
     |  71 | Hami |
     |  69 | Vall |
     |  67 | La P |
     |  67 | Sain |
     +-----+------+
     10 rows in set (0.01 sec)
     ```

   - 5个前缀字母

     ```sq
     mysql> select count(*) as cnt, left(city, 5) as pref from citydemo group by pref order by cnt desc limit 10;
     +-----+-------+
     | cnt | pref  |
     +-----+-------+
     | 123 | Santa |
     | 107 | South |
     |  81 | al-Qa |
     |  71 | Chang |
     |  69 | Valle |
     |  67 | Saint |
     |  64 | Toulo |
     |  60 | Allap |
     |  59 | San F |
     |  59 | Shimo |
     +-----+-------+
     10 rows in set (0.02 sec)
     ```

   - 6个前缀字母

     ```sql
     mysql> select count(*) as cnt, left(city, 6) as pref from citydemo group by pref order by cnt desc limit 10;
     +-----+--------+
     | cnt | pref   |
     +-----+--------+
     | 123 | Santa  |
     |  69 | Valle  |
     |  60 | Allapp |
     |  59 | San Fe |
     |  57 | Santia |
     |  52 | Berham |
     |  48 | Balurg |
     |  48 | Molode |
     |  48 | London |
     |  47 | Kowloo |
     +-----+--------+
     10 rows in set (0.02 sec)
     ```

   - 7个前缀字母

     ```sql
     mysql> select count(*) as cnt, left(city, 7) as pref from citydemo group by pref order by cnt desc limit 10;
     +-----+---------+
     | cnt | pref    |
     +-----+---------+
     |  69 | Valle d |
     |  60 | Allappu |
     |  59 | San Fel |
     |  57 | Santiag |
     |  52 | Berhamp |
     |  48 | London  |
     |  48 | Balurgh |
     |  48 | Molodet |
     |  47 | Kirovo- |
     |  47 | Kowloon |
     +-----+---------+
     10 rows in set (0.02 sec)
     ```

   - 8个前缀字母

     ```sql
     mysql> select count(*) as cnt, left(city, 8) as pref from citydemo group by pref order by cnt desc limit 10;
     +-----+----------+
     | cnt | pref     |
     +-----+----------+
     |  69 | Valle de |
     |  60 | Allappuz |
     |  59 | San Feli |
     |  57 | Santiago |
     |  52 | Berhampo |
     |  48 | Molodetn |
     |  48 | London   |
     |  48 | Balurgha |
     |  47 | Cape Cor |
     |  47 | Kowloon  |
     +-----+----------+
     10 rows in set (0.02 sec)
     ```

     综上，7个前缀字母的选择性接近于完整列的选择性

6. 计算完成之后可以创建前缀索引

   ```sql
   alter table citydemo add key(city(7));
   
   mysql> show index from citydemo;
   +----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
   | Table    | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
   +----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
   | citydemo |          1 | city     |            1 | city        | A         |         596 |        7 | NULL   |      | BTREE      |         |               |
   +----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
   1 row in set (0.00 sec)
   ```

   里面的Cardinality含义：基数

   经常使用HyperLogLog算法

7. 还可以通过另外一种方式计算完整列的选择性，可以看到当前缀长度到达7之后，再增加前缀长度，选择性提升的幅度已经很小了

   ```sql
   mysql> select count(distinct left(city,3))/count(*) as sel3,
       -> count(distinct left(city,4))/count(*) as sel4,
       -> count(distinct left(city,5))/count(*) as sel5,
       -> count(distinct left(city,6))/count(*) as sel6,
       -> count(distinct left(city,7))/count(*) as sel7,
       -> count(distinct left(city,8))/count(*) as sel8
       -> from citydemo;
   +--------+--------+--------+--------+--------+--------+
   | sel3   | sel4   | sel5   | sel6   | sel7   | sel8   |
   +--------+--------+--------+--------+--------+--------+
   | 0.0239 | 0.0293 | 0.0305 | 0.0309 | 0.0310 | 0.0310 |
   +--------+--------+--------+--------+--------+--------+
   1 row in set (0.06 sec)
   ```

### 6.4 利用索引扫描来排序

MySQL有两种方式可以生成有序的结果

- 排序操作
- 按索引顺序扫描

如果explain出来的type列的值为index，则说明了使用了索引扫描来做排序

扫描索引本身是很快的，因为只需要从一条索引记录移动到紧接着下一条记录。但如果索引不能覆盖查询所需的全部列，那么就不得不每扫描一条索引记录就得回表查询一次对应的行，这基本都是随机IO，因此按照索引顺序读取数据的速度通常要比顺序地全表扫描慢。

mysql可以使用同一个索引即满足排序，又用于查找行，如果可能的话，设计索引时应该尽可能地同时满足这两种任务。

只有当索引的列顺序和order by子句的顺序完全一致，并且所有列的排序方式都一样时，mysql才能够使用索引来对结果进行排序，如果查询需要关联多张表，则只有当order by子句引用的字段全部为第一张表时，才能使用索引做排序。order by子句和查找型查询的限制是一样的，需要满足索引的最左前缀的要求，否则，mysql都需要执行顺序操作，而无法利用索引排序。

#### 6.4.1 索引扫描案例

sakila数据库中rental表在rental_date, inventory_id, customer_id上有rental_date的索引（UNIQUE INDEX `rental_date` (`rental_date`, `inventory_id`, `customer_id`),）

rental表建表sql

```sql
CREATE TABLE `rental` (
	`rental_id` INT(11) NOT NULL AUTO_INCREMENT,
	`rental_date` DATETIME NOT NULL,
	`inventory_id` MEDIUMINT(8) UNSIGNED NOT NULL,
	`customer_id` SMALLINT(5) UNSIGNED NOT NULL,
	`return_date` DATETIME NULL DEFAULT NULL,
	`staff_id` TINYINT(3) UNSIGNED NOT NULL,
	`last_update` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
	PRIMARY KEY (`rental_id`),
	UNIQUE INDEX `rental_date` (`rental_date`, `inventory_id`, `customer_id`),
	INDEX `idx_fk_inventory_id` (`inventory_id`),
	INDEX `idx_fk_customer_id` (`customer_id`),
	INDEX `idx_fk_staff_id` (`staff_id`),
	CONSTRAINT `fk_rental_customer` FOREIGN KEY (`customer_id`) REFERENCES `customer` (`customer_id`) ON UPDATE CASCADE,
	CONSTRAINT `fk_rental_inventory` FOREIGN KEY (`inventory_id`) REFERENCES `inventory` (`inventory_id`) ON UPDATE CASCADE,
	CONSTRAINT `fk_rental_staff` FOREIGN KEY (`staff_id`) REFERENCES `staff` (`staff_id`) ON UPDATE CASCADE
)
COLLATE='utf8mb4_general_ci'
ENGINE=InnoDB
AUTO_INCREMENT=16050
;
```

1. 使用rental_date为下面的查询做排序

   ```sql
   mysql> explain select rental_id,staff_id from rental where rental_date = '2005-05-25' order by inventory_id, customer_id\G
   *************************** 1. row ***************************
              id: 1
     select_type: SIMPLE
           table: rental
      partitions: NULL
            type: ref
   possible_keys: rental_date
             key: rental_date
         key_len: 5
             ref: const
            rows: 1
        filtered: 100.00
           Extra: Using index condition
   1 row in set, 1 warning (0.02 sec)
   ```

   Extra: Using index condition，使用了index条件。rental_date是一个常量，order by是inventory_id和customer_id，不会用到索引，不能排序，

2. 该查询为索引的第一列提供了常量条件，而使用第二列进行排序，将两个列组合在一起，就形成了索引的最左前缀

   ```sql
   mysql> explain select rental_id,staff_id from rental where rental_date='2005-05-25' order by inventory_id desc\G
   *************************** 1. row ***************************
              id: 1
     select_type: SIMPLE
           table: rental
      partitions: NULL
            type: ref
   possible_keys: rental_date
             key: rental_date
         key_len: 5
             ref: const
            rows: 1
        filtered: 100.00
           Extra: Using where
   1 row in set, 1 warning (0.00 sec)
   ```

3. 没用到索引

   ```sql
   mysql> explain select rental_id,staff_id from rental where rental_date>'2005-05-25' order by rental_date,inventory_id\G
   *************************** 1. row ***************************
              id: 1
     select_type: SIMPLE
           table: rental
      partitions: NULL
            type: ALL
   possible_keys: rental_date
             key: NULL
         key_len: NULL
             ref: NULL
            rows: 16005
        filtered: 50.00
           Extra: Using where; Using filesort
   1 row in set, 1 warning (0.01 sec)
   ```

4. 该查询使用了两中不同的排序方向，但是索引列都是正序排序的

   - inventory_id desc, customer_id asc 不能使用
   - inventory_id asc, customer_id asc 能使用
   - inventory_id desc, customer_id desc 能使用

   ```sql
   mysql> explain select rental_id,staff_id from rental where rental_date>'2005-05-25' order by inventory_id desc,customer_id asc\G
   *************************** 1. row ***************************
              id: 1
     select_type: SIMPLE
           table: rental
      partitions: NULL
            type: ALL
   possible_keys: rental_date
             key: NULL
         key_len: NULL
             ref: NULL
            rows: 16005
        filtered: 50.00
           Extra: Using where; Using filesort
   1 row in set, 1 warning (0.00 sec)
   ```

5. 该查询中引用了一个不再索引中的列

   ```sql
   mysql> explain select rental_id,staff_id from rental where rental_date > '2005-05-25' order by inventory_id, staff_id\G
   *************************** 1. row ***************************
              id: 1
     select_type: SIMPLE
           table: rental
      partitions: NULL
            type: ALL
   possible_keys: rental_date
             key: NULL
         key_len: NULL
             ref: NULL
            rows: 16005
        filtered: 50.00
           Extra: Using where; Using filesort
   1 row in set, 1 warning (0.00 sec)
   ```

综上应用场景

1. where条件和order by列能组成最左前缀匹配，就能使用索引排序，第一列是范围无法使用（例3）
2. 如果order by和索引升序、降序不一致，无法使用索引排序（例4）

### 6.5 union all、in、or 可以使用索引

union all,in,or都能够使用索引，但是推荐使用in

1. union

   ```sql
   mysql> explain select * from actor where actor_id = 1 union all select * from actor where actor_id = 2\G;
   *************************** 1. row ***************************
              id: 1
     select_type: PRIMARY
           table: actor
      partitions: NULL
            type: const
   possible_keys: PRIMARY
             key: PRIMARY
         key_len: 2
             ref: const
            rows: 1
        filtered: 100.00
           Extra: NULL
   *************************** 2. row ***************************
              id: 2
     select_type: UNION
           table: actor
      partitions: NULL
            type: const
   possible_keys: PRIMARY
             key: PRIMARY
         key_len: 2
             ref: const
            rows: 1
        filtered: 100.00
           Extra: NULL
   2 rows in set, 1 warning (0.00 sec)
   
   ERROR:
   No query specified
   ```

2. in

   ```sql
   mysql> explain select * from actor where actor_id in (1, 2)\G;
   *************************** 1. row ***************************
              id: 1
     select_type: SIMPLE
           table: actor
      partitions: NULL
            type: range
   possible_keys: PRIMARY
             key: PRIMARY
         key_len: 2
             ref: NULL
            rows: 2
        filtered: 100.00
           Extra: Using where
   1 row in set, 1 warning (0.00 sec)
   
   ERROR:
   No query specified
   ```

3. or

   ```sql
   mysql> explain select * from actor where actor_id = 1 or actor_id = 2\G;
   *************************** 1. row ***************************
              id: 1
     select_type: SIMPLE
           table: actor
      partitions: NULL
            type: range
   possible_keys: PRIMARY
             key: PRIMARY
         key_len: 2
             ref: NULL
            rows: 2
        filtered: 100.00
           Extra: Using where
   1 row in set, 1 warning (0.00 sec)
   
   ERROR:
   No query specified
   ```

### 6.6 范围列可以用到索引

范围条件是：<、<=、>、>=、between

范围列可以用到索引，但是范围列后面的列无法用到索引，索引最多用于一个范围列

### 6.7 强制类型转换会全表扫描

```sql
CREATE TABLE `user` (
	`id` INT(11) NULL DEFAULT NULL,
	`name` VARCHAR(10) NULL DEFAULT NULL,
	`phone` VARCHAR(11) NULL DEFAULT NULL,
	INDEX `idx_1` (`phone`)
)
COLLATE='utf8mb4_general_ci'
ENGINE=InnoDB
;
```

不发生强制类型转换，触发索引

```sql
mysql> explain select * from user where phone= '13800001234'\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ref
possible_keys: idx_1
          key: idx_1
      key_len: 47
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)

ERROR:
No query specified
```

发生强制类型转换，不触发索引

```sql
mysql> explain select * from user where phone = 13800001234\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ALL
possible_keys: idx_1
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using where
1 row in set, 3 warnings (0.00 sec)

ERROR:
No query specified
```

### 6.8 更新频繁，数据区分度不高的字段上不宜建立索引

更新十分频繁，数据区分度不高的字段上不宜建立索引

1. 更新会变更B+树，更新频繁的字段建议索引会大大降低数据库性能
2. 类似于性别这类区分不大的属性，建立索引是没有意义的，不能有效的过滤数据
3. 一般区分度在80%以上的时候就可以建立索引，区分度可以使用 count(distinct(列名))/count(*) 来计算

### 6.9 索引列不允许为null

创建索引的列，不允许为null，可能会得到不符合预期的结果

### 6.10 join

当需要进行表连接的时候，最好不要超过三张表，因为需要join的字段，数据类型必须一致

官网算法：https://dev.mysql.com/doc/refman/8.0/en/nested-loop-joins.html

```sql
mysql> show variables like '%join_buffer%';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| join_buffer_size | 262144 |
+------------------+--------+
1 row in set, 10 warnings (0.00 sec)
```

### 6.11 尽量使用limit

能使用limit的时候尽量使用limit

如果能明确知道只有一条返回结果，limit能够提高效率

```sql
mysql> select ename from emp where empno = 7369;
mysql> select ename from emp where empno = 7369 limit 1;

mysql> show profiles;
+----------+------------+--------------------------------------------------+
| Query_ID | Duration   | Query                                            |
+----------+------------+--------------------------------------------------+
|        1 | 0.00025300 | select ename from emp where empno = 7369         |
|        2 | 0.00020500 | select ename from emp where empno = 7369 limit 1 |
+----------+------------+--------------------------------------------------+
```

limit时间明显比较短

### 6.12 单表索引数量

单表索引建议控制在5个以内，合理使用索引

### 6.13 单索引字段数

单索引字段数不允许超过5个（组合索引）

### 6.14 创建索引时注意事项

创建索引的时候应该避免以下错误概念

1. 索引越多越少

2. 过早优化，再不了解系统的情况下进行优化

   面试举例（说场景）：在公司遇到一个查询速度比较慢的sql，通过执行计划查看type类使用的or，调整之后变为contract

## 7 索引监控

官网地址：https://dev.mysql.com/doc/refman/8.0/en/index-extensions.html

索引监控参数

```sql
mysql> show status like 'Handler_read%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| Handler_read_first    | 0     |
| Handler_read_key      | 0     |
| Handler_read_last     | 0     |
| Handler_read_next     | 0     |
| Handler_read_prev     | 0     |
| Handler_read_rnd      | 0     |
| Handler_read_rnd_next | 0     |
+-----------------------+-------+
7 rows in set (0.02 sec)
```

![image-20200815084809626](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200815084809626.png)

### 7.1 参数解释

官网地址：https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_optimizer_switch

1. Handler_read_first

   读取索引第一个条目的次数

2. Handler_read_key

   通过index获取数据的次数

3. Handler_read_last

   读取索引最后一个条目的次数

4. Handler_read_next

   通过索引读取下一条数据的次数

5. Handler_read_prev

   通过索引读取上一条数据的次数

6. Handler_read_rnd

   从固定位置读取数据的次数

7. Handler_read_rnd_next

   从数据节点读取下一条数据的次数

**主要看Handler_read_key和Handler_read_rnd_next，如果Handler_read_key使用比较少证明索引建立的不够好。**

## 8 索引优化案例

建表

```sql
SET FOREIGN_KEY_CHECKS=0;
DROP TABLE IF EXISTS `itdragon_order_list`;
CREATE TABLE `itdragon_order_list` (
  `id` bigint(11) NOT NULL AUTO_INCREMENT COMMENT '主键id，默认自增长',
  `transaction_id` varchar(150) DEFAULT NULL COMMENT '交易号',
  `gross` double DEFAULT NULL COMMENT '毛收入(RMB)',
  `net` double DEFAULT NULL COMMENT '净收入(RMB)',
  `stock_id` int(11) DEFAULT NULL COMMENT '发货仓库',
  `order_status` int(11) DEFAULT NULL COMMENT '订单状态',
  `descript` varchar(255) DEFAULT NULL COMMENT '客服备注',
  `finance_descript` varchar(255) DEFAULT NULL COMMENT '财务备注',
  `create_type` varchar(100) DEFAULT NULL COMMENT '创建类型',
  `order_level` int(11) DEFAULT NULL COMMENT '订单级别',
  `input_user` varchar(20) DEFAULT NULL COMMENT '录入人',
  `input_date` varchar(20) DEFAULT NULL COMMENT '录入时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=10003 DEFAULT CHARSET=utf8;

INSERT INTO itdragon_order_list VALUES ('10000', '81X97310V32236260E', '6.6', '6.13', '1', '10', 'ok', 'ok', 'auto', '1', 'itdragon', '2017-08-28 17:01:49');
INSERT INTO itdragon_order_list VALUES ('10001', '61525478BB371361Q', '18.88', '18.79', '1', '10', 'ok', 'ok', 'auto', '1', 'itdragon', '2017-08-18 17:01:50');
INSERT INTO itdragon_order_list VALUES ('10002', '5RT64180WE555861V', '20.18', '20.17', '1', '10', 'ok', 'ok', 'auto', '1', 'itdragon', '2017-09-08 17:01:49');
```

逐步开始进行优化

### 8.1 案例一

```sql
select * from itdragon_order_list where transaction_id = '81X97310V32236260E';
```

通过查看执行计划发现type = all，需要进行全表扫描

![image-20200815091802219](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200815091802219.png)



**优化一：为transaction_id创建唯一索引**

```sql
create unique index idx_order_txid on itdragon_order_list (transaction_id);
```

![image-20200815092357354](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200815092357354.png)

当创建索引之后，唯一索引对应的type是const，通过索引一次就可以找到结果，普通索引对应的type是ref，表示非唯一性索引，找到值还要进行扫描，直到将索引文件扫描完为止，显而易见，const的性能要高于ref。

![image-20200815092519662](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200815092519662.png)

**优化二：使用覆盖索引**

查询的结果变成transaction_id，当extra出现using index，表示使用了覆盖索引

![image-20200815092917120](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200815092917120.png)

### 8.2 案例二

创建组合索引

```sql
create index idx_order_level_date on itdragon_order_list (order_level, input_date);
```

![image-20200815093246541](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200815093246541.png)

创建索引之后发现跟没有创建索引一样，都是全表扫描，都是文件排序

```sql
explain select * from itdragon_order_list order by order_level, input_date;
```

![image-20200815093625497](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200815093625497.png)

可以使用force index强制指定索引

```sql
explain select * from itdragon_order_list force index(idx_order_level_date) order by order_level, input_date;
```

![image-20200815093644266](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200815093644266.png)

其实给订单排序意义不大，给订单级别添加索引意义也不大，因此可以先确定order_level的值，然后再给input_date排序

```sql
explain select * from itdragon_order_list where order_level = 3 order by input_date;
```

![image-20200815093701781](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200815093701781.png)