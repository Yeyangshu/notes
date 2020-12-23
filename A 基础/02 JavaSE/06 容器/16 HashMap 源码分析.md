# HashMap源码

HashMap 是最常用的 Map 结构，Map 的本质是键值对。它使用**数组**来存放这些键值对，键值对与数组下标的对应关系由 key 值的 hashCode 来决定，这种类型的数据结构可以称之为**哈希桶**（buckets）。

在Java语言中，hashCode 是个 int 值，虽然 int 的取值范围是 [-232，231-1]，但是 Java 的数组下标只能是正数，所以该哈希桶能存储 [0,231-1] 区间的哈希值。这个存储区间可以存储的数据足足有 20 亿之多，可是在实际应用中，hashCode 会倾向于集中在某个区域内，这就导致了大量的 hashCode 重复，这种重复又称为**哈希冲突**。

hashCode 在 HashMap 中的作用：

```java
public class HashMapSample {
    public static void main(String[] args) {
        HashMap<HS, String> map = new HashMap<>();

        // 1 存入hashCode相同的HS对象
        // hashCode一致的HS类并没有发生冲突，两个HS对象都被正常地存入了HashMap。
        map.put(new HS(), "1");
        map.put(new HS(), "2");
        // {HS{}=1, HS{}=2}
        System.out.println(map);

        // 2 存入重写过equals方法的HS对象
        // hashCode一致，同时equals返回true的对象发生了冲突，认为是同一个对象，第三个HS对象替代了第一个。
        map.put(
            new HS() {
                @Override
                public boolean equals(Object obj) {
                    return true;
                }
            }, "3");
        // {HS{}=3, HS{}=2}
        System.out.println(map);

        // 3 存入重写过equals和hashCode方法的HS对象
        // 重写了hashCode使之不一致，同时equals返回true的对象，也没有发生冲突，被正确的存入了HashMap。
        map.put(
            new HS() {
                @Override
                public boolean equals(Object obj) {
                    return true;
                }

                @Override
                public int hashCode() {
                    return 2;
                }
            }, "3");
        // {HS{}=3, HS{}=2, HS{}=3}
        System.out.println(map);
    }
}

// {HS{}=1, HS{}=2}
// {HS{}=3, HS{}=2}
// {HS{}=3, HS{}=2, HS{}=3}
```

这三个现象说明，当且仅当 hashCode 一致，且 equals 一致的对象，才会被 HashMap 认为是同一个对象。

## 1 底层数据结构分析

### 1.1 JDK1.8之前

在 Java7 及之前的版本中，HashMap 的底层实现是**数组和链表**。

#### 1.1.1 成员变量

HashMap的主要成员变量包括：

```java
/** 存储数据的核心成员变量 */
transient Entry<K,V>[] table;
/** 键值对数量 */
transient int size;
/** 加载因子，用于决定table的扩容量 */
final float loadFactor;
```

**底层数据结构**

![image-20201212213659127](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201212213659127.png)

table 是 HashMap 的核心成员变量。该数组用于记录 HashMap 的所有数据，它的每一个下标都对应一条链表。换言之，所有哈希冲突的数据，都会被存放到同一条链表中。Entry<K,V> 则是该链表的结点元素。

Entry<K,V> 包含以下成员变量：

```java
static class Entry<K,V> {
    /** 存放键值对中的关键字 */
    final K key;
    /** 存放键值对中的值 */
    V value;
    /** 指向下一个节点的引用 */
    Entry<K,V> next;
    /** key所对应的hashCode */
    final int hash;
}
```

**通过上述源码可以看出，HashMap 的核心实现是一个单向链表数组（Entry<K,V>[]table）。**



HashMap规定了该数组的两个特性：

1. 会在特定的时刻，根据需要来扩容。

2. 其长度始终保持为 2 的幂次方。



由于数组的查找是比链表要快的，于是我们可以得出一个结论：**尽可能使键值的 hashcode 分散，这样可以提高 HashMap 的查询效率。**

#### 1.1.2 常量

```java
/** 默认的初始化容量，必须是2的幂次方 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
/** 最大容量，在构造函数指定容量的大小时，用于比较 */
static final int MAXIMUM_CAPACITY = 1 << 30;
/** 默认的加载因子，如果没有构造方法指定，那么使用该常量 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

#### 1.1.3 方法

##### 1.1.3.1 put(K key, V value)

put 方法用于向 HashMap 中添加元素

**源码**

```java
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    if (key == null)
        return putForNullKey(value);
    // 根据key计算哈希值
    int hash = hash(key);
    // 根据哈希值和数组长度计算数组下标
    int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        // 哈希值相同再比较key是否相同，相同的话值替换，否则将这个槽转成链表
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    // fast-fail，迭代时响应快速失败，还未添加元素就进行modCount++,将为后续留下很多隐患
    modCount++;
    // 添加元素，注意最后一个参数i是table数组的下标
    addEntry(hash, key, value, i);
    return null;
}
```

**执行流程**

![image-20201212214114582](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201212214114582.png)

###### 1.1.3.1.1 indexFor(int,int) 方法

indexFor(int,int) 方法作用是根据 hashCode 和 table 长度来计算下标。

**源码**

```java
/**
 * Returns index for hash code h.　　返回数组下标
 */
static int indexFor(int h, int length) {
    // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
    return h & (length-1);　// 保证获取的index一定在数组范围内
}
```

h&(length-1) 有什么意义？

- h 小于 length-1 的时候，取 h；
- h 大于 length-1 的时候，取余数。

###### 1.1.3.1.2 hash(Object k) 方法

hash(Object k) 方法，用于计算键值 k 的 hashCode。

**源码**

```java
final int hash(Object k) {
    int h = hashSeed;
    if (0 != h && k instanceof String) {
        return sun.misc.Hashing.stringHash32((String) k);
    }
    // 先取key的hashCode再和hashSeed进行异或运算
    h ^= k.hashCode();

    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

**概念**

松散哈希：松散哈希是指数值尽可能平衡分布的 hashCode。

哈希碰撞：HashMap 默认的容量是 16，同时，在 indexFor(int,int) 方法里，hashCode 如果超出 length-1，那么会执行取余计算。设想一个情况，如果有一类数据，其原始的 hashCode 集中在 000A 0000～FFFF 0000 之间，那么，计算 indexFor 的时候，其结果会全部为 0。这种 hashCode 重复的现象称之为**哈希碰撞**，当发生哈希碰撞的时候，碰撞的键值对都会被存入同一条链表中，导致 HashMap 效率低下。

松散哈希意义：**松散哈希可以尽量减少哈希碰撞的发生**。



useAltHashing 是个标识量，当它的值为 true 时，将启用替代的哈希松散算法，它有以下两个意义：

- 当处理String类型数据时，直接调用 sun.misc.Hashing.stringHash32(String) 方法来获取最终的哈希值。
- 当处理其他类型数据时，提供一个相对于 HashMap 事例唯一且不变的随机值 hashSeed 作为 hashCode 计算的初始量。

###### 1.1.3.1.3 存储数据

put 方法中 for 循环之后的部分

**源码**

```java
for (Entry<K,V> e = table[i]; e != null; e = e.next) {
    Object k;
    // 哈希值相同再比较key是否相同，相同的话值替换，否则将这个槽转成链表
    if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
        V oldValue = e.value;
        e.value = value;
        e.recordAccess(this);
        return oldValue;
    }
}
// fast-fail，迭代时响应快速失败，还未添加元素就进行modCount++,将为后续留下很多隐患
modCount++;
// 添加元素，注意最后一个参数i是table数组的下标
addEntry(hash, key, value, i);
return null;
```

**代码的执行流程**

![image-20201212215830841](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201212215830841.png)

新增 Entry 有以下两种情况：

- table[] 里不存在指定下标，也就是没有发生哈希碰撞。
- table[] 里存在指定下标（发生了哈希碰撞），但是该下标对应的链表上所有结点都和待添加的键值对的 key 值不同，在这种情况下会向这个链表中添加 Entry 结点。

###### 1.1.3.1.4 addEntry(int,K,V,int)和createEntry(int,K,V,int)

由于 HashMap 的核心数据结构是一个数组，所以一定会涉及数组的扩容。是否需要扩容的依据为成员变量：threshold。

当添加键值对的时候，如果键值对将要占用的位置不是 null，并且 size>=threshold，那么会启动 HashMap 的扩容方法 resize(2*table.length)，扩容之后会重新计算一次 hash 和下标。

不论 HashMap 是否扩容，都会执行创建键值对 createEntry(hash,key,value,bucketIndex) 方法，该方法会增加 size。

###### 1.1.3.1.5 resize(int)

resize(int)，用于给 HashMap 扩充容量。

resize主要完成以下工作：

1. 根据新的容量，确定新的扩容阈值（threshold）大小。如果当前的容量已经达到了最大容量（1<<30)，那么把 threshold 设为 Integer 最大值；反之，则用新计算出来的容量乘以加载因子（loadFactor），计算结果和最大容量 +1 比较大小，取较小者为新的扩容阈值。如果 threshold 被设置为最大整型数，那么它必然大于  size，扩容操作不会再次触发。

2. 确定是否要哈希重构（rehash），判断依据是原有的 useAltHashing（是否使用替代哈希算法标识）和新产生的这个值，是否一致。不一致时，需要哈希重构。

3. 使用新容量来构造新的 Entry<K,V>table 数组，调用 transfer(newTable, rehash) 来重新计算当前所有结点转移到新 table 数组后的下标。

###### 1.1.3.1.6 transfer(Entry[], boolean)

该方法会遍历所有的键值对，根据键值的哈希值和新的数组长度来确定新的下标，如果需要哈希重构，那么还需先对所有键值执行哈希重构。

###### 1.1.3.1.7 put方法总结

put 方法是 HashMap 中最常用的方法之一，它的实现相对复杂，整个功能包括：

1. 计算键值（key）的 hash 值。
2. 根据 hash 值和 table 长度来确定下标。
3. 存入数组。
4. 根据 key 值和 hash 值来比对，确定是创建链表结点还是替代之前的链表值。
5. 根据增加后的 size 来扩容，确定下一个扩容阈值，确定是否需要使用替代哈希算法。

##### 1.1.3.2 get方法

get 方法用于取值。

**源码**

```java
public V get(Object key) {
    // 如果key为null，直接去table[0]处去检索即可
    if (key == null)
        return getForNullKey();
    Entry<K,V> entry = getEntry(key); // 根据key去获取Entry数组

    return null == entry ? null : entry.getValue();
}
final Entry<K,V> getEntry(Object key) {
    if (size == 0) {
        return null;
    }
    // 根据key的hashCode重新计算hash值
    int hash = (key == null) ? 0 : hash(key);
    // 获取查找的key所在数组中的索引，然后遍历链表，通过equals方法对比key找到对应的记录
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}
```

**处理流程**

![image-20201212221105864](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201212221105864.png)

每一次 get 操作都要比较对应链表所有结点 key 值，因为链表的遍历操作的时间复杂度为 O(n)，所以，get 方法的性能关键就在链表的长度上。

##### 1.1.3.2 性能优化

通过对 get 和 put 这两个常用方法的分析，可以得出以下推论：

1. HashMap 执行写操作（put）的时候，比较消耗资源的是遍历链表，扩容数组操作。

2. HashMap 执行读操作（get）的时候，比较消耗资源的是遍历链表。

影响遍历链表的因素是链表的长度，在HashMap中，链表的长度被哈希碰撞的频率决定。哈希碰撞的频率受数组长度所决定，长度越长，则碰撞的概率越小，但长度越长，闲置的内存空间越多。所以，扩容数组操作的结果也会影响哈希碰撞的频率，需要在时间和空间上取得一个平衡点。

哈希碰撞的频率又受 key 值的 hashCode() 方法影响，所计算得出的 hashCode 的独特性越高，哈希碰撞的概率也会变低。

链表的遍历中，需要调用 key 值的 equals 方法，不合理的equals实现会导致 HashMap 效率低下甚至调用异常。因此，要提高 HashMap 的使用效率，可以从以下几个方面入手：

1. 根据实际的业务需求，测试出合理的 loadFactor，否则会始终使用 Java 建议的 0.75。

2. 合理的重写键值对象的 hashCode 和 equals 方法，可以参考《Effective Java》的建议：

   ![image-20201212221411699](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201212221411699.png)

### 1.2 JDK1.8之后

Java 8 的 HashMap 数据结构发生了较大的变化，之前的 HashMap 使用的数组+链表来实现，这主要体现在Entry<K,V>[]table 这个成员变量，新的 HashMap 里，虽然依然使用的是 table 数组，但是数据类型发生了变化。

```java
transient Node<K,V>[] table;
```

结点 Node<K,V> 具备链表结点的特性，同时，它还有一个子类 TreeNode<K,V>，是个树结点。

Java 8里的HashMap使用的是**数组+树+链表**的结构

![image-20201211231122208](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201211231122208.png)

#### 1.2.1 方法

##### 1.2.1.1 put(K,V)方法详解

主流程有以下两步：

1. 获取key值的hashCode
2. 调用putVal方法进行存值。

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

###### 1.2.1.1.1 hash方法源码

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

Java 8 也进行了哈希分散，计算过程简单很多

###### 1.2.1.1.2 putVal方法源码

这个方法主要做了以下三件事：

1. 计算下标，计算公式为：下标=table的长度-1&hash，与历史版本一致。
2. 当table为空，或者数据数量（size）超过扩容阈值（threshold）的时候，重新计算table长度。
3. 保存数据。保存数据又分为多种情况：
   - 当下标位置没有结点的时候，直接增加一个链表结点。
   - 当下标位置结点为树结点（TreeNode）的时候，增加一个树结点。
   - 当前面情况都不满足时，则说明当前下标位置有结点，且为链表结点，此时遍历链表，根据hash和key值判断是否重复，以决定是替代某个结点还是新增结点。
   - 在添加链表结点后，如果链表深度达到或超过建树阈值（TREEIFY_THRESHOLD-1)，那么调用treeifyBin方法把整个链表重构为树。注意，TREEIFY_THRESHOLD是一个常量，值固定为8。也就是说，当链表长度达到7的时候，会转化为树结构，为什么要这样设计呢？该树是一棵红黑树，由于链表的查找是O(n)，而红黑树的查找是O(log2n)的，数值太小的时候，它们的查找效率相差无几，Java 8认为7是一个合适的阈值，因此这个值被用来决定是否要从链表结构转化为树结构。

###### 1.2.1.1.3 resize方法源码

resize方法用于重新规划table长度和阈值，如果table长度发生了变化，那么部分数据结点也需要重新进行排列。

两方面讨论：

1. **重新规划table长度和阈值**

   - 当数据数量（size）超出扩容阈值时，进行扩容：把table的容量增加到旧容量的两倍。
   - 如果新的table容量小于默认的初始化容量16，那么将table容量重置为16，阈值重新设置为新容量和加载因子（默认0.75）之积。
   - 如果新的table容量超出或等于最大容量（1<<30），那么将阈值调整为最大整型数，并且return，终止整个resize过程。注意，由于size不可能超过最大整型数，所以之后不会再触发扩容。

2. **重新排列数据结点**，该操作遍历table上的每一个结点，对它们分别进行处理：

   - 如果结点为null，那么不进行处理。

   - 如果结点不为null且没有next结点，那么重新计算该结点的hash值，存入新的table中。

   - 如果结点为树结点(TreeNode)，那么调用该树结点的split方法处理，该方法用于对红黑树进行调整，如果红黑树太小，则将其退化为链表。

   - 如果以上条件都不满足，那么说明结点为链表结点。在上文中提到过，根据hashcode计算出来的下标不会超出table容量，超出的位数会被设为0，而resize进行扩容后，table容量发生了变化，同一个链表里有部分结点的下标也应当发生变化。所以，需要把链表拆成两部分，分别为hashCode超出旧容量的链表和未超出容量的链表。对于hash&oldCap==0的部分，不需要做处理；反之，则需要被存放到新的下标位置上，公式如下所示：

     新下标=原下标+旧容量；

源码：

```java
/**
 * Implements Map.put and related methods
 * 实现Map.put和相关方法
 * 
 * @param hash hash代表key值的hashCode
 * @param key 代表key值
 * @param value 代表value值
 * @param onlyIfAbsent 代表是否取代已存在的值
 * @param evict 在HashMap里并没有特殊含义，是一个为继承预留的布尔值。
 * @return previous value, or null if none
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

##### 1.2.1.2 红黑树相关知识点详解

问题：对于有序数列[10，13，31，72，76，89，91，97]，如何确定31在哪个位置上？

可以依靠遍历来解决这个问题，直接遍历的时间复杂度和数列规模增长一致，也就是O(n)。

更好的方式是二分查找法。

![image-20201211233943681](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201211233943681.png)

对于长度为8的数列，先找到length/2位置的数字，76比较，由于76>31，因此在数列的左半部分继续查找，左半部分长度为4，因此会查找在2/2=2的位置的数据，显然是31，查找结束。

![image-20201211233958170](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201211233958170.png)

二分查找的时间复杂度是O(log2n)。

###### 1.2.1.2.1 二叉查找树

以二分法思想为指导，设计出来一种快速查找树，就是二分查找树。

**特点**

- 每一个结点关键字只会在树中出现一次。
- 任何一个结点，如果它有子结点，那么左侧的关键字一定比较小，右侧的关键字一定比较大。



![image-20201211234220533](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201211234220533.png)

如果把之前的数列转化为这种结构，每次都从根结点开始查找，就算查找到叶子结点，也只是进行了log2n次比较，效率明显高于顺序/倒序遍历。



**缺点**

同样的一个数列，可以对应不同高度的二叉树，二叉树也可以被表示为图5-25所示的形状。

![image-20201211234456973](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201211234456973.png)

在实际应用中，对一棵二分查找树进行多次插入和删除后，它往往会朝着链表的方向退化，查找效率比较低下。。

###### 1.2.1.2.2 平衡二叉树（AVL树）

**特点**

- 它是一棵空树或者二分树查找树。
- 左右两个子树的高度差的绝对值不超过1。
- 左右两个子树都是一棵平衡二叉树。

满足这些条件的二叉树，查找的时间复杂度不会超过O(log2n)。

在对二叉查找树做插入或删除的时候，需要通过一系列旋转操作（自平衡），让其始终满足平衡二叉树的条件，从而可以达到查找效率最优。

![image-20201211235015617](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201211235015617.png)

决定该树是否失衡的关键变量为平衡因子bf。

bf(p) = p左子树高度 - p右子树高度，它有以下特性：

- 结点平衡因子变化后，回溯修改父结点的平衡因子。
- 当平衡因子等于-2或者2的时候，认为以该结点为根结点的树失衡。
- 失衡后需要进行修复，修复完成后，停止回溯修改父结点的平衡因子。



下面重点介绍对二叉树进行不同的操作后，如何维持平衡二叉树的特性。

**插入操作**

**删除操作**



###### 1.2.1.2.3 红黑树（R-B树）

红黑树也是一种自平衡二叉树，它的实现原理和平衡二叉树类似，但在统计上，它的性能要优于平衡二叉树。

**特性**

- 结点是红色或者黑色。
- 根结点是黑色。
- 每个叶子结点（NIL结点）为黑色。
- 每个红色结点的两个子结点都是黑色。
- 从任一结点到其每个叶子的所有路径都包含相同数目的黑色结点。

**插入操作**

##### 1.2.1.3 红黑树在HashMap中的体现

###### 1.2.1.3.1 treeifyBin方法

当table容量小于最小建树容量（64）时，则调整table大小（resize）。由于resize的过程可以分解链表，所以无需转化链表为树。

如果table容量超出64，那么调用TreeNode.treeify方法把链表转化为红黑树。

###### 1.2.1.3.2 putTreeVal方法

putTreeVal方法用于保存树结点。该方法执行二叉树查找，每一次都比较当前结点和待插入结点的大小，如果待插入结点较小，那么在当前结点左子树查找，否则在右子树查找。

待找到空位可以存放结点值之后，执行两个方法：

1. alanceInsertion(root,x)，平衡插入，一方面把结点插入红黑树中，另一方面对红黑树进行转换，使之平衡。
2. moveRootToFront(table,root)，由于红黑树重新平衡之后，root结点可能发生了变化，table里记录的结点不再是红黑树的root，需要重置。

###### 1.2.1.3.3 balanceInsertion平衡插入方法



###### 1.2.1.3.4 rotateLeft和rotateRight方法

这两个方法用于旋转