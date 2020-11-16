# 内存屏障与JMM指令

JMM模型

## 1 硬件

### 1.1 硬件内存模型

![image-20201110232508047](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201110232508047.png)

### 1.2 缓存行

### 1.3 多线程环境下出现的问题

#### 1.3.1 数据一致性

- 老CPU数据一致性实现：使用总线锁锁住总线，使得其他CPU不能访问内存中的其他地址，效率低下
- 新CPU数据一致性实现：使用缓存锁（例如MESI，https://www.cnblogs.com/z00377750/p/9180644.html）+ 总线锁

#### 1.3.2 指令重排序问题

- 读乱序：读指令的同时可以执行不影响的其他指令

- 合并写：WC Buffer，写的时候可以进行合并写

CPU执行就是乱序的，根本原因就是CPU与内存、缓存的速度读取和写入处理速度上的差别

必须使用内存屏障（Memory Barriers，Intel称之为Memory Fence）指令来做好指令重排序，volatile底层实现

##### 1.3.2.1 如何保证特定情况下的不乱序？

硬件层面

- 第一种方式就是加锁
- 在指令级别加内存屏障
  - sfence：save fence，写屏障指令。在sfence指令前的写操作必须在sfence指令后的写操作前完成。
  - ifence：load fence，读屏障指令。在lfence指令前的读操作必须在lfence指令后的读操作前完成。
  - mfence：在mfence指令前得读写操作必须在mfence指令后的读写操作前完成。

## 2 JVM 上的 Memory Barrier

- LoadLoad Barriers

### 2.1 volatile实现细节

### 2.2 synchronized实现细节