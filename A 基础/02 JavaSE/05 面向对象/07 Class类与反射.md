# Java反射机制

JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

Java反射机制提供的主要功能有：

- 得到一个对象所属的类
- 获取一个类的所有成员变量和方法
- 在运行时创建对象，在运行时调用对象的方法

所有Java类均继承了 Object 类，在 Object 类中定义了一个 getClass() 方法，该方法返回一个类型为 Class 的对象。代码如下

```java
Class textFiledC = textField.getClass();
```

Class 类的描述信息：

![image-20201126080117903](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201126080117903.png)



在反射机制中，Class 是一个非常重要的类，如何才能获取 Class 类？总共有三种方法可以获取：

1. Class.forName(“类的全路径”)

   Person类

   ```java
   public class Person {
   	private int id;
   	private String name;
   
   	public Person() {
   		System.out.println("person");
   	}
   }
   ```

   测试代码：

   ```java
   public class T {
   
   	public static void main(String[] args) {
   		Class clazz = null;
   		try {
               // 获取 Class
   			clazz = Class.forName("com.yeyangshu.Person");
               // 动态创建对象
   			Person person = (Person)clazz.newInstance();
   		} catch (Exception e) {
   			e.printStackTrace();
   		}
   	}
   	
   	/**
   	 * person
   	 */
   }
   ```

2. 类名.Class

   测试代码：

   ```java
   public class T {
   
   	public static void main(String[] args) {
   		Class clazz = null;
   		try {
               // 获取 Class
   			clazz = Person.class;
               // 动态创建对象
   			Person person = (Person)clazz.newInstance();
   		} catch (Exception e) {
   			e.printStackTrace();
   		}
   	}
   
   	/**
   	 * person
   	 */
   }
   ```

3. 实例.getClass

   测试代码：

   ```java
   public class T {
   
   	public static void main(String[] args) {
   		Class clazz = null;
   		try {
               // 获取 Class
   			Person person = new Person();
   			clazz = person.getClass();
               // 动态创建对象
   			Person p = (Person)clazz.newInstance();
   		} catch (Exception e) {
   			e.printStackTrace();
   		}
   	}
   
   	/**
   	 * person
   	 * person
   	 */
   }
   ```

   

## 1 访问构造方法

## 2 访问成员变量

## 3 访问方法

