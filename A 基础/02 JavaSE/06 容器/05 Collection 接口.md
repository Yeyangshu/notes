# Collection

集合层次中的根接口

## 1 Collection接口类图

![](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201119233411381.png)

## 2 Collection的功能方法

- 具有作为容器应该具有的功能（增、删、改、查）
- 但是不一定全有
- 容器的基本操作：增加、删除、判断、取出

```java
public interface Collection<E> extends Iterable<E> {
    
    boolean add(E e);
    boolean addAll(Collection<? extends E> c);
    void clear();
    boolean contains(Object o);
    boolean containsAll(Collection<?> c);
    boolean isEmpty();
    Iterator<E> iterator();
    boolean remove(Object o);
    boolean removeAll(Collection<?> c);
    boolean retainAll(Collection<?> c);
    int size();
    Object[] toArray();
    <T> T[] toArray(T[] a);
    
    // jdk1.8
    default Stream<E> parallelStream() {
        return StreamSupport.stream(spliterator(), true);
    }
    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, 0);
    }
    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }
}
```

详细介绍：

- boolean add(E e)：将一个元素添加到容器中，确保容器持有具有泛型类型 E 的参数。如果没有此参数添加进容器，则返回 false（可算的方法）
- boolean addAll(Collection<? extends E> c)：添加参数中的所有元素。只要添加了任意元素就返回 true（可选的）
- void clear()：移除容器中的所有元素（可选的）。
- boolean contains(Object o)：如果容器中包含了一个与 o 相等的对象，则返回 ture。
- boolean containsAll(Collection<?> c)：如果容器持有 c 容器中的所有参数，则返回 true。
- boolean isEmpty()：如果容器中没有元素，则返回 true。
- Iterator<E> iterator()：返回一个 Iterator<E>，可以用来遍历容器中的元素。
- boolean remove(Object o);：如果参数在容器中，则移除此元素的一个实例。如果做了移除动作，则返回 true（可选的）。
- boolean removeAll(Collection<?> c)：移除参数中的所有元素。只要有移除动作就返回 true（可选的）。
- boolean retainAll(Collection<?> c)：只保存参数中的元素。只要容器发生了改变就返回 true（可选的）；
- int size()：返回容器中元素的数目。
- Object[] toArray()： 返回一个对象数组，该数组包含容器中的所有元素。
-  <T> T[] toArray(T[] a)：返回一个对象数组，该数组包含容器中的所有元素。返回结果的运行时类型与参数数组 a 的类型相同，而是不单纯的 Object。

### 3 Collection的子接口和实现类

- AbstractCollection：此类提供 `Collection` 接口的骨干实现，以最大限度地减少了实现此接口所需的工作。 

  - 要实现一个不可修改的 collection，编程人员只需扩展此类，并提供 `iterator` 和 `size`  方法的实现。（`iterator` 方法返回的迭代器必须实现 `hasNext` 和 `next`。）

  - 要实现可修改的 collection，编程人员必须另外重写此类的 `add` 方法（否则，会抛出  `UnsupportedOperationException`），`iterator` 方法返回的迭代器还必须另外实现其  `remove` 方法。

- List
- Set
- Queue