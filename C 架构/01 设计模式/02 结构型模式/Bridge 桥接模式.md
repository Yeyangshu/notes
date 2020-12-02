# 桥接模式

桥梁模式（Bridge Pattern）也称桥接模式，是一种简单的、不常使用的设计模式。

## 1 桥梁模式的定义

桥梁模式的英文原话是：

> Decouple an abstraction from its implementation so that the two can varyindependently.

意思是：将抽象和实现解耦，使得两者可以独立地变化。

桥梁模式有以下4个角色：

- 抽象化（Abstraction）角色：该角色抽象化的给出定义，并保存一个对实现化对象的引用。

- 实现化（Implementor）角色：该角色给出实现化角色的接口，但不给出具体的实现。

- 修正抽象化（RefinedAbstraction）角色：该角色扩展抽象化角色，它引用实现化角色并对抽象化角色进行修正。

- 具体实现化（ConcreteImplementor）角色：该角色对实现化角色接口中的方法进行具体实现。



代码直接copy

礼物有各种分类

书也有各种分类



**聚合代替继承**

v1：哥哥追美眉，送礼物

v2：如果礼物分为温柔型的礼物和狂野型的礼物

- WarmGift

  ```java
  public class WarmGift extends Gift {
  }
  ```

- WildGift

  ```java
  public class WildGift extends Gift {
  }
  ```

v3：如果礼物分为温柔的礼物和狂野的礼物

- WarmGift WildGift

  这时Flower应该分为

- WarmFlower WildFlower

- WarmBook WildBook

如果再有别的礼物，比如抽象类型：ToughGift、ColdGift
或者具体的某种实现：Ring Car

就会产生类的爆炸

- WarmCar ColdRing WildCar WildFlower ...

v4：使用桥接模式：分离抽象与具体实现，让他们可以独自发展

- Gift -> WarmGift ColdGift WildGift

- GiftImpl -> Flower Ring Car

哥哥可以送出温暖型（抽象类的实现：温暖型）的花（具体实现礼物的子类：花）