# Dubbo

## 1 Dubbo介绍

Apache dubbo是一高性能、轻量级的开源Java框架，它提供了三大核心框架：

- 面向接口的远程方法调用
- 智能容错和负载均衡
- 服务自动注册和发现

·Dubbo是阿里巴巴公司开源的一个高性能优秀的[服务框架](https://baike.baidu.com/item/服务框架)，使得应用可通过高性能的 RPC 实现服务的输出和输入功能，可以和[Spring](https://baike.baidu.com/item/Spring)框架无缝集成。Dubbo框架，是基于容器运行的.。容器是Spring。

## 2 Dubbo框架结构

## 3 角色

### 3.1 registry

注册中心。是用于发布和订阅服务的一个平台。用于替代SOA结构体系框架中的ESB服务总线的

#### 3.1.1 发布

开发服务端代码完毕后，将信息发布出去，实现一个服务的公开。

#### 3.1.2 订阅

客户端程序，从注册中心下载服务内容，这个过程是订阅

订阅服务的时候，会将发布的服务所有信息 ，一次性下载到客户端。

客户端也可以自定义，修改部分服务配置信息，如：超时的时长、调用的重试次数等。

### 3.2 consumer

服务的消费者，就是服务的客户端

消费者必须使用Dubbo技术开发部分代码，基本上都是配置文件定义

### 3.3 provider

服务的提供者，就是服务端

服务端必须使用Dubbo技术开发部分代码.，以配置文件为主。

### 3.4 container

容器

 Dubbo技术的服务端（Provider），在启动执行的时候, 必须依赖容器才能正常启动。

默认依赖的就是spring容器，且Dubbo技术不能脱离spring框架。

在2.5.3版本的dubbo中，默认依赖的是spring2.5版本技术，可以选用spring4.5以下版本。

在2.5.7版本的dubbo中，默认依赖的是spring4.3.10版本技术，可以选择任意的spring版本。

### 3.5 monitor dubbo admin

监控中心.。是Dubbo提供的一个jar工程。

主要功能是监控服务端（Provider）和消费端（Consumer）的使用数据的，如: 服务端是什么、有多少接口、多少方法、调用次数、压力信息等。客户端有多少，调用过哪些服务端，调用了多少次等。

 ## 4 执行流程

- start：启动Spring容器时，自动启动Dubbo的Provider
- register：Dubbo 的 Provider 在启动后自动会去注册中心注册内容，注册的内容包括：
  - Provider 的 IP
  - Provider 的端口
  - Provider 对外提供的接口列表.哪些方法.哪些接口类



## 5 SpringBoot+Dubbo案例

### 5.1 服务提供方 Provider

整体目录结构

--dubbo-provider

  --com

    --yeyangshu
    
      --service
    
        --**IDemoService**
    
        --**DemoServiceImpl**

#### 5.1.1 pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.7.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.yeyangshu</groupId>
	<artifactId>dubbo-provider</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>com.yeyangshu</name>
	<description>dubbo demo provider</description>

	<properties>
		<java.version>1.8</java.version>
		<dubbo.version>2.7.7</dubbo.version>
		<spring-boot.version>2.3.0.RELEASE</spring-boot.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
		</dependency>

		<!--Dubbo-->
		<dependency>
			<groupId>org.apache.dubbo</groupId>
			<artifactId>dubbo-spring-boot-starter</artifactId>
			<version>${dubbo.version}</version>
		</dependency>

		<dependency>
			<groupId>org.apache.dubbo</groupId>
			<artifactId>dubbo</artifactId>
			<version>${dubbo.version}</version>
		</dependency>

		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-framework</artifactId>
			<version>4.2.0</version>
		</dependency>

		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-recipes</artifactId>
			<version>4.2.0</version>
			<exclusions>
				<exclusion>
					<groupId>org.apache.zookeeper</groupId>
					<artifactId>zookeeper</artifactId>
				</exclusion>
			</exclusions>
		</dependency>

		<dependency>
			<groupId>org.apache.zookeeper</groupId>
			<artifactId>zookeeper</artifactId>
			<version>3.4.14</version>
		</dependency>

	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```

#### 5.1.2 配置文件

```properties
#服务端口号
server.port=8081
spring.application.name=DemoProvider

dubbo.scan.base-packages=com.yeyangshu.service //包路径要与consumer一致
dubbo.protocol.name=dubbo
dubbo.protocol.port=666
dubbo.protocol.host=127.0.0.1
#dubbo注册中心
dubbo.registry.address=zookeeper://127.0.0.1:2181
```

#### 5.1.3 服务接口

IDemoService.java

```java
package com.yeyangshu.service;

public interface IDemoService {
    public String say(String name);
}
```

#### 5.1.4 服务实现接口

DemoServiceImpl.java

```java
package com.yeyangshu.service;

import org.apache.dubbo.config.annotation.Service;

@Service(version = "1.0.0", timeout = 10000,interfaceClass = IDemoService.class)
public class DemoServiceImpl implements IDemoService {
    @Override
    public String say(String name) {
        System.out.println("hi! " + name);
        return "h1! " + name;
    }
}
```

### 5.2 服务消费方 Consumer

创建SpringBoot-web项目

整体目录结构

--dubbo-consumer

  --com

    --yeyangshu
    
      --controller
    
        --**MainController**
    
      --service
    
        --**IDemoService**

#### 5.1.1 pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.7.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.yeyangshu</groupId>
	<artifactId>dubbo-consumer</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>dubbo-consumer</name>
	<description>dubbo demo consumer</description>

	<properties>
		<java.version>1.8</java.version>
		<dubbo.version>2.7.7</dubbo.version>
		<spring-boot.version>2.3.0.RELEASE</spring-boot.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<!--Dubbo-->
		<dependency>
			<groupId>org.apache.dubbo</groupId>
			<artifactId>dubbo-spring-boot-starter</artifactId>
			<version>${dubbo.version}</version>
		</dependency>

		<dependency>
			<groupId>org.apache.dubbo</groupId>
			<artifactId>dubbo</artifactId>
			<version>${dubbo.version}</version>
		</dependency>

		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-framework</artifactId>
			<version>4.2.0</version>
		</dependency>

		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-recipes</artifactId>
			<version>4.2.0</version>
			<exclusions>
				<exclusion>
					<groupId>org.apache.zookeeper</groupId>
					<artifactId>zookeeper</artifactId>
				</exclusion>
			</exclusions>
		</dependency>

		<dependency>
			<groupId>org.apache.zookeeper</groupId>
			<artifactId>zookeeper</artifactId>
			<version>3.4.14</version>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

	<repositories>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
		</repository>
		<repository>
			<id>spring-snapshots</id>
			<name>Spring Snapshots</name>
			<url>https://repo.spring.io/snapshot</url>
			<snapshots>
				<enabled>true</enabled>
			</snapshots>
		</repository>
	</repositories>
	<pluginRepositories>
		<pluginRepository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
		</pluginRepository>
		<pluginRepository>
			<id>spring-snapshots</id>
			<name>Spring Snapshots</name>
			<url>https://repo.spring.io/snapshot</url>
			<snapshots>
				<enabled>true</enabled>
			</snapshots>
		</pluginRepository>
	</pluginRepositories>

</project>
```

#### 5.1.2 配置文件

```properties
server.port=8082
spring.application.name=DemoConsumer
dubbo.scan.base-packages=com.yeyangshu.service
#dubbo注册中心
dubbo.registry.address=zookeeper://127.0.0.1:2181
```

#### 5.1.3 服务接口

IDemoService.java

```java
package com.yeyangshu.service;

public interface IDemoService {
    public String say(String name);
}
```

#### 5.1.4 Controller

MainController.java

重点：

```java
@Reference(version = "1.0.0")
IDemoService iDemoService;
```



```java
package com.yeyangshu.controller;

import com.yeyangshu.service.IDemoService;
import org.apache.dubbo.config.annotation.Reference;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/")
public class MainController {

    @Reference(version = "1.0.0")
    IDemoService iDemoService;

    @RequestMapping("say")
    public String say() {
        return iDemoService.say("Hello World?");
    }
}
```

### 5.3 服务调用

首先启动Zookeeper，再启动服务提供者，在启动服务消费者

打开127.0.0.1:8082/say





