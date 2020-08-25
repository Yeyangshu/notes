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

























