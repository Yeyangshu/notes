# Java基础知识面试题

## 1 基本概念

### 1.1 Java语言有哪些特点？

1. Java是面向对象的语言（封装、继承、多态）

   > 《Java编程思想》：“Everything is object”。

2. 平台无关性

   “一次编译，到处运行”，Java虚拟机把Java代码编译字节码文件，在JVM上解释执行。

3. 安全性和健壮性

4. 支持网络编程

### 1.2 Java与C++有什么不同？

### 1.3 为什么说Java语言“编译与解释并存”？

### 1.4 main方法

### 1.5 Java程序初始化顺序

### 1.6 Java中的作用域

 ### 1.7 反射

### 1.8 Object 类的常见方法总结

### 1.9 “==”与“equals”

### 1.10 “hashcode”与“equals”

### 1.11 泛型

泛型的主要目的之一就是用来指定容器要持有什么类型的对象，而且由编译器来保证类型的正确性

### Jdk1.8种的stream有用过吗，stream的并行操作原理，stream并行的线程池是从哪里来的

### Java中的静态方法只有一个实例，如果想用多个实例怎么办？

### Java中的代理有几种实现方式？

### 字符型常量和字符串常量的区别?

字符常量相当于一个整形值（ASCII 码），可以参加表达式运算

字符串常量代表一个地址值（该字符串在内存中存放位置）

```java
public static void main(String[] args) {
    // 字符类型可以进行运算
    char a = 91;
    char b = (char) (a + 6);
    System.out.println(b);

    // 字符串常量代表一个地址值
    String[] abc = {"abc"};
    System.out.println(abc);

    /**
     * a
     * [Ljava.lang.String;@6ff3c5b5
     */
}
```

### 自增自减运算符

符号在前就先加/减，符号在后就后加/减

```java
public static void main(String[] args) {
    // 先赋值，再自增
    int a = 10;
    int b = a++;
    // 11
    System.out.println(a);
    // 10
    System.out.println(b);

    // 先自增，再赋值
    int c = 10;
    int d = ++c;
    // 11
    System.out.println(c);
    // 11
    System.out.println(d);
}
```

### continue、break、和 return 的区别是什么？

return：跳出所在方法，结束该方法的运行。return 一般有两种用法：

1. `return;` ：直接使用 return 结束方法执行，用于没有返回值函数的方法
2. `return value;` ：return 一个特定值，用于有返回值函数的方法

break：中断语句，指跳出整个循环体，继续执行循环下面的语句。

continue：指跳出当前的这一次循环，继续下一次循环。

```java
public static void main(String[] args) {
    int i;
    for (i = 1; i <= 10; i++) {
        if (i % 3 == 0) {
            break;
        }
        System.out.println("i=" + i);
    }
    System.out.println("循环中断，i=" + i);
    /**
     * i=1
     * i=2
     * 循环中断，i=3
     */

    for (i = 1; i <= 10; i++) {
        if (i % 3 == 0) {
            continue;
        }
        System.out.println("i=" + i);
    }
    System.out.println("循环中断，i=" + i);
    /**
     * i=1
     * i=2
     * i=4
     * i=5
     * i=7
     * i=8
     * i=10
     * 循环中断，i=11
     */
}
```

### ==和 equals 的区别

1. **`==`** 

   `==`运算符用来比较两个变量的值是否相等。也就是说，该运算符比较变量对应的内存所存储的数值是否相同。

   - 如果两个变量是基本数据

     可以直接使用`==`运算符来比较对应的值是否相等。

   - 如果两个变量是对象

     如果一个变量指向的数据是对象（引用类型），那么此时涉及了两块内存：对象占用内存（堆内存）和变量占用的内存。变量所对应的内存所存储的数值就是对象占用内存的首地址。对于指向对象类型的变量，如果要比较两个变量是否指向同一块存储空间，可以用`==`运算符进行比较，如果要比较这两个对象的内容是否相等，就不能用`==`进行比较。

2. `equals()` 

   `equals()` 是Object类提供的方法之一。每一个Java类都继承自Object类，所以每一个对象都具有`equals()` 方法。

   `Object`类`equals()`方法

   ```java
   public boolean equals(Object obj) {
        return (this == obj);
   }
   ```

   `equals()` 方法存在两种使用情况：

   - 情况 1：类没有覆盖 `equals()`方法。

     在没有覆盖`equals()` 方法的情况下，`equals()` 与`==`一样，比较的是引用。

   - 情况 2：类覆盖了 `equals()`方法。

     可以覆盖 `equals()`方法来两个对象的内容相等；若它们的内容相等，则返回 true（即，认为这两个对象相等）。

     例如String类的`equals()`用于比较两个独立对象的内容是否相等。

测试案例：

```java
public static void main(String[] args) {
        // true，基本数据类型可以直接使用==
        System.out.println(42 == 42.0);

        // String测试
        // 此时字符串常量池没有"Hello"，堆内存创建一个String对象，字符串常量池创建"Hello"，a引用指向String对象
        String a = new String("Hello");
        // 此时字符串常量池已有"Hello"，堆内存创建一个String对象，字符串常量池不再创建，b引用指向String对象
        String b = new String("Hello");
        // aa指向常量池中"Hello"
        String c = "Hello";
        // bb指向常量池中"Hello"
        String d = "Hello";

        // false，比较的是对象的引用，非同一对象
        System.out.println(a == b);
        
        // true，比较的是常量池位置（引用）
        System.out.println(c == d);
        
        // true，String类重写了equals方法，比较的是内容，a和b内容是相同的
        System.out.println(a.equals(b));

        // false
        System.out.println(a == c);

    }
```

### hashCode()与 equals()

返回对象在内存中地址转换成的一个int值，所以，如果没有重新写hashCode方法，任何对象的hashCode方法都是不相等的。

hashCode()方法的返回值和equals()方法的关系如下：

如果x.equals(y)返回true，即对象根据equals()方法比较是相等的，那么调用这两个对象的hashCode()方法都必须返回相同的整数结果。

如果x.equals(y)返回false，即两个对象根据

### JDK动态代理和CGlib的区别？

Java动态代理是利用反射机制生成的一个代理接口的匿名类，在调用具体的方法前调用invokeHandler来处理

cglib动态代理是利用asm，对代理类的class对象文件加载进来，通过修改其字节码文件生成子类来处理。

### 自动装箱和拆箱

https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/basis/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86.md#13-%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B

### 为什么 Java 中只有值传递？

### 深拷贝 vs 浅拷贝

- 浅拷贝是创建一个新对象，这个对象有着原始对象属性值的一份精确拷贝。如果属性是基本类型，拷贝的就是基本类型的值，如果属性是引用类型，拷贝的就是内存地址 ，所以**如果其中一个对象改变了这个地址，就会影响到另一个对象**。

- 深拷贝是将一个对象从内存中完整的拷贝一份出来,从堆内存中开辟一个新的区域存放新对象,且**修改新对象不会影响原对象**。

### 反射的原理，反射创建类实例的三种方式是什么？

java反射机制是在运行当中，对任意一个类来说，能够知道它的所有属性和方法，都能调用它的任意一个属性和方法。

三种方式：getClass()方式，调用类的静态属性class，Class.forName()。

- Class class1 = Foo.class;
- Class class2 = foo1.getClass();
- Class class3 = Class.forName("com.imooc.reflect.Foo");

### Arrays.sort 实现原理和 Collections.sort 实现原理

Collection.sort()底层会调用Arrays.sort()，Arrays.sort()底层实现是TimeSort，TimeSort的算法就是先找到已经排好序数据的子序列，然后对剩余部分数据进行排序，然后在合并起来。

### foreach和while的区别(编译之后)

while会读一行输入，把它存入某个变量并执行循环体，然后再找其他行的输入，适用于不确定循环次数的情况；foreach是增强for循环，它是逐条读取，在循环开始前会将所有输入全部读入，适用于数组、集合等确定长度的情况；当输入内容非常大的时候foreach会非常占内存。

### 动态代理的几种方式

jdk动态代理和cglib动态代理。jdk动态代理是有java内部的反射机制实现的，前提是代理类和目标类必须实现统一的接口；cglib动态代理是借助asm来实现的。

### 反射中，Class.forName和ClassLoader区别

java类加载的过程包括：加载->验证->准备->解析->初始化->使用->卸载，而初始化就是激活java类中静态变量初始化代码和静态代码块，并初始化程序设置的变量值。

Class.forName会执行类的初始化，而ClassLoader不会执行类的初始化。

## 2 面向对象

### 2.1 面向对象和面向过程有什么区别？

### 2.2 面向对象有哪些特征？

面向对象的主要特征有抽象、继承、封装和多态

1. 抽象
2. 继承
3. 封装
4. 多态

### 2.3 组合和继承有什么区别？

### 2.4 多态的实现机制是什么？

### 2.5 重写和重载的区别是什么？

### 2.6 抽象类和接口有什么区别？

### 2.7 this和super有什么区别？

### 2.8 怎样声明一个类不会被继承？

## 3 关键字

### 3.1 break、continue和return有什么区别？

### 3.2 static关键字

### 3.3 final关键字

## 4 基本类型与运算

### 4.1 Java提供了哪些基本数据类型？

### 4.2 自动装箱和拆箱

### 4.3 ++i与i++有什么区别？

### 4.4 值传递与引用传递

## 5 字符串与数组

### 5.1 字符串的创建与存储机制

### 5.3 String、StringBuffer、StringBuilder区别？

## 6 容器

HashMap和ConcurrentHashMap区别（必考）

ConcurrentHashMap的数据结构（必考）

Atomic类如何保证原子性（CAS操作）（必考）

### HashMap底层数据结构，以及解决hash碰撞的方法

### Hashmap为什么要使用红黑树

### 集合类怎么解决高并发问题

### 队列的使用问题

### concurrenthashMap 的底层实现原理，是如何实现线程安全的？

### Java中的自增是线程安全的吗，如何实现线程安全的自增

### HashMap中jdk1.7与jdk1.8的区别

## 7 异常处理

### 7.1 Java异常类层次结构

### 7.2 运行时异常和普通异常有什么区别？

### 7.3 try、cath、finally

### 7.4 throw 和 throws 的区别是什么？**

throw 关键字用来抛出方法或代码块中的异常，受查异常和非受查异常都可以被抛出。
throws 关键字用在方法签名处，用来标识该方法可能抛出的异常列表。一个方法用 throws 标识了可能抛出的异常列表，调用该方法的方法中必须包含可处理异常的代码，否则也要在方法签名中用 throws 关键字声明相应的异常。

### 7.5 自定义异常的应用场景

## 8 输入输出流

### BIO、NIO、AIO有什么区别?



## 9 1.8的新特性

### Jdk1.8的completableFuture有用过吗？

