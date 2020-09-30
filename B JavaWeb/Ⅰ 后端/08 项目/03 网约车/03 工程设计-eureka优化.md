# 优化

## 1 生产优化

### 1.1 server端优化

```yaml
# 自我保护
enable-replicated-request-compression: false
# 自我保护阈值
renewal-percent-threshold: 0.85
# 剔除失效服务间隔，ms
eviction-interval-timer-in-ms: 1000
# 关闭从 readOnly 读取注册表
use-read-only-response-cache: false
# rewrite 和 readonly 同步时间间隔，ms
response-cache-update-interval-ms: 1000
```

**生产中的问题：**

1. 优化目的：减少服务上下线的延时
2. 自我保护的选择：看网络和服务情况
3. 服务更新：停止，再发送先下线请求

#### 1.1 自我保护

1. 开关

2. 阈值

10个微服务 - 3 = 7 70%

100个 -7 = 93 93%

阈值设置为80%，触发保护

AbstractInstanceRegistry.evict：维护服务注册表

```java
//
int registrySize = (int) getLocalRegistrySize();
// getRenewalPercentThreshold默认0.85
int registrySizeThreshold = (int) (registrySize * serverConfig.getRenewalPercentThreshold());
int evictionLimit = registrySize - registrySizeThreshold;

int toEvict = Math.min(expiredLeases.size(), evictionLimit);
```

#### 1.2 cap

##### 1.2.1 三级缓存 

eureka为什么是ap

优先保证可用性 (A) 和分区容错性P，不保证强一致性C，只保证最终一致性，因此在架构中设计了较多缓存。

##### 1.2.2 从其他peer拉取注册表

`registry.syncUp()`没有满足 C 的地方

##### 1.2.3 p，分区可用性

网络不好的情况下，由于自我保护机制，还是可以拉取到注册表的信息进行调用的

### 1.2 client端优化

```
eureka:
  instance:
    # 心跳续约间隔
    lease-renewal-interval-in-seconds: 10
    # 缺心跳过期时间
    lease-expiration-duration-in-seconds: 10
  client:
    # 刷新注册表（拉取注册表）间隔
    registry-fetch-interval-seconds: 5
```

可以设置饥饿加载，防止第一次请求超时

## 2 Eureka源码

### 2.1 注册

集群同步：第一个节点注册进来，null（第二次true），true.equals后面的那个值是false（二次true）

### 2.2 拉取

### 2.3 剔除

集群同步：没有这项业务

剔除，长时间没有心跳服务，eureka server将它从注册表剔除

### 2.4 续约

集群同步：一直同步，所有集群

renewLease

### 2.5 下线

集群同步：一直同步，所有集群

![image-20200919140800873](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200919140800873.png)

### 2.6 集群同步

 

### 2.7 增量拉取

浏览器访问`http://euk.com:7900/eureka/apps/delta`

debug `ApplicationsResource.getContainerDifferential`



全量拉取

``ApplicationsResource.getContainers`



## 3 服务测算

网约车共有18个服务，假设20个，每个服务部署五个 eureka client，一共100个

注册，30s一次，一分钟2次，共200次

心跳，一分钟两次

```
200*24*60*60=
```

设计访问量：几十万次，对server而言

## 4 生产中的问题

### 4.1 服务重启时，先停服，再手动触发下线

如果不停服手动触发下线的话，就会重新续约

面试：debug源码发现重新续约

### 4.2 打乱集群url

serviceUrl：实际工作中要把集群中的url随机打乱，因为拉取的时候去第一个，第一个挂掉之后才去第二个

## 4 其他学习

### 1 Eureka源码学习

```java
AbstractInstanceRegistry

Lease<InstanceInfo> existingLease = gMap.get(registrant.getId());
```

Lease类，将InstanceInfo类加在 Lease 里，频繁的续约操作不影响服务实例

```java
public class Lease<T> {
    
    private T holder;
    private long evictionTimestamp;
    private long registrationTimestamp;
    private long serviceUpTimestamp;
    private volatile long lastUpdateTimestamp;
    private long duration;
}
```

如果是我们自己设计，对比一下

```java
收到服务实例，保存。心跳时间，xxx时间。
服务实例，有 xxxx时间。
class 租约{
	long 到期time;
	long 续约时间time;
	long 心跳时间time;
	T 服务实例 holder。setter
}
```

### 2 client 注册服务 debug

debug 启动 server，打断点，postman发起post请求： `127.0.0.1:7900/eureka/apps/MyApp01`

```xml
<instance> 
  <instanceId>MyApp01</instanceId>  
  <hostName>localhost</hostName>  
  <app>MyApp01</app>  
  <ipAddr>127.0.0.1</ipAddr>  
  <status>UP</status>  
  <port enabled="true">8085</port>  
  <dataCenterInfo class="com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo"> 
    <name>MyOwn</name> 
  </dataCenterInfo> 
</instance>
```

### 3 client renew服务 debug

启动server，InstanceResource.renewLease 打断点，postman发起post请求： `http://127.0.0.1:7900/eureka/apps/my-service/MyApp01`

```xml
<instance> 
  <instanceId>MyApp01</instanceId>  
  <hostName>localhost</hostName>  
  <app>MyApp01</app>  
  <ipAddr>127.0.0.1</ipAddr>  
  <status>UP</status>  
  <port enabled="true">7900</port>  
  <dataCenterInfo class="com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo"> 
    <name>MyOwn</name> 
  </dataCenterInfo> 
</instance>
```

# 

### 地区、可用区

减少网络延迟

server

```yaml
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
    region: bj
    availability-zones:
      bj: z1, z2
    sevice-url:
      z1: http://localhost:7911/eureka/,http://localhost:7912/eureka/
      z2: http://localhost:7921/eureka/,http://localhost:7922/eureka/
```

provider

```yaml
eureka:
  client:
    region: bj
    availability-zones:
      bj: z2
    perfer-same-zone-eureka: true
```

eureka server

操作：单体，高可用，分区域

源码：6个，注册、心跳、下线、剔除、拉取注册表、集群同步