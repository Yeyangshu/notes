# 责任链模式

## 1 责任链模式的定义

责任链模式的英文原话是

> Avoid coupling the sender of a request to its receiver by giving more thanone object a chance to handle the request. Chain the receiving objects andpass the request along the chain until an object handles it.

意思是：使多个对象都有机会处理请求，从而避免了请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递该请求，直到有对象处理它为止。







![image-20201128234752738](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201128234752738.png)

![image-20201129191717957](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201129191717957.png)

https://javaee.github.io/javaee-spec/javadocs/

# 适配器模式

适配器模式（Adapter Pattern）又叫做变压器模式，变压器把一种电压变换为另一种电压。

## 1 适配器模式的定义

适配器模式的英文原话是：

> Convert the interface of a class into another interface clients expect.Adapter lets classes work together that couldn't otherwise because ofincompatible interfaces.

意思是：将一个类的接口变换成客户端所期待的另一种接口，从而使原本因接口不匹配而无法在一起工作的两个类能够在一起工作。



接口转换器

常见例子：

- 电压转换头
- java.io
- jdbc-odbc bridge（不是桥接模式）
- ASM transfer

误区：

- 常见的Adapter类反而不是Adapter
- WindowAdapter
- KeyAdapter



```java
import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.InputStreamReader;

public class Client {
    public static void main(String[] args) throws Exception {
        FileInputStream fis = new FileInputStream("c:/test.text");
        InputStreamReader isr = new InputStreamReader(fis);
        BufferedReader br = new BufferedReader(isr);
        String line = br.readLine();
        while (line != null && !line.equals("")) {
            System.out.println(line);
        }
        br.close();
    }

}
```





# 