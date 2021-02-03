# MySQL乐观锁和悲观锁

## 1 悲观锁

### 1.1 定义

悲观锁，正如其名，具有强烈的独占和排他特性。它指的是对数据被外界（包括本系统当前的其他事务，以及来自外部系统的事务处理）修改持保守态度，因此，在整个数据处理过程中，将数据处于锁定状态。悲观锁的实现，往往依靠数据库提供的锁机制（也只有数据库层提供的锁机制才能真正保证数据访问的排他性，否则，即使在本系统中实现了加锁机制，也无法保证外部系统不会修改数据）。

#### 1.1.1 悲观锁分类

- 共享锁（Pessimistic Lock）

  共享锁又称为读锁，简称S锁，顾名思义，共享锁就是多个事务对于同一数据可以共享一把锁，都能访问到数据，但是只能读不能修改。

- 排它锁（Optimistic Lock）

  排他锁又称为写锁，简称X锁，顾名思义，排他锁就是不能与其他所并存，如一个事务获取了一个数据行的排他锁，其他事务就不能再获取该行的其他锁，包括共享锁和排他锁，但是获取排他锁的事务是可以对数据就行读取和修改。

问题

并发数据一致性问题

### 1.2 案例

表结构和数据

```sql
CREATE TABLE `book` (
	`id` INT(10) NOT NULL,
	`book_name` VARCHAR(10) NULL DEFAULT NULL,
	`price` DOUBLE NULL DEFAULT NULL,
	PRIMARY KEY (`id`)
)
COLLATE='utf8mb4_general_ci'
ENGINE=InnoDB
;
INSERT INTO `book` (`id`, `book_name`, `price`) VALUES (1, '西游记', 100);
INSERT INTO `book` (`id`, `book_name`, `price`) VALUES (2, '水浒传', 100);
INSERT INTO `book` (`id`, `book_name`, `price`) VALUES (3, '三国演义', 100);
INSERT INTO `book` (`id`, `book_name`, `price`) VALUES (4, '红楼梦', 100);

mysql> select * from book;
+----+-----------+-------+
| id | book_name | price |
+----+-----------+-------+
|  1 | 西游记    |   100 |
|  2 | 水浒传    |   100 |
|  3 | 三国演义  |   100 |
|  4 | 红楼梦    |   100 |
+----+-----------+-------+
```

#### 1.2.1 排它锁（select for update;）

1. 首先关闭数据库自动提交

   ```sql
   set autocommit=0
   ```

2. session1开启事务，session2开始事务

   ```sql
   start transaction;
   ```

3. session1获取西游记的时候对该行加锁，期间session2用户阻塞

   ```sql
   mysql> select book_name, price from book where id = 1 for update;
   +-----------+-------+
   | book_name | price |
   +-----------+-------+
   | 西游记    |   100 |
   +-----------+-------+
   1 row in set (0.00 sec)
   
   # session2
   mysql> select book_name, price from book where id = 1 for update;
   -
   ```

4. session1提交，session2查询到数据

   ```sql
   # session1
   mysql> commit;
   Query OK, 0 rows affected (0.01 sec)
   
   # session2
   mysql> select book_name, price from book where id = 1 for update;
   +-----------+-------+
   | book_name | price |
   +-----------+-------+
   | 西游记    |   100 |
   +-----------+-------+
   1 row in set (19.80 sec)
   ```

综上

| 时间轴 |                            事务A                             |                            事务B                             |
| :----: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|   T1   |                      start transaction;                      |                                                              |
|   T2   | select book_name, price from book where id = 1 for update;（得到数据） |                                                              |
|   T3   |                                                              |                      start transaction;                      |
|   T4   |                                                              | select book_name, price from book where id = 1 for update;（waiting……） |
|   T5   |                           commit;                            |                           得到数据                           |
|   T6   |                                                              |                           commit;                            |

#### 1.2.2 共享锁案例（lock in share mode;）

1. 首先关闭数据库自动提交

   ```sql
   set autocommit=0
   ```

2. session1开启事务，session2开始事务

   ```sql
   start transaction;
   ```

3. session1获取西游记的时候对该行加锁，期间session2可以进行读取，不可以进行修改

   ```sql
   # session1
   mysql> select book_name, price from book where id = 1 lock in share mode;
   +-----------+-------+
   | book_name | price |
   +-----------+-------+
   | 西游记    |   100 |
   +-----------+-------+
   1 row in set (0.00 sec)
   
   # session2，修改数据阻塞
   mysql> select book_name, price from book where id = 1 lock in share mode;
   +-----------+-------+
   | book_name | price |
   +-----------+-------+
   | 西游记    |   100 |
   +-----------+-------+
   1 row in set (0.00 sec)
   mysql> update book set price = 101 where id = 1;
   -
   ```

4. session1提交，session2修改数据成功

   ```sql
   # session1
   mysql> commit;
   Query OK, 0 rows affected (0.01 sec)
   
   # session2
   mysql> update book set price = 101 where id = 1;
   Query OK, 1 row affected (25.99 sec)
   Rows matched: 1  Changed: 1  Warnings: 0
   ```

综上

| 时间轴 |                            事务A                             |                            事务B                             |
| :----: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|   T1   |                      start transaction;                      |                                                              |
|   T2   | select book_name, price from book where id = 1 lock in share mode;（得到数据） |                                                              |
|   T3   |                                                              |                      start transaction;                      |
|   T4   |                                                              | select book_name, price from book where id = 1 lock in share mode;（得到数据） |
|   T5   |                                                              |     update book set price = 101 where id = 1;（阻塞……）      |
|   T6   |                           commit;                            |                                                              |
|   T7   |                                                              |                           修改成功                           |
|   T8   |                                                              |                           commit;                            |

## 2 乐观锁

乐观锁机制采取了更加宽松的加锁机制。乐观锁是相对悲观锁而言，也是为了避免数据库幻读、业务处理时间过长等原因引起数据处理错误的一种机制，但乐观锁不会刻意使用数据库本身的锁机制，而是依据数据本身来保证数据的正确性。

### 2.1 乐观锁实现方式

- 使用版本号

  使用数据版本（Version）记录机制实现，这是乐观锁最常用的一种实现方式。何谓数据版本？即为数据增加一个版本标识，一般是通过为数据库表增加一个数字类型的 “version” 字段来实现。当读取数据时，将version字段的值一同读出，数据每更新一次，对此version值加一。当我们提交更新的时候，判断数据库表对应记录的当前版本信息与第一次取出来的version值进行比对，如果数据库表当前版本号与第一次取出来的version值相等，则予以更新，否则认为是过期数据。

- 使用时间戳

  乐观锁定的第二种实现方式和第一种差不多，同样是在需要乐观锁控制的table中增加一个字段，名称无所谓，字段类型使用时间戳（timestamp）, 和上面的version类似，也是在更新提交的时候检查当前数据库中数据的时间戳和自己更新前取到的时间戳进行对比，如果一致则OK，否则就是版本冲突。

### 2.2 案例

数据库表

```sql
CREATE TABLE `book` (
	`id` INT(10) NOT NULL,
	`book_name` VARCHAR(10) NULL DEFAULT NULL,
	`price` DOUBLE NULL DEFAULT NULL,
	`version` INT(11) NULL DEFAULT NULL,
	PRIMARY KEY (`id`)
)
COLLATE='utf8mb4_general_ci'
ENGINE=InnoDB
;

INSERT INTO `book` (`id`, `book_name`, `price`, `version`) VALUES (1, '西游记', 101, 1);
INSERT INTO `book` (`id`, `book_name`, `price`, `version`) VALUES (2, '水浒传', 100, 1);
INSERT INTO `book` (`id`, `book_name`, `price`, `version`) VALUES (3, '三国演义', 100, 1);
INSERT INTO `book` (`id`, `book_name`, `price`, `version`) VALUES (4, '红楼梦', 100, 1);

mysql> select * from book;
+----+-----------+-------+---------+
| id | book_name | price | version |
+----+-----------+-------+---------+
|  1 | 西游记    |   101 |       1 |
|  2 | 水浒传    |   100 |       1 |
|  3 | 三国演义  |   100 |       1 |
|  4 | 红楼梦    |   100 |       1 |
+----+-----------+-------+---------+
4 rows in set (0.01 sec)
```

#### 2.2.1 开启事务使用乐观锁

| 时间轴 |                            事务A                             |                            事务B                             |
| :----: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|   T1   |                      start transaction;                      |                                                              |
|   T2   | select book_name, price from book where id = 1;（得到数据）<br/>price=100, version =1 |                                                              |
|   T3   | mysql> update book set price = 101, version = version + 1 where id = 1 and version = 1;<br/>Query OK, 1 row affected (0.01 sec)<br/>Rows matched: 1  Changed: 1  Warnings: 0<br/>受影响的行数为1 |                                                              |
|   T4   |                                                              |                      start transaction;                      |
|   T5   |                                                              | select book_name, price from book where id = 1;（得到数据）<br/>price=100, version =1 |
|   T6   |                                                              | mysql> update book set price = 101, version = version + 1 where id =1 and version =1;<br/>阻塞…… |
|   T7   |                           commit;                            |                                                              |
|   T8   |                                                              | mysql> update book set price = 101, version = version + 1 where id =1 and version =1;<br/>Query OK, 0 rows affected (7.40 sec)<br/>Rows matched: 0  Changed: 0  Warnings: 0<br/>受影响的行数为0 |
|   T9   |                                                              |                           commit;                            |

#### 2.2.1 不开启事务使用乐观锁

| 时间轴 |                            用户A                             |                            用户B                             |
| :----: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|   T1   | select book_name, price from book where id = 1;（得到数据）<br/>price=100, version =1 |                                                              |
|   T2   | mysql> update book set price = 101, version = version + 1 where id = 1 and version = 1;<br/>Query OK, 1 row affected (0.01 sec)<br/>Rows matched: 1  Changed: 1  Warnings: 0<br/>受影响的行数为1 |                                                              |
|   T3   |                                                              | select book_name, price from book where id = 1;（得到数据）<br/>price=100, version =1 |
|   T4   |                                                              | mysql> update book set price = 101, version = version + 1 where id =1 and version =1;<br/>Query OK, 0 rows affected (7.40 sec)<br/>Rows matched: 0  Changed: 0  Warnings: 0<br/>受影响的行数为0 |

**乐观锁小结**

- 用户B修改数据的时候，受影响行数为0，对业务来说，即更新失败。这时候我们只需要告诉用户更新失败，重新查询一遍即可。
- 对比第一种和第二种测试，我们会发现第一种测试，将update语句放入事务中会出现阻塞的情况，而第二种测试不会出现阻塞情况。这是为什么呢？
  update其实在不在事务中都无所谓，在内部是这样的：update是单线程的，及如果一个线程对一条数据进行update操作，会获得锁，其他线程如果要对同一条数据操作会阻塞，直到这个线程update成功后释放锁。

## 3 适用场景

**悲观锁**

比较适合写入操作比较频繁的场景，如果出现大量的读取操作，每次读取的时候都会进行加锁，这样会增加大量的锁的开销，降低了系统的吞吐量。

**乐观锁**

比较适合读取操作比较频繁的场景，如果出现大量的写入操作，数据发生冲突的可能性就会增大，为了保证数据的一致性，应用层需要不断的重新获取数据，这样会增加大量的查询操作，降低了系统的吞吐量。





悲观锁

```xml
<!-- mapped statement for IbatisCideAppDAO.lockAppByAlias -->
<select id="lock" resultMap="" >
    select
        tnt_inst_id,
        app_id,
        xxx
    from
        table
    where
        alias = #alias# and version = #version#
    FOR UPDATE
</select>
```

乐观锁更新数据

```sql
update table set 
    parent_id=#parentId#,
    xxx
where 
    composite_id = #compositeId# and version=#version#
```

