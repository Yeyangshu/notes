# Lists

## 1 Lists简介

官网：

Redis lists基于Linked Lists实现。这意味着即使在一个list中有数百万个元素，在头部或尾部添加一个元素的操作，其时间复杂度也是常数级别的。用LPUSH 命令在十个元素的list头部添加新元素，和在千万元素list头部添加新元素的速度相同。

那么，坏消息是什么？在数组实现的list中利用索引访问元素的速度极快，而同样的操作在linked list实现的list上没有那么快。

Redis Lists用linked list实现的原因是：对于数据库系统来说，至关重要的特性是：能非常快的在很大的列表上添加元素。另一个重要因素是，正如你将要看到的：Redis lists能在常数时间取得常数长度。

![image-20200707204400692](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200707204400692.png)

## 2 List命令

### 2.1 help @list

```
127.0.0.1:6379> help @list

  BLPOP key [key ...] timeout
  summary: Remove and get the first element in a list, or block until one is available
  since: 2.0.0

  BRPOP key [key ...] timeout
  summary: Remove and get the last element in a list, or block until one is available
  since: 2.0.0

  BRPOPLPUSH source destination timeout
  summary: Pop a value from a list, push it to another list and return it; or block until one is available
  since: 2.2.0

  LINDEX key index
  summary: Get an element from a list by its index
  since: 1.0.0

  LINSERT key BEFORE|AFTER pivot value
  summary: Insert an element before or after another element in a list
  since: 2.2.0

  LLEN key
  summary: Get the length of a list
  since: 1.0.0

  LPOP key
  summary: Remove and get the first element in a list
  since: 1.0.0

  LPUSH key value [value ...]
  summary: Prepend one or multiple values to a list
  since: 1.0.0

  LPUSHX key value
  summary: Prepend a value to a list, only if the list exists
  since: 2.2.0

  LRANGE key start stop
  summary: Get a range of elements from a list
  since: 1.0.0

  LREM key count value
  summary: Remove elements from a list
  since: 1.0.0

  LSET key index value
  summary: Set the value of an element in a list by its index
  since: 1.0.0

  LTRIM key start stop
  summary: Trim a list to the specified range
  since: 1.0.0

  RPOP key
  summary: Remove and get the last element in a list
  since: 1.0.0

  RPOPLPUSH source destination
  summary: Remove the last element in a list, prepend it to another list and return it
  since: 1.2.0

  RPUSH key value [value ...]
  summary: Append one or multiple values to a list
  since: 1.0.0

  RPUSHX key value
  summary: Append a value to a list, only if the list exists
  since: 2.2.0
```

### 2.2 LPUSH、RPUSH、LPOP、RPOP

格式

```txt
LPUSH key value [value ...]
summary: Prepend one or multiple values to a list

RPUSH key value [value ...]
summary: Append one or multiple values to a list

LPOP key
summary: Remove and get the first element in a list

RPOP key
summary: Remove and get the last element in a list
```

可以实现两种数据结构：

- 栈
- 队列

案例

```shell
127.0.0.1:6379> LPUSH k1 a b c d e f
(integer) 6
127.0.0.1:6379> RPUSH k2 a b c d e f
(integer) 6
127.0.0.1:6379> LPOP k1 //后进先出，栈
"f"
127.0.0.1:6379> RPOP k1 //先进先出，队列
"a"
```

### 2.3 LRANGE、LINDEX、LSET

格式

```
LRANGE key start stop
summary: Get a range of elements from a list
```

案例

```shell
127.0.0.1:6379> LPUSH k1 a b c d e f
(integer) 6
127.0.0.1:6379> LRANGE k1 0 -1
1) "f"
2) "e"
3) "d"
4) "c"
5) "b"
6) "a"

127.0.0.1:6379> RPUSH k2 a b c d e f
(integer) 6
127.0.0.1:6379> LRANGE k2 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"
6) "f"

127.0.0.1:6379> LINDEX k1 2
"d"
127.0.0.1:6379> LINDEX k1 -1
"a"
127.0.0.1:6379> LSET k1 3 X
OK
127.0.0.1:6379> LRANGE k1 0 -1
1) "f"
2) "e"
3) "d"
4) "X"
5) "b"
6) "a"
```

### 2.4 LREM、LINSERT 

**支持数组**

127.0.0.1:6379> LPUSH k3 1 a 2 b 3 c 4 a 5 e 6 a 7
(integer) 13
127.0.0.1:6379> LRANGE k3 0 -1
 1) "7"
 2) "a"
 3) "6"
 4) "e"
 5) "5"
 6) "a"
 7) "4"
 8) "c"
 9) "3"
10) "b"
11) "2"
12) "a"
13) "1"

127.0.0.1:6379> LREM k3 2 k3 // 移除前两个a
(integer) 0

127.0.0.1:6379> LREM k3 -2 k3 // 移除后两个a
(integer) 0



127.0.0.1:6379> LINSERT k3 AFTER 7 a
(integer) 12
127.0.0.1:6379> LINSERT k3 BEFORE 4 a
(integer) 13
127.0.0.1:6379> LRANGE k3 0 -1
 1) "7"
 2) "a"
 3) "6"
 4) "e"
 5) "5"
 6) "a"
 7) "4"
 8) "c"
 9) "3"
10) "b"
11) "2"
12) "a"
13) "1" 

 ### 2.5 LLEN

```
127.0.0.1:6379> LLEN k3
(integer) 13
```

### 2.5 BLPOP

**阻塞、单播队列**

client1阻塞等待：

```
127.0.0.1:6379> BLPOP b1 0
```

client2阻塞等待:

```
127.0.0.1:6379> BLPOP b1 0
```

client3

```
127.0.0.1:6379> RPUSH b1 hello
(integer) 1
```

此时client1

<img src="https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200707220037491.png"  align="left"/>

```
127.0.0.1:6379> RPUSH b1 world
(integer) 1
```

此时client2

<img src="https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200707220218395.png"  align="left"/>

### 2.6 LTRIM

```
127.0.0.1:6379> LPUSH k1 0  1 2 3 4 5 6 7 8
(integer) 9
127.0.0.1:6379> LTRIM k1 2 6
OK
127.0.0.1:6379> LRANGE k1 0 -1
1) "6"
2) "5"
3) "4"
4) "3"
5) "2"
```

## 3 Lists应用场景

官网：

正如你可以从上面的例子中猜到的，list可被用来实现聊天系统。还可以作为不同进程间传递消息的队列。关键是，你可以每次都以原先添加的顺序访问数据。这不需要任何SQL ORDER BY 操作，将会非常快，也会很容易扩展到百万级别元素的规模。

例如在评级系统中，比如社会化新闻网站 reddit.com，你可以把每个新提交的链接添加到一个list，用LRANGE可简单的对结果分页。

在博客引擎实现中，你可为每篇日志设置一个list，在该list中推入博客评论，等等。