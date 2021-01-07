# Spring

## 1 Spring基础

### 什么是Spring框架？Spring框架主要包含哪些模块？

Spring是一个开源框架，Spring是一个轻量级的Java开发框架。它是为了解决企业应用开发的复杂性而创建的。框架的主要优势之一就是其分层架构，分层架构允许使用者选择使用哪一个组件，同时为J2EE应用程序开发提供集成的框架。Spring使用基本的Javabean来完成以前可能只有EJB完成的事情。然而，Spring的用途不仅限于服务端的开发。从简单性。可测试性和松耦合的角度而言，任何Java应用都可以从Spring中受益。Spring的核心是控制反转（IoC）和面向切面（AOP）。简单来说，Spring是一个分层的full-stack（一站式）轻量级开源框架。

模块图位置：https://lfvepclr.gitbooks.io/spring-framework-5-doc-cn/content/2/2-2.html

**文字不需要背，模块图要记住**：JDBC、ORM、Transactions、Servlet、Web、AOP、Aspects、Beans、Core、Context、SpEL必须记住

### Spring框架的优势（背）

1. Spring通过DI、AOP和消除样板式代码来简化企业级Java开发
2. Spring框架之外还存在一个构建在核心框架之上的庞大生态圈，它将Spring扩展到不同的领域，如Web服务、REST、移动开发以及NoSQL

3. 低侵入式设计，代码的污染极低

4. 独立于各种应用服务器，基于Spring框架的应用，可以真正实现Write Once，Run Anywhere的承诺

5. Spring的IoC容器降低了业务对象替换的复杂性，提高了组件之间的解耦

6. Spring的AOP允许将一些通用任务如安全、事务、日志等进行集中式处理，从而提供了更好的复用

7. Spring的ORM和DAO提供了与第三方持久层框架的的良好整合，并简化了底层的数据库访问

8. Spring的高度开放性，并不强制应用完全依赖于Spring，开发者可自由选用Spring框架的部分或全部

### IoC和DI是什么？

控制反转是就是应用本身不负责依赖对象的创建和维护,依赖对象的创建及维护是由外部容器负责的,这样控制权就有应用转移到了外部容器,控制权的转移就是控制反转。

依赖注入是指：在程序运行期间，由外部容器动态地将依赖对象注入到组件中如：一般通过构造函数注入或者setter注入。

面试：

IoC思想

DI实现方式

说不清楚用婚介所案例

### 描述下Spring IoC容器初始化过程

Spring IoC容器的初始化可以分为三个过程：

1. 第一个过程是资源定位。这个Resource指的是BeanDefinition的资源定位。这个过程就是容器找到数据的过程。

2. 第二个就是BeanDefination的载入过程。这个载入过程是把用户定义好的Bean表示成IoC容器内部的数据结构，这个容器的内部数据结构就是BeanDefinition。

3. 第三个过程就是向IoC容器注入这些BeanDefinition的过程，这个过程就是将前面的BeanDefinition保存到HashMap中的过程。

   反射

### Spring框架中IOC的原理是什么?

Spring ioc容器原理是根据Java的反射机制，获取类的所有信息，再通过xml或者注解配置获取类与类之间的关系，最后根据以上信息构建类与类之间的依赖。

### Spring的依赖注入有哪几种方式

- 构造方法注入
- setter注入
- 基于注解的注入

### BeanFactory和FactoryBean的区别？



### BeanFactory和ApplicationContext的异同

### Spring中Bean的生命周期

Bean 的生命周期概括起来就是 **4 个阶段**：

1. 实例化（Instantiation）
2. 属性赋值（Populate）
3. 初始化（Initialization）
4. 销毁（Destruction）

https://chaycao.github.io/2020/02/15/%E5%A6%82%E4%BD%95%E8%AE%B0%E5%BF%86Spring-Bean%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F/

bean的生命周期图：https://juejin.cn/post/6844903793709023240

### Spring AOP的实现原理

Spring AOP采用的是动态代理，在**运行期间**对业务方法进行增强，所以不会生成新类，对于动态代理技术，Spring AOP提供了对JDK动态代理的支持以及CGLib的支持。

- JDK动态代理只能为接口创建动态代理实例，而不能对类创建动态代理。需要获得被目标类的接口信息（应用Java的反射技术），生成一个实现了代理接口的动态代理类（字节码），再通过反射机制获得动态代理类的构造函数，利用构造函数生成动态代理类的实例对象，在调用具体方法前调用invokeHandler方法来处理。
- CGLib动态代理需要依赖asm包，把被代理对象类的class文件加载进来，修改其字节码生成子类。

### Spring AOP解决了什么问题？

AOP可以解决非核心代码（比如日志）代码重复的问题。

### Spring 是如何管理事务的

### Spring 的不同事务传播行为有哪些，干什么用的？

### Spring 中用到了那些设计模式？

- 工厂模式
- 单例设计
- 代理设计 
- 模板方法
- 观察者模式
- 适配器模式
- 装饰者模式

### Spring到底是怎么循环依赖问题

Spring 为了解决单例的循环依赖问题，使用了 三级缓存 ，递归调用时发现 Bean 还在创建中即为循环依赖

检测循环依赖的过程如下：

- A 创建过程中需要 B，于是 **A 将自己放到三级缓里面** ，去实例化 B

- B 实例化的时候发现需要 A，于是 B 先查一级缓存，没有，再查二级缓存，还是没有，再查三级缓存，找到了！

- - **然后把三级缓存里面的这个 A 放到二级缓存里面，并删除三级缓存里面的 A**
  - B 顺利初始化完毕，**将自己放到一级缓存里面**（此时B里面的A依然是创建中状态）

- 然后回来接着创建 A，此时 B 已经创建结束，直接从一级缓存里面拿到 B ，然后完成创建，**并将自己放到一级缓存里面**

- 如此一来便解决了循环依赖的问题

一句话：先让最底层对象完成初始化，通过三级缓存与二级缓存提前曝光创建中的 Bean，让其他 Bean 率先完成初始化。

### Spring事务的传播属性是怎么回事？它会影响什么？Spring的事务隔离级别，实现原理

七个事务传播属性

- **PROPAGATION_REQUIRED** -- 支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。
- **PROPAGATION_SUPPORTS** -- 支持当前事务，如果当前没有事务，就以非事务方式执行。
- **PROPAGATION_MANDATORY** -- 支持当前事务，如果当前没有事务，就抛出异常。
- **PROPAGATION_REQUIRES_NEW** -- 新建事务，如果当前存在事务，把当前事务挂起。
- **PROPAGATION_NOT_SUPPORTED** -- 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- **PROPAGATION_NEVER** -- 以非事务方式执行，如果当前存在事务，则抛出异常。
- **PROPAGATION_NESTED**--如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则进行与PROPAGATION_REQUIRED类似的操作。

五种隔离级别

隔离级别是指若干个并发的事务之间的隔离程度。

- **ISOLATION_DEFAULT**--这是一个PlatfromTransactionManager默认的隔离级别，使用数据库默认的事务隔离级别.另外四个与JDBC的隔离级别相对应；
  **ISOLATION_READ_UNCOMMITTED**--这是事务最低的隔离级别，它充许别外一个事务可以看到这个事务未提交的数据。这种隔离级别会产生脏读，不可重复读和幻像读。

- **ISOLATION_READ_COMMITTED**--保证一个事务修改的数据提交后才能被另外一个事务读取。另外一个事务不能读取该事务未提交的数据。这种事务隔离级别可以避免脏读出现，但是可能会出现不可重复读和幻像读。

- **ISOLATION_REPEATABLE_READ**--这种事务隔离级别可以防止脏读，不可重复读。但是可能出现幻像读。它除了保证一个事务不能读取另一个事务未提交的数据外，还保证了避免下面的情况产生(不可重复读)。

- **ISOLATION_SERIALIZABLE**--这是花费最高代价但是最可靠的事务隔离级别。事务被处理为顺序执行。除了防止脏读，不可重复读外，还避免了幻像读。

### Spring 如何实现数据库事务?

事务控制的核心——Connection

用AOP技术保持当前的Connection

采用的是ThreadLocal本地线程变量，Service层的Connection存起来

动态代理的实现方式（必考）

- jdk动态代
- cglib借助asm

### 谈谈对SpringAOP Weaving（织入）的理解？

### Spring的IOC/AOP的实现（必考）

### Spring如何解决循环依赖（三级缓存）（必考）

### Spring的@Transactional如何实现的（必考）

### 讲解Spring 框架中如何基于 AOP 实现的事务管理？

### Spring的事务是如何回滚的

### @Bean这个注解时如何实现Bean的注入的？

## 2 SpringMVC

### MVC框架原理，他们都是怎么做url路由的

## 3 SpringBoot

### Springboot启动过程中做了哪些事情？

### Springboot 启动类上的注解 @Spring boot Application说明？

### Springboot如何判断当前应用是否时web应用？

### Spring boot整合jsp的流程，需要注意哪些点

## 4 SpringCloud

SpringCloud和dubbo的区别

项目中用到了哪些组件

eureka的原理，如何保证高可用性，和Zookeeper有什么区别

feign如何调用的

处理生产环境上配置生效问题

hystrix的降级策略有哪些

Springcloud eureka是如何注册服务、如何监测心跳的，它注册的流程是怎么样的

在分布式环境中如何快速发现某一台服务有问题

分布式集群系统对外提供接口的时候如何验证对方的身份

Eureka和zookeeper作为注册中心有什么区别