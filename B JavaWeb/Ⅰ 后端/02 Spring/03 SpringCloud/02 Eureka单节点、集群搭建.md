# 服务注册与发现 Eureka

## 1 Eureka介绍

## 2 Eureka单节点搭建

1. pom.xml

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
   </dependency>
   ```

2. application.yml

   ```yaml
   eureka:
     client:
     	// 是否将自己注册到Eureka Server，默认为true，由于自己就是server，所以设置为false，表明该服务不会向Eureka Server注册自己的信息
       register-with-eureka: false
       // 是否从Eureka Server获取注册信息，由于单节点不需要同步其他节点数据，设置为false
       fetch-registry: false
       // 设置服务注册中心的URL，用于client和server端交流
       server-url:
         defaultZone: http://localhost:7900/eureka/
   ```

3. 启动类

   启动类添加注解@EnableEurekaServer

   ```java
   @SpringBootApplication
   @EnableEurekaServer
   public class EurekaServerApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(EurekaServerApplication.class, args);
       }
   
   }
   ```

4. 启动运行，访问本地8080端口，可以看见控制台

## 3 高可用

### 3.1 搭建步骤

#### 3.1.1 准备

准备2个节点部署eureka，也可以单机部署

修改本机host文件，绑定一个主机名，单机部署时使用ip地址会有问题

#### 3.1.2 配置文件

**节点 1:**

```
#是否将自己注册到其他Eureka Server,默认为true 需要
eureka.client.register-with-eureka=true
#是否从eureka server获取注册信息， 需要
eureka.client.fetch-registry=true
#设置服务注册中心的URL，用于client和server端交流
#此节点应向其他节点发起请求
eureka.client.serviceUrl.defaultZone=http://ek2.com:7902/eureka/
#主机名，必填
eureka.instance.hostname=ek1.com
management.endpoint.shutdown.enabled=true
#web端口，服务是由这个端口处理rest请求的
server.port=7901
```

**节点 2:**

```
#是否将自己注册到其他Eureka Server,默认为true 需要
eureka.client.register-with-eureka=true
#是否从eureka server获取注册信息， 需要
eureka.client.fetch-registry=true
#设置服务注册中心的URL，用于client和server端交流
#此节点应向其他节点发起请求
eureka.client.serviceUrl.defaultZone=http://ek1.com:7902/eureka/
#主机名，必填
eureka.instance.hostname=ek2.com
management.endpoint.shutdown.enabled=true
#web端口，服务是由这个端口处理rest请求的
server.port=7902
```

**节点 1:**

如果有节点3，配置同上 改一下主机名和端口

略。。。



两个节点的话，如下图内容 就算成功了

![image-20200403193147121](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200403193147121.png)



