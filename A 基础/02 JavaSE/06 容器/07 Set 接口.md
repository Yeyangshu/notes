# Set

Set接口：

- 存储一组唯一，无序的对象
- 存入和取出的顺序不一定一致
- 操作数据的方法与List类似，Set接口丌存在get()方法

## 1 Set接口类图

![image-20201120002839495](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201120002839495.png)

## 2 Set接口的功能方法

```java
public interface Set<E> extends Collection<E> {
    
}
```

方法详解：



## 3 Set接口实现类

### 3.1 HashSet

HashSet采用Hashtable哈希表存储结构

- 优点：添加速度快，查询速度快，删除速度快

- 缺点：无序

### 3.2 LinkedHashSet

LinkedHashSet采用哈希表存储结构，同时使用链表维护次序

- 有序（添加顺序）

### 3.3 TreeSet

TreeSet采用二叉树（红黑树）的存储结构

- 优点：有序（排序后的升序）查询速度比List快

- 缺点：查询速度没有HashSet快