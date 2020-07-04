[TOC]

# 一、JVM实战

## 1 目标

- 降低gc次数时间
- 降低FullGC几率

## 2 情景

### 2.1 停机或延迟过长？

- 有GC详细日志？先tail GC日志看最近GC的Cause，看不出来再用工具分析GC日志
- 无GC详细日志？jstat -gc(具体请查看man jstat)
- java内存泄漏？分析heap dump
- native泄漏？先确定是不是Xmx配大了，也许并没有泄漏，然后尝试排查direct memory泄漏
   load高？性能差？

### 2.2 重启load高？

- top -H看热点线程，然后jstack看是否C2编译线程（注意十进制十六进制转化）
- 峰值性能差？JInsight

### 2.3 挂了？JVM进程还活着

- 线程死锁？分析jstack日志
- 从ps看进程状态有"T"字? 用kill -CONT [pid]恢复
- 怀疑codecache问题? 应用日志codecache满或者JInsight

### 2.4 挂了？JVM进程消失了

- 有crash日志？自己试着读下crash日志
- 无crash日志,有JavaCore文件? $JAVA_HOME/bin/jstack $JAVA_HOME/bin/java core.xxx看jstack，是否有无限递归
- 无crash日志,无JavaCore文件? dmesg | grep java 看是不是被oom killer杀死了，可能是内存配置太大。

## 3 常见问题

1. Young GC时间过长，十几秒几十秒，不正常，一般是打开了swap。
2. Young GC过于频繁，调大堆或某个代。
3. CMS/FullGC过于频繁，是调大堆或者old

