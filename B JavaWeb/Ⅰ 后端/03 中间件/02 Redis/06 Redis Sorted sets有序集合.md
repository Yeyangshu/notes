# Sorted Sets 有序集合

## 1 Sorted Sets简介

Redis 有序集合和集合一样也是 string 类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个 double 类型的分数。redis 正是通过分数来为集合中的成员进行从小到大的排序。

有序集合的成员是唯一的,但分数(score)却可以重复。

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。 集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。

## 2 Sorted Sets命令

### 2.1 help @sorted_set

```
127.0.0.1:6379> help @sorted_set

  ZADD key [NX|XX] [CH] [INCR] score member [score member ...]
  summary: Add one or more members to a sorted set, or update its score if it already exists
  since: 1.2.0

  ZCARD key
  summary: Get the number of members in a sorted set
  since: 1.2.0

  ZCOUNT key min max
  summary: Count the members in a sorted set with scores within the given values
  since: 2.0.0

  ZINCRBY key increment member
  summary: Increment the score of a member in a sorted set
  since: 1.2.0

  ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM|MIN|MAX]
  summary: Intersect multiple sorted sets and store the resulting sorted set in a new key
  since: 2.0.0

  ZLEXCOUNT key min max
  summary: Count the number of members in a sorted set between a given lexicographical range
  since: 2.8.9

  ZRANGE key start stop [WITHSCORES]
  summary: Return a range of members in a sorted set, by index
  since: 1.2.0

  ZRANGEBYLEX key min max [LIMIT offset count]
  summary: Return a range of members in a sorted set, by lexicographical range
  since: 2.8.9

  ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
  summary: Return a range of members in a sorted set, by score
  since: 1.0.5

  ZRANK key member
  summary: Determine the index of a member in a sorted set
  since: 2.0.0

  ZREM key member [member ...]
  summary: Remove one or more members from a sorted set
  since: 1.2.0

  ZREMRANGEBYLEX key min max
  summary: Remove all members in a sorted set between the given lexicographical range
  since: 2.8.9

  ZREMRANGEBYRANK key start stop
  summary: Remove all members in a sorted set within the given indexes
  since: 2.0.0

  ZREMRANGEBYSCORE key min max
  summary: Remove all members in a sorted set within the given scores
  since: 1.2.0

  ZREVRANGE key start stop [WITHSCORES]
  summary: Return a range of members in a sorted set, by index, with scores ordered from high to low
  since: 1.2.0

  ZREVRANGEBYLEX key max min [LIMIT offset count]
  summary: Return a range of members in a sorted set, by lexicographical range, ordered from higher to lower strings.
  since: 2.8.9

  ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]
  summary: Return a range of members in a sorted set, by score, with scores ordered from high to low
  since: 2.2.0

  ZREVRANK key member
  summary: Determine the index of a member in a sorted set, with scores ordered from high to low
  since: 2.0.0

  ZSCAN key cursor [MATCH pattern] [COUNT count]
  summary: Incrementally iterate sorted sets elements and associated scores
  since: 2.8.0

  ZSCORE key member
  summary: Get the score associated with the given member in a sorted set
  since: 1.2.0

  ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM|MIN|MAX]
  summary: Add multiple sorted sets and store the resulting sorted set in a new key
  since: 2.0.0
```

### 2.2 ZADD

```
127.0.0.1:6379> help zadd

  ZADD key [NX|XX] [CH] [INCR] score member [score member ...]
  summary: Add one or more members to a sorted set, or update its score if it already exists
  since: 1.2.0
  group: sorted_set
```

ZADD：将一个或多个 `member` 元素及其 `score` 值加入到有序集 `key` 当中。

- 如果某个 `member` 已经是有序集的成员，那么更新这个 `member` 的 `score` 值，并通过重新插入这个 `member` 元素，来保证该 `member` 在正确的位置上。
- `score` 值可以是整数值或双精度浮点数。

案例：

```
# apple 8，banana 2，orange 3
127.0.0.1:6379> ZADD k1 8 apple 2 banana 3 orange
(integer) 3
```

### 2.3 ZRANGE&ZRANGEBYSCORE&ZREVRANGE

```
127.0.0.1:6379> help zrange

  ZRANGE key start stop [WITHSCORES]
  summary: Return a range of members in a sorted set, by index
  since: 1.2.0
  group: sorted_set

127.0.0.1:6379> help zrangebyscore

  ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
  summary: Return a range of members in a sorted set, by score
  since: 1.0.5
  group: sorted_set
```

**取值相关命令**

ZRANGE：返回有序集 `key` 中，指定区间内的成员。

- 其中成员的位置按 `score` 值递增(从小到大)来排序。
- 具有相同 `score` 值的成员按字典序来排列。
- 可以通过使用 `WITHSCORES` 选项，来让成员和它的 `score` 值一并返回，返回列表以 `value1,score1, ..., valueN,scoreN` 的格式表示。

ZRANGEBYSCORE：返回有序集 `key` 中，所有 `score` 值介于 `min` 和 `max` 之间(包括等于 `min` 或 `max` )的成员。有序集成员按 `score` 值递增(从小到大)次序排列。

- 具有相同 `score` 值的成员按字典序来排列(该属性是有序集提供的，不需要额外的计算)。
- 可选的 `WITHSCORES` 参数决定结果集是单单返回有序集的成员，还是将有序集成员及其 `score` 值一起返回。

ZREVRANGE：ZREVRANGE返回有序集 `key` 中，指定区间内的成员。

- 其中成员的位置按 `score` 值递减(从大到小)来排列。
- 具有相同 `score` 值的成员按字典序的逆序EVRANGE 命令的其他方面和 ZRANGE 命令一样。

ZREVRANGEBYSCORE：返回有序集 `key` 中， `score` 值介于 `max` 和 `min` 之间(默认包括等于 `max` 或 `min` )的所有的成员。有序集成员按 `score` 值递减(从大到小)的次序排列。

- 具有相同 `score` 值的成员按字典序的逆序排列。
- 除了成员按 `score` 值递减的次序排列这一点外，ZREVRANGEBYSCORE 命令的其他方面和ZRANGEBYSCORE命令一样。

```
# ZRANGE 按照 score 正序排列
127.0.0.1:6379> ZRANGE k1 0 -1
1) "banana"
2) "orange"
3) "apple"
# 使用 WITHSCORES，成员和 score 一起返回
127.0.0.1:6379> ZRANGE k1 0 -1 withscores
1) "banana"
2) "2"
3) "orange"
4) "3"
5) "apple"
6) "8"
# 使用下标
127.0.0.1:6379> ZRANGE k1 -2 -1
1) "orange"
2) "apple"

# 返回score 3~8区间的值
127.0.0.1:6379> ZRANGEBYSCORE k1 3 8
1) "orange"
2) "apple"

# ZRANGE 按照 score 逆序排列
127.0.0.1:6379> ZREVRANGE k1 0 1
1) "apple"
2) "orange"
# ZREVRANGEBYSCORE 逆序排列
127.0.0.1:6379> ZREVRANGEBYSCORE k1 8 3
1) "apple"
2) "orange"
```

### 2.4 ZSCORE

```
127.0.0.1:6379> help zscore

  ZSCORE key member
  summary: Get the score associated with the given member in a sorted set
  since: 1.2.0
  group: sorted_set
```

ZSCORE：返回有序集 `key` 中，成员 `member` 的 `score` 值。

案例：

```
127.0.0.1:6379> ZSCORE k1 apple
"8"
```

### 2.5 ZRANK&ZREVRANK

```
127.0.0.1:6379> help zrank

  ZRANK key member
  summary: Determine the index of a member in a sorted set
  since: 2.0.0
  group: sorted_set
```

ZRANK：返回有序集 `key` 中成员 `member` 的排名。其中有序集成员按 `score` 值递增(从小到大)顺序排列。

ZREVRANK：返回有序集 `key` 中成员 `member` 的排名。其中有序集成员按 `score` 值递减(从大到小)排序。

案例：

```
127.0.0.1:6379> ZRANK k1 apple
(integer) 2
```

### 2.6 ZINCRBY

```
127.0.0.1:6379> help zincrby

  ZINCRBY key increment member
  summary: Increment the score of a member in a sorted set
  since: 1.2.0
  group: sorted_set
```

ZINCRBY：为有序集 `key` 的成员 `member` 的 `score` 值加上增量 `increment` 。

- 可以通过传递一个负数值 `increment` ，让 `score` 减去相应的值，比如 `ZINCRBY key -5 member` ，就是让 `member` 的 `score` 值减去 `5` 。

算数运算

案例：

```
127.0.0.1:6379> ZINCRBY k1 2.5 banana
"4.5"
127.0.0.1:6379> ZRANGE k1 0 -1 withscores
1) "orange"
2) "3"
3) "banana"
4) "4.5"
5) "apple"
6) "8"
```

### 2.7 ZUNIONSTORE&**ZINTERSTORE** 

```
127.0.0.1:6379> help zunionstore

  ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM|MIN|MAX]
  summary: Add multiple sorted sets and store the resulting sorted set in a new key
  since: 2.0.0
  group: sorted_set

127.0.0.1:6379> help zinterstore

  ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM|MIN|MAX]
  summary: Intersect multiple sorted sets and store the resulting sorted set in a new key
  since: 2.0.0
  group: sorted_set
```

ZUNIONSTORE：计算给定的一个或多个有序集的并集，其中给定 `key` 的数量必须以 `numkeys` 参数指定，并将该并集(结果集)储存到 `destination` 。

- 默认情况下，结果集中某个成员的 `score` 值是所有给定集下该成员 `score` 值之和 。

- **WEIGHTS**

  使用 `WEIGHTS` 选项，你可以为 *每个* 给定有序集 *分别* 指定一个乘法因子(multiplication factor)，每个给定有序集的所有成员的 `score` 值在传递给聚合函数(aggregation function)之前都要先乘以该有序集的因子。

  如果没有指定 `WEIGHTS` 选项，乘法因子默认设置为 `1` 。

- **AGGREGATE**

  使用 `AGGREGATE` 选项，你可以指定并集的结果集的聚合方式。

  默认使用的参数 `SUM` ，可以将所有集合中某个成员的 `score` 值之 *和* 作为结果集中该成员的 `score` 值；使用参数 `MIN` ，可以将所有集合中某个成员的 *最小* `score` 值作为结果集中该成员的 `score` 值；而参数 `MAX` 则是将所有集合中某个成员的最大 `score` 值作为结果集中该成员的 `score` 值。

ZINTERSTORE ：计算给定的一个或多个有序集的交集，其中给定 `key` 的数量必须以 `numkeys` 参数指定，并将该交集(结果集)储存到 `destination` 。

- 默认情况下，结果集中某个成员的 `score` 值是所有给定集下该成员 `score` 值之和.

案例：

```powershell
# 假设有两个学科的考试分数
# 第一科分数
127.0.0.1:6379> ZADD k1 80 tom 60 sean 70 baby
(integer) 3
# 第二科分数
127.0.0.1:6379> ZADD k2 60 tom 100 sean 40 peter
(integer) 3
# numkeys为2，k1和k2进行整合
127.0.0.1:6379> ZUNIONSTORE unkey 2 k1 k2
(integer) 4
# 查看整合后的分值,默认SUM相加，正序排列
127.0.0.1:6379> ZRANGE unkey 0 -1 withscores
1) "peter"
2) "40"
3) "baby"
4) "70"
5) "tom"
6) "140"
7) "sean"
8) "160"

# 测试权重，k1权重为1，k2权重为0.5
127.0.0.1:6379> ZUNIONSTORE unkey1 2 k1 k2 WEIGHTS 1 0.5
(integer) 4
# 例如sean分数：60+100*0.5=110
127.0.0.1:6379> ZRANGE unkey1 0 -1 withscores
1) "peter"
2) "20"
3) "baby"
4) "70"
5) "sean"
6) "110"
7) "tom"
8) "110"

# 测试聚合，使用max
127.0.0.1:6379> ZUNIONSTORE unkey2 2 k1 k2 aggregate max
(integer) 4
# 返回k1、k2分数最大值
127.0.0.1:6379> ZRANGE unkey2 0 -1 withscores
1) "peter"
2) "40"
3) "baby"
4) "70"
5) "tom"
6) "80"
7) "sean"
8) "100"

# 并集测试，不存在的值直接删掉
127.0.0.1:6379> ZINTERSTORE interkey 2 k1 k2
(integer) 2
127.0.0.1:6379> ZRANGE interkey 0 -1 withscores
1) "tom"
2) "140"
3) "sean"
4) "160"
```

## 3 Sorted Sets使用场景

歌曲排行榜前十名

歌曲排行榜，第一天上线，所有的歌曲得分值都是0，排行榜按什么排名？可能是播放量、下载数、点播量其中一个，是正序还是倒序的？

可以使用Redis的ZINCYBY，某一首歌播放了一次，就加1，也可以非常快的使用ZRANGE&ZREVRANGE正序或倒序排序。

## 4 面试： 排序是怎么实现的、增删改查的速度？

数据结构：skip list，跳跃表