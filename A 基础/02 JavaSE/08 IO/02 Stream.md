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

### 2.3 按功能划分

- 节点流：可以直接从数据源或目的地读写数据
- 处理流（包装流）：不直接连接到数据源或数据库，是其他流进行封装。目的主要是简化操作和提高性能

节点流和处理流的关系

- 节点流处于io操作的第一线，所有操作必须通过他们进行
- 处理流可以对其他流进行处理（提高效率或操作灵活性）

![image-20200716003918168](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200716003918168.png)

## 3 流的使用

### 3.1 流使用的步骤

1. 选择合适的io流对象
2. 创建对象
3. 传输数据
4. 关闭流对象（）

### 3.2 inputStream read

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

#### 3.2.4 练习

复制abc.txt内容到文件aaa.txt

```java
public class CopyFile {
    public static void main(String[] args) {
        // 定义数据文件
        File src = new File("abc.txt");

        File dest = new File("aaa.txt");
        // 创建输入流对象
        InputStream inputStream = null;
        // 创建输出流对象
        OutputStream outputStream = null;

        try {
            inputStream = new FileInputStream(src);
            outputStream = new FileOutputStream(dest);

            // 带缓冲的输入输出方式
            byte[] buffer = new byte[1024];
            int length = 0;
            while ((length = inputStream.read(buffer)) != -1) {
                // 完成数据传输的过程
                outputStream.write(buffer);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                outputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                inputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## 4 文件的读写

### 4.1 文本文件的读写

- 用FileInputStream和FileOutputStream读写文本文件
- 用读写文本文件

### 4.2 二进制文件的读写

### 4.3 对象的读写

### 4.4 字节流练习

#### 4.4.1 FileReader读文件练习、FileWriter写文件练习

文件内容：中国abcdefghijklmnopqrstuvwxyz

**读文件**

1. 单字符读取

   ```java
   /**
    * 字符流可以直接读取中文汉字，而字节流在处理的时候会出现中文乱码
    */
   public class ReaderDemo {
       public static void main(String[] args) {
           Reader reader = null;
           try {
               reader = new FileReader("abc.txt");
               int read = reader.read();
               System.out.println((char)read);
           } catch (FileNotFoundException e) {
               e.printStackTrace();
           } catch (IOException e) {
               e.printStackTrace();
           } finally {
               try {
                   reader.close();
               } catch (IOException e) {
                   e.printStackTrace();
               }
           }
       }
   }
   控制台：
       中
   ```

2. 循环读取

   ```java
   public class ReaderDemo {
       public static void main(String[] args) {
           Reader reader = null;
           try {
               reader = new FileReader("abc.txt");
               int read = 0;
               while ((read = reader.read()) != -1) {
                   System.out.println((char)read);
               }
           } catch (FileNotFoundException e) {
               e.printStackTrace();
           } catch (IOException e) {
               e.printStackTrace();
           } finally {
               try {
                   reader.close();
               } catch (IOException e) {
                   e.printStackTrace();
               }
           }
       }
   }
   控制台：
       中
       国
       a
       ...
   ```

3. 带缓冲区读取

   ```java
   /**
    * 字符流可以直接读取中文汉字，而字节流在处理的时候会出现中文乱码
    */
   public class ReaderDemo {
       public static void main(String[] args) {
           Reader reader = null;
           try {
               reader = new FileReader("abc.txt");
               int length = 0;
               char[] buffer = new char[1024];
               while ((length = reader.read(buffer)) != -1) {
                   System.out.println(new String(buffer, 0, length));
               }
           } catch (FileNotFoundException e) {
               e.printStackTrace();
           } catch (IOException e) {
               e.printStackTrace();
           } finally {
               try {
                   reader.close();
               } catch (IOException e) {
                   e.printStackTrace();
               }
           }
       }
   }
   控制台：
       中国abcdefghijklmnopqrstuvwxyz
   ```

**写文件**

1. 不关闭writer对象，文件内没有内容

   ```java
   public class WriterDemo {
       public static void main(String[] args) {
           File file = new File("writer.txt");
           Writer writer = null;
           try {
               writer = new FileWriter(file);
               writer.write("www.");
               writer.write("baidu.com");
           } catch (IOException e) {
               e.printStackTrace();
           } finally {
               
           }
       }
   }
   // 有文件生成，但是文件内没有内容
   ```

2. 关闭writer对象或者使用flush方法，文件内有内容

   什么时候需要加flush，什么时候不加flush？

   - 最保险的方式，在输出流关闭之前每次都flush，然后再关闭
   - 当某一个输出流带有缓冲区的时候，就需要进行flush

   ```java
   public class WriterDemo {
       public static void main(String[] args) {
           File file = new File("writer.txt");
           Writer writer = null;
           try {
               writer = new FileWriter(file);
               writer.write("www.");
               writer.write("baidu.com");
               writer.flush();
           } catch (IOException e) {
               e.printStackTrace();
           } finally {
               try {
                   writer.close();
               } catch (IOException e) {
                   e.printStackTrace();
               }
           }
       }
   }
   writer.txt文件内容：
       www.baidu.com
   ```

#### 4.1.2 图片、视频及其他格式的处理

文本文件可以使用字节流和字符流，图片、视频最好使用字节流

```java
public class WriterDemo {
    public static void main(String[] args) {
        FileInputStream fileInputStream = null;
        FileOutputStream fileOutputStream = null;

        try {
            fileInputStream = new FileInputStream("1.jpg");
            fileOutputStream = new FileOutputStream("2.jpg");
            int length = 0;
            byte[] buffer = new byte[1024];
            while ((length = fileInputStream.read(buffer)) != -1) {
                fileOutputStream.write(buffer);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                fileOutputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                fileInputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

#### 4.1.3 InputStreamReader、OutputStreamWriter练习

- InputStreamReader：InputStreamReader是从字节流到字符流的桥：它读取字节，并使用指定的charset将其解码为字符 。 它使用的字符集可以由名称指定，也可以被明确指定，或者可以接受平台的默认字符集。
- OutputStreamWriter：OutputStreamWriter是字符的桥梁流以字节流：向其写入的字符编码成使用指定的字节charset 。 它使用的字符集可以由名称指定，也可以被明确指定，或者可以接受平台的默认字符集。

```java
public class OutputStreamWriterDemo {
    public static void main(String[] args) {
        File file = new File("abc.txt");
        OutputStreamWriter outputStreamWriter = null;
        FileOutputStream fileOutputStream = null;

        try {
            fileOutputStream = new FileOutputStream(file);
            outputStreamWriter = new OutputStreamWriter(fileOutputStream, "UTF-8");
            outputStreamWriter.write("www");
            outputStreamWriter.write(".baidu.com");
            outputStreamWriter.write("123456789", 0, 5);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                outputStreamWriter.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                fileOutputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
控制台：
    www.baidu.com12345
```

#### 4.1.4 ByteArrayInputStream、ByteArrayOutputStream练习

先写后读

1. ByteArrayInputStream单个字节

   ```java
   public class ByteArrayInputStreamDemo {
       public static void main(String[] args) {
           String str = "www.baidu.com";
           byte[] buffer = str.getBytes();
           ByteArrayInputStream byteArrayInputStream = null;
           
           byteArrayInputStream = new ByteArrayInputStream(buffer);
           int read = byteArrayInputStream.read();
           System.out.println((char)read);
   
           try {
               byteArrayInputStream.close();
           } catch (IOException e) {
               e.printStackTrace();
           }
       }
   }
   ```

2. ByteArrayInputStream循环打印，skip跳过1个字节

   skip：从此输入流跳过 `n`个字节的输入。

   ```java
   public class ByteArrayInputStreamDemo {
       public static void main(String[] args) {
           String str = "www.baidu.com";
           byte[] buffer = str.getBytes();
           ByteArrayInputStream byteArrayInputStream = null;
           byteArrayInputStream = new ByteArrayInputStream(buffer);
           int read = 0;
           while ((read = byteArrayInputStream.read()) != -1) {
               byteArrayInputStream.skip(1);
               System.out.println((char)read);
           }
   
           try {
               byteArrayInputStream.close();
           } catch (IOException e) {
               e.printStackTrace();
           }
       }
   }
   控制台：
       w
       w
       b
       i
       u
       c
       m
   ```

3. ByteArrayOutputStream

   ```java
   public class ByteArrayOutputStreamDemo {
       public static void main(String[] args) {
           ByteArrayOutputStream byteArrayOutputStream = null;
           byteArrayOutputStream = new ByteArrayOutputStream();
           byteArrayOutputStream.write(123);
   
           try {
               byteArrayOutputStream.write("www.baidu.com".getBytes());
               System.out.println(byteArrayOutputStream.toString());
           } catch (IOException e) {
               e.printStackTrace();
           } finally {
               try {
                   byteArrayOutputStream.close();
               } catch (IOException e) {
                   e.printStackTrace();
               }
           }
       }
   }
   ```

#### 4.1.5 BufferedInputStream、BufferedOutputStream

先写后读

```java
public class BufferedInputStreamDemo {
    public static void main(String[] args) {
        File file = new File("abc.txt");
        FileInputStream fileInputStream = null;
        BufferedInputStream bufferedInputStream = null;
        try {
            fileInputStream = new FileInputStream(file);
            bufferedInputStream = new BufferedInputStream(fileInputStream);
            int read = 0;
            while ((read = bufferedInputStream.read()) != -1) {
                System.out.print((char) read);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                bufferedInputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }

            try {
                fileInputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

#### 4.1.6 DataInputStream、DataOutputStream

```java
public class DataInputStreamDemo {
    public static void main(String[] args) {
        FileOutputStream fileOutputStream = null;
        DataOutputStream dataOutputStream = null;
        FileInputStream fileInputStream = null;
        DataInputStream dataInputStream = null;
        try {
            // 向文件写入数据源
            fileOutputStream = new FileOutputStream("abc.txt");
            dataOutputStream = new DataOutputStream(fileOutputStream);
            dataOutputStream.writeBoolean(true);
            dataOutputStream.writeInt(123);
            dataOutputStream.writeDouble(10.10);
            dataOutputStream.writeUTF("www.baidu.com");
            // 从文件读取数据源
            fileInputStream = new FileInputStream("abc.txt");
            dataInputStream = new DataInputStream(fileInputStream);
            System.out.println(dataInputStream.readBoolean());
            System.out.println(dataInputStream.readInt());
            System.out.println(dataInputStream.readDouble());
            System.out.println(dataInputStream.readUTF());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

#### 4.1.7 FileOutputStream、FileInputStream

- 如果需要将对象通过io进行传输，那么必须实现序列化
- 当前类中必须声明一个serialVersionUID的值，值随意必须有
- transient关键字修饰的变量，在进行序列化的时候不会被序列化

```java
public class ObjectInputStreamDemo {
    public static void main(String[] args) {
        FileOutputStream fileOutputStream = null;
        ObjectOutputStream objectOutputStream = null;

        FileInputStream fileInputStream = null;
        ObjectInputStream objectInputStream = null;

        try {
            fileOutputStream = new FileOutputStream("abc.txt");
            objectOutputStream = new ObjectOutputStream(fileOutputStream);
            objectOutputStream.writeObject(new Person("小明"));

            fileInputStream = new FileInputStream("abc.txt");
            objectInputStream = new ObjectInputStream(fileInputStream);
            Person person = (Person)objectInputStream.readObject();
            System.out.println(person.toString());
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } finally {
            try {
                objectInputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                fileInputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                objectOutputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                fileOutputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
控制台：
    Person{name='小明'}
```

### 4.5 字符流练习

#### 4.5.1 CharArrayReader、CharArrayWriter

该类实现了一个字符缓冲区，可以用作字符输入流。

```java
public class CharArrayReaderDemo {
    public static void main(String[] args) {
        char[] chars = "叶杨树".toCharArray();
        CharArrayReader charArrayReader = new CharArrayReader(chars);
        int read = 0;
        try {
            while ((read = charArrayReader.read()) != -1) {
                System.out.println((char)read);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            charArrayReader.close();
        }
    }
}
```