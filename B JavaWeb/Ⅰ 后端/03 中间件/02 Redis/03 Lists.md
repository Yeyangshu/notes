# Lists 列表

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

### 2.2 LPUSH&RPUSH&LPOP&RPOP

```
127.0.0.1:6379> help lpush

  LPUSH key value [value ...]
  summary: Prepend one or multiple values to a list
  since: 1.0.0
  group: list

127.0.0.1:6379> help rpush

  RPUSH key value [value ...]
  summary: Append one or multiple values to a list
  since: 1.0.0
  group: list

127.0.0.1:6379> help lpop

  LPOP key
  summary: Remove and get the first element in a list
  since: 1.0.0
  group: list

127.0.0.1:6379> help rpop

  RPOP key
  summary: Remove and get the last element in a list
  since: 1.0.0
  group: list
```

LPUSH：将一个或多个值 `value` 插入到列表 `key` 的表头

- 如果有多个 `value` 值，那么各个 `value` 值按从左到右的顺序依次插入到表头

RPUSH：将一个或多个值 `value` 插入到列表 `key` 的表尾(最右边)。

- 如果有多个 `value` 值，那么各个 `value` 值按从左到右的顺序依次插入到表尾

LPOP：移除并返回列表 `key` 的头元素。

RPOP：移除并返回列表 `key` 的尾元素。



**可以实现两种数据结构：**

- **栈：同向命令实现，LPUSH+LPOP**
- **队列：反向命令实现，LPUSH+RPOP**

案例：

```powershell
# 此时链表排列顺序为 f e d c b a
127.0.0.1:6379> LPUSH k1 a b c d e f
(integer) 6
# 此时链表排列顺序为 a b c d e f
127.0.0.1:6379> RPUSH k2 a b c d e f
(integer) 6
# 后进先出，栈
127.0.0.1:6379> LPOP k1
"f"
# 先进先出，队列
127.0.0.1:6379> RPOP k1
"a"
```

### 2.3 LRANGE

```
127.0.0.1:6379> help lrange

  LRANGE key start stop
  summary: Get a range of elements from a list
  since: 1.0.0
  group: list
```

LRANGE：返回列表 `key` 中指定区间内的元素，区间以偏移量 `start` 和 `stop` 指定。可以使用正负索引。

案例：

```powershell
# 此时链表排列顺序为 f e d c b a
127.0.0.1:6379> LPUSH k1 a b c d e f
(integer) 6
127.0.0.1:6379> LRANGE k1 0 -1
1) "f"
2) "e"
3) "d"
4) "c"
5) "b"
6) "a"

# 此时链表排列顺序为 a b c d e f
127.0.0.1:6379> RPUSH k2 a b c d e f
(integer) 6
127.0.0.1:6379> LRANGE k2 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"
6) "f"
```

### 2.4 LINDEX&LSET

```
127.0.0.1:6379> help lindex

  LINDEX key index
  summary: Get an element from a list by its index
  since: 1.0.0
  group: list
  
127.0.0.1:6379> help lset

  LSET key index value
  summary: Set the value of an element in a list by its index
  since: 1.0.0
  group: list
```

LINDEX：返回列表 `key` 中，下标为 `index` 的元素。将列表 `key` 下标为 `index` 的元素的值设置为 `value` 。

LSET：将列表 `key` 下标为 `index` 的元素的值设置为 `value` 。

- 当 `index` 参数超出范围，或对一个空列表( `key` 不存在)进行 LSET 时，返回一个错误。



**可以实现的数据结构**

- **数组：LINDEX+LSET，都是对下标进行操作**

案例：

```powershell
# 此时链表排列顺序为 f(0) e(1) d(2) c(3) b(4) a(5)
127.0.0.1:6379> LPUSH k1 a b c d e f
(integer) 6
# 取下标为2的元素
127.0.0.1:6379> LINDEX k1 2
"d"
# 取列表最后一位元素
127.0.0.1:6379> LINDEX k1 -1
"a"

# 此时链表排列顺序为 f(0) e(1) d(2) c(3) b(4) a(5)
# 设置下标为3的元素的值为X
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

### 2.5 LREM&LINSERT

```
127.0.0.1:6379> help lrem

  LREM key count value
  summary: Remove elements from a list
  since: 1.0.0
  group: list
  
127.0.0.1:6379> help linsert

  LINSERT key BEFORE|AFTER pivot value
  summary: Insert an element before or after another element in a list
  since: 2.2.0
  group: list
```

LREM：根据参数 `count` 的值，移除列表中与参数 `value` 相等的元素。

`count` 的值可以是以下几种：

- `count > 0` : 从表头开始向表尾搜索，移除与 `value` 相等的元素，数量为 `count` 。
- `count < 0` : 从表尾开始向表头搜索，移除与 `value` 相等的元素，数量为 `count` 的绝对值。
- `count = 0` : 移除表中所有与 `value` 相等的值。

LINSERT：将值 `value` 插入到列表 `key` 当中，位于值 `pivot` 之前或之后。

- 当 `pivot` 不存在于列表 `key` 时，不执行任何操作。

- 当 `key` 不存在时， `key` 被视为空列表，不执行任何操作。

- 如果 `key` 不是列表类型，返回一个错误。

案例：

```powershell
# 此时链表排列顺序为 7 a 6 e 5 a 4 c 3 b 2 a 1
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

# 移除链表元素，count>0，从表头开始向表尾开始移除
# 此时链表排列顺序为 7 6 e 5 4 c 3 b 2 a 1
127.0.0.1:6379> LREM k3 2 a
(integer) 2
127.0.0.1:6379> LRANGE k3 0 -1
 1) "7"
 2) "6"
 3) "e"
 4) "5"
 5) "4"
 6) "c"
 7) "3"
 8) "b"
 9) "2"
10) "a"
11) "1"

# 添加链表元素
# 在元素6后面添加a，此时链表排列顺序为 7 6 a e 5 4 c 3 b 2 a 1
127.0.0.1:6379> LINSERT k3 after 6 a
(integer) 12
127.0.0.1:6379> LRANGE k3 0 -1
 1) "7"
 2) "6"
 3) "a"
 4) "e"
 5) "5"
 6) "4"
 7) "c"
 8) "3"
 9) "b"
10) "2"
11) "a"
12) "1"
# 在元素3前面添加a，此时链表排列顺序为 7 6 a e 5 4 c a 3 b 2 a 1
127.0.0.1:6379> LINSERT k3 before 3 a
(integer) 13
127.0.0.1:6379> LRANGE k3 0 -1
 1) "7"
 2) "6"
 3) "a"
 4) "e"
 5) "5"
 6) "4"
 7) "c"
 8) "a"
 9) "3"
10) "b"
11) "2"
12) "a"
13) "1"
```

### 2.6 LLEN

```
127.0.0.1:6379> help llen

  LLEN key
  summary: Get the length of a list
  since: 1.0.0
  group: list
```

LLEN：返回列表 `key` 的长度。

- 如果 `key` 不存在，则 `key` 被解释为一个空列表，返回 `0` .

- 如果 `key` 不是列表类型，返回一个错误。

案例：

```powershell
127.0.0.1:6379> LRANGE k3 0 -1
 1) "7"
 2) "6"
 3) "a"
 4) "e"
 5) "5"
 6) "4"
 7) "c"
 8) "a"
 9) "3"
10) "b"
11) "2"
12) "a"
13) "1"
# 返回k3的元素长度
127.0.0.1:6379> LLEN k3
(integer) 13
```

### 2.7 LTRIM

```
127.0.0.1:6379> help ltrim

  LTRIM key start stop
  summary: Trim a list to the specified range
  since: 1.0.0
  group: list
```

LTRIM：对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。

案例：

```powershell
127.0.0.1:6379> LPUSH k1 0 1 2 3 4 5 6 7 8
(integer) 9
# 此时链表排列顺序为 8(0) 7(1) 6(2) 5(3) 4(4) 3(5) 2(6) 1(7) 0(8)
# 将下标不在2-6的元素删除
127.0.0.1:6379> LTRIM k1 2 6
OK
127.0.0.1:6379> LRANGE k1 0 -1
1) "6"
2) "5"
3) "4"
4) "3"
5) "2"
```

### 2.8 BLPOP&BRPOP

```
127.0.0.1:6379> help blpop

  BLPOP key [key ...] timeout
  summary: Remove and get the first element in a list, or block until one is available
  since: 2.0.0
  group: list
  
127.0.0.1:6379> help brpop

  BRPOP key [key ...] timeout
  summary: Remove and get the last element in a list, or block until one is available
  since: 2.0.0
  group: list
```

BLPOP：列表的阻塞式(blocking)弹出原语，超时参数 `timeout` 接受一个以秒为单位的数字作为值。超时参数设为 `0` 表示阻塞时间可以无限期延长(block indefinitely) 。

- 它是 LPOP 命令的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被 BLPOP 命令阻塞，直到等待超时或发现可弹出元素为止。

- 当给定多个 key 参数时，按参数 key 的先后顺序依次检查各个列表，弹出第一个非空列表的头元素。
- 如果所有给定 key 都不存在或包含空列表，那么 BLPOP 命令将阻塞连接，直到等待超时，或有另一个客户端对给定 key 的任意一个执行 LPUSH 或 RPUSH 命令为止。



可以实现的数据结构：

- **阻塞、单播队列，FIFO：BLPOP**

案例：

client1阻塞等待：

```powershell
# 无限期阻塞
127.0.0.1:6379> BLPOP b1 0
```

client2阻塞等待:

```powershell
# 无限期阻塞
127.0.0.1:6379> BLPOP b1 0
```

client3

```powershell
# client3向b1添加"hello"
127.0.0.1:6379> RPUSH b1 hello
(integer) 1
```

此时client1接收到值

<img src="https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200707220037491.png"  align="left"/>

```powershell
127.0.0.1:6379> BLPOP b1 0
1) "b1"                          # 这里被 push 的是 b1
2) "hello"            			 # 被弹出的值
(26.97s)                         # 等待的秒数
```

此时client2还在阻塞

```powershell
# 无限期阻塞
127.0.0.1:6379> BLPOP b1 0
```

client3再向b1添加值

```powershell
# client3向b1添加"world"
127.0.0.1:6379> RPUSH b1 world
(integer) 1
```

此时client2

<img src="https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200707220218395.png"  align="left"/>

```powershell
127.0.0.1:6379> BLPOP b1 0
1) "b1"                          # 这里被 push 的是 b1
2) "world"            			 # 被弹出的值
(35.33s)                         # 等待的秒数
```

## 3 Lists应用场景

官网：

正如你可以从上面的例子中猜到的，list可被用来实现聊天系统。还可以作为不同进程间传递消息的队列。关键是，你可以每次都以原先添加的顺序访问数据。这不需要任何SQL ORDER BY 操作，将会非常快，也会很容易扩展到百万级别元素的规模。

例如在评级系统中，比如社会化新闻网站 reddit.com，你可以把每个新提交的链接添加到一个list，用LRANGE可简单的对结果分页。

在博客引擎实现中，你可为每篇日志设置一个list，在该list中推入博客评论，等等。

