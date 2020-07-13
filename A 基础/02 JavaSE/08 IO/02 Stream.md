# 流(Stream)

## 1 流的基本概念

如何读写文件？

- 通过流来读写文件

  流是指一连串流动的字符，是以先进先出的方式发送信息的通道

数据源：datasource，提供原始数据的原始媒介

- 数据库
- 文件
- 其他程序
- 内存
- 网络设备
- IO设备

截一下ppt的图。。。。。。



## 2 流的分类

### 2.1 按流向区分

输入输出流是相对于计算机内存来说的，而不是相对于源和目标

1. 输入流

   InputStream和Reader作为基类

2. 输出流

   OnputStream和Writer作为基类

### 2.2 按处理数据单位划分

字节流是8位通用字节流，字符流是16位Unicode字符流

1. 字节流
   - 字节输入流InputStream作为基类
   - 字节输出流OnputStream作为基类
2. 字符流
   - 字符输入流Reader作为基类
   - 字符输出流Writer作为基类

## 3 流的使用

### 3.1 流使用的步骤

1. 选择合适的io流对象
2. 创建对象
3. 传输数据
4. 关闭流对象（）

### 3.2 read

#### 3.2.1 read()

从输入流读取数据的下一个字节

```java
public class StreamDemo {
    public static void main(String[] args) {
        InputStream inputStream = null;
        try {
            inputStream = new FileInputStream("src/abc.txt");
            int length = 0;
            while ((length = inputStream.read()) != -1) {
                System.out.println((char)length);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                inputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

    }
}
控制台：
    a
    b
    c
    d
    ...
    x
    y
    z
```

**存在问题**：每次只能读取一个字节，效率比较低，需要循环多次

#### 3.2.2 read(byte[] b)

从输入流读取一些字节数，并将它们存储到缓冲区 b

好处：添加缓冲区的方式读取，会将数据添加到缓冲区，当缓冲区满了之后，依次读取，而不是每一个字节进行读取

```java
ublic class StreamDemo {
    public static void main(String[] args) {
        InputStream inputStream = null;
        try {
            inputStream = new FileInputStream("src/abc.txt");
            // 添加缓冲区
            byte[] buffer = new byte[1024];
            // 读取到缓冲区的总字节数，或者如果没有更多的数据，因为已经到达流的末尾，则是-1
            int read = 0;
            while ((read = inputStream.read(buffer)) != -1) {
                System.out.println(new String(buffer, 0, read));
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                inputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
控制台：
    abcdefghijklmnopqrstuvwxyz
```

#### 3.2.3 read(byte[] b, int off, int len)

从输入流读取最多 `len`字节的数据到一个字节数组

```java
public class StreamDemo {
    public static void main(String[] args) {
        InputStream inputStream = null;
        try {
            inputStream = new FileInputStream("src/abc.txt");
            // 添加缓冲区
            byte[] buffer = new byte[1024];
            // 读取到缓冲区的总字节数，或者如果没有更多的数据，因为已经到达流的末尾，则是-1
            int read = 0;
            while ((read = inputStream.read(buffer, 5 , 5)) != -1) {
                System.out.println(new String(buffer, 5, read));
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                inputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
控制台：
    abcde
    fghij
    klmno
    pqrst
    uvwxy
    z
```