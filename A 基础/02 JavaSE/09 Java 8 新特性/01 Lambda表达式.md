# Lambda表达式

## 1 Lambda表达式介绍

Lambda表达式是Java8最重要的新功能之一。使用Lambda表达式可以替代只有一个抽象函数的接口实现，告别匿名内部类，代码看起来更简洁易懂。Lambda表达式同时还提升了对集合、框架的迭代、遍历、过滤数据的操作。

## 2 Lambda表达式的特点

1. 函数式编程
2. 参数类型自动推断
3. 代码量少、简洁

## 3 Lambda表达式应用场景
任何有**函数式接口**的地方

## 4 函数式接口
java.util.function 它包含了很多类，用来支持 Java的 函数式编程，该包中的函数式接口有：
菜鸟教程：https://www.runoob.com/java/java8-functional-interfaces.html

Supplier 代表一个输出

Consumer 代表一个输入
BiConsumer 代表两个输出

Function 代表一个输入，一个输出（一般输入和输出是不同类型的）
UnaryOperater 代表一个输入，一个输出（输入和输出是相同类型的）

BiFunction 代表两个输入，一个输出（一般输入和输出是不同类型的）
BinaryOperater 代表两个输入，一个输出（输入和输出是相同类型的）

## 5 方法的引用
方法的引用是用来直接访问类或者实例的已经存在的方法或者构造方法，方法引用提供了一种引用而不是执行的方式，如果抽象方法的实现恰好可以使用调用另一个方法实现，就有可能可以使用方法引用。
### 5.1 方法引用的分类

|     类型     |        语法        |        对应的Lambda表达式         |
| :----------: | :----------------: | :-------------------------------: |
| 静态方法引用 | 类名::staticMethod | (args) -> 类名.staticMethod(args) |
| 实例方法引用 |  inst::instMethod  |  (args) -> inst.instMethod(args)  |
| 对象方法引用 |  类名::instMethod  |  (args) -> 类名.instMethod(args)  |
| 构造方法引用 |     类名::new      |     (args) -> new 类名(args)      |

- 静态方法的引用
如果函数式接口的实现可以通过调用一个静态方法来实现，那么就可以使用静态方法引用
- 实例方法的引用
如果函数式接口的实现恰好可以通过一个实例的类型方法来实现，那么就可以使用实例方法引用
- 对象方法的引用
抽象方法的第一个参数类型刚好是实例方法的类型，抽象方法剩余的参数恰好可以当做实例方法的参数。如果函数式接口的实现能由上面说的实例方法调用来实现的话，那么就可以使用对象方法引用。
- 构造方法的引用
如果函数式接口的实现恰好可以通过一个类的构造方法来实现，那么就可以使用构造方法的引用

### 5.3 方法引用的案例

#### 5.3.1 静态方法引用
```java
package com.yeyangshu.study.javaSE.lambda;

import java.util.concurrent.Callable;
import java.util.function.BiFunction;
import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Supplier;

/**
 * 静态方法引用
 */
public class LambdaClass_StaticMethod {
    public static void main(String[] args) throws Exception {
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("running1...");
            }
        };
        runnable.run();

        Runnable runnable2 = () -> {
            System.out.println("running2...");
        };
        runnable2.run();

        Runnable runnable3 = () -> System.out.println("running3...");
        runnable3.run();


        //-----------------
        Callable callable = new Callable() {
            @Override
            public String call() throws Exception {
                return "Hello World";
            }
        };
        System.out.println(callable.call());

        Callable callable2 = () -> {
            return "Hello World";
        };
        System.out.println(callable2.call());

        Callable callable3 = () -> "Hello World";
        System.out.println(callable3.call());

        // 自定义FunctionalInterface
        StudentDao studentDao = new StudentDao() {
            @Override
            public void insert(Student student) {
                System.out.println("插入学生:yeyangshu");
            }
        };
        studentDao.insert(new Student());

        StudentDao studentDao1 = (student) -> {
            System.out.println("插入学生:yeyangshu");
        };
        studentDao1.insert(new Student());

        // 自动识别类型
        StudentDao studentDao2 = (student) -> System.out.println("插入学生:yeyangshu");
        studentDao2.insert(new Student());


        // Supplier
        Supplier supplier = () -> {return "Supplier";};
        System.out.println(supplier.get());
        Supplier supplier1 = () -> "Supplier";
        System.out.println(supplier1.get());

        // Consumer 代表一个输入
        Consumer<String> consumer = (str -> System.out.println(str));
        consumer.accept("Consumer");

        // Function
        Function<String, Integer> function = (str) -> {
            return str.length();
        };
        System.out.println("str length: " + function.apply("Hello World"));

        //
        BiFunction<String, String, Integer> biFunction = (a, b) -> a.length() + b.length();
        System.out.println(biFunction.apply("Hello", "World"));
        BiFunction<String, String, Integer> biFunction1 = LambdaClass_StaticMethod::getLength;
        System.out.println(biFunction1.apply("Hello", "World"));

    }

    static int getLength(String s1, String s2) {
        return s1.length() + s2.length();
    }
}
```

#### 5.3.2 实例方法引用
```
package com.yeyangshu.study.javaSE.lambda;

import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Supplier;

/**
 * 实例方法引用
 */
public class LambdaInst_InstMethod {
    public String put() {
        return "put...";
    }

    public void getSize(int size) {
        System.out.println("size：" + size);
    }

    public String toUpperCase(String s) {
        return s.toUpperCase();
    }

    public static void main(String[] args) {
        //-------------无参方法-----------------
        // 1.最原始
        System.out.println(new LambdaInst_InstMethod().put());
        // 2.Supplier Lambda表达式第一种写法
        Supplier<String> supplier1 = () -> new LambdaInst_InstMethod().put();
        // 3.Supplier Lambda表达式第二种写法
        Supplier<String> supplier2 = () -> {
            return new LambdaInst_InstMethod().put();
        };
        // 4.实例方法引用
        Supplier<String> supplier3 = new LambdaInst_InstMethod()::put;
        System.out.println(supplier1.get());
        System.out.println(supplier2.get());
        System.out.println(supplier3.get());

        //System.out::printf 也是对象实例


        //-------------有参方法-----------------
        // 1.Consumer Lambda表达式写法
        Consumer<Integer> consumer = (size) -> new LambdaInst_InstMethod().getSize(size);
        // 2.实例方法引用写法一
        Consumer<Integer> consumer1 = new LambdaInst_InstMethod()::getSize;
        // 创建唯一的LambdaInst_InstMethod对象
        LambdaInst_InstMethod lambdaInstInstMethod = new LambdaInst_InstMethod();
        // 3.实例方法引用写法二
        Consumer<Integer> consumer2 = lambdaInstInstMethod::getSize;
        consumer.accept(123);
        consumer1.accept(123);
        consumer2.accept(123);


        //-------------有参方法-----------------
        // 1.字符串调用自己的toUpperCase方法
        Function<String, String> function = (str) -> str.toUpperCase();
        // 2.Lambda表达式写法
        Function<String, String> function1 = (str) -> lambdaInstInstMethod.toUpperCase(str);
        // 3.实例方法引用写法一
        Function<String, String> function2 = new LambdaInst_InstMethod()::toUpperCase;
        // 4.实例方法引用写法二
        Function<String, String> function3 = lambdaInstInstMethod::toUpperCase;
        System.out.println(function.apply("beijing"));
        System.out.println(function1.apply("beijing"));
        System.out.println(function2.apply("beijing"));
        System.out.println(function3.apply("beijing"));

    }
}
```
#### 5.3.3 静态方法引用
```
package com.yeyangshu.study.javaSE.lambda;

import java.util.concurrent.Callable;
import java.util.function.BiFunction;
import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Supplier;

/**
 * 静态方法引用
 */
public class LambdaClass_StaticMethod {
    public static void main(String[] args) throws Exception {
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("running1...");
            }
        };
        runnable.run();

        Runnable runnable2 = () -> {
            System.out.println("running2...");
        };
        runnable2.run();

        Runnable runnable3 = () -> System.out.println("running3...");
        runnable3.run();


        //-----------------
        Callable callable = new Callable() {
            @Override
            public String call() throws Exception {
                return "Hello World";
            }
        };
        System.out.println(callable.call());

        Callable callable2 = () -> {
            return "Hello World";
        };
        System.out.println(callable2.call());

        Callable callable3 = () -> "Hello World";
        System.out.println(callable3.call());

        // 自定义FunctionalInterface
        StudentDao studentDao = new StudentDao() {
            @Override
            public void insert(Student student) {
                System.out.println("插入学生:yeyangshu");
            }
        };
        studentDao.insert(new Student());

        StudentDao studentDao1 = (student) -> {
            System.out.println("插入学生:yeyangshu");
        };
        studentDao1.insert(new Student());

        // 自动识别类型
        StudentDao studentDao2 = (student) -> System.out.println("插入学生:yeyangshu");
        studentDao2.insert(new Student());


        // Supplier
        Supplier supplier = () -> {return "Supplier";};
        System.out.println(supplier.get());
        Supplier supplier1 = () -> "Supplier";
        System.out.println(supplier1.get());

        // Consumer 代表一个输入
        Consumer<String> consumer = (str -> System.out.println(str));
        consumer.accept("Consumer");

        // Function
        Function<String, Integer> function = (str) -> {
            return str.length();
        };
        System.out.println("str length: " + function.apply("Hello World"));

        //
        BiFunction<String, String, Integer> biFunction = (a, b) -> a.length() + b.length();
        System.out.println(biFunction.apply("Hello", "World"));
        BiFunction<String, String, Integer> biFunction1 = LambdaClass_StaticMethod::getLength;
        System.out.println(biFunction1.apply("Hello", "World"));

    }

    static int getLength(String s1, String s2) {
        return s1.length() + s2.length();
    }
}
```
#### 5.3.4 构造方法引用
```
package com.yeyangshu.study.javaSE.lambda;

import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;
import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Supplier;

public class LambdaClass_New {
    public static void main(String[] args) {
        //-------------不带参数-----------------
        // 原来方法
        Supplier<Person> supplier = () -> new Person();
        supplier.get();
        // 构造方法的引用
        Supplier<Person> supplier1 = Person::new;
        supplier1.get();

        Supplier<List> listSupplier = ArrayList::new;
        Supplier<Set> setSupplier = HashSet::new;
        Supplier<Thread> threadSupplier = Thread::new;
        Supplier<String> stringSupplier = String::new;

        //-------------带参数-----------------
        Consumer<Integer> consumer = (age) -> new Account(age);
        Consumer<Integer> consumer1 = Account::new;
        consumer.accept(123);
        consumer1.accept(456);

        Function<String, Account> function = (str) -> new Account(str);
        Function<String, Account> function1 = Account::new;
        function.apply("123");
        function1.apply("456");
    }
}

class Person {
    public Person() {
        System.out.println("调用无参的构造方法");
    }
}
class Account {
    public Account() {
        System.out.println("调用无参的构造方法");
    }
    public Account(int age) {
        System.out.println("age 参数构造方法：" + age);
    }
    public Account(String str) {
        System.out.println("str 参数构造方法：" + str);
    }
}
```

