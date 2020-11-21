# List

List承诺可以将元素维护在特定的序列中。List接口在Collection的基础上添加了大量的方法，使得可以在List的中间插入和移动元素。

**List特点:有序，不唯一（可重复）**

## 1 List接口类图

![image-20201120002233889](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201120002233889.png)

## 2 List接口的功能方法

```java
public interface List<E> extends Collection<E> {
    
}
```

方法详解：



## 3 List接口实现类

有两种基本类型的List：

- ArrayList
- LinkedList

### 3.1 ArrayList

![image-20201121103132152](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201121103132152.png)

ArrayList实现了长度可变的数组，在内存中分配连续的空间。

- 优点：遍历元素和随机访问元素的效率比较高
- 缺点：添加和删除需要大量移动元素效率低，按照内容查询效率低

![image-20201121102740187](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201121102740187.png)

### 3.2 LinkedList

![image-20201121103302263](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201121103302263.png)

LinkedList采用链表存储方式。

- 优点：插入、删除元素时效率比较高
- 缺点：遍历和随机访问元素效率低下

![image-20201121102833742](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201121102833742.png)