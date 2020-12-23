文章链接：http://tutorials.jenkov.com/java-io/inputstream.html

# InputStream

InputStream类是Java IO API中所有输入流的基类。

## 1 InputStream子类

InputStream的每个子类都有非常特定的用途，但可以用作InputStream。 

InputStream子类有：

- ByteArrayInputStream
- FileInputStream
- PipedInputStream
- BufferedInputStream
- FilterInputStream
- PushbackInputStream
- DataInputStream
- ObjectInputStream
- SequenceInputStream

## 2 输入流和源

Java InputStream通常连接到某些数据源，例如文件，网络连接，管道等。

## 3 Java InputStream方法

InputStream用于读取基于字节的数据，一次读取一个字节，这是一个InputStream的例子：

```java
InputStream inputStream = new FileInputStream("C:\\Study\\io.txt");

int data = inputStream.read();
while (data != -1) {
    // do something with data...

    // read next byte.
    data = inputStream.read();
}
inputStream.close();
```

本示例创建一个新的FileInputStream实例。 FileInputStream是InputStream的子类，因此可以安全地将FileInputStream的实例分配给InputStream变量（inputstream变量）。

### 3.1 read()

InputStream的read()方法返回一个int值，其中包含读取的字节的字节值。 这是一个InputStream.read()示例：

```java
int data = inputStream.read();
```

要读取Java InputStream中的所有字节，必须继续读取直到返回值-1。 此值意味着没有更多字节可从InputStream中读取。 这是从Java InputStream读取所有字节的示例：

```java
int data = inputStream.read();
while(data != -1) {
    // do something with data variable

    // read next byte
    data = inputStream.read();
}
```

InputStream的子类可能会包含read()方法的替代方法。比如，DataInputStream允许你利用readBoolean()，readDouble()等方法读取Java基本类型变量int，long，float，double和boolean。

### 3.2 流末尾

如果read()方法返回-1，意味着程序已经读到了流的末尾，此时流内已经没有多余的数据可供读取了。-1是一个int类型，不是byte或者char类型，这是不一样的。

当达到流末尾时，你就可以关闭流了。

### 3.3 read(byte[])

InputStream包含了2个从InputStream中读取数据并将数据存储到缓冲数组中的read()方法，他们分别是：

- int read(byte b[])

  read(byte[])方法会尝试读取与给定字节数组容量一样大的字节数，返回值说明了已经读取过的字节数。如果InputStream内可读的数据不足以填满字节数组，则数组剩余部分将保留参数传入时的原始数据，记得检查有多少数据实际被写入到了字节数组中，以防读取无效数据。

- int read(byte b[], int offset, int length)

  read(byte, int offset, int length)方法同样将数据读取到字节数组中，不同的是，该方法从数组的offset位置开始，并且最多将length个字节写入到数组中。同样地，read(byte, int offset, int length)方法返回一个int变量，告诉你已经有多少字节已经被写入到字节数组中，所以请记得在读取数据前检查上一次调用read(byte, int offset, int length)的返回值。

这两个方法都会在读取到达到流末尾时返回-1。

一次性读取一个字节数组的方式，比一次性读取一个字节的方式快的多，所以，尽可能使用这两个方法代替read()方法。


这是一个使用InputStream的read(byte[])的例子：

```java
InputStream inputStream = new FileInputStream("C:\\Study\\io.txt");

byte[] data = new byte[1024];
int bytesRead = inputStream.read(data);
while (bytesRead != -1) {
    // doSomethingWithData(data, bytesRead)...

    // read next byte.
    bytesRead = inputStream.read(data);
}
inputStream.close();
```

### 3.4 readAllBytes()

Java InputStream类包含一个名为readAllBytes()的方法（自Java 9开始）。 此方法读取InputStream中所有可用的字节，并返回包含字节的单个字节数组。如果您需要通过FileInputStream将文件中的所有字节读入字节数组，则此方法很有用。 这是一个通过readAllBytes()从Java InputStream读取所有字节的示例：

```java
byte[] fileBytes = null;
try(InputStream input = new FileInputStream("myfile.txt")) {
   fileBytes = input.readALlBytes();
}
```

### 3.5 mark() and reset()

InputStream类有两个方法，分别为mark()和reset()，InputStream的子类可能支持也可能不支持。

如果InputStream子类支持mark()和reset()方法，则该子类应重写markSupported()以返回true。 如果markSupported()方法返回false，则不支持mark()和reset()。

mark()在InputStream中内部设置一个标记，该标记标记流中到目前为止已读取数据的点。 然后，使用InputStream的代码可以继续从中读取数据。 如果使用InputStream的代码想要返回到流中设置标记的点，则该代码在InputStream上调用reset()。 然后，InputStream“倒带”并返回标记，并再次从该点开始返回（读取）数据。 当然，这将导致某些数据从InputStream中返回多次。

在实现解析器时，通常使用mark()和reset()方法。 有时解析器可能需要在InputStream中提前读取，如果解析器没有找到期望的内容，则可能需要后退并尝试将读取的数据与其他内容进行匹配。

### 3.6 close()

完成Java InputStream后，必须将其关闭。 您可以通过调用InputStream close（）方法来关闭InputStream。 这是打开InputStream，从中读取所有数据，然后关闭它的示例：

```java
InputStream inputstream = new FileInputStream("c:\\data\\input-text.txt");

int data = inputstream.read();
while(data != -1) {
    data = inputstream.read();
}
inputstream.close();
```

注意while循环如何继续，直到从InputStream read（）方法中读取-1值为止。 之后，while循环退出，并调用InputStream close（）方法。

上面的代码不是100％健壮的。 如果从InputStream读取数据时引发了异常，则永远不会调用close（）方法。 为了使代码更健壮，您将必须使用Java try-with-resources构造。 我在Java IO异常处理的教程中也解释了使用Java IO类的正确异常处理。

这是使用try-with-resources构造关闭Java InputStream的示例：

```java
try (InputStream inputstream = new FileInputStream("file.txt")) {

    int data = inputstream.read();
    while (data != -1) {
        data = inputstream.read();
    }
}
```

请注意，现在在try关键字后的括号内声明InputStream。 这向Java发出信号，该InputStream将由try-with-resources构造进行管理。

一旦执行线程退出try块，就关闭inputstream变量。 如果从try块内部引发了异常，则捕获该异常，关闭InputStream，然后重新抛出该异常。 因此，可以保证在try-with-resources块中使用InputStream时将其关闭。

## 4 读取性能

一次读取一个字节数组要比一次从Java InputStream读取单个字节快。 通过读取字节数组而不是一次读取单个字节，该差异很容易成为性能提高的10倍或更多。

实际获得的加速取决于读取的字节数组的大小以及运行代码的计算机的OS，硬件等。 在决定之前，您应该研究目标系统的硬盘缓冲区大小等。 但是，如果缓冲区大小为8KB或更高，则可以实现很好的加速。 但是，一旦您的字节数组超出了底层操作系统和硬件的容量，就不会从更大的字节数组中获得更大的加速。

您可能必须尝试不同的字节数组大小并测量读取性能，以找到最佳的字节数组大小。

## 5 通过BufferedInputStream进行透明缓冲

您可以使用Java BufferedInputStream从InputStream添加透明的自动读取和缓冲字节数组。 BufferedInputStream从基础InputStream读取字节块到字节数组。 然后，您可以从BufferedInputStream逐个读取字节，并且仍然可以通过读取字节数组而不是一次读取一个字节来获得很多加速。 这是将Java InputStream包装在BufferedInputStream中的示例：

```java
// buffer size
InputStream input = new BufferedInputStream(new FileInputStream("c:\\data\\input-file.txt"), 1024 * 1024);
```

## 6 将InputStream转换为Reader

Java InputStream是基于字节的数据流。 您可能知道，Java IO API还具有一个基于字符的输入流集，称为“读取器”。 您可以使用Java InputStreamReader将Java InputStream转换为Java Reader。

下面是将InputStream转换为InputStreamReader的快速示例：

```java
InputStream inputStream       = new FileInputStream("c:\\data\\input.txt");
Reader      inputStreamReader = new InputStreamReader(inputStream);
```