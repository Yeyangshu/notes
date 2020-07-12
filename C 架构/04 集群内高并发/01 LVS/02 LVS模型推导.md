# LVS模型推导

负载均衡分硬件层和软件层两种。

硬件负载均衡将4-7层负载均衡功能做到一个硬件里面，如F5，梭子鱼

软件负载均衡分为四层和七层，所谓四层就是基于IP+端口的负载均衡；七层就是基于URL等应用层信息的负载均衡

LVS：Linux Virtual Server
![image-20200712112146442](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200712112146.png)

## 1 NAT模型

![image-20200712104840836](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200712104841.png)

## 2 LVS

### 2.1 D-NAT

![image-20200712104955925](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200712104955.png)

### 2.2 DR

![image-20200712105029937](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200712105030.png)

### 2.3 TUN隧道技术

![image-20200712105554603](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200712105554.png)



