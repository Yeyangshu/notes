# 中介者模式

中介者模式（Mediator）也称调停者模式，是一种比较简单的模式。

## 1 中介者模式的定义

中介者模式的英文原话是：

> Define an object that encapsulates how a set of objects interact. Mediatorpromotes loose coupling by keeping objects from referring to each otherexplicitly, and it lets you vary their interaction independently.

意思是：用一个中介对象封装一系列对象（同事）的交互，中介者使各对象不需要显式地相互作用，从而使其耦合松散，而且可以独立地改变它们之间的交互。

中介者模式有以下4个角色：

- 抽象中介者（Mediator）角色：该角色定义出同事对象到中介者对象的统一接口，用于各同事角色之间的通信。

- 具体中介者（Concrete Mediator）角色：该角色实现抽象中介者，它依赖于各个同事角色，并通过协调各同事角色实现协作行为。

- 抽象同事（Colleague）角色：该角色定义出中介者到同事对象的接口，同事对象只知道中介者而不知道其余的同事对象。

- 具体同事（Concrete Colleague）角色：该角色实现抽象同事类，每一个具体同事类都清楚自己在小范围内的行为，而不知道大范围内的目的。