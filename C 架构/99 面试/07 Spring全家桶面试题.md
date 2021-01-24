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

控制反转是就是应用本身不负责依赖对象的创建和维护，依赖对象的创建及维护是由外部容器负责的,这样控制权就有应用转移到了外部容器,控制权的转移就是控制反转。

依赖注入是指：在程序运行期间，由外部容器动态地将依赖对象注入到组件中如：一般通过构造函数注入或者setter注入。

面试：

IoC思想

DI实现方式

说不清楚用婚介所案例

### Spring IoC容器初始化过程

Spring IoC容器的初始化可以分为三个过程：

1. 第一个过程是资源定位。这个Resource指的是BeanDefinition的资源定位。这个过程就是容器找到数据的过程。

2. 第二个就是BeanDefination的载入过程。这个载入过程是把用户定义好的Bean表示成IoC容器内部的数据结构，这个容器的内部数据结构就是BeanDefinition。

3. 第三个过程就是向IoC容器注入这些BeanDefinition的过程，这个过程就是将前面的BeanDefinition保存到HashMap中的过程。

   反射

### Spring的IOC容器实现原理，为什么可以通过byName和byType找到Bean



### Spring框架中IOC的原理是什么?

实例化策略 源码112页

- 判断如果beanDefinition.getMethodOverrides()为空也就是用户没有使用replace或者lookup的配置方法，直接使用反射的方式
- 如果使用了这两个特性，因为需要将这两个配置提供的功能增强进去，所以就必须使用动态代理的方式将包含这两个特性所对应的逻辑的拦截增强器设置进去，这样才可以保证在调用方法的时候会被相应的拦截器增强、返回值为包含拦截器的代理实例。

Spring ioc容器原理是根据Java的反射机制，获取类的所有信息，再通过xml或者注解配置获取类与类之间的关系，最后根据以上信息构建类与类之间的依赖。

### Spring的依赖注入有哪几种方式

- 构造方法注入
- setter注入
- 基于注解的注入

### BeanFactory和FactoryBean的区别？

### Spring如何解决循环依赖（三级缓存）（必考）

Spring循环依赖包括构造器循环依赖和setter循环依赖。

#### 构造器循环依赖

Spring容器将每一个正在创建的bean标识符放在一个“当前创建池”中，bean标识符在创建过程中将一直保持在这个池中，如果在创建bean过程中发现自己已经在“当前创建池”里时，将抛出`BeanCurrerntlyCreationException`异常表示循环依赖；而对于创建完成的bean将从“当前创建池”中清除掉。

#### setter循环依赖

通过setter注入方式构成的循环依赖是通过Spring容器提前暴露刚完成构造器注入但未完成其他步骤的bean来完成的，而且只能解决单例作用域的bean循环依赖。通过提前暴露一个单例工厂方法，从而使其他bean能引用到该bean，如下代码所示：

```java
addSingletonFactory(beanName, 
                    new ObjectFactory() {
                        public Object getObject() throws BeansException {
                            return getEarlyBeanRefrence(beanName, mbd, bean);
                        });

```

#### prototype范围的依赖处理

对于prototype作用域的bean，Spring容器无法完成依赖注入，因为Spring容器不允许缓存prototype作用域的bean，因此无法提前暴露一个创建中的bean。



Spring 为了解决单例的循环依赖问题，使用了三级缓存 ，递归调用时发现 Bean 还在创建中即为循环依赖

检测循环依赖的过程如下：

Spring内部维护了三个Map

- singletonObjects：它是我们最熟悉的朋友，俗称“单例池”“容器”，缓存创建完成单例Bean的地方。
- earlySingletonObjects：映射Bean的早期引用，也就是说在这个Map里的Bean不是完整的，甚至还不能称之为“Bean”，只是一个Instance。
- singletonFactories：映射创建Bean的原始工厂

首先尝试从singletonObjects里面获取实例，如果获取不到再从earlySingletonObjects里面获取，如果还是获取不到，在尝试从singletonFactories里面获取beanName对应的ObjectFactory，然后调用这个ObjectFactory的getObject来创建bean，并放到earlySingletonObjects里面去，并且从singletonFactories里面remove掉这个ObjectFactory。

一句话：先让最底层对象完成初始化，通过三级缓存与二级缓存提前曝光创建中的 Bean，让其他 Bean 率先完成初始化。

BeanFactory是个Factory，也就是IOC容器或对象工厂，FactoryBean是个Bean。在Spring中，所有的Bean都是由BeanFactory(也就是IOC容器)来进行管理的。但对FactoryBean而言，这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean，它的实现与设计模式中的工厂模式和修饰器模式类似

- BeanFactory定义了IOC容器的最基本形式，并提供了IOC容器应遵守的的最基本的接口，也就是Spring IOC所遵守的最底层和最基本的编程规范。

  ```java
  package org.springframework.beans.factory;  
  import org.springframework.beans.BeansException;  
  public interface BeanFactory {  
      String FACTORY_BEAN_PREFIX = "&";  
      Object getBean(String name) throws BeansException;  
      <T> T getBean(String name, Class<T> requiredType) throws BeansException;  
      <T> T getBean(Class<T> requiredType) throws BeansException;  
      Object getBean(String name, Object... args) throws BeansException;  
      boolean containsBean(String name);  
      boolean isSingleton(String name) throws NoSuchBeanDefinitionException;  
      boolean isPrototype(String name) throws NoSuchBeanDefinitionException;  
      boolean isTypeMatch(String name, Class<?> targetType) throws NoSuchBeanDefinitionException;  
      Class<?> getType(String name) throws NoSuchBeanDefinitionException;  
      String[] getAliases(String name);  
  }   
  ```

- 一般情况下，Spring通过反射机制利用`<bean>`的class属性指定实现类实例化Bean，在某些情况下，实例化Bean过程比较复杂，如果按照传统的方式，则需要在`<bean>`中提供大量的配置信息。配置方式的灵活性是受限的，这时采用编码的方式可能会得到一个简单的方案。Spring为此提供了一个org.springframework.bean.factory.FactoryBean的工厂类接口，用户可以通过实现该接口定制实例化Bean的逻辑。

  ```java
  package org.springframework.beans.factory;  
  public interface FactoryBean<T> {  
      T getObject() throws Exception;  
      Class<?> getObjectType();  
      boolean isSingleton();  
  }
  ```

当使用BeanFactory的时候必须要遵循完整的创建过程，这个过程是由spring来管理控制的

而使用FactoryBean只需要调用getObject就可以返回具体的对象，整个对象的创建过程是由用户自己来控制的，更加灵活

### BeanFactory和ApplicationContext的异同

BeanFactory和ApplicationContext两者都是用于加载bean，相比之下，ApplicationContext提供了更多的扩展功能。

### Spring中Bean的生命周期

Bean 的生命周期概括起来就是 **4 个阶段**：

1. 实例化（Instantiation）
2. 属性赋值（Populate）
3. 初始化（Initialization）
4. 销毁（Destruction）

https://chaycao.github.io/2020/02/15/%E5%A6%82%E4%BD%95%E8%AE%B0%E5%BF%86Spring-Bean%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F/

![image-20210109235408148](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210109235408148.png)

### Spring AOP解决了什么问题？

AOP可以解决非核心代码（比如日志）代码重复的问题。****

### Spring AOP的实现原理

源码191页

Spring AOP采用的是动态代理，在**运行期间**对业务方法进行增强，所以不会生成新类，对于动态代理技术，Spring AOP提供了对JDK动态代理的支持以及CGLib的支持。

- JDK动态代理只能为接口创建动态代理实例，而不能对类创建动态代理。需要获得被目标类的接口信息（应用Java的反射技术），生成一个实现了代理接口的动态代理类（字节码），再通过反射机制获得动态代理类的构造函数，利用构造函数生成动态代理类的实例对象，在调用具体方法前调用invokeHandler方法来处理。
- CGLib动态代理需要依赖asm包，把被代理对象类的class文件加载进来，修改其字节码生成子类。

### Spring 是如何管理事务的

![image-20210110215032266](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210110215032266.png)

由不同的传播特性来决定不同的方法的事务应该如何获取

### Spring 的不同事务传播行为有哪些，干什么用的？

### Spring 中用到了那些设计模式？

- 工厂模式
- 单例设计
- 代理设计 ：getObject->getBean
- 模板方法
- 观察者模式
- 适配器模式
- 装饰者模式
- 策略模式：初始化bean的时候可以使用反射，也可是使用cglib，spring源码112页



### Spring事务的传播属性？它会影响什么？Spring的事务隔离级别，实现原理

七个事务传播属性，分为两大类

保证同一个事务中，支持外层事务

- PROPAGATION_REQUIRED：支持当前事务，如果不存在就新建一个（默认）
- PROPAGATION_SUPPORTS：支持当前事务，如果不存在就不使用事务，不使用事务执行
- PROPAGATION_MANDATORY：支持当前事务，如果没有当前事务，就抛出异常

保证没有在同一个事务中，不支持外层事务

- PROPAGATION_REQUIRES_NEW：新建事务，如果当前存在事务，就把当前事务挂起
- PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。
- PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则进行与PROPAGATION_REQUIRED类似的操作。

五种隔离级别

隔离级别是指若干个并发的事务之间的隔离程度。

- **ISOLATION_DEFAULT**--这是一个PlatfromTransactionManager默认的隔离级别，使用数据库默认的事务隔离级别.另外四个与JDBC的隔离级别相对应；
- **ISOLATION_READ_UNCOMMITTED**--这是事务最低的隔离级别，它充许别外一个事务可以看到这个事务未提交的数据。这种隔离级别会产生脏读，不可重复读和幻像读。
- **ISOLATION_READ_COMMITTED**--保证一个事务修改的数据提交后才能被另外一个事务读取。另外一个事务不能读取该事务未提交的数据。这种事务隔离级别可以避免脏读出现，但是可能会出现不可重复读和幻像读。
- **ISOLATION_REPEATABLE_READ**--这种事务隔离级别可以防止脏读，不可重复读。但是可能出现幻像读。它除了保证一个事务不能读取另一个事务未提交的数据外，还保证了避免下面的情况产生(不可重复读)。
- **ISOLATION_SERIALIZABLE**--这是花费最高代价但是最可靠的事务隔离级别。事务被处理为顺序执行。除了防止脏读，不可重复读外，还避免了幻像读。

### Spring 如何实现数据库事务?

事务控制的核心——Connection

用AOP技术保持当前的Connection

采用的是ThreadLocal本地线程变量，Service层的Connection存起来

### Spring的@Transactional如何实现的（必考）

### 讲解Spring 框架中如何基于 AOP 实现的事务管理？

### Spring的事务是如何回滚的

## 2 SpringMVC

### MVC框架原理，他们都是怎么做url路由的

## 3 SpringBoot

### Springboot启动过程中做了哪些事情？

### Springboot 启动类上的注解 @Spring boot Application说明？

### Springboot如何判断当前应用是否时web应用？

### Spring boot整合jsp的流程，需要注意哪些点

## 4 SpringCloud

### SpringCloud和dubbo的区别

### 项目中用到了哪些组件

### Eureka的原理，如何保证高可用性？

集群同步

![image-20210123102753019](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210123102753019.png)



![image-20210123102830117](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210123102830117.png)

### Springcloud eureka是如何注册服务、如何监测心跳的，它注册的流程是怎么样的？

#### 服务注册

![image-20210123103437130](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210123103437130.png)

#### 心跳机制

在应用启动后，节点们将会向Eureka Server定时发送心跳，默认周期为30秒，如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，Eureka Server将会从服务注册表中把这个服务节点移除(默认90秒)。

![image-20210123103746791](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210123103746791.png)

### Eureka和Zookeeper作为注册中心有什么区别

#### Zookeeper保证CP

当向注册中心查询服务列表时，我们可以容忍注册中心返回的是几分钟以前的注册信息，但不能接受服务直接down掉不可用。也就是说，**服务注册功能对可用性的要求要高于一致性**。但是zk会出现这样一种情况，当master节点因为网络故障与其他节点失去联系时，剩余节点会重新进行leader选举。问题在于，选举leader的时间太长，30 ~ 120s，且选举期间整个zk集群都是不可用的，这就导致在选举期间注册服务瘫痪。在云部署的环境下，因网络问题使得zk集群失去master节点是较大概率会发生的事，虽然服务能够最终恢复，但是漫长的选举时间导致的注册长期不可用是不能容忍的。

#### Eureka保证AP

Eureka看明白了这一点，因此在设计时就优先保证可用性。Eureka各个节点都是平等的，几个节点挂掉不会影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。而Eureka的客户端在向某个Eureka注册或如果发现连接失败，则会自动切换至其它节点，只要有一台Eureka还在，就能保证注册服务可用(保证可用性)，只不过查到的信息可能不是最新的(不保证强一致性)。除此之外，Eureka还有一种自我保护机制，如果在15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时会出现以下几种情况：

- Eureka不再从注册列表中移除因为长时间没收到心跳而应该过期的服务
- Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上(即保证当前节点依然可用)
- 当网络稳定时，当前实例新的注册信息会被同步到其它节点中

因此， Eureka可以很好的应对因网络故障导致部分节点失去联系的情况，而不会像zookeeper那样使整个注册服务瘫痪。



feign如何调用的

处理生产环境上配置生效问题

### Hystrix的降级策略有哪些



在分布式环境中如何快速发现某一台服务有问题

分布式集群系统对外提供接口的时候如何验证对方的身份

## 