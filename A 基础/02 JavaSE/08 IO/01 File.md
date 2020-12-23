# File类

## 1 文件

1. 什么是文件？

   文件可以认为是相关记录或放在一起的数据的集合

2. 文件一般存储在哪里？

   硬盘、光盘、磁盘等等

3. Java程序如何访问文件属性？

   File类：File类可以创建文件对象，操作文件的目录或属性

请注意：**File只能访问文件以及文件系统的元数据。**如果你想读写文件内容，需要使用FileInputStream、FileOutputStream或者RandomAccessFile。

## 2 File类方法

### 2.1 构造方法

- File(File parent, String child)

  从父抽象路径名和子路径名字符串创建新的 File 实例。

- File(String pathname)

  通过将给定的路径名字符串转换为抽象路径名来创建新的 File 实例。

- File(String parent, String child)

  从父路径名字符串和子路径名字符串创建新的 File 实例。

- File(URI uri)
  通过将给定的 file: URI 转换为抽象路径名来创建新的 File 实例。

### 2.2 常用方法

#### 2.2.1 exists()

当你获得一个File对象之后，可以检测相应的文件是否存在。当文件不存在的时候，构造函数并不会执行失败。

```java
// 磁盘没有对应文件
File file = new File("C:\\file.txt");
boolean fileExists = file.exists();
System.out.println(fileExists);
// false

// 磁盘新建对应文件
File file = new File("C:\\file.txt");
boolean fileExists = file.exists();
System.out.println(fileExists);
// true
```

#### 2.2.2 length()

通过调用length()可以获得文件的字节长度

```java
File file = new File("C:\\Study\\io.txt");
long length = file.length();
System.out.println(length);
// 38
```

#### 2.2.3 renameTo()

通过调用File类中的renameTo()方法可以重命名(或者移动)文件

```java
File file = new File("C:\\Study\\io.txt");
boolean success = file.renameTo(new File("C:\\Study\\new-file.txt"));
System.out.println(success);
// true
// 此时文件重命名为new-file.txt
```

#### 2.2.4 delete()

通过调用delete()方法可以删除文件

```java
File file = new File("C:\\Study\\io.txt");
boolean success = file.delete();
System.out.println(success);
```

#### 2.2.5 isDirectory()

File对象既可以指向一个文件，也可以指向一个目录。可以通过调用isDirectory()方法，可以判断当前File对象指向的是文件还是目录。当方法返回值是true时，File指向的是目录，否则指向的是文件。

```java
File file = new File("C:\\Study\\io.txt");
boolean isDirectory = file.isDirectory();
System.out.println(isDirectory);
// false
```

#### 2.2.6 list()

你可以通过调用list()或者listFiles()方法获取一个目录中的所有文件列表。list()方法返回当前File对象指向的目录中所有文件与子目录的字符串名称(不会返回子目录下的文件及其子目录名称)。listFiles()方法返回当前File对象指向的目录中所有文件与子目录相关联的File对象(与list()方法类似，不会返回子目录下的文件及其子目录)。代码如下：

```java
File file = new File("C:\\Study\\io");
String[] fileNames = file.list();
for (String filename : fileNames) {
    System.out.println(filename);
}
File[] files = file.listFiles();
for (File fileObject : files) {
    System.out.println(fileObject);
}

// HelloWorld.txt
// io.txt
// C:\Study\io\HelloWorld.txt
// C:\Study\io\io.txt
```

#### 2.2.7 其他方法（暂时不列）

```java
import java.io.File;
import java.io.IOException;

public class FileDemo {
    public static void main(String[] args) {
        File file = new File("abc.txt");

        try {
            file.createNewFile();
        } catch (IOException e) {
            e.printStackTrace();
        }

        // 判断文件的属性，返回Boolean类型的值
        System.out.println(file.canExecute());
        System.out.println(file.canRead());
        System.out.println(file.canWrite());

        // 判断当前文件是否存在
        System.out.println(file.exists());

        // 获取当前文件的名称
        System.out.println(file.getName());
        // 获取当前文件的绝对路径
        System.out.println(file.getAbsolutePath());
        // 获取文件的父路径名称，如果文件的路径中只包含文件名称，则显示为空
        System.out.println(file.getParent());
        // 返回此抽象路径名的规范形式。
        try {
            System.out.println(file.getCanonicalPath());
        } catch (IOException e) {
            e.printStackTrace();
        }
        // 返回文件的分隔符
        System.out.println(File.separator);

        // ----------------------------------
        // 无论当前文件是否存在，只要给定具体的文件名，都可以返回响应的路径名称
        File file1 = new File("c:/");
        System.out.println(file1.getAbsolutePath());
        // 判断文件是否是文件或目录
        System.out.println(file1.isDirectory());
        System.out.println(file1.isFile());

        // 返回一个字符串数组，命名由此抽象路径名表示的目录中满足指定过滤器的文件和目录。
        String[] list = file1.list();
        for (String str : list) {
            System.out.println(list.toString());
        }
        // 返回一个抽象路径名数组，表示由该抽象路径名表示的目录中的文件。用的较多
        File[] files = file1.listFiles();
        for (File f : files) {
            System.out.println(f);
        }
        // 打印当前系统所有盘符
        File[] files1 = File.listRoots();
        for (int i = 0; i < files1.length; i++) {
            System.out.println(files1[i]);
        }

        // 创建单级目录
        //file1.mkdir();
        // 创建多级目录
        //file1.mkdirs();

        // 循环遍历输出c盘的所有文件的绝对路径
        // 使用递归的方式
        printFile(new File("c:/git"));
    }

    /**
     * 文件在遍历的时候会出现空指针的问题，原因在于文件系统受到保护，某些文件没有访问权限，此时会报空指针异常
     * @param file
     */
    public static void printFile(File file) {
        if (file.isDirectory()) {
            File[] files = file.listFiles();
            for (File f : files) {
                printFile(f);
            }
        } else {
            System.out.println("此文件是一个具体的文件，只有一个文件名称");
            System.out.println(file.getAbsolutePath());
        }
    }
}
```