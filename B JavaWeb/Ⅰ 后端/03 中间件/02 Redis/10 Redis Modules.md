英文官网：https://redis.io/modules

# 1 布隆过滤器

## 1.1 布隆过滤器作用 

解决缓存穿透

![image-20200715235020381](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200715235020381.png)

可能出现的情况：

![image-20200715235939264](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200715235939264.png)

## 1.2 安装

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

6. 启动Redis，挂载布隆过滤器

   ```
   redis-server --loadmodule /opt/soft/redis/redisbloom.so
   ```

7. 简单使用

   ```
   127.0.0.1:6379> BF.ADD ooxx abc
   (integer) 1
   127.0.0.1:6379> BF.EXISTS ooxx abc
   (integer) 1
   127.0.0.1:6379> BF.EXISTS ooxx def
   (integer) 0
   ```

   

## 1.3 布隆过滤器架构

![image-20200715235204309](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200715235204309.png)

## 1.4 布隆/布谷鸟过滤器

自己百度学习面试！！！