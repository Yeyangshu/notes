# 装饰者模式

装饰模式（Decorator Pattern）是一种比较常见的模式。

## 1 装饰模式的定义

装饰模式的英文原话是：

> Attach additional responsibilities to an object dynamically keeping thesame interface. Decorators provide a flexible alternative to subclassing forextending functionality.

意思是：动态地给一个对象添加一些额外的职责。就增加功能来说，装饰模式比生成子类更为灵活。

装饰模式有以下4个角色。

- 抽象构件（Component）角色：该角色用于规范需要装饰的对象（原始对象）。

- 具体构件（Concrete Component）角色：该角色实现抽象构件接口，定义一个需要装饰的原始类。

- 装饰（Decorator）角色：该角色持有一个构件对象的实例，并定义一个与抽象构件接口一致的接口。

- 具体装饰（Concrete Decorator）角色：该角色负责对构件对象进行装饰。



**关键代码：** 1、Component 类充当抽象角色，不应该具体实现。 2、修饰类引用和继承 Component 类，具体扩展类重写父类方法。

TankDecorator聚合GameObject

![image-20201128234640367](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201128234640367.png)