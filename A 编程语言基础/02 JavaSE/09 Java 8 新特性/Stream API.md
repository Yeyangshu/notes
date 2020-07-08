# Stream

A sequences of elements supporting sequential and parallel aggregate operations. 

Stream是一组用来处理数组、集合的API。

优点：

1. 函数式语言。使用stream接口可以告别 for 循环
2. 多核友好。parallel()

## 1 Stream特性

1. 不是数据结构，没有内部存储
2. 不支持索引访问
3. 延迟计算
4. 支持并行
5. 很容易生成数组或集合（List、Set）
6. 支持过滤、查找、转换、汇总、聚合等操作

## 2 Stream运行机制

Stream分为源source，中间操作，终止操作

流的源可以是一个数组、一个集合、一个生成器方法，一个I/O通道等等。

一个流可以有零个或者多个中间操作，每个中间操作都会返回一个新的流，供下一个操作使用。一个流指挥有一个终止操作。

Stream只有遇到终止操作，它的源才开始执行遍历操作。

## 3 Stream的创建

1. 通过数组创建

   ```java
   public static void main(String[] args) {
       gen01();
   }
   
   // 通过数组创建
   static void gen01() {
       String[] strings = {"a", "b", "c", "d"};
       Stream<String> stream = Stream.of(strings);
       stream.forEach(System.out::println);
       // 相当于fori遍历
       //for (int i = 0; i < strings.length; i++) {
       //System.out.println(strings[i]);
       //}
   }
   控制台:
       a
       b
       c
       d
   ```

   

2. 通过集合创建

   ```java
   public static void main(String[] args) {
       gen02();
   }
   
   // 通过集合创建
   static void gen02() {
       List<String> list = Arrays.asList("1", "2", "3", "4");
       Stream<String> stream = list.stream();
       stream.forEach(System.out::println);
   }
   控制台:
       1
       2
       3
       4
   ```

   

3. 通过Stream.generate方法创建

   ```java
   public static void main(String[] args) {
       gen03();
   }
   
   // 通过Stream.generate创建
   static void gen03() {
       Stream<Integer> stream = Stream.generate(() -> 1);
       stream.limit(10).forEach(System.out::println);
   }
   
   控制台:
       1
       1
       1
       1
       1
       1
       1
       1
       1
       1
   ```

   

4. 通过Stream.iterate方法创建

   ```java
   public static void main(String[] args) {
       gen04();
   }
   
   // 通过Stream.iterate创建
   static void gen04() {
       Stream<Integer> iterate = Stream.iterate(1, x -> x + 1);
       iterate.limit(10).forEach(System.out::println);
   }
   控制台:
       1
       2
       3
       4
       5
       6
       7
       8
       9
       10
   ```

   

5. 通过其他API创建

   ```java
   public static void main(String[] args) {
       gen05();
   }
   
   // 通过其他API创建
   static void gen05() {
       String string = "abcd";
       IntStream intStream = string.chars();
       intStream.forEach(System.out::println);
   }
   
   控制台:
       97
       98
       99
       100
   ```


## 4 Stream常用API

### 4.1 中间操作

- 过滤 filter
- 去重 distinct
- 排序 sorted
- 截取 limit、skip
- 转换 map、flatMap
- 其他 peek

### 4.2 终止操作

- 循环 forEach
- 计算 min、max、count、average
- 匹配 anyMatch、allMatch、noneMatch、findFirst、findAny
- 汇聚 reduce
- 收集器 toArray、collect

## 5 流实例

中间操作：如果调用方法之后返回的结果是Stream对象就意味着是一个中间操作。

### 5.1 filter

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
list.stream().filter(x -> x % 2 == 0).forEach(System.out::println);

控制台：
    2
    4
    6
    8
```

### 5.2 distinct

```java
Stream.of(1,2,3,4,5,4,4,3,2,5,6,4).distinct().forEach(System.out::println);

控制台：
    1
    2
    3
    4
    5
    6
```

### 5.3 sorted

```java
Stream.of("java", "c", "python", "php").sorted().forEach(System.out::println);

控制台：
c
java
php
python

Stream.of("java", "c", "python", "php").sorted((a, b) -> a.length() - b.length()).forEach(System.out::println);

控制台：
c
php
java
python

```

### 5.4 limit

```java
Stream.iterate(1, x -> x + 1).limit(5).forEach(System.out::println);

控制台：
    1
    2
    3
    4
    5
```

### 5.5 skip

```java
Stream.iterate(1, x -> x + 1).skip(5).limit(5).forEach(System.out::println);

控制台：
    6
    7
    8
    9
    10
```

### 5.6 map

```java
String str = "11,22,33,44,55";
Stream.of(str.split(",")).map(x -> Integer.valueOf(x)).forEach(System.out::println);

控制台：
    11
    22
    33
    44
    55
```

### 5.7 peek

```java
System.out.println(Stream.of(str.split(",")).peek(System.out::println).mapToInt(Integer::valueOf).sum());

控制台：
    11
    22
    33
    44
    55
    165
```

### 5.8 综合实例

StreamDemo

```java
import java.util.Arrays;
import java.util.HashSet;
import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class StreamDemo {
    public static void main(String[] args) {

        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);

        // 中间操作：如果调用方法之后返回的结果是Stream对象就意味着是一个中间操作
        list.stream().filter(x -> x % 2 == 0).forEach(System.out::println);
        // 集合中偶数的个数
        long count = list.stream().filter(x -> x % 2 == 0).count();
        System.out.println(count);
        // 集合中偶数的和
        int sum = list.stream().filter(x -> x % 2 == 0).mapToInt(x -> x).sum();
        System.out.println(sum);


        // 求集合中最大值
        Optional<Integer> max = list.stream().max((a, b) -> a - b);
        System.out.println(max.get());
        // 求集合中最小值
        Optional<Integer> min = list.stream().min((a, b) -> a - b);
        System.out.println(min.get());

        Optional<Integer> any = list.stream().findAny();
        System.out.println("findAny:" + any.get());

        Optional<Integer> first = list.stream().findFirst();
        System.out.println("findFirst:" + first.get());

        Stream<Integer> integerStream = list.stream().filter(i -> {
            System.out.println("运行代码");
            return i % 2 == 0;
        });
        System.out.println("integerStream:" + integerStream.findFirst().get());

        // 不使用 max 和 min 求最大值最小值
        Optional<Integer> sortedMin = list.stream().sorted().findFirst();
        System.out.println("sortedMin:" + sortedMin.get());
        Optional<Integer> sortedMax = list.stream().sorted((a, b) -> b - a).findFirst();
        System.out.println("sortedMax:" + sortedMax.get());

        Stream.of("java", "c", "python", "php").sorted().forEach(System.out::println);
        Stream.of("java", "c", "python", "php").sorted((a, b) -> a.length() - b.length()).forEach(System.out::println);

        // 将集合中的元素进行过滤同时返回一个集合对象
        List<Integer> collect = list.stream().filter(x -> x % 2 == 0).collect(Collectors.toList());
        collect.forEach(System.out::println);
        // 去重
        Stream.of(1, 2, 3, 4, 5, 4, 4, 3, 2, 5, 6, 4).distinct().forEach(System.out::println);
        new HashSet<>(Arrays.asList(1, 2, 3, 4, 5, 4, 4, 3, 2, 5, 6, 4)).forEach(System.out::println);

        // 打印20~30集合数据
        Stream.iterate(1, x -> x + 1).limit(50).skip(20).limit(10).forEach(System.out::println);

        String str = "11,22,33,44,55";
        System.out.println(Stream.of(str.split(",")).mapToInt(x -> Integer.valueOf(x)).sum());
        System.out.println(Stream.of(str.split(",")).mapToInt(Integer::valueOf).sum());
        System.out.println(Stream.of(str.split(",")).map(x -> Integer.valueOf(x)).mapToInt(x -> x).sum());
        System.out.println(Stream.of(str.split(",")).map(Integer::valueOf).mapToInt(x -> x).sum());

        // 创建一组自定义对象
        String string = "java,python,php";
        Stream.of(string.split(",")).map(x -> new Person(x)).forEach(System.out::println);
        Stream.of(string.split(",")).map(Person::new).forEach(System.out::println);
        Stream.of(string.split(",")).map(x -> Person.build(x)).forEach(System.out::println);
        Stream.of(string.split(",")).map(Person::build).forEach(System.out::println);

        System.out.println(Stream.of(str.split(",")).peek(System.out::println).mapToInt(Integer::valueOf).sum());

        System.out.println(list.stream().allMatch(x -> x >= 0));
    }
}
```

Person类

```java
public class Person {
    private String name;

    public static Person build(String name) {
        Person p = new Person();
        p.setName(name);
        return p;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Person(String name) {
        this.name = name;
    }

    public Person() {
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                '}';
    }
}
```