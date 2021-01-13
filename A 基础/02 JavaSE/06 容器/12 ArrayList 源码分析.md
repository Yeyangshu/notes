# ArrayList源码

![image-20201121103132152](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201121103132152.png)

## 父类和接口

- java.util.AbstractList

  该抽象类是大部分List的共同父类，它提供了一些基本的方法封装，以及通用的迭代器实现。

- java.util.List

  列表标准接口，列表是一个有序集合，又被称为序列。该接口对它内部的每一个元素的插入位置都有精确控制，用户可以使用整数索引（index）来查询。

- java.util.RandomAccess

  这是一个标记性质的随机访问接口，它没有提供任何方法。如果一个类实现了这个接口，那么表示这个类使用索引遍历比迭代器要更快（ArrayList、CopyOnWriteArrayList、Stack和Vector都实现了这个接口）。

  测试

  ```java
  public static void main(String[] args) {
      // 数组大小
      int size = 100000000;
      // 向数组中添加2000000条数据
      List<String> list = new ArrayList<>(size);
      for (int i = 0; i < size; i++) {
          list.add("i：" + i);
      }
  
      // 计算索引遍历时间
      long start = System.currentTimeMillis();
      String str;
      for (int i = 0; i < size; i++) {
          str = list.get(i);
      }
      System.out.println("索引遍历耗时：" + (System.currentTimeMillis() - start));
  
      // 计算迭代器遍历时间
      start = System.currentTimeMillis();
      Iterator<String> iterator = list.iterator();
      while (iterator.hasNext()) {
          str = iterator.next();
      }
      System.out.println("迭代器遍历耗时：" + (System.currentTimeMillis() - start));
  }
  // 索引遍历耗时：26
  // 迭代器遍历耗时：27
  ```

- java.lang.Cloneable

  用于标记可克隆对象，是一个常见接口，没有实现该接口的对象在调用Object.clone()方法时会抛出异常。

- java.io.Serializable

  序列化标记接口，是一个常见接口，被此接口标记的类可以实现Java序列化和反序列化。该接口没有任何内容，但是Java序列化里有一些默认成员变量和默认方法，会在序列化和反序列化的时候调用到。主要有如下几个方法：

## 成员变量和常量

### 无参构造

无参构造函数创建的 ArrayList

```java
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {}
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

### add

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

此时 size 为 0，ensureCapacityInternal(1)

### ensureCapacityInternal 得到最小扩容量

```java
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

// 得到最小扩容量
// minCapacity=1
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        // 此时是空数组{}，DEFAULT_CAPACITY=10，与传入参数1进行比较，获取最大值，为最小扩容量10
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}
```

### ensureExplicitCapacity 判断是否需要扩容

```java
// 判断是否需要扩容
private void ensureExplicitCapacity(int minCapacity) {
    // 已对该列表进行结构修改的次数。 结构修改是指更改列表大小或以其他方式干扰列表的方式。
    modCount++;

    // overflow-conscious code
    // 第一次添加元素，此时判断 1-10>0
    if (minCapacity - elementData.length > 0)
        // 调用grow方法进行扩容，调用此方法代表已经开始扩容了
        grow(minCapacity);
}
```

- 当我们要 add 进第 1 个元素到 ArrayList 时，elementData.length 为 0 （因为还是一个空的 list），因为执行了 `ensureCapacityInternal()` 方法 ，所以 minCapacity 此时为 10。此时，`minCapacity - elementData.length > 0`成立，所以会进入 `grow(minCapacity)` 方法。
- 当 add 第 2 个元素时，minCapacity 为 2，此时 e lementData.length(容量)在添加第一个元素后扩容成 10 了。此时，`minCapacity - elementData.length > 0` 不成立，所以不会进入 （执行）`grow(minCapacity)` 方法。
- 添加第 3、4···到第 10 个元素时，依然不会执行 grow 方法，数组容量都为 10。

直到添加第 11 个元素，minCapacity(为 11)比 elementData.length（为 10）要大。进入 grow 方法进行扩容。

### grow 扩容

```java
rivate void grow(int minCapacity) {
    // overflow-conscious code
    // oldCapacity为旧容量，此时为10
    int oldCapacity = elementData.length;
    // 将oldCapacity右移一位，等价于oldCapacity*1.5，此时是15
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    // 如果新容量大于MAX_ARRAY_SIZE，hugeCapacity()方法来比较minCapacity和MAX_ARRAY_SIZE
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

当数组里面超过10个的时候，会进行扩容。

### System.arraycopy() 和 Arrays.copyOf()方法

```java
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
```

**区别：**

`arraycopy()`：需要目标数组，将原数组**拷贝**到你自己定义的数组里或者原数组，而且可以选择拷贝的起点和长度以及放入新数组中的位置 

`copyOf()` ：是系统自动在内部新建一个数组，并返回该数组。

## 