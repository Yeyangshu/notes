# Iterator

**迭代器是一个对象，它的工作是遍历选择序列中的对象**，而客户端程序员不必知道或关心该序列底层的结构。迭代器通常被称为轻量级对象：创建它的代价小，Iterator只能用来：

1. 使用方法`iterator()`要求容器返回一个Iterator。Iterator将准备好返回序列的第一个元素。
2. 使用`next()`获得序列的下一个元素。
3. 使用`hasNext()`检查序列中是否还有元素。
4. 使用`remove()`将迭代器新近元素删除。

有了Iterator就不必为元素中的元素数量操心了，由`hasNext()`和`next()`实现。

如果只是向前遍历List，并不打算修改List对象本身，foreach语法会更加简洁。

Iterator还可以移除由`next()`产生的最后一个元素，这意味着在调用`remove()`之前**必须**先调用`next()`。

## 1 Iterator接口类图

## 2 Iterator的功能方法

```java
public interface Iterator<E> {
	boolean hasNext();
    E next();
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }
    // jdk1.8
    default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
}
```

### 2.1 详细方法介绍

- hasNext()：判断是否有元素没有被遍历
- next()：返回游标当前位置的元素并将游标移动到下一个位置
- remove()：删除游标左面的元素，在执行完next之后。该操作只能执行一次

### 2.2 方法测试

案例来自《Java编程思想》

```java‘
package holding;

import typeinfo.pets.*;

import java.util.*;

public class SimpleIteration {
    public static void main(String[] args) {
        List<Pet> pets = Pets.arrayList(12);
        System.out.println(pets);
        Iterator<Pet> it = pets.iterator();
        while (it.hasNext()) {
            Pet p = it.next();
            System.out.print(p.id() + ":" + p + " ");
        }
        System.out.println();
        
        // A simpler approach, when possible:
        // 如果可能的话，一种更简单的方法
        for (Pet p : pets)
            System.out.print(p.id() + ":" + p + " ");
        System.out.println();
        
        // An Iterator can also remove elements:
        it = pets.iterator();
        for (int i = 0; i < 6; i++) {
            it.next();
            it.remove();
        }
        System.out.println(pets);
    }
}

结果：
[Rat, Manx, Cymric, Mutt, Pug, Cymric, Pug, Manx, Cymric, Rat, EgyptianMau, Hamster]
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx 8:Cymric 9:Rat 10:EgyptianMau 11:Hamster 
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx 8:Cymric 9:Rat 10:EgyptianMau 11:Hamster 
[Pug, Manx, Cymric, Rat, EgyptianMau, Hamster]
```

## 3 Iterator实现类

### 3.1 ListIterator

ListIterator：ListIterator是一个更加强大的Iterator的子类型，它只能用于各种List类的访问。ListIterator可以双向移动，它还可以相对于迭代器在列表中指向的当前位置的前一个和后一个元素的索引，并且可以使用set()方法替换它访问过的最后一个元素。你还可以通过调用listIterator()方法产生一个指向List开始处的ListIterator，并且还可以通过调用listIterator(n)方法创建一个一开始就指向列表元素处的ListIterator。

#### 3.1.1 ListIterator使用

```java
public class ListIteration {
    public static void main(String[] args) {
        List<Pet> pets = Pets.arrayList(8);
        ListIterator<Pet> it = pets.listIterator();
        while (it.hasNext())
            System.out.print(it.next() + ", " + it.nextIndex() +
                    ", " + it.previousIndex() + "; ");
        System.out.println();
        // Backwards:
        while (it.hasPrevious()) {
          System.out.print(it.previous().id() + " ");
        }
        System.out.println();
        System.out.println(pets);
        
        it = pets.listIterator(3);
        while (it.hasNext()) {
            it.next();
            it.set(Pets.randomPet());
        }
        System.out.println(pets);
    }
}

Rat, 1, 0; Manx, 2, 1; Cymric, 3, 2; Mutt, 4, 3; Pug, 5, 4; Cymric, 6, 5; Pug, 7, 6; Manx, 8, 7; 
7 6 5 4 3 2 1 0 
[Rat, Manx, Cymric, Mutt, Pug, Cymric, Pug, Manx]
[Rat, Manx, Cymric, Cymric, Rat, EgyptianMau, Hamster, EgyptianMau]
```

#### 3.1.2 为什么需要ListIterator？

在迭代过程中，准备添加或者删除元素

#### 3.1.3 ListIterator的作用 -> 解决并发操作异常

在迭代时，不可能通过集合对象的方法(al.add(?))操作集合中的元素，

- 会发生并发修改异常。

- 所以，在迭代时只能通过迭代器的方法操作元素，但是Iterator的方法是有限的，只能进行判断(hasNext)，取出(next)，删除(remove)的操作

- 如果想要在迭代的过程中进行向集合中添加，修改元素等就需要使用ListIterator接口中的方法

```java
ListIterator li=al.listIterator();
while(li.hasNext()){
    Object obj=li.next();
    if ("java2".equals(obj)) {
        li.add("java9994");
        li.set("java002");
    }
}
```

