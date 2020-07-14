# Sort Set

## 1 Sort Set简介

## 2 Sort Set命令

### 2.1 help @sorted_set

```shell
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

添加、更新

- ZADD：向有序集合添加一个或多个成员，或者更新已存在成员的分数

```shell
127.0.0.1:6379> ZADD k1 8 apple 2 banana 3 orange
(integer) 3
```

### 2.3 ZRANGE、ZRANGEBYSCORE、ZREVRANGE、ZSCORE、ZRANK

取值相关

- ZRANGE：通过索；引区间返回有序集合指定区间内的成员
- ZRANGEBYSCORE：通过分数返回有序集合指定区间内的成员

```she
127.0.0.1:6379> ZRANGE k1 0 -1
1) "banana"
2) "orange"
3) "apple"
127.0.0.1:6379> ZRANGE k1 0 -1 withscores
1) "banana"
2) "2"
3) "orange"
4) "3"
5) "apple"
6) "8"
127.0.0.1:6379> ZRANGE k1 -2 -1
1) "orange"
2) "apple"

127.0.0.1:6379> ZRANGEBYSCORE k1 3 8
1) "orange"
2) "apple"

127.0.0.1:6379> ZREVRANGE k1 0 1
1) "apple"
2) "orange"

127.0.0.1:6379> ZSCORE k1 apple
"8"

127.0.0.1:6379> ZRANK k1 apple
(integer) 2
```

### 2.4 ZINCRBY

算数运算

- ZINCRBY

```shell
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

### 2.5 ZUNIONSTORE

- ZUNIONSTORE

```shell
127.0.0.1:6379> ZADD k1 80 tom 60 sean 70 baby
(integer) 3
127.0.0.1:6379> ZADD k2 60 tom 100 sean 40 peter
(integer) 3
127.0.0.1:6379> ZUNIONSTORE unkey 2 k1 k2
(integer) 4
// 默认相加
127.0.0.1:6379> ZRANGE unkey 0 -1 withscores
1) "peter"
2) "40"
3) "baby"
4) "70"
5) "tom"
6) "140"
7) "sean"
8) "160"

// 权重
127.0.0.1:6379> ZUNIONSTORE unkey1 2 k1 k2 WEIGHTS 1 0.5
(integer) 4
127.0.0.1:6379> ZRANGE unkey1 0 -1 withscores
1) "peter"
2) "20"
3) "baby"
4) "70"
5) "sean"
6) "110"
7) "tom"
8) "110"
```

面试：  排序是怎么实现的、增删改查的速度？