## 1 Spring

动态代理的实现方式（必考）

- jdk动态代
- cglib借助asm

### 为什么需要代理模式？

谈谈对SpringAOP Weaving（织入）的理解？

Spring的IOC/AOP的实现（必考）

Spring如何解决循环依赖（三级缓存）（必考）

Spring的@Transactional如何实现的（必考）

讲解Spring 框架中如何基于 AOP 实现的事务管理？

1、Spring的底层代码，

2、bean的生命周期 

3、循环引用问题，以及Spring中用到的设计模式

4、Spring和SpringBoot的区别

5、Spring的AOP的底层实现原理

6、Spring的事务是如何回滚的

7、Spring 是如何解决循环依赖的问题的？

8、Spring IOC的理解，原理与实现

9、Bean Factory与FactoryBean有什么区别？@Bean这个注解时如何实现Bean的注入的？

## 2 SpringBoot

1、Springboot启动过程中做了哪些事情？

2、Springboot 启动类上的注解 @Spring boot Application说明？

3、Springboot如何判断当前应用是否时web应用？

4、Spring boot整合jsp的流程，需要注意哪些点

## 3 SpringCloud

1、SpringCloud和dubbo的区别

2、项目中用到了哪些组件

3、eureka的原理，如何保证高可用性，和Zookeeper有什么区别

4、feign如何调用的

5、处理生产环境上配置生效问题

6、hystrix的降级策略有哪些

7、Springcloud eureka是如何注册服务、如何监测心跳的，它注册的流程是怎么样的

8、在分布式环境中如何快速发现某一台服务有问题

9、分布式集群系统对外提供接口的时候如何验证对方的身份

10、Eureka和zookeeper作为注册中心有什么区别



# Spring原理讲解

## 1 什么是Spring框架？Spring框架主要包含哪些模块？

Spring是一个开源框架，Spring是一个轻量级的Java开发框架。它是为了解决企业应用开发的复杂性而创建的。框架的主要优势之一就是其分层架构，分层架构允许使用者选择使用哪一个组件，同时为J2EE应用程序开发提供集成的框架。Spring使用基本的Javabean来完成以前可能只有EJB完成的事情。然而，Spring的用途不仅限于服务端的开发。从简单性。可测试性和松耦合的角度而言，任何Java应用都可以从Spring中受益。Spring的核心是控制反转（IoC）和面向切面（AOP）。简单来说，Spring是一个分层的full-stack（一站式）轻量级开源框架。

模块图位置：https://lfvepclr.gitbooks.io/spring-framework-5-doc-cn/content/2/2-2.html

**文字不需要背，模块图要记住**：JDBC、ORM、Transactions、Servlet、Web、AOP、Aspects、Beans、Core、Context、SpEL必须记住

## 2 Spring框架的优势（背）

1. Spring通过DI、AOP和消除样板式代码来简化企业级Java开发
2. Spring框架之外还存在一个构建在核心框架之上的庞大生态圈，它将Spring扩展到不同的领域，如Web服务、REST、移动开发以及NoSQL

3. 低侵入式设计，代码的污染极低

4. 独立于各种应用服务器，基于Spring框架的应用，可以真正实现Write Once，Run Anywhere的承诺

5. Spring的IoC容器降低了业务对象替换的复杂性，提高了组件之间的解耦

6. Spring的AOP允许将一些通用任务如安全、事务、日志等进行集中式处理，从而提供了更好的复用

7. Spring的ORM和DAO提供了与第三方持久层框架的的良好整合，并简化了底层的数据库访问

8. Spring的高度开放性，并不强制应用完全依赖于Spring，开发者可自由选用Spring框架的部分或全部

## 3 IoC和DI是什么？

控制反转是就是应用本身不负责依赖对象的创建和维护,依赖对象的创建及维护是由外部容器负责的,这样控制权就有应用转移到了外部容器,控制权的转移就是控制反转。

依赖注入是指：在程序运行期间，由外部容器动态地将依赖对象注入到组件中如：一般通过构造函数注入或者setter注入。

面试：

IoC思想

DI实现方式

说不清楚用婚介所案例

## 4 描述下Spring IoC容器初始化过程

Spring IoC容器的初始化可以分为三个过程：

1. 第一个过程是资源定位。这个Resource指的是BeanDefinition的资源定位。这个过程就是容器找到数据的过程。

2. 第二个就是BeanDefination的载入过程。这个载入过程是把用户定义好的Bean表示成IoC容器内部的数据结构，这个容器的内部数据结构就是BeanDefinition。

3. 第三个过程就是向IoC容器注入这些BeanDefinition的过程，这个过程就是将前面的BeanDefinition保存到HashMap中的过程。

   反射

## 5 BeanFactory和FactoryBean的区别？



## 6 BeanFactory和ApplicationContext的异同

## 7 Spring Bean 的生命周期



## 8 Spring AOP的实现原理



## 9 Spring 是如何管理事务的

## 10 Spring 的不同事务传播行为有哪些，干什么用的？

## 11 Spring 中用到了那些设计模式？

- 工厂模式
- 单例设计
- 代理设计 
- 模板方法
- 观察者模式
- 适配器模式
- 装饰者模式