## Redis三种情况

### 1.1 缓存击穿

前提：肯定已经发生了高并发

某一个key过期

### 1.2 缓存穿透

从业务接收查询的是你系统根本不存在的数据



解决方案

### 1.3 缓存雪崩

什么时候会发生雪崩？

1. 零点

2. 与时点性无关

   设置过期时间

## 分布式锁