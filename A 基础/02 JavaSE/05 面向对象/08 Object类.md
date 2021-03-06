# Object类

Object 类是 Java 中所有类的父类，在 Java 中每个类都是由它扩展而来的。如果一个类没有使用 `extends` 关键词明确标识继承另一个类，那么这个类就默认继承 Object 类。

在 Java 中，只有基本类型不是对象，例如，数值、字符和布尔类型都不是对象。

所有的数组类型，不管是对象数组还是基本类型数组都扩展了 Object 类。

## 1 Object 类所有的方法

```java
public class Object {
    protected native Object clone();
    public boolean equals(Object obj) {
        return (this == obj);
    }
    protected void finalize() throws Throwable { }
    public final native Class<?> getClass();
    public native int hashCode();
    public final native void notify();
    public final native void notifyAll();
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
    public final void wait() throws InterruptedException {
        wait(0);
    }
    public final native void wait(long timeout) throws InterruptedException;
    public final void wait(long timeout, int nanos) throws InterruptedException {}
}
```

## 2 equals方法

Object 类中的 equals 方法用于检测一个对象是否等于另外一个对象。在 Object 类中，**这个方法将判断两个对象是否具有相同的引用**。如果两个对象具有相同的引用，它们一定是相等的。

然而，对于多数类来说，这种判断并没有什么意义。经常需要检测两个对象状态的相等性，如果两个对象的状态相等，就认为这两个对象是相等的。

Java语言规范`equals`方法具有下面的特性：

1. 自反性：对于任何非空引用x，x.queals(x)应该返回true。
2. 对称性：对于任何引用x和y，当且仅当y.queals(x)返回true，x.queals(y)也应该返回true。
3. 传递性：对于任何引用x、y和z，如果x.queals(x)应该返回true，y.queals(z)应该返回true，x.queals(z)也应该返回true。
4. 一致性：如果x和y引用的对象没有发生变化，反复调用x.queals(x)应该返回同样的结果。
5. 对于任意非空引用x，x.queals(null)应该返回false。

## 3 hashCode 方法

散列码是由对象导出的一个整型值。散列码是没有规律的。如果 x 和 y 是两个不同的对象，x.hashCode( ) 与 y.hashCode( ) 基本上不会相同。

例，String 类的 hashCode() 得到的散列码：

```java
System.out.println("Hello".hashCode());
System.out.println("World".hashCode());

/**
 * 69609650
 * 83766130
 */
```

由于hashCode方法定义在Object类中，因此每个对象都有一个默认的散列码，其值为对象的存储地址。

如果重新定义 equals 方法，就必须重新定义 hashCode 方法，以便用户可以将对象插入到散列码中。

## toString方法

返回表示对象值的字符串

