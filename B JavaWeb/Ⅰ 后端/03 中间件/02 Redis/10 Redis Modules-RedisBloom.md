# RedisBloom 布隆过滤器

Redis Modules英文官网：https://redis.io/modules	

RedisBloom github：https://github.com/RedisBloom/RedisBloom

## 1 布隆过滤器

### 1.1 布隆过滤器作用

**布隆过滤器解决的问题：解决缓存穿透**

什么是缓存穿透？

正常情况下，查询的数据都存在，如果请求一个不存在的数据，也就是缓存和数据库都查不到这个数据，每次都会去数据库查询，这种查询不存在数据的现象我们称为缓存穿透。

### 1.2 布隆过滤器原理

![image-20201119220806700](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201119220806700.png)

## 2 布隆过滤器安装

1. wget https://github.com/RedisBloom/RedisBloom/archive/master.zip

2. 如果没有unzip，下载

   ```
   yum install unzip
   ```

   已安装，解压mater.zip

   ```
   unzip master.zip
   ```

3. ```
   cd RedisBloom-master/
   make
   ```

   会产生一个redisbloom.so扩展库

4. 将redisbloom.socopy至Redis安装目录下

   ```
   cp redisbloom.so /opt/soft/redis/
   ```

5. 停止Redis服务

   ```
   server redis_6379 stop
   ```

6. 启动Redis，挂载布隆过滤器（也可以配置配置文件）

   ```
   redis-server --loadmodule /opt/soft/redis/redisbloom.so
   ```

7. 简单使用

   ```
   # 布隆过滤器添加abc
   127.0.0.1:6379> BF.ADD ooxx abc
   (integer) 1
   # 判断是否存在abc
   127.0.0.1:6379> BF.EXISTS ooxx abc
   (integer) 1
   # 判断是否存在def
   127.0.0.1:6379> BF.EXISTS ooxx def
   (integer) 0
   ```

## 3 布隆过滤器架构设计

- 客户端实现布隆算法，自己承载bitmap，Redis只做缓存
- 客户端实现布隆算法，Redis实现bitmap和缓存
- 客户端不做任何实现，Redis实现布隆和bitmap

![image-20201119220720470](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201119220720470.png)

## 4 布隆过滤器可能出现的问题

可能出现的情况：

两种情况：

1. 发生缓存穿透，查询的值不存在，client端增加redis中的key/value标记，其他客户端下次查询直接访问到key，不走布隆过滤器了
2. 数据库可能脱离了Redis，数据库独自增加了元素，就必须对布隆过滤器添加元素，别人请求的时候才能查到数据，其实就是双写，既要写数据库还要写布隆。

## 4 布隆/布谷鸟过滤器

自己百度学习面试！！！