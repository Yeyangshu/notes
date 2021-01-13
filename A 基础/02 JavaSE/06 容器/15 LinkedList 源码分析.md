## LinkedList

### Node

双端链表的节点Node

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

### 构造方法

```java
// 空构造
public LinkedList() {
}
// 用已有的集合创建链表的构造方法
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

### add方法

```java
// 将指定的元素追加到此列表的末尾
public boolean add(E e) {
    linkLast(e);
    return true;
}
// 将e链接为最后一个元素
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        // 指向后继元素，也就是指向下一个元素
        l.next = newNode;
    size++;
    modCount++;
}
```
