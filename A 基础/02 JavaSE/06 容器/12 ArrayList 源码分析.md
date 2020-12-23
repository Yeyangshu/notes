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