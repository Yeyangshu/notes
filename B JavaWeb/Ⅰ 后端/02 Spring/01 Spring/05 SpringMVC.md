# SpringMVC

## 1 什么是MVC？

MVC是模型(Model)、视图(View)、控制器(Controller)的简写，是一种软件设计规范。就是将业务逻辑、数据、显示分离的方法来组织代码。MVC主要作用是**降低了视图与业务逻辑间的双向偶合**。MVC不是一种设计模式，**MVC是一种架构模式**。当然不同的MVC存在差异。

**Model（模型）：**数据模型，提供要展示的数据，因此包含数据和行为，可以认为是领域模型或JavaBean组件（包含数据和行为），不过现在一般都分离开来：Value Object（数据Dao） 和 服务层（行为Service）。也就是模型提供了模型数据查询和模型数据的状态更新等功能，包括数据和业务。

**View（视图）：**负责进行模型的展示，一般就是我们见到的用户界面，客户想看到的东西。

**Controller（控制器）：**接收用户请求，委托给模型进行处理（状态改变），处理完毕后把返回的模型数据返回给视图，由视图负责展示。 也就是说控制器做了个调度员的工作。



最典型的MVC就是JSP + servlet + javabean的模式

![image-20201119224353715](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201119224353715.png)

## 2 SpringMVC

### 2.1 SpringMVC介绍

```
Spring Web MVC is the original web framework built on the Servlet API and has been included in the Spring Framework from the very beginning. The formal name, “Spring Web MVC,” comes from the name of its source module (spring-webmvc), but it is more commonly known as “Spring MVC”.
Spring Web MVC是构建在Servlet API上的原始Web框架，从一开始就包含在Spring Framework中。 正式名称 “Spring Web MVC,” 来自其源模块(spring-webmvc)的名称，但它通常被称为“Spring MVC”。
```

简而言之，SpringMVC是Spring框架的一部分，是基于java实现的一个轻量级web框架。

学习SpringMVC框架最核心的就是DispatcherServlet的设计，掌握好DispatcherServlet是掌握SpringMVC的核心关键。

### 2.2 SpringMVC的优点

1. 清晰的角色划分

   控制器(controller)、验证器(validator)、命令对象(command obect)、表单对象(form object)、模型对象(model object)、Servlet分发器(DispatcherServlet)、处理器映射(handler mapping)、视图解析器(view resoler)等等。每一个角色都可以由一个专门的对象来实现。

2. 强大而直接的配置方式

   将框架类和应用程序类都能作为JavaBean配置，支持跨多个context的引用，例如，在web控制器中对业务对象和验证器(validator)的引用。

3. 可适配、非侵入

   可以根据不同的应用场景，选择何事的控制器子类(simple型、command型、from型、wizard型、multi-action型或者自定义)，而不是一个单一控制器(比如Action/ActionForm)继承。

4. 可重用的业务代码

   可以使用现有的业务对象作为命令或表单对象，而不需要去扩展某个特定框架的基类。

5. 可定制的绑定(binding)和验证(validation)

   比如将类型不匹配作为应用级的验证错误，这可以保证错误的值。再比如本地化的日期和数字绑定等等。在其他某些框架中，你只能使用字符串表单对象，需要手动解析它并转换到业务对象。

6. 可定制的handler mapping和view resolution

   Spring提供从最简单的URL映射，到复杂的、专用的定制策略。与某些web MVC框架强制开发人员使用单一特定技术相比，Spring显得更加灵活。

7. 灵活的model转换

   在Springweb框架中，使用基于Map的键/值对来达到轻易的与各种视图技术集成。

8. 可定制的本地化和主题(theme)解析

   支持在JSP中可选择地使用Spring标签库、支持JSTL、支持Velocity(不需要额外的中间层)等等。

9. 简单而强大的JSP标签库(Spring Tag Library)

   支持包括诸如数据绑定和主题(theme)之类的许多功能。他提供在标记方面的最大灵活性。

10. JSP表单标签库

    在Spring2.0中引入的表单标签库，使用在JSP编写表单更加容易。

11. Spring Bean的生命周期

    可以被限制在当前的HTTp Request或者HTTp Session。准确的说，这并非Spring MVC框架本身特性，而应归属于Spring MVC使用的WebApplicationContext容器。

### 2.3 SpringMVC的实现原理

SpringMVC的MVC模式，如下图：

![image-20201119225058495](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201119225058495.png)

SpringMVC的具体执行流程：

当发起请求时被前置的控制拦截器拦截到请求，根据请求参数生成代理请求，找到请求对应的实际控制器，控制器处理请求，创建数据模型，访问数据库，将模型响应给中心控制器，控制器使用模型与视图渲染器渲染视图结果，将结果返回给中心控制器，再将结果返回给请求者。

![image-20201119225609816](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201119225609816.png)

1. DispatcherServlet表示前置控制器，是整个SpringMVC的控制中心。用户发出请求，DispatcherServlet接收请求并拦截请求。
2. HandlerMapping为处理器映射。DispatcherServlet调用HandlerMapping，HandlerMapping根据请求url查找Handler。
3. 返回处理器执行链，根据url查找控制器，并且将解析后的信息传递给DispatcherServlet
4. HandlerAdapter表示处理器适配器，其按照特定的规则去执行Handler。
5. 执行handler找到具体的处理器
6. Controller将具体的执行信息返回给HandlerAdapter，如ModelAndView。
7. HandlerAdapter将视图逻辑名或模型传递给DispatcherServlet。
8. DispatcherServlet调用视图解析器(ViewResolver)来解析HandlerAdapter传递的逻辑视图名。
9. 视图解析器将解析的逻辑视图名传给DispatcherServlet。
10. DispatcherServlet根据视图解析器解析的视图结果，调用具体的视图，进行试图渲染。
11. 将响应数据返回给客户端