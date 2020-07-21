# 哨兵（Sentinel）模式

Redis中文文档：http://redis.cn/topics/sentinel.html

## 1 Redis哨兵简介

Redis的哨兵系统用于管理多个Redis服务器，该系统执行以下三个任务：

- 监控
- 提醒
- 自动故障转移

## 2 Redis哨兵使用

### 2.1 获取 Sentinel

### 2.2 启动 Sentinel

### 2.3 配置 Sentinel

新建三个sentinel配置文件，例26379.conf、26380.conf、26381.conf监听主机，内容为

```
port 26379/26380/26381
sentinal monitor mymaster 127.0.0.1 6379 2
```











