# 原型模式

原型模式（Prototype Pattern）是一种简单的设计模式。

1 原型模式的定义原型模式的英文原话是：

> Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype.

意思是：用原型实例指定创建对象的种类，并且通过复制这些原型创建新的对象。

原型模式涉及3个角色：

- 客户（Client）角色：该角色提出创建对象的请求。

- 抽象原型（Prototype）角色：该角色是一个抽象角色，通常由一个Java接口或抽象类实现，给出所有的具体原型类所需的接口。

- 具体原型（Concrete Prototype）角色：该角色是被复制的对象，必须实现抽象原型接口。

Java中内置了克隆机制，Object类具有一个clone()方法，能够实现对象的克隆，使一个类支持克隆需要以下两步。

- 实现Cloneable接口；

- 覆盖Object的clone()方法，完成对象的克隆操作，通常只需要调用Object的clone()方法即可。为了使外部能够调用此类的clone()方法，可以将可访问性修改为public。

Java中的原型模式

- 自带

  - 实现原型模式需要实现标记型接口Cloneable
  - 一般会重写clone()方法
    - 如果只是重写clone()方法，而没有实现接口，调用时会报异常
  - 一般用于一个对象的属性已经确定，需要产生很多相同对象的时候
  - 需要区分深克隆与浅克隆

