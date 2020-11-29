# 外观模式

门面模式（Facade Pattern）也叫外观模式，是一种比较常用也非常简单的设计模式。

门面模式（Facade Pattern）隐藏系统的复杂性，并向客户端提供了一个客户端可以访问系统的接口。它向现有的系统添加一个接口，来隐藏系统的复杂性。

## 1 门面模式的定义

外观模式的英文原话是：

> Provide a unified interface to a set of interfaces in a subsystem. Facadedefines a higher-level interface that makes the subsystem easier to use.

意思是：要求一个子系统的外部与其内部的通信必须通过一个统一的对象进行。外观模式提供一个高层次的接口，使得子系统更易使用。

外观模式注重“统一的对象”，即提供一个访问子系统的接口，只有通过该接口（Façade）才能允许访问子系统的行为发生，其示意图如图所示。

![image-20201129000757778](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201129000757778.png)

外观模式具有以下两个角色：

- 外观（Facade）角色：客户端可以调用该角色的方法，该角色知晓相关子系统的功能和责任。正常情况下，本角色会将所有从客户端发来的请求委派到相应的子系统，即该角色没有实际的业务逻辑，只是一个委托类。
- 子系统（Subsystem）角色：可以同时有一个或多个子系统，每一个子系统都不是一个单独的类，而是一个类的集合。子系统不知道外观角色的存在，对于子系统而言，外观角色仅仅是另外一个客户端而已。