# Stream

A sequences of elements supporting sequential and parallel aggregate operations. 

Stream是一组用来处理数组、集合的API。

优点：

1. 函数式语言。使用stream接口可以告别 for 循环
2. 多核友好。parallel()

从支持数据处理操作的源生成的元素序列

- 元素序列：集合讲的是数据，流讲的是计算
- 源：流会使用一个提供数据的源，如集合、数组或输入/输出资源。
- 数据处理操作：流的数据处理功能支持类似于数据库的操作，以及函数式编程语言中的常用操作，如filter、map、reduce、find、match、sort等。流操作可以顺序执行，也可并行执行。

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

### 3.1 通过数组创建流

可以使用静态方法Stream.of，通过显式值创建一个流。它可以接受任意数量的参数。

也使用empty得到一个空流

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

case02

```java
public static void main(String[] args) {
    Stream<String> stream = Stream.of("Hello", "World");
    stream.map(String::toLowerCase).forEach(System.out::println);
    // 空流
    Stream<String> emptyStream = Stream.empty();
}
// hello
// world
```

### 3.2 通过集合创建

可以使用静态方法Arrays.stream从数组创建一个流。它接受一个数组作为参数。

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

case02

```java
public static void main(String[] args) {
    int[] numbers = {1, 2, 3, 4, 5};
    int sum = Arrays.stream(numbers).sum();
    System.out.println(sum);
}
// 15
```

### 3.3 由文件生成流

Java中用于处理文件等I/O操作的NIO API（非阻塞I/O）已更新，以便利用StreamAPI。java.nio.file.Files中的很多静态方法都会返回一个流。例如，一个很有用的方法是Files.lines，它会返回一个由指定文件中的各行构成的字符串流。

你可以使用Files.lines得到一个流，其中的每个元素都是给定文件中的一行。然后，你可以对line调用split方法将行拆分成单词。应该注意的是，你该如何使用flatMap产生一个扁平的单词流，而不是给每一行生成一个单词流。最后，把distinct和count方法链接起来，数数流中有多少各不相同的单词。

### 3.4 由函数生成流：创建无限流

Stream API提供了两个静态方法来从函数生成流：Stream.iterate和Stream.generate。这两个操作可以创建所谓的无限流：不像从固定集合创建的流那样有固定大小的流。由iterate和generate产生的流会用给定的函数按需创建值，因此可以无穷无尽地计算下去！一般来说，应该使用limit(n)来对这种流加以限制，以避免打印无穷多个值。

#### 3.4.1 迭代

iterate方法接受一个初始值（在这里是0），还有一个依次应用在每个产生的新值上的Lambda（UnaryOperator<t>类型）。这里，我们使用Lambda n-> n+2，返回的是前一个元素加上2。因此，iterate方法生成了一个所有正偶数的流：流的第一个元素是初始值0。然后加上2来生成新的值2，再加上2来得到新的值4，以此类推。

```java
public static void main(String[] args) {
    Stream<Integer> numbers = Stream.iterate(0, n -> n + 2).limit(10);
    System.out.println(Arrays.toString(numbers.toArray()));
}
// [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

#### 3.4.2 生成

generate方法也可让你按需生成一个无限流。但generate不是依次对每个新生成的值应用函数的。它接受一个Supplier<T>类型的Lambda提供新的值。

```java
public static void main(String[] args) {
    Stream<Double> randoms = Stream.generate(Math::random).limit(5);
    System.out.println(Arrays.toString(randoms.toArray()));
}
// [0.578188424543552, 0.6351151398438128, 0.037347390817622395, 0.759188135257225, 0.6516852439800229]
```

### 3.5 通过其他API创建

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

## 5 使用流

- 筛选、切片和匹配

  如何选择流中的元素：用谓词筛选，筛选出各不相同的元素，忽略流中的头几个元素，或将流截短至指定长度。

- 查找、匹配和归约

- 使用数值范围等数值流

- 从多个源创建流

- 无限流

中间操作：如果调用方法之后返回的结果是Stream对象就意味着是一个中间操作。

### 5.1 筛选和切片

#### 5.1.1 filter 用谓词筛选

Streams接口支持filter方法（你现在应该很熟悉了）。该操作会接受一个谓词（一个返回boolean的函数）作为参数，并返回一个包括所有符合谓词的元素的流。

#### 5.1.2 distinct 筛选各异的元素 

流还支持一个叫作distinct的方法，它会返回一个元素各异（根据流所生成元素的hashCode和equals方法实现）的流，没有重复。

#### 5.1.3 limit 截短流

流支持limit(n)方法，该方法会返回一个不超过给定长度的流。所需的长度作为参数传递给limit。如果流是有序的，则最多会返回前n个元素

#### 5.1.4 skip 跳过元素

流还支持skip(n)方法，返回一个扔掉了前n个元素的流。如果流中元素不足n个，则返回一个空流。请注意，limit(n)和skip(n)是互补的！

```java
public static void main(String[] args) {
    List<Integer> integerList = Arrays.asList(1, 2, 2, 3, 3, 3, 4, 4, 4, 5, 6, 7, 8, 9);
    List<Integer> collect = integerList.stream()
        // 筛选，filter，boolean类型参数
        .filter(integer -> integer >= 2)
        // 去重
        .distinct()
        // 跳过前n个
        .skip(2)
        // 保留前n个
        .limit(5)
        .collect(Collectors.toList());
    System.out.println(collect);
}
// [4, 5, 6, 7, 8]
```

### 5.2 映射

#### 5.2.1 map 对流中每一个元素应用函数

流支持map方法，它会接受一个函数作为参数。这个函数会被应用到每个元素上，并将其映射成一个新的元素（使用映射一词，是因为它和转换类似，但其中的细微差别在于它是“创建一个新版本”而不是去“修改”）。

```java
public static void main(String[] args) {
    List<Integer> integerList = Arrays.asList(1, 2, 2, 3, 3, 3, 4, 4, 4, 5, 6, 7, 8, 9);
    List<Integer> collect = integerList.stream()
        .map(integer -> integer - 1)
        .collect(Collectors.toList());
    System.out.println(collect);
}
// [0, 1, 1, 2, 2, 2, 3, 3, 3, 4, 5, 6, 7, 8]
```

#### 5.2.2 flatMap 流的扁平化

使用flatMap方法的效果是，各个数组并不是分别映射成一个流，而是映射成流的内容。所有使用map(Arrays::stream)时生成的单个流都被合并起来，即扁平化为一个流。	

```java
public static void main(String[] args) {
    List<String> stringStream = Arrays.asList("Hello", "World");
    List<String> stringList = stringStream.stream()
        .map(String -> String.split(""))
        // 合并成一个流
        .flatMap(Arrays::stream)
        .collect(toList());
    System.out.println(stringList.toString());
}
// [H, e, l, l, o, W, o, r, l, d]
```

### 5.3 查找和匹配

#### 5.3.1 检查谓词是否至少匹配一个元素

anyMatch方法可以回答“流中是否有一个元素能匹配给定的谓词”。anyMatch方法返回一个boolean，因此是一个终端操作。

```java
public static void main(String[] args) {
    List<String> stringStream = Arrays.asList("Hello", "World");
    boolean isSuccess = stringStream.stream()
        .anyMatch(s -> s.equals("Hello"));
    System.out.println(isSuccess);
}
// true
```

#### 5.3.2 检查谓词是否匹配所有元素

allMatch：流中的元素是否都能匹配给定的谓词。

```java
public static void main(String[] args) {
    List<String> stringStream = Arrays.asList("Hello", "World");
    boolean isSuccess = stringStream.stream()
        .allMatch(s -> s.length() >= 5);
    System.out.println(isSuccess);
}
// true
```

和allMatch相对的是noneMatch。它可以确保流中没有任何元素与给定的谓词匹配

```java
public static void main(String[] args) {
    List<String> stringStream = Arrays.asList("Hello", "World");
    boolean isSuccess = stringStream.stream()
        .noneMatch(s -> s.length() >= 5);
    System.out.println(isSuccess);
}
// false
```

#### 5.3.3 查找元素

findAny方法将返回当前流中的任意元素。它可以与其他流操作结合使用。

```java
public static void main(String[] args) {
    List<String> stringStream = Arrays.asList("Hello", "World");
    Optional<String> optional = stringStream.stream()
        .filter(s -> s.equals("Hello"))
        .findAny();
    System.out.println(optional.get());
}
// Hello
```

#### 5.3.4 查找第一个元素

有些流有一个出现顺序（encounter order）来指定流中项目出现的逻辑顺序（比如由List或排序好的数据列生成的流）。对于这种流，你可能想要找到第一个元素。

```java
public static void main(String[] args) {
    List<String> stringStream = Arrays.asList("Hello", "World");
    Optional<String> optional = stringStream.stream()
        .findFirst();
    System.out.println(optional.get());
}
// Hello
```

何时使用findFirst和findAny

你可能会想，为什么会同时有findFirst和findAny呢？答案是并行。找到第一个元素在并行上限制更多。如果你不关心返回的元素是哪个，请使用findAny，因为它在使用并行流时限制较少。

### 5.4 归约 reduce

#### 5.4.1 元素求和

reduce

reduce接受两个参数：

- 一个初始值，这里是0；
- 一个BinaryOperator<T>来将两个元素结合起来产生一个新值，这里我们用的是lambda (a, b)-> a+b。

```java
public static void main(String[] args) {
    List<Integer> integerList = Arrays.asList(4, 5, 3, 9);
    int i = integerList.stream()
        .reduce(0, (a, b) -> a + b);
    System.out.println(i);
}
// 21
```

如何对一个数字流求和的。

首先，0作为Lambda（a）的第一个参数，从流中获得4作为第二个参数（b）。0+4得到4，它成了新的累积值。然后再用累积值和流中下一个元素5调用Lambda，产生新的累积值9。接下来，再用累积值和下一个元素3调用Lambda，得到12。最后，用12和流中最后一个元素9调用Lambda，得到最终结果21。

使用方法引用让这段代码更简洁。

```java
public static void main(String[] args) {
    List<Integer> integerList = Arrays.asList(4, 5, 3, 9);
    int i = integerList.stream()
        .reduce(0, Integer::sum);
    System.out.println(i);
}
```

##### 无初始值

reduce还有一个重载的变体，它不接受初始值，但是会返回一个Optional对象

为什么它返回一个`Optional<Integer>`呢？考虑流中没有任何元素的情况。

```java
public static void main(String[] args) {
    List<Integer> integerList = Arrays.asList(4, 5, 3, 9);
    Optional<Integer> integer = integerList.stream()
        .reduce(Integer::sum);
    System.out.println(integer.get());
}
// 21
```

无值

```java
public static void main(String[] args) {
    List<Integer> integerList = Arrays.asList(4, 5, 3, 9);
    Optional<Integer> intValue = integerList.stream()
        // 过滤大于10的数，此时返回空集合
        .filter(integer -> integer > 10)
        .reduce(Integer::sum);
    System.out.println(intValue.orElse(0));
}
// 0
```

#### 5.4.2 最大值和最小值

reduce接受两个参数：

- 一个初始值
- 一个Lambda来把两个流元素结合起来并产生一个新值

```java
public static void main(String[] args) {
    List<Integer> integerList = Arrays.asList(4, 5, 3, 9);
    Integer max = integerList.stream()
        .reduce(4, Integer::max);
    System.out.println(max);
    Integer min = integerList.stream()
        .reduce(4, Integer::min);
    System.out.println(min);
}
// 9
// 3
```

### 5.5 综合实例

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

IBM文档：https://developer.ibm.com/zh/articles/j-lo-java8streamapi/