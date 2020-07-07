# Stream

Stream是一组用来处理数组、集合的API。

### Stream特性

1. 不是数据结构，没有内部存储
2. 不支持索引访问
3. 延迟计算
4. 支持并行
5. 很容易生成数组或集合（List、Set）
6. 支持过滤、查找、转换、汇总、聚合等操作

### Stream运行机制

Stream分为源source，中间操作，终止操作

流的源可以是一个数组、一个集合、一个生成器方法，一个I/O通道等等。

一个流可以有零个或者多个中间操作，每个中间操作都会返回一个新的流，供下一个操作使用。一个流指挥有一个终止操作。

Stream只有遇到终止操作，它的源才开始执行遍历操作。

### Stream的创建

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

   

