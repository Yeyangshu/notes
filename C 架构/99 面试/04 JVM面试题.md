# JVM面试题

## 1 类加载过程

### Java是解释还是编译执行？

33页

Java是混合模式：解释器 + 热点代码编译（JIT，Just In Time）

- 解释器
- JIT
- 混合模式
  - 热点代码检测

指令：

- -Xmixed：默认混合模式，开始解释执行，启动速度较快，对热点代码实行检测和编译
- -Xint：使用解释模式，启动很快，执行稍慢
- -Xcomp：使用纯编译模式，执行很快，启动很慢

### class文件格式

- 魔数
- 次版本号
- 主版本号
- 常量池大小
- 访问标识
- 类索引
- 字段表
- 方法表

### 类加载过程

一个java文件从编码到最终运行，一般主要包括两个过程：

- 编译：将写好的 .java 文件，通过 javac 命令编译成字节码，也就是 .class 文件。
- 运行：JVM 把编译生成好的 .class 文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的 Java 类型的过程叫做虚拟机的类加载机制。

与其他语言不同，Java 类型的加载、连接和初始化是在程序运行期间完成的。

![image-20210107225639880](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210107225639880.png)

==类加载的过程主要分为三个部分==：

#### 加载

1. 通过全类名获取定义此类的二进制字节流
2. 将字节流所代表的静态存储结构转换为方法区的运行时数据结构
3. 在内存中生成一个代表该类的 Class 对象,作为方法区这些数据的访问入口

总结：二进制文件加载到内存，生成class对象

#### 链接

- 验证：校验class文件是否符合标准

  ![image-20210107225716408](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210107225716408.png)

- 准备

  **为类变量分配内存并设置类变量初始值的阶段**，这些内存都将在方法区中分配。

  静态变量赋默认值，static int i =1，此时int = 0

  ![image-20210107225811455](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210107225811455.png)

- 解析

  解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程，也就是得到类或者字段、方法在内存中的指针或者偏移量。

#### 初始化

静态变量赋初始值。**真正执行类中定义的** Java 程序代码（字节码），初始化阶段是执行初始化方法 `<clinit>()`方法的过程。

对于初始化阶段，虚拟机严格规范了有且只有5种情况下，必须对类进行初始化（只有主动去使用类才会初始化类）：

1. 当遇到 new 、 getstatic、putstatic或invokestatic 这4条直接码指令时，比如 new 一个类，读取一个静态字段(未被 final 修饰)、或调用一个类的静态方法时。
   - 当jvm执行new指令时会初始化类。即当程序创建一个类的实例对象。
   - 当jvm执行getstatic指令时会初始化类。即程序访问类的静态变量(不是静态常量，常量会被加载到运行时常量池)。
   - 当jvm执行putstatic指令时会初始化类。即程序给类的静态变量赋值。
   - 当jvm执行invokestatic指令时会初始化类。即程序调用类的静态方法。
2. 使用 `java.lang.reflect` 包的方法对类进行反射调用时如Class.forname("..."),newInstance()等等。 ，如果类没初始化，需要触发其初始化。
3. 初始化一个类，如果其父类还未初始化，则先触发该父类的初始化。
4. 当虚拟机启动时，用户需要定义一个要执行的主类 (包含 main 方法的那个类)，虚拟机会先初始化这个类。
5. MethodHandle和VarHandle可以看作是轻量级的反射调用机制，而要想使用这2个调用， 就必须先使用findStaticVarHandle来初始化要调用的类。

### 类加载器

JVM 中内置了三个重要的 ClassLoader，除了 BootstrapClassLoader 其他类加载器均由 Java 实现且全部继承自`java.lang.ClassLoader`：

1. **BootstrapClassLoader(启动类加载器)** ：最顶层的加载类，由C++实现，负责加载 `%JAVA_HOME%/lib`目录下的jar包和类或者或被 `-Xbootclasspath`参数指定的路径中的所有类。
2. **ExtensionClassLoader(扩展类加载器)** ：主要负责加载目录 `%JRE_HOME%/lib/ext` 目录下的jar包和类，或被 `java.ext.dirs` 系统变量所指定的路径下的jar包。
3. **AppClassLoader(应用程序类加载器)** :面向我们用户的加载器，负责加载当前应用classpath下的所有jar包和类。
4. 自定义加载器

### 双亲委派模型

·JVM是按需动态加载，采用双亲委派模型

系统中的 ClassLoder 在协同工作的时候会默认使用 **双亲委派模型** 。即在类加载的时候，系统会首先判断当前类是否被加载过。已经被加载的类会直接返回，否则才会尝试加载。加载的时候，首先会把该请求委派该父类加载器的 `loadClass()` 处理，因此所有的请求最终都应该传送到顶层的启动类加载器 `BootstrapClassLoader` 中。当父类加载器无法处理时，才由自己来处理。当父类加载器为null时，会使用启动类加载器 `BootstrapClassLoader` 作为父类加载器。

> Java平台使用委托模型来加载类。 基本思想是每个类加载器都有一个“父”类加载器。 加载类时，类加载器首先将对类的搜索“委派”给其父类加载器，然后再尝试查找类本身。

#### 源码

```java
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
        // First, check if the class has already been loaded
        // 首先，检查请求的类是否已经被加载过
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    // 父加载器不为空，调用父加载器loadClass()方法处理
                    c = parent.loadClass(name, false);
                } else {
                    // //父加载器为空，使用启动类加载器 BootstrapClassLoader 加载
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException异常
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }
			// 尝试自己加载
            if (c == null) {
                // If still not found, then invoke findClass in order
                // to find the class.
                long t1 = System.nanoTime();
                c = findClass(name);

                // this is the defining class loader; record the stats
                PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
    }
}
```

loadClass()方法把类加载到内存中，返回一个class对象

Class<?> c = findLoadedClass(name);

### 为什么要实现双亲委派？

主要是为了安全，可以避免类的重复加载（JVM 区分不同类的方式不仅仅根据类名，相同的类文件被不同的类加载器加载产生的是两个不同的类），也保证了 Java 的核心 API 不被篡改。

反证法：

例如类java.lang.Object，它存放在rt.jar中，无论哪个类加载器要加载这个类，最终都会委派给启动类加载器进行加载，因此Object类在程序的各种类加载器环境中都是同一个类。相反，如果用户自己写了一个名为java.lang.Object的类，并放在程序的Classpath中，那系统中将会出现多个不同的Object类，java类型体系中最基础的行为也无法保证，应用程序也会变得一片混乱。

### 自定义类加载器

- 继承ClassLoader

- 重写模板方法findClass()

  打破双亲委派模型则需要重写 loadClass() 方法

```java
// 尝试自己加载
if (c == null) {
    c = findClass(name);
}

// protected，模板方法，只能去子类找
protected Class<?> findClass(String name) throws ClassNotFoundException {
    throw new ClassNotFoundException(name);
}
```

### 对象的创建过程

虚拟机遇到一条new指令的时候，首先去检查这个指令的参数是否能在常量池中定位到一个类的引用，并且检查这个符号引用所代表的类是否已经被加载、解析和初始化过。如果没有必须先执行相应的类加载过程。

1. 第一步将class 加载到内存
2. 第二步linking的过程
   - verification校验
   - preparation把类的静态变量设置默认值
   - resolution做解析
3. 类的初始化把静态变量设为初始值，同时执行静态语句块
4. 申请对象内存
5. 成员变量赋默认值
6. 设置对象头信息：
7. 调用构造方法`<init>`
   - 成员变量顺序赋初始值
   - 执行构造方法语句

### 对象怎么分配

1. new一个对象的时候首先在栈上分配，栈上能分配就在栈上分配，栈弹出对象就没了
2. 栈TLAB分配
3. 堆分配

### 对象在内存中的存储布局？

对象头、实例数据和对齐填充

![image-20210107215558149](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210107215558149.png)

对象的内存布局分为两类：

#### 普通对象

- 对象头，在Hotspot里面称为markword，8个字节

  **一部分用于存储对象自身的运行时数据**（哈希码、GC 分代年龄、锁状态标志等等），**另一部分是类型指针**，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是那个类的实例。

  ClassPointer指针：-XX:+UseCompressedClassPointers为4个字节，不开启为8个字节

- 实例数据：

  - 引用类型：-XX:+UseCompressedOops为四个字节，不开启为8个字节，Oops（Ordinary Object Pointers）

  - 成员变量大小

    | Primitive Type | Memory Required(bytes) |
    | -------------- | ---------------------- |
    | boolean        | 1                      |
    | byte           | 1                      |
    | short          | 2                      |
    | char           | 2                      |
    | int            | 4                      |
    | float          | 4                      |
    | long           | 8                      |
    | double         | 8                      |

- Padding对齐，是8的倍数

普通对象，首先有一个对象头markword，8个字节，第二这个对象是属于哪个class的，ClassPointer指针指向你要的class对象，实例数据就是成员变量还有引用类型，第四个Padding对齐，算出来正好是15个字节，64位机器按照块来读取，所以是8的倍数

#### 数组对象

- 对象头，8
- ClassPointer指针
- 数组长度：4字节
- 数组数据
- 对齐8的倍数

### 对象头具体包括什么？

![image-20210107215558149](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210107215558149.png)

- **Mark Word**：包含一系列的标记位，比如轻量级锁的标记位，偏向锁标记位等等。在32位系统占4字节，在64位系统中占8字节；

  锁定信息：两位代表对象有没有被锁定

  GC的标记：被回收多少次分带年龄

- **Class Pointer**：用来指向对象对应的Class对象（其对应的元数据对象）的内存地址。在32位系统占4字节，在64位系统中占8字节；

- **Length**（数组）：如果是数组对象，还有一个保存数组长度的空间，占4个字节；

深入理解Java虚拟机 47页

对象头分为两部分：

第一部分用于存储对象自身的运行时数据，如：

- 哈希码
- GC分代年龄
- 锁状态标志
- 线程持有的锁
- 偏向线程ID
- 偏向时间戳
- ...

第二部分是类型指针

即对象只想它的类型指针，虚拟机通过这个指针来确定这个对象是那个类的实例

https://blog.csdn.net/lkforce/article/details/81128115

### 对象怎么定位？

摘自《深入理解Java虚拟机》

创建对象自然是为了后续使用该对象，Java程序会通过栈上的refrence数据来操作栈上的具体对象。refrence类型在《Java虚拟机规范》里面只规定了它是一个指向对象的引用，并没有定义这个引用应该通过什么方式去定位、访问到堆中对象的具体位置，所以访问对象方式也是由虚拟机实现而定的，主流的访问方式有两种：

1. 句柄池

   如果使用句柄访问的话，Java堆中将可能会划分出一块内存来作为句柄池，refrence中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的地址信息，**有图**

2. 直接指针

   如果使用直接指针访问的话，Java堆中对象的内存布局就必须考虑如何放置访问类型数据的相关信息，refrence中存储的直接就是对象地址，如果只是访问对象本身的话，就不需要间接访问的开销，有图

这两种访问方式各有优势，使用句柄来访问最大的好处就是refrence中存储的是稳定句柄地址，在对象被移动（垃圾回收的时候移动对象是非常普遍的）时只会改变句柄中的实例数据指针，而refrence本身不需要被修改。使用直接指针来访问的最大好处就是速度更快，它节省了一次指针定位的时间开销，由于对象访问在Java非常频繁，因此这类开销积少成多也是一项极为可观的执行成本。Hotspot主要使用第二种方式进行对象定位，但是从整个软件开发的范围来看，在各类语言、框架中使用句柄来访问的情况也是非常常见的。

### Object o = new Object在内存中占用了多少字节？

书58-59

### 如果对象的引用被置为 null，垃圾收集器是否会立即释放对象占用的内存

不会，在下一个垃圾回调周期中，这个对象将是被可回收的。

也就是说并不会立即被垃圾收集器立刻回收，而是在下一次垃圾回收时才会释放其占用的内存。

### Java栈什么时候会内存溢出，java堆呢，说一种场景？

Java 虚拟机栈会出现两种错误：StackOverFlowError 和 OutOfMemoryError。

- **`StackOverFlowError`：** 若 Java 虚拟机栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度的时候，就抛出 StackOverFlowError 错误。
- **`OutOfMemoryError`：** 若 Java 虚拟机堆中没有空闲内存，并且垃圾回收器也无法提供更多内存的话。就会抛出 OutOfMemoryError 错误。

## 2 Java内存模型（JMM）

### 硬件缓存行 cache line

一次读取64字节

### 硬件伪共享

缓存 系统中是以缓存行（cache line）为单位存储的，当多线程修改互相独立的变量时，如果这些变量共享同一个缓存行，就会无意中影响彼此的性能，这就是伪共享。

### 硬件缓存一致性协议

![image-20210108002412179](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210108002412179.png)

MESI

现代CPU的数据一致性的实现=缓存锁+总线锁

### 硬件乱序执行

CPU为了提高指令的执行效率，会在一条指令的执行过程中，同时执行另一条指令，前提是两条指令没有依赖关系

原因：CPU与内存的速度差

### 硬件合并写

读指令的同时可以同时执行不同的其它指令

写的同时可以进行合并写

### 硬件内存屏障

X86 CPU内存屏障

- sfence：存屏障
- ifence：读屏障
- mfence：上面两个加起来

### 硬件lock

硬件使用lock指令去锁住内存，保证执行顺序

### JVM内存屏障

- LoadLoad
- StoreStore
- LoadStore
- StoreLoad

### volatile底层实现细节

1. 字节码层面

   volatile编译成字节码文件，增加一个访问符ACC_VOLATILE

2. jvm层面

   volatile写操作前面加了一个StoreStore，后面加了一个StoreLoad

   读操作前加一个LoadLoad，后面加了一个LoadStore

   综上，volatile内存区的读写都加内存屏障

3. os和硬件层面

   windows加lock指令

源码编译完只是在字节码上加ACC_VOLATILE标识，当虚拟机读到这个标识的时候就会在内存区读写之前加屏障，加完之后虚拟机和操作系统去执行这个指令，硬件层面加lock（windows）

### synchronized底层实现细节

1. 字节码层面

   synchronized编译成字节码文件

   - 同步语句块增加monitorenter和monitorexit
   - 方法加ACC_SYNCHRONIZED修饰符

2. jvm层面

   实际是C和C++调用了操作系统上提供的同步机制

3. os和硬件层面

   调用硬件的lock指令

源码编译完只是在字节码上加ACC_VOLATILE标识，当虚拟机读到这个标识的时候就会在内存区读写之前加屏障，加完之后虚拟机和操作系统去执行这个指令，硬件层面加lock（windows）

### JMM

![image-20210108002439657](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210108002439657.png)

Java线程之间的通信由Java内存模型（简称为JMM）控制，JMM决定一个线程对共享变量的写入何时对另一个线程可见。

JMM定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地内存（local memory），本地内存中存储了该线程以读/写共享变量的副本。

### happens-before原则

JVM规定重排序必须遵守的规则



## 3 JVM内存区域

### 介绍下 Java 内存区域（运行时数据区），以及这些空间的存放内容

< 1.8

![image-20210107215944888](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210107215944888.png)

1.8

![image-20210107220024525](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210107220024525.png)

**线程私有的：**

- 程序计数器
- 虚拟机栈：Java 虚拟机栈也是线程私有的，它的生命周期和线程相同，描述的是 Java 方法执行的内存模型，每次方法调用的数据都是通过栈传递的。
- 本地方法栈

**线程共享的：**

- 堆
- 方法区
- 直接内存 (非运行时数据区的一部分)

### 堆内存划分的空间

- 新生代
  - Eden 区
  - From Survivor 区
  - To Survivor 区

- 老年代

### JVM为什么要增加元空间？

## 4 GC与调优

### 怎样找到垃圾？

-  引用计数器：给对象加一个计数器，引用它就加一

  ![image-20210108004216984](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210108004216984.png)

  不能解决的问题

  ![image-20210108004236946](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210108004236946.png)

- 可达性分析算法：

  使用GC ROOTS作为起点，从这些起点开始向下搜索，搜索走过的路径叫引用连，当一个对象到GC ROOTS没有任何引用链时，证明此对象不可用

### GC ROOTS

在Java语言中，可作为GC Roots的对象包括以下几种：

- 虚拟机栈（栈帧中的本地变量表）中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中JNI（native方法）引用的对象

![image-20210108004318744](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210108004318744.png)

### 垃圾回收算法

#### 标记清除法 Mark-Sweep（老年代）

标记和清除两个阶段：首先标记除所有需要回收的对象，标记完成之后统一回收所有的被标记的对象。

不足：

- 效率问题，标记和清除两个阶段效率低
- 空间问题，标记清楚之后会产生大量不连续的内存碎片

#### 复制算法 Copying（新生代）

将可用的内存分为大小相等的两块，每次只使用一块，当这一块内存用完之后，把存活的对象复制到另一块上去，然后再把已使用过的内存空间一次清理掉。

实现简单，运行高效

不需要按1:1的比例来划分内存空间，HotSpot默认Eden:Survivor比例为8:1。

#### 标记整理法 Mark-Compact（老年代）

首先标记除所有需要回收的对象，让所有的存活的对象都向一端移动，然后直接清理掉端边界以外的内存。

### 常见垃圾回收器

- Serial：单线程，STW长
- Serial Old：单线程，STW长
- Parallel Scavenge：多线程清理垃圾
- Parallel Old：多线程清理垃圾
- ParNew：多线程清理垃圾，配合CMS
- CMS：concurrent mark sweep，CMS是HotSpot在JDK1.5推出的第一款真正意义上的并发（Concurrent）收集器，第一次实现了让垃圾收集线程与用户线程（基本上）同时工作；

### PS为什么不能和CMS搭配

CMS作为老年代收集器，但却无法与JDK1.4已经存在的新生代收集器Parallel Scavenge配合工作；这是因为Parallel Scavenge（以及G1）没有使用传统的GC收集器代码框架，而另外独立实现，而其余几种收集器则共用了部分的框架代码。

PN、CMS是分带框架下开发的

### CMS解决什么问题

STW，无法并发收集垃圾的问题

### CMS回收停顿了几次，说一下回收的过程？

七次，大致上有四个步骤：

- initial mark（初始标记）：STW，找到根上的对象
- concurrent mark（并发标记）：标记垃圾，与用户线程同时运行
- remark（重新标记）：重新标记一下垃圾，会导致STW
- concurrent-sweep（并发清除）：并发清理

### CMS初始标记的对象都有哪些？

标记所有GCRoot直达的老年代对象和被年轻代对象直接引用的老年代对象。

说明初始标记阶段只标记老年代对象。

### CMS缺点？

- 内存碎片：内存比较大的时候，会产生内存碎片，这时会使用Serial Old做标记压缩
- 浮动垃圾：



### CMS（Comcurrent Mark Sweep）缺点

深入理解Java虚拟机 83

- 无法处理浮动垃圾，可能出现出现 Concurrent Mode Failure 失败而导致另一次Full GC的产生，这时虚拟机将启动后备预案：临时启用Serial Old收集器来重新进行老年代的垃圾收集。
- 收集结束时会有大量的空间碎片产生。空间碎片过多时，将会给大对象分配带来很多麻烦，往往会出现老年代还有很大的空间剩余，但是无法找到足够大的连续空间来分配当前对象，不得不提前触发一次Full GC

### G1（Garbage First）特点

- 并行与并发
- 分代收集
- 空间整合
- 可预测的停顿

- 但是无法找到足够大的连续空间来分配当前对象，不得不提前触发一次Full GC

### G1 GC

美团文章：

- Young GC：选定所有年轻代里的Region。通过控制年轻代的region个数，即年轻代内存大小，来控制young GC的时间开销
- Mixed GC：选定所有年轻代里的Region，外加根据global concurrent marking统计得出收集收益高的若干老年代Region。在用户指定的开销目标范围内尽可能选择收益高的老年代Region。

Mixed GC只回收部分old region。G1没有 fullGC 概念，需要fullGC时，调用Serial Old GC进行全堆扫描（包括eden、survivor、o、perm）。

### G1 如果产生FullGC，怎么办

1. 扩内存
2. 提高CPU性能
3. **降低Mixed GC触发的阈值，让Mixed GC提早发生**

### global concurrent marking运作步骤

- 初始标记：标记一下GC Roots能够直接关联到的对象
- 并发标记：从GC Roots开始对堆中对象进行可达性分析，找出存活对象
- 最终筛选：修正并发标记期间因用户程序继续运作而导致的产生变动的那部分记录，标记那些在并发标记阶段发生变化的对象，将被回收
- 筛选回收：对各个region的回收价值和成本进行排序，根据用户所期望的gc停顿时间制定回收计划

### 并发标记的算法

CMS和G1核心在于并发标记，难点在于在标记对象的过程中，对象引用关系正在改变

#### CMS和G1：三色标记

#### ZGC：颜色指针

### Garbage First (G1)了解吗？

G1逻辑上分代，物理上不分带。

G1是一种主要运行在服务器端的一个垃圾回收器，目标是用在多核、大内存的机器上，它在大多数情况下可以实现指定的GC暂停时间，同时还能保证较高的吞吐量。

分而治之思想：G1将堆空间划分成了互相独立的区块，把内存分为Region，块大小1Mb ~32Mb，大约2014块。每一块Region在逻辑上依然属于某一个分代，分代分四种：

- Old区：放老对象
- Survivor：放存活对象
- Eden区：新生对象
- Humongous：大对象区域

G1特点：

- 并发与并行
- 空间整合，基于标记-整理算法，解决了内存碎片的问题
- 可以建立可预测的停顿模型
- 将整个java堆内存模型划分为多个大小相等的Region，使得年轻代和老年代不再物理隔离开来

### JVM调优，如何解决线上gc问题

- 运维团队接收到报警信息
- 首先使用top命令查看CPU占比高的进程
- top -Hp查看进程中的线程CPU使用占比
- jps定位Java进程
- jstat -gc动态观察gc情况
- jmap -dump 文件

### Java中会存在内存泄漏吗

长生命周期的对象持有短生命周期对象的引用，这样就很可能发生内存泄露，尽管短生命周期对象已经不再需要，但是因为长生命周期对象持有它的引用而导致不能被回收，这就是java中内存泄露的发生场景。

### 内存溢出的原因，如何排查线上问题

内存溢出指程序在申请内存后无法释放已申请的内存空间
