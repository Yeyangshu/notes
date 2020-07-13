# File类

## 1 文件

1. 什么是文件？

   文件可以认为是相关记录或放在一起的数据的集合

2. 文件一般存储在哪里？

   硬盘、光盘、磁盘等等

3. Java程序如何访问文件属性？

   File类

## 2 File类

### 2.1 构造方法

```
构造方法
Constructor and Description
File(File parent, String child)
从父抽象路径名和子路径名字符串创建新的 File实例。
File(String pathname)
通过将给定的路径名字符串转换为抽象路径名来创建新的 File实例。
File(String parent, String child)
从父路径名字符串和子路径名字符串创建新的 File实例。
File(URI uri)
通过将给定的 file: URI转换为抽象路径名来创建新的 File实例。
```

### 2.2 常用方法

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