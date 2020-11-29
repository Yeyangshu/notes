#  工厂方法设计模式、

工厂方法模式（Factory Method Pattern）又叫虚拟构造函数（VirtualConstructor）模式或者多态性工厂（Polymorphic Factory）模式。工厂方法模式的用意是定义一个创建产品对象的工厂接口，将实际创建性工作推迟到子类中。

## 1 工厂方法模式的定义

工厂模式的英文原话是：

> Define an interface for creating an object, but let subclasses decide whichclass to instantiate. Factory Method lets a class defer instantiation tosubclasses.

意思是：定义一个用于创建对象的接口，让子类决定实例化哪个类。工厂方法使一个类的实例化延迟到其子类。

