# 代理模式

代理模式（Proxy Pattern）也叫委托模式，是一个使用率非常高的模式。

## 1 代理模式的定义

代理模式的英文原话是：

> Provide a surrogate or placeholder for another object to control access toit.

意思是：为其他对象提供一种代理以控制对这个对象的访问。

代理模式提供以下3个角色：

- 抽象主题（Subject）角色：该角色是`RealSubject`和`Proxy`的**共用接口**，以便在任何可以使用`RealSubject`的地方都可以使用`Proxy`。

- 代理主题（Proxy Subject）角色：也叫做委托类、代理类，该角色负责控制对真实主题的`Proxy`，负责在需要的时候创建或删除真实主题对象，并且在真实主题角色处理完毕前后做预处理和善后处理工作。

- 真实主题（Real Subject）角色：该角色也叫做被委托角色、被代理角色，是业务逻辑的具体执行者。

抽象主题Subject的代码如下所示：

```java
/**
 * 抽象主题
 * 代理对象和被代理对象共同实现的接口
 */
public interface Subject {

    /**
     * 定义一个请求方法
     * 就是被代理时执行的方法
     */
    public void request();
}
```

真实主题RealSubject的代码如下所示：

```java
public class RealSubject implements Subject {
    @Override
    public void request() {
        // 具体逻辑处理
        System.out.println("=========================");
    }
}
```

代理主题ProxySubject的代码如下所示：

```java
/**
 * 代理对象
 */
public class ProxySubject implements Subject {

    /** 使用聚合保存一个引用  */
    private Subject subject;

    /**
     * 构造方法
     *
     * @param subject 被代理对象
     */
    public ProxySubject(Subject subject) {
        this.subject = subject;
    }

    /**
     * 控制实体
     */
    @Override
    public void request() {
        this.beforeRequest();
        subject.request();
        this.afterRequest();
    }

    /**
     * 请求前的操作
     */
    private void beforeRequest() {
        // 预处理
        System.out.println("before request");
    }

    /**
     * 请求后的操作
     */
    private void afterRequest() {
        // 善后处理
        System.out.println("after request");
    }
}
```

客户端代码：

```java
public class Client {
    public static void main(String[] args) {
        RealSubject realSubject = new RealSubject();
        ProxySubject proxy = new ProxySubject(realSubject);
        proxy.request();
    }

    /**
     * before request
     * =========================
     * after request
     */
}
```

### 1.1 静态代理

### 1.2 动态代理

代码里面添加

```java
System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
```

可以打印代理类

## 2 代理模式案例