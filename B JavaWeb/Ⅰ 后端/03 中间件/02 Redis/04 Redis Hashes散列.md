# Hashes 哈希

## 1 Hashes简介

Redis hash 是一个 string 类型的 field（字段） 和 value（值） 的映射表，hash 特别适合用于存储对象。

Redis 中每个 hash 可以存储 232 - 1 键值对（40多亿）。

## 2 Hashes命令

### 2.1 help @hash

```
127.0.0.1:6379> help @hash

  HDEL key field [field ...]
  summary: Delete one or more hash fields
  since: 2.0.0

  HEXISTS key field
  summary: Determine if a hash field exists
  since: 2.0.0

  HGET key field
  summary: Get the value of a hash field
  since: 2.0.0

  HGETALL key
  summary: Get all the fields and values in a hash
  since: 2.0.0

  HINCRBY key field increment
  summary: Increment the integer value of a hash field by the given number
  since: 2.0.0

  HINCRBYFLOAT key field increment
  summary: Increment the float value of a hash field by the given amount
  since: 2.6.0

  HKEYS key
  summary: Get all the fields in a hash
  since: 2.0.0

  HLEN key
  summary: Get the number of fields in a hash
  since: 2.0.0

  HMGET key field [field ...]
  summary: Get the values of all the given hash fields
  since: 2.0.0

  HMSET key field value [field value ...]
  summary: Set multiple hash fields to multiple values
  since: 2.0.0

  HSCAN key cursor [MATCH pattern] [COUNT count]
  summary: Incrementally iterate hash fields and associated values
  since: 2.8.0

  HSET key field value
  summary: Set the string value of a hash field
  since: 2.0.0

  HSETNX key field value
  summary: Set the value of a hash field, only if the field does not exist
  since: 2.0.0

  HSTRLEN key field
  summary: Get the length of the value of a hash field
  since: 3.2.0

  HVALS key
  summary: Get all the values in a hash
  since: 2.0.0
```

### 2.2 HSET&HGET

```
127.0.0.1:6379> help hset

  HSET key field value
  summary: Set the string value of a hash field
  since: 2.0.0
  group: hash

127.0.0.1:6379> help hget

  HGET key field
  summary: Get the value of a hash field
  since: 2.0.0
  group: hash
```

HSET：将哈希表 `key` 中的域 `field` 的值设为 `value` 。

- 如果 `key` 不存在，一个新的哈希表被创建并进行 HSET 操作。

- 如果域 `field` 已经存在于哈希表中，旧值将被覆盖。

HGET：返回哈希表 `key` 中给定域 `field` 的值。

- 当给定域不存在或是给定 `key` 不存在时，返回 `nil` 。

案例：

```powershell
# 将yeyangshu name赋值yeyangshu
127.0.0.1:6379> HSET yeyangshu name yeyangshu
(integer) 1
127.0.0.1:6379> HGET yeyangshu name
"yeyangshu"
# 将yeyangshu name覆盖赋值yeyang
127.0.0.1:6379> HSET yeyangshu name yeyang
(integer) 0
127.0.0.1:6379> HGET yeyangshu name
"yeyang"

# 取不存在的域
127.0.0.1:6379> HGET yeyangshu age
(nil)
```

### 2.3 HMSET&HMGET

```
127.0.0.1:6379> help hmset

  HMSET key field value [field value ...]
  summary: Set multiple hash fields to multiple values
  since: 2.0.0
  group: hash

127.0.0.1:6379> help hmget

  HMGET key field [field ...]
  summary: Get the values of all the given hash fields
  since: 2.0.0
  group: hash
```

HMSET：同时将多个 `field-value` (域-值)对设置到哈希表 `key` 中。

- 此命令会覆盖哈希表中已存在的域。

HMGET：返回哈希表 `key` 中，一个或多个给定域的值。

案例：

```
127.0.0.1:6379> HMSET sean age 20 address beijing
OK
127.0.0.1:6379> HMGET sean name
"yeyangshu"
127.0.0.1:6379> HMGET sean name age address
1) "yeyangshu"
2) "20"
3) "beijing"
```

### 2.4 HGETALL

```
127.0.0.1:6379> help hgetall

  HGETALL key
  summary: Get all the fields and values in a hash
  since: 2.0.0
  group: hash
```

HGETALL：返回哈希表 `key` 中，所有的域和值。

- 在返回值里，紧跟每个域名(field name)之后是域的值(value)，所以返回值的长度是哈希表大小的两倍。

案例：

```powershell
127.0.0.1:6379> HGETALL sean
1) "name"
2) "yeyangshu"
3) "age"
4) "20"
5) "address"
```

### 2.5 HKEYS&HVALS

```
127.0.0.1:6379> help hkeys

  HKEYS key
  summary: Get all the fields in a hash
  since: 2.0.0
  group: hash

127.0.0.1:6379> help hvals

  HVALS key
  summary: Get all the values in a hash
  since: 2.0.0
  group: hash
```

HKEYS：返回哈希表 `key` 中的所有域。

HVALS：返回哈希表 `key` 中所有域的值。

案例：

```powershell
127.0.0.1:6379> HKEYS sean
1) "name"
2) "age"
3) "address"
127.0.0.1:6379> HVALS sean
1) "yeyangshu"
2) "20"
3) "beijing"
```

### 2.6 HINCRBY&HINCRBYFLOAT

```
127.0.0.1:6379> help hincrby

  HINCRBY key field increment
  summary: Increment the integer value of a hash field by the given number
  since: 2.0.0
  group: hash

127.0.0.1:6379> help hincrbyfloat

  HINCRBYFLOAT key field increment
  summary: Increment the float value of a hash field by the given amount
  since: 2.6.0
  group: hash
```

HINCRBY：为哈希表 `key` 中的域 `field` 的值加上增量 `increment` 。

- 增量也可以为负数，相当于对给定域进行减法操作。

HINCRBYFLOAT：为哈希表 `key` 中的域 `field` 加上浮点数增量 `increment` 。

案例：

```powershell
127.0.0.1:6379> HINCRBYFLOAT sean age 0.5
"20.5"
127.0.0.1:6379> HGET sean age
"20.5"
127.0.0.1:6379> HINCRBYFLOAT sean age -1
"19.5"
```

## 3 Hashes使用场景

1. Redis是内存的数据库，所有对值的操作都是非常快的，一般的商品详情页都会有很多字段信息，如果客户端请求这么多数据，每一个数据都需要请求一次数据库，可以使用Redis hashes，将所有的信息进行整合，数据整合，调用次数会变低。
2. 数据都会变化，比如微博的个人关注、点赞或者商品详情页的浏览次数、被收藏的次数，数据不仅要被查询，还要进行计算，hash还支持数值计算。







## 