# Redis作为LRU算法缓存使用

如何配置并且将Redis当做缓存来使用，通过限制内存及自动回收键。

Redis中文网址：http://redis.cn/topics/lru-cache.html

## 1 Redis缓存热数据

Redis作为缓存保留热数据有两种逻辑：

- 业务逻辑：人为规定过期时间

- 业务运转：内存有限，随访问变化淘汰冷数据

![image-20201120225745037](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201120225745037.png)

### 1.1 业务逻辑

Redis中文官网：http://redis.cn/commands/expire.html

#### 1.1.1 Keys的过期时间

通常Redis keys创建时没有设置相关过期时间。他们会一直存在，除非使用显示的命令移除，例如，使用`DEL`命令。

`EXPIRE`一类命令能关联到一个有额外内存开销的key。当key执行过期操作时，Redis会确保按照规定时间删除他们。

key的过期时间和永久有效性可以通过`EXPIRE`和`PERSIST`命令（或者其他相关命令）来进行更新或者删除过期时间。



key的时间有效期测试

1. key的时间有效期会随着访问延长吗？

   答：不会！

   ```powershell
   # 先给k1设置20s有效期
   127.0.0.1:6379> set k1 aa ex 20
   OK
   127.0.0.1:6379> ttl k1
   (integer) 14
   127.0.0.1:6379> ttl k1
   (integer) 2
   127.0.0.1:6379> ttl k1
   (integer) 0
   # k1时间过期
   127.0.0.1:6379> ttl k1
   (integer) -2
   # 有效期时间之后key不存在
   127.0.0.1:6379> get k1 
   (nil)
   ```

2. 在有效期内发生写会剔除有效期时间，EXPIRE

   ```powershell
   # 设置k1
   127.0.0.1:6379> set k1 aa
   OK
   # 此时k1有效期是永久值
   127.0.0.1:6379> ttl k1
   (integer) -1
   # 人为为k1设置时间，30s
   127.0.0.1:6379> EXPIRE k1 30
   (integer) 1
   # 倒计时
   127.0.0.1:6379> ttl k1
   (integer) 28
   # 存活时间内写新值
   127.0.0.1:6379> set k1 bbb
   OK
   # k1有效期时间被剔除，重新为永久值
   127.0.0.1:6379> ttl k1
   (integer) -1
   # 查询是新值
   127.0.0.1:6379> get k1 
   "bbb"
   ```

3. 倒计时`EXPIREAT`，时间不会延长

   ```powershell
   127.0.0.1:6379> SET k1 aaa
   OK
   # 查看当前时间戳
   127.0.0.1:6379> TIME
   1) "1605838194"
   2) "36051"
   # 为k1设置倒计时
   127.0.0.1:6379> EXPIREAT k1 1605838294
   (integer) 1
   127.0.0.1:6379> TTL k1
   (integer) 65
   127.0.0.1:6379> TTL k1
   (integer) 58
   127.0.0.1:6379> TTL k1
   (integer) 27
   # k1过期
   127.0.0.1:6379> TTL k1
   (integer) -2
   ```

4. 定时，时间不会延长

**所以在写业务逻辑时自己补全，若更新再去设置存活时间**

#### 1.1.2 过期精度

在 Redis 2.4 及以前版本，过期期时间可能不是十分准确，有0-1秒的误差。

从 Redis 2.6 起，过期时间误差缩小到0-1毫秒。

#### 1.1.3 过期和持久

Keys的过期时间使用Unix时间戳存储(从Redis 2.6开始以毫秒为单位)。这意味着即使Redis实例不可用，时间也是一直在流逝的。

要想过期的工作处理好，计算机必须采用稳定的时间。 如果你将RDB文件在两台时钟不同步的电脑间同步，有趣的事会发生（所有的 keys装载时就会过期）。

即使正在运行的实例也会检查计算机的时钟，例如如果你设置了一个key的有效期是1000秒，然后设置你的计算机时间为未来2000秒，这时key会立即失效，而不是等1000秒之后。

#### 1.1.4 Redis key过期删除策略

Redis keys过期有两种方式：

- 被动方式

  官网：当一些客户端尝试访问它时，key会被发现并主动的过期。

  《Redis设计与实现》：放任键过期不管，但是每次从键空间中获取键时，都检查取得的键是否过期，如果过期的话，就删除该键；如果没有过期，就返回该键。

- 主动方式

  当然，这样是不够的，因为有些过期的keys，永远不会访问他们。 无论如何，这些keys应该过期，所以定时随机测试设置keys的过期时间。所有这些过期的keys将会从密钥空间删除。

  具体就是Redis每秒10次做的事情：

  1. 测试随机的20个keys进行相关过期检测。
  2. 删除所有已经过期的keys。
  3. 如果有多于25%的keys过期，重复步奏1.

  这是一个平凡的概率算法，基本上的假设是，我们的样本是这个密钥控件，并且我们不断重复过期检测，直到过期的keys的百分百低于25%,这意味着，在任何给定的时刻，最多会清除1/4的过期keys。

### 1.2 业务运转

#### 1.2.1 内存大小

maxmemory配置指令用于配置Redis存储数据时指定限制的内存大小

#### 1.2.2 淘汰策略

当达到maxmemory限制的最大值的时候，由maxmemory-policy配置指令来指定Redis的回收策略。

- volatile-lru：采用最近使用最少的淘汰策略，Redis将回收那些超时的（仅仅是超时的）键值对，也就是它只淘汰那些超时的键值对。
- allkeys-lru：采用最近最少使用的淘汰策略，Redis将对所有（不仅仅是超时的）的键值对采用最近最少使用的淘汰策略。
- volatile-lfu：采用最近最不常用的淘汰策略，所谓最近最不常用，也就是一定时期内被访问次数最少的。Redis将回收超时的键值对。
- allkeys-lfu：采用最近最不常用的淘汰策略，Redis将对所有的键值对采用最近最不常用的淘汰策略。
- volatile-random：采用随机淘汰策略删除超时的键值对。
- allkeys-random：采用随机淘汰策略删除所有的键值对，这个策略不常用。
- volatile-ttl：采用删除存活时间最短的键值对策略。
- noeviction：不淘汰任何键值对，当内存满时，如果进行读操作，例如get命令，它将正常工作，而做写操作，它将返回错误，也就是说，当Redis采用这个策略内存达到最大的时候，它就只能读不能写了。

如果没有键满足回收的前提条件的话，策略**volatile-lru**, **volatile-random**以及**volatile-ttl**就和noeviction 差不多了。

选择正确的回收策略是非常重要的，这取决于你的应用的访问模式，不过你可以在运行时进行相关的策略调整，并且监控缓存命中率和没命中的次数，通过RedisINFO命令输出以便调优。

一般的经验规则:

- 使用**allkeys-lru**策略：当你希望你的请求符合一个幂定律分布，也就是说，你希望部分的子集元素将比其它其它元素被访问的更多。如果你不确定选择什么，这是个很好的选择。.
- 使用**allkeys-random**：如果你是循环访问，所有的键被连续的扫描，或者你希望请求分布正常（所有元素被访问的概率都差不多）。
- 使用**volatile-ttl**：如果你想要通过创建缓存对象时设置TTL值，来决定哪些对象应该被过期。

**allkeys-lru** 和 **volatile-random**策略对于当你想要单一的实例实现缓存及持久化一些键时很有用。不过一般运行两个实例是解决这个问题的更好方法。

为了键设置过期时间也是需要消耗内存的，所以使用**allkeys-lru**这种策略更加高效，因为没有必要为键取设置过期时间当内存有压力时。

## 2 Maxmemory配置实战

### 2.1 配置文件配置

通过配置文件redis.conf可以设置该指令

- maxmemory
- maxmemory-policy noeviction
- maxmemory-samples 5

```powershell
############################## MEMORY MANAGEMENT ################################
#

# Set a memory usage limit to the specified amount of bytes.
# When the memory limit is reached Redis will try to remove keys
# according to the eviction policy selected (see maxmemory-policy).
#
# If Redis can't remove keys according to the policy, or if the policy is
# set to 'noeviction', Redis will start to reply with errors to commands
# that would use more memory, like SET, LPUSH, and so on, and will continue
# to reply to read-only commands like GET.
#
# This option is usually useful when using Redis as an LRU or LFU cache, or to
# set a hard memory limit for an instance (using the 'noeviction' policy).
#
# WARNING: If you have replicas attached to an instance with maxmemory on,
# the size of the output buffers needed to feed the replicas are subtracted
# from the used memory count, so that network problems / resyncs will
# not trigger a loop where keys are evicted, and in turn the output
# buffer of replicas is full with DELs of keys evicted triggering the deletion
# of more keys, and so forth until the database is completely emptied.
# In short... if you have replicas attached it is suggested that you set a lower
# limit for maxmemory so that there is some free RAM on the system for replica
# output buffers (but this is not needed if the policy is 'noeviction').
#
# maxmemory <bytes>

# MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
# is reached. You can select among five behaviors:
#
# volatile-lru -> Evict using approximated LRU among the keys with an expire sett
.
# allkeys-lru -> Evict any key using approximated LRU.
# volatile-lfu -> Evict using approximated LFU among the keys with an expire sett
.
# allkeys-lfu -> Evict any key using approximated LFU.
# volatile-random -> Remove a random key among the ones with an expire set.
# allkeys-random -> Remove a random key, any key.
# volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
# noeviction -> Don't evict anything, just return an error on write operations.
#
# LRU means Least Recently Used
# LFU means Least Frequently Used
#
# Both LRU, LFU and volatile-ttl are implemented using approximated
# randomized algorithms.
#
# Note: with any of the above policies, Redis will return an error on write
#       operations, when there are no suitable keys for eviction.
#
#       At the date of writing these commands are: set setnx setex append
#       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
#       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
#       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
#       getset mset msetnx exec sort
#
# The default is:
#
# maxmemory-policy noeviction
# LRU, LFU and minimal TTL algorithms are not precise algorithms but approximatee
d
# algorithms (in order to save memory), so you can tune it for speed or
# accuracy. For default Redis will check five keys and pick the one that was
# used less recently, you can change the sample size using the following
# configuration directive.
#
# The default of 5 produces good enough results. 10 Approximates very closely
# true LRU but costs more CPU. 3 is faster but not very accurate.
#
# maxmemory-samples 5

# Starting from Redis 5, by default a replica will ignore its maxmemory setting
# (unless it is promoted to master after a failover or manually). It means
# that the eviction of keys will be just handled by the master, sending the
# DEL commands to the replica as keys evict in the master side.
#
# This behavior ensures that masters and replicas stay consistent, and is usualll
y
# what you want, however if your replica is writable, or you want the replica too
 have
# a different memory setting, and you are sure all the writes performed to the
# replica are idempotent, then you may change this default (but be sure to underr
stand
# what you are doing).
#
# Note that since the replica by default does not evict, it may end using more
# memory than the one set via maxmemory (there are certain buffers that may
# be larger on the replica, or data structures may sometimes take more memory ann
d so
# forth). So make sure you monitor your replicas and make sure they have enough
# memory to never hit a real out-of-memory condition before the master hits
# the configured maxmemory setting.
#
# replica-ignore-maxmemory yes
```

### 2.2 运行时配置

使用CONFIG SET命令来进行运行时配置

## 3 近似LRU算法

**LRU算法或者TTL算法都是不精确的算法，而是一个近似算法。**

Redis的LRU算法并非完整的实现。这意味着Redis并没办法选择最佳候选来进行回收，也就是最久未被访问的键。相反它会尝试运行一个近似LRU的算法，通过对少量keys进行**取样**，然后回收其中一个最好的key（被访问时间较早的）。

Redis LRU有个很重要的点，你通过调整每次回收时检查的**采样数量**，以实现**调整**算法的精度。这个参数可以通过以下的配置指令调整:

```
maxmemory-samples 5
```

LRU，LFU和最小TTL算法不是精确算法，而是近似值算法（以节省内存），因此您可以对其进行调整以提高速度或准确性。默认情况下，Redis将检查五个键并选择一个使用较少，您可以使用以下方法更改样本大小配置指令。

默认值5产生足够好的结果。 10非常接近真正的LRU，但需要更多的CPU， 3更快，但不是很准确。
