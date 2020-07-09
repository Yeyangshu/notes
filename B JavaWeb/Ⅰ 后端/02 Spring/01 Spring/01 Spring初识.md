[TOC]

# Spring初识

## 1 架构

随着互联网的发展，网站应用的规模不断扩大，常规的垂直应用架构已无法应对，分布式服务架构以及流动计算架构势在必行，亟需一个治理系统确保架构有条不紊的演进。

![image-20200707002843819](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200707002843819.png)

### 1 单一应用架构

当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。此时，用于简化增删改查工作量的数据访问框架(ORM)是关键。

### 2 垂直应用架构

当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，提升效率的方法之一是将应用拆成互不相干的几个应用，以提升效率。此时，用于加速前端页面开发的Web框架(MVC)是关键。

### 3 分布式服务架构

当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。此时，用于提高业务复用及整合的分布式服务框架(RPC)是关键。

### 4 流动计算架构

当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率。此时，用于提高机器利用率的资源调度和治理中心(SOA)是关键。

## 2 Java主流框架演变之路

1. JSP + Servlet + JavaBean

2. MVC三层架构

3. EJB，重量级框架，麻烦

4. Struts1/Struts2 + Hibernate + Spring
5. SpringMVC + Mybatis + Spring
6. SpringBoot开发，约定大于配置

## 3 Spring

官网地址：https://spring.io/projects/spring-framework#overview

压缩包下载地址：https://repo.spring.io/release/org/springframework/spring/

源码地址：https://github.com/spring-projects/spring-framework

### 3.1 核心解释

spring是一个开源框架。

​		spring是为了简化企业开发而生的，使得开发变得更加优雅和简洁。

​		spring是一个**IOC**和**AOP**的容器框架。

​				IOC：控制反转

​				AOP：面向切面编程

​				容器：包含并管理应用对象的生命周期，就好比用桶装水一样，spring就是桶，而对象就是水

### 3.2 使用spring的优点

1. Spring通过DI、AOP和消除样板式代码来简化企业级Java开发
2. Spring框架之外还存在一个构建在核心框架之上的庞大生态圈，它将Spring扩展到不同的领域，如Web服务、REST、移动开发以及NoSQL
3. 低侵入式设计，代码的污染极低
4. 独立于各种应用服务器，基于Spring框架的应用，可以真正实现Write Once,Run Anywhere的承诺
5. Spring的IoC容器降低了业务对象替换的复杂性，提高了组件之间的解耦
6. Spring的AOP支持允许将一些通用任务如安全、事务、日志等进行集中式处理，从而提供了更好的复用
7. Spring的ORM和DAO提供了与第三方持久层框架的的良好整合，并简化了底层的数据库访问
8. Spring的高度开放性，并不强制应用完全依赖于Spring，开发者可自由选用Spring框架的部分或全部

### 3.3 如何简化开发

- 基于POJO的轻量级和最小侵入性编程
- 通过依赖注入和面向接口实现松耦合
- 基于切面和惯例进行声明式编程
- 通过切面和模板减少样板式代码

### 3.4 Spring模块划分

![image-20200707002826910](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200707002826910.png)

>模块解释：
>Test:Spring的单元测试模块
>Core Container:核心容器模块
>AOP+Aspects:面向切面编程模块
>Instrumentation:提供了class instrumentation支持和类加载器的实现来在特定的应用服务器上使用,几乎不用
>Messaging:包括一系列的用来映射消息到方法的注解,几乎不用
>Data Access/Integration:数据的获取/整合模块，包括了JDBC,ORM,OXM,JMS和事务模块
>Web:提供面向web整合特性

## 4 IOC初识

### 4.1 基本概念

先弄清楚如下几个问题：

1. 谁控制谁
2. 控制什么
3. 什么是反转

4. 哪些方面被反转

> Spring官方文档：Core Technologies

```txt
 IoC is also known as dependency injection (DI). It is a process whereby objects define their dependencies (that is, the other objects they work with) only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method. The container then injects those dependencies when it creates the bean. This process is fundamentally the inverse (hence the name, Inversion of Control) of the bean itself controlling the instantiation or location of its dependencies by using direct construction of classes or a mechanism such as the Service Locator pattern.
 
 IOC与大家熟知的依赖注入同理。这是一个通过依赖注入对象的过程，也就是说，它们所使用的对象，是通过构造函数参数，工厂方法的参数或这是从工厂方法的构造函数或返回值的对象实例设置的属性，然后容器在创建bean时注入这些需要的依赖。这个过程相对普通创建对象的过程是反向的（因此称之为IoC），bean本身通过直接构造类来控制依赖关系的实例化或位置，或提供诸如服务定位器模式之类的机制。
```

1. 谁控制谁：在之前的编码过程中，都是需要什么对象自己去创建什么对象，有程序员自己来控制对象，而有了IOC容器之后，就会变成由IOC容器来控制对象，
2. 控制什么：在实现过程中所需要的对象及需要依赖的对象
3. 什么是反转：在没有IOC容器之前我们都是在对象中主动去创建依赖的对象，这是正转的，而有了IOC之后，依赖的对象直接由IOC容器创建后注入到对象中，由主动创建变成了被动接受，这是反转
4. 哪些方面被反转：依赖的对象

### 4.2 IOC和DI

很多人把IOC和DI说成一个东西，笼统来说的话是没有问题的，但是本质上还是有所区别的，希望大家能够严谨一点，IOC和DI是从不同的角度描述的同一件事，IOC是从容器的角度描述，而DI是从应用程序的角度来描述，也可以这样说，IOC是设计思想，而DI是具体的实现方式。

## 5 总结

### 5.1 解耦

在面向对象设计的软件系统中，底层的实现都是由N个对象组成的，所有的对象通过彼此的合作，最终实现系统的业务逻辑。

![image-20200709235035764](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200709235035764.png)

需要注意的是，在这样的组合关系中，一旦某一个对象出现了问题，那么其他对象肯定回有所影响，这就是耦合性太高的缘故，但是对象的耦合关系是无法避免的，也是必要的。随着应用程序越来越庞大，对象的耦合关系可能越来越复杂，经常需要多重依赖关系，因此，无论是架构师还是程序员，在面临这样的场景的时候，都需要减少这些对象的耦合性。

![image-20200709235101841](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200709235101841.png)

耦合的关系不仅仅是对象与对象之间，也会出现在软件系统的各个模块之间，是我们需要重点解决的问题。而为了解决对象之间的耦合度过高的问题，我们就可以通过IOC来实现对象之间的解耦，spring框架就是IOC理论最最广泛的应用。

![image-20200709234954146](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200709234954146.png)

### 5.2 生态

任何一个语言或者任何一个框架想要立于不败之地，那么很重要的就是它的生态。