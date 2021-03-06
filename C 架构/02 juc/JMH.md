# JMH

## 一、JMH简介

## 二、JMH测试

### 2.1 创建maven项目，添加依赖

```xml
<!-- https://mvnrepository.com/artifact/org.openjdk.jmh/jmh-core -->
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.21</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.openjdk.jmh/jmh-generator-annprocess -->
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>1.21</version>
    <scope>test</scope>
</dependency>
```

### 2.2 IDEA安装JMH插件

![image-20200709230803175](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200709230803175.png)

### 2.3 由于用到了注解，打开运行程序注解配置

setting -> Build, Execution, Deployment -> Compiler -> Annotation Processors -> Enable Annotation Processing

![image-20200709230821115](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200709230821115.png)

### 2.4 定义需要测试类PS (ParallelStream)

```java
public class PS {
    static List<Integer> nums = new ArrayList<>();
    static {
        Random r = new Random();
        for (int i = 0; i < 10000; i++) {
            nums.add(1000000 + r.nextInt(1000000));
        }
    }
    static void foreach() {
        nums.forEach(v -> isPrime(v));
    }

    static void parallel() {
        nums.parallelStream().forEach(PS::isPrime);
    }

    static boolean isPrime(int num) {
        for (int i = 2; i < num/2; i++) {
            if (num % i == 0) {
                return false;
            }
        }
        return true;
    }
}
```

### 2.5 写单元测试

这个测试类一定要在test package下面

```java
public class PSTest {

    @Benchmark
    public void testForeach() {
        PS.foreach();
    }
}
```



### 2.6 运行测试类，如果遇到下面的错误

```
ERROR: org.openjdk.jmh.runner.RunnerException: ERROR: Exception while trying to acquire the JMH lock (C:\WINDOWS\/jmh.lock): C:\WINDOWS\jmh.lock (拒绝访问。), exiting. Use -Djmh.ignoreLock=true to forcefully continue.
	at org.openjdk.jmh.runner.Runner.run(Runner.java:216)
	at org.openjdk.jmh.Main.main(Main.java:71)
```

这个错误是因为JMH运行需要访问系统的TMP目录，解决办法是：

打开RunConfiguration -> Environment Variables -> include system environment viables

![image-20200709230841190](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200709230841190.png)

### 2.7 阅读测试报告



## 三、JMH中的基本概念

1. Warmup 预热，由于JVM中对于特定代码会存在优化（本地化），预热对于测试结果很重要
2. Mesurement 总共执行多少次测试
3. Timeout
4. Threads 线程数，由fork指定
5. Benchmark mode 基准测试的模式
6. Benchmark 测试哪一段代码

## 四、Next

官方样例： http://hg.openjdk.java.net/code-tools/jmh/file/tip/jmh-samples/src/main/java/org/openjdk/jmh/samples/