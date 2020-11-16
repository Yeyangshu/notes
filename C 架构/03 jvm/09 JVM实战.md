# JVM调优

## 1 JVM参数总结

官方网址：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html

![image-20201114225211396](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201114225211396.png)

![image-20201114225229015](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201114225229015.png)

![image-20201114225248127](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201114225248127.png)



- -XX:+UseSerialGC ：这个等于Serial New + Serial Old组合
- -XX:+UseParNewGC：ParNew+Serial Old，这个组合已经很少在用了
- -XX:+UseConcMarkSweepGC：ParNew+CMS+Serial Old
- -XX:+UseParallelGC：默认，Parallel Scavenge+Parallel Old
- -XX:+UseParallelOldGC：Parallel Scavenge+Parallel Old
- -XX:+UseG1GC：G1
- Linux没有找到默认的GC查看方式，Windows会打印UseParallelGC
  - Java：+XX:PrintCommandLineFlags-version
  - 通过GC的日志来分辨

JVM调优第一步：了解JVM常用命令参数

Hotspot参数分类：

- 标准：`-`开头，所有的Hotspot都支持
- 非标准：`-X`开头，特定版本Hotspot支持特定命令
- 不稳定：`-XX`开头，下个版本可能取消



区分两个概念：

- 内存泄漏（Memory leak）：有一块内存无人占用
- 内存溢出（Out Of Memory，简称OOM）：不断产生对象，程序运行要用到的[内存](https://baike.baidu.com/item/内存/103614)大于能提供的最大内存。

## 2 GC日志



GC日志

```
33.125: [GC [DefNew: 3324K->152K(3712K), 0.0025925 secs] 3324K->152K(11904K), 0.0031680 secs]
100.667: [Full GC [Tenured: 0K->210K(1024K), 0.0149142 secs] 4603K->210K(19456K), [Perm: 2999K->2999K(21248)], 0.0150007 secs] [Times:user=0.01 sys=0.00, real=0.02 secs]
```

- 最前面的数字：“33.125:”和“100.667:”代表了GC发生的时间，这个数字的含义是从Java虚拟机启动以来经过的秒数。

- GC日志开头的“[GC”和“[Full GC”说明了这次垃圾手机的停顿类型，而不是用来区分新生代GC还是老年代GC的。如果有Full，说明这次GC是发生了 Stop-The-World的，例如下面这段新生代收集器ParNew的日志也会出现“Full GC（这一般是因为出现了分配担保失败之类的问题，所以才导致STW）”。如果是调用System.gc()方法所触发的收集，那么在这里将显示“Full GC(System)”

  ```
  [Full GC 283.736]:[ParNew: 261599K->261599K(261952K), 0,0000288 secs]
  ```

一旦产生内存溢出，会把整个堆heap dump下来

![image-20201114210336341](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201114210336341.png)



![image-20201114210120010](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201114210120010.png)

## 3 调优前期

### 3.1 两个重要概念

调优前两个重要概念：

- 吞吐量：用户代码时间/(用户代码执行时间+垃圾回收时间)
- 响应时间：STW越短，响应时间越好

所谓调优，首先确定追求什么？吞吐量优先还是响应时间优先？还是在满足一定条件下，要求达到多大的吞吐量？每个项目的具体要求不一样，性能达到要求就可以，达不到好好调优，实在是调不了，加CPU加内存。

问题：科学计算，吞吐量，数据挖掘，吞吐量一般很简单：先选定垃圾收集器（PS+PO）；网站、带界面的程序（GUI）、对外提供的相应API，这类的服响应时间要短。

### 3.2 什么是调优？

三大类：

- 根据需求进行JVM规划和预调优
- 优化运行JVM运行环境（慢、卡顿）
- 解决JVM运行环境过程中出现的各种问题（OOM）

### 3.3 调优，从规划开始

- 调优，从业务场景开始，没有业务场景的调优都是耍流氓

- 无监控（指的是压力测试，能看到结果）不调优
- 调优步骤
  1. 熟悉业务场景（选定垃圾回收器，没有最好的垃圾回收器，只有最适合的垃圾回收器）
     - 响应时间、停顿时间（需要给客户做响应）
     - 吞吐量
  2. 选择回收器组合
  3. 计算内存需求（经验值 1.5G 16G）内存需求弹性需求大，内存小，回收快也能承受，所以内存大小没有一定的规定
  4. 选定CPU（越高越好，按照预算来）
  5. 设定年代大小，升级年龄
  6. 设置日志参数

## 4 调优案例

### 4.1 预调优案例

#### 4.1.1 案例一：垂直电商，最高每日百万订单，处理订单系统需要什么样的服务器配置？

这个问题比较业余，因为很多不同的服务器配置都能支持，一小时360000集中时间段，100个订单/秒，（找一小时内的高峰期，1000订单/秒），

例：每天100万订单，每小时不会产生很高的并发量，我们寻找高峰时间，做一个假设100万订单有72万订单在高峰期产生，比如一小时平均36万订单，所以我们的内存选择是按照巅峰时间选择的。

好多时候就是经验值拿来做压力测试，实在不行加CPU加内存

非要计算：一个订单产生需要多少内存？512K*10000，500M内存，一秒钟250订单，我只要不到一秒时间处理完成，这里垃圾回收就ok了，所以这一块难于估计

专业一点问法：要求响应时间100ms

#### 4.1.2 案例二：12306遭遇春节大规模抢票应该如何支撑？

这个类似架构设计，跟预调优不是很有关系

12036应该是中国并发量最大的秒杀网址，号称最高100W并发量

CDN->LVS->NGINX->业务系统->每台机器1W并发（10K问题，单机10K问题已被解决，Redis是其中的关键）100台机器装Redis就搞定了

大流量的处理方式：分而治之

### 4.2 优化环境

#### 4.2.1 案例一

有一个50万



#### 4.2.2 案例二（面试高频）

系统CPU经常100%，如何调优？（高频）

步骤：

1. 找出哪个进程CPU高（top）
2. 该进程中哪个线程cpu高（top -Hp）
3. 导出该线程的堆栈（jstack）
4. 查找哪个方法（栈帧）消耗时间（jstack）
5. 工作线程占比高、垃圾回收线程占比高

#### 4.2.3 案例三

系统内存飙高，如何查找问题？（高频）

1. 堆栈比较多，导出堆内存（jmap）
2. 分析（jhat、jvisual、mat、jprofiler...）

#### 4.2.4 案例四

如何监控JVM？

jstat、jvisualvm、jprofiler、arths、top



##### 4.2.4.1 运行Java文件

```java
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.concurrent.ScheduledThreadPoolExecutor;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class T15_FullGC_Problem01 {

    private static class CardInfo {
        BigDecimal price = new BigDecimal(0.0);
        String name = "张三";
        int age = 5;
        Date birthdate = new Date();

        public void m() {

        }
    }

    private static ScheduledThreadPoolExecutor executor = new ScheduledThreadPoolExecutor(50,
            new ThreadPoolExecutor.DiscardOldestPolicy());

    public static void main(String[] args) throws Exception {
        executor.setMaximumPoolSize(50);

        for (; ; ) {
            modelFit();
            Thread.sleep(100);
        }
    }

    private static void modelFit() {
        List<CardInfo> taskList = getAllCardInfo();
        taskList.forEach(info -> {
            // do something
            executor.scheduleWithFixedDelay(() -> {
                //do sth with info
                info.m();

            }, 2, 3, TimeUnit.SECONDS);
        });
    }

    private static List<CardInfo> getAllCardInfo() {
        List<CardInfo> taskList = new ArrayList<>();

        for (int i = 0; i < 100; i++) {
            CardInfo ci = new CardInfo();
            taskList.add(ci);
        }

        return taskList;
    }
}
```

将文件上传至Linux服务器，然后编译文件`javac T15_FullGC_Problem01.java`

然后运行此文件`java -Xms200M -Xmx200M -XX:+PrintGC T15_FullGC_Problem01`

##### 4.2.4.2 运维团队收到告警信息，`top -c`查看进程，观察到问题，内存不断增长，CPU占用率居高不下

使用`top`命令查看进程CPU占比，可以看到进程`49404`CPU占用了飙高

```
top - 03:11:46 up 6 days, 16:37,  0 users,  load average: 2.24, 1.44, 0.88
Tasks:  60 total,   1 running,  59 sleeping,   0 stopped,   0 zombie
%Cpu(s):  3.8 us,  1.7 sy,  0.0 ni, 94.4 id,  0.0 wa,  0.0 hi,  0.0 si,  0.1 st
KiB Mem :  8388608 total,   468148 free,  5050644 used,  2869816 buff/cache
KiB Swap:  2097148 total,  2097148 free,        0 used.   468148 avail Mem 

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                               
 49404 log       20   0 1871880 231360  16004 S 251.8  2.8  10:49.84 java                                                                                                                                  
 71674 admin     20   0 3704448  26032  15572 S   3.0  0.3   0:00.09 java                                                                                                                                  
   566 root      20   0   61388   6328   5492 S   1.0  0.1  97:35.79 alisentry_cli                                                                                                                         
  3275 admin     20   0 8843504 4.114g  50444 S   1.0 51.4  99:24.85 java                                                                                                                                  
```

##### 4.2.4.3 使用`top -Hp 49404`查看进程里线程

`top -Hp 49404`查看进程里线程哪一个线程CPU和内存占比高

```
top - 03:11:31 up 6 days, 16:37,  0 users,  load average: 2.15, 1.37, 0.85
Threads:  67 total,   1 running,  66 sleeping,   0 stopped,   0 zombie
%Cpu(s):  3.6 us,  1.6 sy,  0.0 ni, 94.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.1 st
KiB Mem :  8388608 total,   479160 free,  5039932 used,  2869516 buff/cache
KiB Swap:  2097148 total,  2097148 free,        0 used.   479160 avail Mem 

   PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                                                                                
 49410 log       20   0 1871880 231360  16004 S 94.0  2.8   1:19.12 java                                                                                                                                   
 49425 log       20   0 1871880 231360  16004 S  3.3  2.8   0:10.70 java                                                                                                                                   
 49432 log       20   0 1871880 231360  16004 S  3.3  2.8   0:10.65 java                                                                                                                                   
 49434 log       20   0 1871880 231360  16004 S  3.3  2.8   0:10.65 java          	
```

##### 4.2.4.4 找Java进程，jps和jstack

为什么阿里规范线程的名字（尤其是线程池）都要写有意义的名称，怎样自定义线程池里的线程名称？

jps（JVM ProcessStatus Tool）：虚拟机进程状况工具，可以列出正在运行的虚拟机进程，并显示虚拟机执行主类（Main Class，main()函数所在的类）名称以及这些进程的本地虚拟机唯一ID（LVMID，LocalVirtual Machine Identifier）。

jstack：Java堆栈跟踪工具，重点关注`WAITING`、`BLOCKED`，例如`waiting on condition [0x00007fbfd24f3000]`

作业：

1. 写一个死锁程序，用`jstack`观察
2. 写一个程序，一个线程持有锁不释放，其他线程等待

##### 4.2.4.5 jindo pid

##### 4.2.4.6 jstat -gc动态观察gc情况

jstat -gc动态观察gc情况，阅读gc日志发现频繁gc/arthas观察/jconsule/jvisualVM/jprofiler（收费、最好用）

面试问怎么定位OOM问题的？

1. 一定不能答使用图像界面，图形界面最大的用途是系统还未上线，进行本地压力测试，看jvm的响应，发现问题能及时解决

2. 已经上线的系统使用命令行去定位问题

##### 4.2.4.7 jmap -histo 4665 | grep -20查找有多少对象生成

##### 4.2.4.8 jmap dump文件，使用MAT/jhat进行dump文件分析

jmap（Memory Map for Java）：Java内存映像工具

`jmap -dump:format=b,file=xxxx pid;`

也可以设置jvm参数

```
-XX:+HeapDumpOnOutOfMemoryError
```

##### 4.2.4.9 找代码的问题，解决问题

