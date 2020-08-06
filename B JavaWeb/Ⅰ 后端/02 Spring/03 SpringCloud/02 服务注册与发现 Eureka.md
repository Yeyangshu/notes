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

