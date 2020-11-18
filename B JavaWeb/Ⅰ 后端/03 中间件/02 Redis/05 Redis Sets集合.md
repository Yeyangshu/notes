# Sets 集合

## 1 Sets简介

Redis Set 是 String 的无序排列。集合成员是唯一的，这就意味着集合中不能出现重复的数据。

Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。

集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。

## 2 Sets命令

### 2.1 help @set

```
127.0.0.1:6379> help @set

  SADD key member [member ...]
  summary: Add one or more members to a set
  since: 1.0.0

  SCARD key
  summary: Get the number of members in a set
  since: 1.0.0

  SDIFF key [key ...]
  summary: Subtract multiple sets
  since: 1.0.0

  SDIFFSTORE destination key [key ...]
  summary: Subtract multiple sets and store the resulting set in a key
  since: 1.0.0

  SINTER key [key ...]
  summary: Intersect multiple sets
  since: 1.0.0

  SINTERSTORE destination key [key ...]
  summary: Intersect multiple sets and store the resulting set in a key
  since: 1.0.0

  SISMEMBER key member
  summary: Determine if a given value is a member of a set
  since: 1.0.0

  SMEMBERS key
  summary: Get all the members in a set
  since: 1.0.0

  SMOVE source destination member
  summary: Move a member from one set to another
  since: 1.0.0

  SPOP key [count]
  summary: Remove and return one or multiple random members from a set
  since: 1.0.0

  SRANDMEMBER key [count]
  summary: Get one or multiple random members from a set
  since: 1.0.0

  SREM key member [member ...]
  summary: Remove one or more members from a set
  since: 1.0.0

  SSCAN key cursor [MATCH pattern] [COUNT count]
  summary: Incrementally iterate Set elements
  since: 2.8.0

  SUNION key [key ...]
  summary: Add multiple sets
  since: 1.0.0

  SUNIONSTORE destination key [key ...]
  summary: Add multiple sets and store the resulting set in a key
  since: 1.0.0
```

### 2.2 SADD&SMEMBERS

```
127.0.0.1:6379> help sadd

  SADD key member [member ...]
  summary: Add one or more members to a set
  since: 1.0.0
  group: set

127.0.0.1:6379> help smembers

  SMEMBERS key
  summary: Get all the members in a set
  since: 1.0.0
  group: set
```

SADD：将一个或多个 `member` 元素加入到集合 `key` 当中，已经存在于集合的 `member` 元素将被忽略。

- 假如 `key` 不存在，则创建一个只包含 `member` 元素作成员的集合。

- 当 `key` 不是集合类型时，返回一个错误。

SMEMBERS：返回集合 `key` 中的所有成员。

案例：

```powershell
# 向k1添加重复元素
127.0.0.1:6379> SADD k1 sean tom peter tom ooxx ooxx
(integer) 4
# k1取出所有元素
127.0.0.1:6379> SMEMBERS k1
1) "ooxx"
2) "peter"
3) "tom"
4) "sean"
```

### 2.3 SREM

```
127.0.0.1:6379> help srem

  SREM key member [member ...]
  summary: Remove one or more members from a set
  since: 1.0.0
  group: set
```

SREM：移除集合 `key` 中的一个或多个 `member` 元素，不存在的 `member` 元素会被忽略。

案例：

```powershell
# k1取出所有元素
127.0.0.1:6379> SMEMBERS k1
1) "ooxx"
2) "peter"
3) "tom"
4) "sean"
127.0.0.1:6379> SREM k1 ooxx
(integer) 1
127.0.0.1:6379> SMEMBERS k1
1) "tom"
2) "sean"
3) "peter"
```

### 2.4 SINTER&SUNION&SDIFF（重要）

```
127.0.0.1:6379> help sinter

  SINTER key [key ...]
  summary: Intersect multiple sets
  since: 1.0.0
  group: set

127.0.0.1:6379> help sinterstore

  SINTERSTORE destination key [key ...]
  summary: Intersect multiple sets and store the resulting set in a key
  since: 1.0.0
  group: set

127.0.0.1:6379> help sunion

  SUNION key [key ...]
  summary: Add multiple sets
  since: 1.0.0
  group: set

127.0.0.1:6379> help sdiff

  SDIFF key [key ...]
  summary: Subtract multiple sets
  since: 1.0.0
  group: set
```

SINTER：返回一个集合的全部成员，该集合是所有给定集合的交集。

SINTERSTORE：这个命令类似于 SINTER 命令，但它将结果保存到 `destination` 集合，而不是简单地返回结果集。

- 如果 `destination` 集合已经存在，则将其覆盖。

- `destination` 可以是 `key` 本身。

SUNION：返回一个集合的全部成员，该集合是所有给定集合的并集。

SUNIONSTORE：这个命令类似于 SUNION 命令，但它将结果保存到 `destination` 集合，而不是简单地返回结果集。

- 如果 `destination` 已经存在，则将其覆盖。

- `-destination` 可以是 `key` 本身。

SDIFF：返回一个集合的全部成员，该集合是所有给定集合之间的差集。

SDIFFSTORE：这个命令的作用和 SDIFF 类似，但它将结果保存到 `destination` 集合，而不是简单地返回结果集。

- 如果 `destination` 集合已经存在，则将其覆盖。

- `destination` 可以是 `key` 本身。

案例：

交集

```powershell
127.0.0.1:6379> SADD k2 1 2 3 4 5 6
(integer) 6
127.0.0.1:6379> SADD k3 3 4 5 6 7 8
(integer) 6
# k3与k3交集
127.0.0.1:6379> SINTER k2 k3
1) "3"
2) "4"
3) "5"
4) "6"

# 将交集结果存入dest
127.0.0.1:6379> SINTERSTORE dest k2 k3
(integer) 4
127.0.0.1:6379> SMEMBERS dest
1) "3"
2) "4"
3) "5"
4) "6"
```

并集

```powershell
127.0.0.1:6379> SUNION k2 k3
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
7) "7"
8) "8"
```

差集

```powershell
127.0.0.1:6379> SDIFF k2 k3
1) "1"
2) "2"
127.0.0.1:6379> SDIFF k3 k2
1) "7"
2) "8"
```

### 2.5 SRANDMEMBER&SPOP

```
127.0.0.1:6379> help srandmember

  SRANDMEMBER key [count]
  summary: Get one or multiple random members from a set
  since: 1.0.0
  group: set

127.0.0.1:6379> help spop

  SPOP key [count]
  summary: Remove and return one or multiple random members from a set
  since: 1.0.0
  group: set
```

SRANDMEMBER：如果命令执行时，只提供了 `key` 参数，那么返回集合中的一个随机元素。

从 Redis 2.6 版本开始， SRANDMEMBER 命令接受可选的 `count` 参数：

- 如果 `count` 为正数，且小于集合基数，那么命令返回一个包含 `count` 个元素的数组，数组中的元素**各不相同**。如果 `count` 大于等于集合基数，那么返回整个集合。
- 如果 `count` 为负数，那么命令返回一个数组，数组中的元素**可能会重复出现多次**，而数组的长度为 `count` 的绝对值。

SPOP：移除并返回集合中的一个随机元素。

案例：

SRANDMEMBER

```powershell
127.0.0.1:6379> SADD k2 1 2 3 4 5 6
(integer) 6
# count正数：取出一个去重的结果集（不能超过已有集）
127.0.0.1:6379> SRANDMEMBER k2 3
1) "2"
2) "4"
3) "5"
# count负数：取出一个带重复的结果集
127.0.0.1:6379> SRANDMEMBER k2 -3
1) "6"
2) "1"
3) "6"
# count为0，不返回
127.0.0.1:6379> SRANDMEMBER k2 0
(empty list or set)

127.0.0.1:6379> SRANDMEMBER k2 10
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
127.0.0.1:6379> SRANDMEMBER k2 -10
 1) "3"
 2) "3"
 3) "3"
 4) "4"
 5) "3"
 6) "4"
 7) "6"
 8) "4"
 9) "3"
10) "1"
```

SPOP

```powershell
127.0.0.1:6379> SPOP k2 1
1) "1"
127.0.0.1:6379> SPOP k2 1
1) "5"
127.0.0.1:6379> SPOP k2 1
1) "6"
127.0.0.1:6379> SPOP k2 1
1) "4"
127.0.0.1:6379> SPOP k2 1
1) "3"
127.0.0.1:6379> SPOP k2 1
1) "2"
127.0.0.1:6379> SPOP k2 1
(empty list or set)
```

## 3 Sets使用场景

1. 随机抽奖，一次出结果

   情景一 人多奖品少：3个奖品，10个人抽奖，抽3个并且没有重复，使用SRANDMEMBER key 3

   情景二 人少奖品多：10个奖品，3个人抽奖，抽10个可以有重复，使用SRANDMEMBER key -10

2. 随机抽奖，一次一个

   奖品分为一、二、三等奖，每次只抽一个人，SPOP key 1
