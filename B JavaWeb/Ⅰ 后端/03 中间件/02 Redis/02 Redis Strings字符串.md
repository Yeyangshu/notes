# Strings

## 1 Strings简介

官网：

这是最简单Redis类型。如果你只用这种类型，Redis就像一个可以持久化的memcached服务器

## 2 String命令

### 2.1 help @string

```she
127.0.0.1:6379> help @string

  APPEND key value
  summary: Append a value to a key
  since: 2.0.0

  BITCOUNT key [start end]
  summary: Count set bits in a string
  since: 2.6.0

  BITFIELD key [GET type offset] [SET type offset value] [INCRBY type offset increment] [OVERFLOW WRAP|SAT|FAIL]
  summary: Perform arbitrary bitfield integer operations on strings
  since: 3.2.0

  BITOP operation destkey key [key ...]
  summary: Perform bitwise operations between strings
  since: 2.6.0

  BITPOS key bit [start] [end]
  summary: Find first bit set or clear in a string
  since: 2.8.7

  DECR key
  summary: Decrement the integer value of a key by one
  since: 1.0.0

  DECRBY key decrement
  summary: Decrement the integer value of a key by the given number
  since: 1.0.0

  GET key
  summary: Get the value of a key
  since: 1.0.0

  GETBIT key offset
  summary: Returns the bit value at offset in the string value stored at key
  since: 2.2.0

  GETRANGE key start end
  summary: Get a substring of the string stored at a key
  since: 2.4.0

  GETSET key value
  summary: Set the string value of a key and return its old value
  since: 1.0.0

  INCR key
  summary: Increment the integer value of a key by one
  since: 1.0.0

  INCRBY key increment
  summary: Increment the integer value of a key by the given amount
  since: 1.0.0

  INCRBYFLOAT key increment
  summary: Increment the float value of a key by the given amount
  since: 2.6.0

  MGET key [key ...]
  summary: Get the values of all the given keys
  since: 1.0.0

  MSET key value [key value ...]
  summary: Set multiple keys to multiple values
  since: 1.0.1

  MSETNX key value [key value ...]
  summary: Set multiple keys to multiple values, only if none of the keys exist
  since: 1.0.1

  PSETEX key milliseconds value
  summary: Set the value and expiration in milliseconds of a key
  since: 2.6.0

  SET key value [expiration EX seconds|PX milliseconds] [NX|XX]
  summary: Set the string value of a key
  since: 1.0.0

  SETBIT key offset value
  summary: Sets or clears the bit at offset in the string value stored at key
  since: 2.2.0

  SETEX key seconds value
  summary: Set the value and expiration of a key
  since: 2.0.0

  SETNX key value
  summary: Set the value of a key, only if the key does not exist
  since: 1.0.0

  SETRANGE key offset value
  summary: Overwrite part of a string at key starting at the specified offset
  since: 2.2.0

  STRLEN key
  summary: Get the length of the value stored in a key
  since: 2.2.0
```

### 2.2 Strings字符串操作命令详解

#### 2.2.1 SET&GET

```
127.0.0.1:6379> help set

  SET key value [EX seconds] [PX milliseconds] [NX|XX]
  summary: Set the string value of a key
  since: 1.0.0
  group: string

127.0.0.1:6379> help get

  GET key
  summary: Get the value of a key
  since: 1.0.0
  group: string
```

通常用SET command 和 GET command来设置和获取字符串值，值可以是任何种类的字符串（包括二进制数据）

- EX：过期时间

- PX：时间单位

- NX ：key未创建时则创建，已经创建时不创建。

  用处：分布式锁，一堆人都想删一个文件，很多链接对单线程redis操作，谁成功就拿到锁，其余人失败

- XX：只能更新，key存在时更新，key不存在时不能更新

案例：

```shell
# set测试
127.0.0.1:6379> set k1 hello
OK
# get测试
127.0.0.1:6379> get k1
"hello"

# nx测试
127.0.0.1:6379> set k1 ooxx nx
(nil)
127.0.0.1:6379> get k1
"hello"

# xx测试
127.0.0.1:6379> set k2 hello xx
(nil)
127.0.0.1:6379> get k2
(nil)
```

#### 2.2.2 MSET&MGET

同时设置一个或多个 key-value 对。

```
127.0.0.1:6379> help mset

  MSET key value [key value ...]
  summary: Set multiple keys to multiple values
  since: 1.0.1
  group: string

127.0.0.1:6379> help mget

  MGET key [key ...]
  summary: Get the values of all the given keys
  since: 1.0.0
  group: string
```

案例：

```shell
# 设置[k3,a]，[k4,b]
127.0.0.1:6379> mset k3 a k4 b
OK
# 获取k3，k4的value
127.0.0.1:6379> mget k3 k4
1) "a"
2) "b"
```

#### 2.2.3 APPEND

如果 key 已经存在并且是一个字符串， APPEND 命令将指定的 value 追加到该 key 原来值（value）的末尾。

```
127.0.0.1:6379> help append

  APPEND key value
  summary: Append a value to a key
  since: 2.0.0
  group: string
```

案例：

```shell
127.0.0.1:6379> get k1
"hello"
127.0.0.1:6379> append k1 " world"
(integer) 11
127.0.0.1:6379> get k1
"hello world"
```

#### 2.2.4 GETRANGE&SETRANGE

```shell
127.0.0.1:6379> help getrange

  GETRANGE key start end
  summary: Get a substring of the string stored at a key
  since: 2.4.0
  group: string
  
127.0.0.1:6379> help setrange

  SETRANGE key offset value
  summary: Overwrite part of a string at key starting at the specified offset
  since: 2.2.0
  group: string
```

GETRANGE：获取存储在指定 key 中字符串的子字符串。字符串的截取范围由 start 和 end 两个偏移量决定(包括 start 和 end 在内)。

SETRANGE：用 value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始。

```shell
# 此时k1 value="hello world"
127.0.0.1:6379> get k1
"hello world"

# 取子字符串"world"
127.0.0.1:6379> GETRANGE k1 6 10
"world"
# 逆向索引取值
127.0.0.1:6379> GETRANGE k1 6 -1
"world"
# 正向索引取值
127.0.0.1:6379> GETRANGE k1 0 -1
"hello world"

# SETRANGE
127.0.0.1:6379> SETRANGE k1 6 yeyangshu
(integer) 15
127.0.0.1:6379> get k1
"hello yeyangshu"
```

#### 2.2.5 STRLEN

```
127.0.0.1:6379> help strlen

  STRLEN key
  summary: Get the length of the value stored in a key
  since: 2.2.0
  group: string
```

获取指定 key 所储存的字符串值的长度。当 key 储存的不是字符串值时，返回一个错误。

案例：

```shell
127.0.0.1:6379> get k1
"hello yeyangshu"
127.0.0.1:6379> STRLEN k1
(integer) 15
```

#### 2.2.6 GETSET

```
127.0.0.1:6379> help getset

  GETSET key value
  summary: Set the string value of a key and return its old value
  since: 1.0.0
  group: string
```

设置指定 key 的值，并返回 key 的旧值。

案例：

```shell
127.0.0.1:6379> GET k1
"100"
127.0.0.1:6379> GETSET k1 200
"100"
127.0.0.1:6379> GET k1
"200"
```

#### 2.2.7 MSETNX

```
127.0.0.1:6379> help MSETNX

  MSETNX key value [key value ...]
  summary: Set multiple keys to multiple values, only if none of the keys exist
  since: 1.0.1
  group: string
```

同时设置一个或多个 key-value 对，当且仅当所有给定 key 都**不存在**。

```shell
# 设置k1、k2的值
127.0.0.1:6379> MSETNX k1 a k2 b
(integer) 1
127.0.0.1:6379> MGET k1 k2
1) "a"
2) "b"
# 设置k2、k3的值，k2已经存在整体返回失败
127.0.0.1:6379> MSETNX k2 c k3 d
(integer) 0
```

### 2.3 Strings数值操作命令详解

#### 2.3.1 INCR&DECR

```
127.0.0.1:6379> help incr

  INCR key
  summary: Increment the integer value of a key by one
  since: 1.0.0
  group: string
  
127.0.0.1:6379> help decr

  DECR key
  summary: Decrement the integer value of a key by one
  since: 1.0.0
  group: string
```

INCR：将 key 中储存的数字值增一。

- 如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 INCR 操作。
- 如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

DECR：将 key 中储存的数字值减一。

- 如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 DECR 操作。

- 如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

案例：

```shell
# k1原值99
127.0.0.1:6379> INCR k1
(integer) 100
127.0.0.1:6379> GET k1
"100"
# 不存在k6这个key
127.0.0.1:6379> incr k6
(integer) 1
127.0.0.1:6379> get k6
"1"

# k1原值100
127.0.0.1:6379> DECR k1
(integer) 99
# 不存在k7这个key
127.0.0.1:6379> DECR k7
(integer) -1
```

#### 2.3.2 INCRBY&DECRBY

```
127.0.0.1:6379> help incrby

  INCRBY key increment
  summary: Increment the integer value of a key by the given amount
  since: 1.0.0
  group: string
  
127.0.0.1:6379> help decrby

  DECRBY key decrement
  summary: Decrement the integer value of a key by the given number
  since: 1.0.0
  group: string
```

INCRBY：将 key 中储存的数字加上指定的增量值。

- 如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 INCRBY 命令。

- 如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

DECRBY：将 key 所储存的值减去指定的减量值。

- 如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 DECRBY 操作。

- 如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

案例：

```powershell
127.0.0.1:6379> GET k1
"1"
127.0.0.1:6379> INCRBY k1 10
(integer) 11
127.0.0.1:6379> GET k1
"11"
127.0.0.1:6379> DECRBY k1 10
(integer) 1
127.0.0.1:6379> GET k1
"1"
```

#### 2.3.3 INCRBYFLOAT

```
127.0.0.1:6379> help incrbyfloat

  INCRBYFLOAT key increment
  summary: Increment the float value of a key by the given amount
  since: 2.6.0
  group: string
```

INCRBYFLOAT：为 key 中所储存的值加上指定的浮点数增量值。

- 如果 key 不存在，那么 INCRBYFLOAT 会先将 key 的值设为 0 ，再执行加法操作

案例：

```shell
127.0.0.1:6379> GET k1
"1"
127.0.0.1:6379> INCRBYFLOAT k1 1.1
"2.1"
```

### 2.4 Strings位图操作命令详解

#### 2.4.1 SETBIT

```
127.0.0.1:6379> help setbit

  SETBIT key offset value
  summary: Sets or clears the bit at offset in the string value stored at key
  since: 2.2.0
  group: string
```

对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。

**注意：是二进制位上的偏移，非字节数组**

案例：

```shell
# 一个字节有八个二进制位，二进制的值只有0和1

# 将k1二进制位的第2位变为1，00000000 -> 01000000
127.0.0.1:6379> SETBIT k1 1 1
(integer) 0
# k1长度为一个字节
127.0.0.1:6379> STRLEN k1
(integer) 1
# 01000000在ASCII码中代表'@'
127.0.0.1:6379> GET k1
"@"

# 将k1二进制位的第8位变为1，01000000 -> 01000001
127.0.0.1:6379> SETBIT k1 7 1
(integer) 0
# 此时还没有超过8位，k1长度为1个字节
127.0.0.1:6379> STRLEN k1
(integer) 1
# 01000001在ASCII码中代表'@'
127.0.0.1:6379> GET k1
"A"

# 将k1二进制位的第10位变为1，01000001 -> 01000001 01000000
127.0.0.1:6379> SETBIT k1 9 1
(integer) 0
# 此时已经超过8位，k1长度为2个字节
127.0.0.1:6379> STRLEN k1
(integer) 2
# 01000001 01000000在ASCII码中代表'A@'
127.0.0.1:6379> GET k1
"A@"
```

#### 2.4.2 BITPOS

```
127.0.0.1:6379> help bitpos

  BITPOS key bit [start] [end]
  summary: Find first bit set or clear in a string
  since: 2.8.7
  group: string
```

二进制位`1`在第几个字节和第几个字节之间最开始的位置

**注意：是字节，不是二进制位**

案例：

```shell
# 01000001 01000000
127.0.0.1:6379> GET k1
"A@"
# 在第一个字节中寻找二进制位为1的第一个位置，结果：下标为1的位置
127.0.0.1:6379> BITPOS k1 1 0 0
(integer) 1
# 在第二个字节中寻找二进制位为1的第一个位置，结果：下标为9的位置
127.0.0.1:6379> BITPOS k1 1 1 1
(integer) 9
# 在第一、二个字节中寻找二进制位为1的第一个位置，结果：下标为1的位置
127.0.0.1:6379> BITPOS k1 1 0 1
(integer) 1
```

#### 2.4.3 BITCOUNT

```
127.0.0.1:6379> help bitcount

  BITCOUNT key [start end]
  summary: Count set bits in a string
  since: 2.6.0
  group: string
```

计算给定字符串中，被设置为 `1` 的比特位的数量。

- 一般情况下，给定的整个字符串都会被进行计数，通过指定额外的 `start` 或 `end` 参数，可以让计数只在特定的位上进行。

- 不存在的 `key` 被当成是空字符串来处理，因此对一个不存在的 `key` 进行 `BITCOUNT` 操作，结果为 `0` 。

案例：

```shell
# 01000001 01000000
127.0.0.1:6379> GET k1
"A@"
# 在第一个字节中计算二进制位为1的数量，结果：2
127.0.0.1:6379> BITCOUNT k1 0 0
(integer) 2
# 在第二个字节中计算二进制位为1的数量，结果：1
127.0.0.1:6379> BITCOUNT k1 1 1 
(integer) 1
# 在第一、二个字节中计算二进制位为1的数量，结果：3
127.0.0.1:6379> BITCOUNT k1 0 1 
(integer) 3
```

#### 2.4.4 BITOP

```
127.0.0.1:6379> help bitop

  BITOP operation destkey key [key ...]
  summary: Perform bitwise operations between strings
  since: 2.6.0
  group: string
```

对一个或多个保存二进制位的字符串 `key` 进行位元操作，并将结果保存到 `destkey` 上。

`operation` 可以是 `AND` 、 `OR` 、 `NOT` 、 `XOR` 这四种操作中的任意一种：

- `BITOP AND destkey key [key ...]` ，对一个或多个 `key` 求逻辑并，并将结果保存到 `destkey` 。
- `BITOP OR destkey key [key ...]` ，对一个或多个 `key` 求逻辑或，并将结果保存到 `destkey` 。
- `BITOP XOR destkey key [key ...]` ，对一个或多个 `key` 求逻辑异或，并将结果保存到 `destkey` 。
- `BITOP NOT destkey key` ，对给定 `key` 求逻辑非，并将结果保存到 `destkey` 。

除了 `NOT` 操作之外，其他操作都可以接受一个或多个 `key` 作为输入。

案例：

```shell
127.0.0.1:6379> SETBIT k1 1 1
(integer) 0
127.0.0.1:6379> SETBIT k1 7 1
(integer) 0
127.0.0.1:6379> get k1
"A"
127.0.0.1:6379> SETBIT k2 1 1
(integer) 0
127.0.0.1:6379> SETBIT k2 6 1
(integer) 0
127.0.0.1:6379> get k2
"B"
# 按位与计算k1、k2
127.0.0.1:6379> bitop and andkey k1 k2
(integer) 1
127.0.0.1:6379> get andkey
"@"
# 按位或计算k1、k2
127.0.0.1:6379> bitop or orkey k1 k2
(integer) 1
127.0.0.1:6379> get orkey
"C"
```

## 2.3 String应用场景

### 2.3.1 字符串

### 2.3.2 数值

### 2.3.3 bitmap

#### 2.3.3.1 用户系统，统计用户登陆天数，且窗口随机

公司的用户系统，统计未来用户的登陆天数，且窗口随机。比如说在电商的公司当中，统计双十一前一周和后一周用户的登陆天数。

1. 解决方案一：MySQL数据库

   使用MySQL数据库，创建一张用户登录表，用户每次登录可以产生一行记录，登记登陆时间。

   问题：

   - 关系型数据库表与表之间会有主外键或者关联，关联的 `id` 可能3、4个字节
   - 成本复杂度，每张表每行需要存储一个日期，登陆时间，日期也需要至少4个字节

   一个用户的一笔登录就需要消耗8个字节，电商可能有几十万人，每人一年至少登陆200天，这张表的数据量可能非常大；随机窗口进行查询的时候，需要遍历所有的数据，成本非常高！

2. 解决方案二：Redis数据库

   成本计算，两个固定的数值，第一个固定的数值就是一年的天数，365或者366，假设一年400天，如果每一天从左向右对应一个二进制位，第一个二进制位代表第一天，第二个二进制位代表第二天，一共400个二进制位，400(位)/ 8=50(字节)，使用50个字节可以记录一个用户全年365天的登录状态。

   ```shell
   # yeyangshu第2天登录
   127.0.0.1:6379> SETBIT yeyangshu 1 1
   # yeyangshu第8天登录
   (integer) 0
   127.0.0.1:6379> SETBIT yeyangshu 7 1
   (integer) 0
   # yeyangshu第365天登录
   127.0.0.1:6379> SETBIT yeyangshu 364 1
   (integer) 0
   # 计算总共占用的空间
   127.0.0.1:6379> STRLEN yeyangshu
   (integer) 46
   ```

   统计最后两周用户登录的天数

   ```shell
   # 统计最后两周yeyangshu登陆天数
   127.0.0.1:6379> BITCOUNT yeyangshu -2 -1
   (integer) 1
   ```

   优点：CPU对二进制位的计算速度是最快的，关系型数据库需要从磁盘读取文件，第一个会产生IO，第二个读取磁盘之后，需要将数据解码再参与计算，而且不是二进制位的计算

#### 2.3.3.2 电商628做活动，该准备多少礼物（面试必问）

电商做活动，当天登陆就送礼物，准备多少礼物？假设京东有2亿用户，618登录送礼物，每个人只能送一件，请问库存需要准备多少礼物？

电商里面有一个基本常识，用户分为僵尸用户、冷热用户（忠诚用户）。

过往一年中或同比去年的时间范围内或上一个月窗口内，网站有1亿活跃用户或100万活跃用户经常登录，其实根本不是2亿用户，所以最终的目标是活跃用户。

这也是需要经常做的一个统计：活跃用户统计。活跃用户统计的本质是什么？比如说1号到3号，1号里面有多少人，2号里面有多少人并且还需要去重，如何去设计？

需求综上：活跃用户统计，随机窗口，连续登录并且去重。



Redis做法

日期作为key，20200101，用户id映射到二进制位上。假设小明的id是10，小李的id是1000，二进制位1代表登录，二进制位0代表未登录，假设第一天只有小明登录了，`setbit 20200101 10 1`，第二天小明和小李都登陆了，`setbit 20200102 1000 1`，计算1号和2号的活跃用户。

```shell
# SETBIT date OFFSET id
# 1号小明登录
127.0.0.1:6379> SETBIT 20200101 10 1
(integer) 0

# 2号小明登录
127.0.0.1:6379> SETBIT 20200102 10 1
(integer) 0
# 2号小李登录
127.0.0.1:6379> SETBIT 20200102 1000 1
(integer) 0

# 或运算，参加运算的两个对象，一个为1，其值为1，目的是统计这两天的登录人数
127.0.0.1:6379> BITOP or destkey 20200101 20200102
(integer) 126

# BITCOUNT计算出活跃人数
127.0.0.1:6379> BITCOUNT destkey 0 -1
(integer) 2
```





