## 使用Spring Boot2.x Actuator监控应用

## 1 Actuator

### 1.1 开启监控

pom依赖

   ```xml
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-actuator</artifactId>
 </dependency>
   ```

服务器访问：http://euk1.com:7001/actuator

显示所有的查看项

![image-20200915001347085](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200915001347085.png)

### 1.2 默认端点

Spring Boot 2.0 的 Actuator 只暴露了 health 和 info 端点，提供的监控信息无法满足我们的需求

在1.x中有n多可供我们监控的节点，官方的回答是为了安全….

### 1.3 开启所有端点

在application.yml中加入如下配置信息，*代表所有节点都加载

```properties
#开启所有端点
management.endpoints.web.exposure.include=*
```

所有端点都开启后的api列表

```
{"_links":{"self":{"href":"http://localhost:8080/actuator","templated":false},"archaius":{"href":"http://localhost:8080/actuator/archaius","templated":false},"beans":{"href":"http://localhost:8080/actuator/beans","templated":false},"caches-cache":{"href":"http://localhost:8080/actuator/caches/{cache}","templated":true},"caches":{"href":"http://localhost:8080/actuator/caches","templated":false},"health":{"href":"http://localhost:8080/actuator/health","templated":false},"health-path":{"href":"http://localhost:8080/actuator/health/{*path}","templated":true},"info":{"href":"http://localhost:8080/actuator/info","templated":false},"conditions":{"href":"http://localhost:8080/actuator/conditions","templated":false},"configprops":{"href":"http://localhost:8080/actuator/configprops","templated":false},"env":{"href":"http://localhost:8080/actuator/env","templated":false},"env-toMatch":{"href":"http://localhost:8080/actuator/env/{toMatch}","templated":true},"loggers":{"href":"http://localhost:8080/actuator/loggers","templated":false},"loggers-name":{"href":"http://localhost:8080/actuator/loggers/{name}","templated":true},"heapdump":{"href":"http://localhost:8080/actuator/heapdump","templated":false},"threaddump":{"href":"http://localhost:8080/actuator/threaddump","templated":false},"metrics":{"href":"http://localhost:8080/actuator/metrics","templated":false},"metrics-requiredMetricName":{"href":"http://localhost:8080/actuator/metrics/{requiredMetricName}","templated":true},"scheduledtasks":{"href":"http://localhost:8080/actuator/scheduledtasks","templated":false},"mappings":{"href":"http://localhost:8080/actuator/mappings","templated":false},"refresh":{"href":"http://localhost:8080/actuator/refresh","templated":false},"features":{"href":"http://localhost:8080/actuator/features","templated":false},"service-registry":{"href":"http://localhost:8080/actuator/service-registry","templated":false}}}
```

### 1.4  配置文件属性

![image-20200915001602430](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200915001602430.png)

### 1.5 api端点功能

#### 1.5.1 health

访问地址：http://euk1.com:7001/actuator/health，会显示系统状态

```json
{"status":"UP","components":{"discoveryComposite":{"status":"UP","components":{"discoveryClient":{"status":"UP","details":{"services":["eureka-server","consumer","provider"]}},"eureka":{"description":"Remote status from Eureka server","status":"UP","details":{"applications":{"EUREKA-SERVER":1,"CONSUMER":1,"PROVIDER":1}}}}},"diskSpace":{"status":"UP","details":{"total":106473451520,"free":98129141760,"threshold":10485760,"exists":true}},"hystrix":{"status":"UP"},"ping":{"status":"UP"},"refreshScope":{"status":"UP"}}}
```

图片

![image-20200915002005227](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200915002005227.png)

#### 1.5.2 shutdown 

用来关闭节点，开启远程关闭功能

```properties
management.endpoint.shutdown.enabled=true
```

配置文件添加此配置之后，服务列表会显示`shundown`的url

![image-20200915002316953](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200915002316953.png)

使用Post方式请求端点

```json
{
  "message": "Shutting down, bye..."
}
```

![image-20200915002435621](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200915002435621.png)

服务被关闭

![image-20200915002525415](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200915002525415.png)

#### 1.5.3 autoconfig 

获取应用的自动化配置报告 

#### 1.5.4 beans 

获取应用上下文中创建的所有Bean 

#### 1.5.5 configprops 

获取应用中配置的属性信息报告 

#### 1.5.6 env 

获取应用所有可用的环境属性报告 

#### 1.5.7 Mappings

 获取应用所有Spring Web的控制器映射关系报告

####  1.5.8 info 

获取应用自定义的信息 

#### 1.5.9 metrics

返回应用的各类重要度量指标信息 

**Metrics**节点并没有返回全量信息，我们可以通过不同的**key**去加载我们想要的值

 metrics/jvm.memory.max

#### 1.5.10 Threaddump

1.x 中为**dump**

返回程序运行中的线程信息 

## 2 手动控制服务上下线

由于server和client通过心跳保持 服务状态，而只有状态为UP的服务才能被访问。看eureka界面中的status。

比如心跳一直正常，服务一直UP，但是此服务DB连不上了，无法正常提供服务。

此时，我们需要将 微服务的健康状态也同步到server。只需要启动eureka的健康检查就行。这样微服务就会将自己的健康状态同步到eureka。配置如下即可。

### 2.1 Client 开启手动控制

在client端配置：将自己真正的健康状态传播到server。

```yaml
eureka:
  client:
    healthcheck:
      enabled: true
```

### 2.2 Client 端配置 Actuator

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 2.3 改变健康状态的 Service

```java
@Service
public class HealthStatusService implements HealthIndicator{

	private Boolean status = true;

	public void setStatus(Boolean status) {
		this.status  = status;
	}

	@Override
	public Health health() {
		// TODO Auto-generated method stub
		if(status)
		return new Health.Builder().up().build();
		return new Health.Builder().down().build();
	}

	public String getStatus() {
		// TODO Auto-generated method stub
		return this.status.toString();
	}
```

### 2.2 测试用的Controller

```java
@GetMapping("/health")
public String health(@RequestParam("status") Boolean status) {

    healthStatusSrv.setStatus(status);
    return healthStatusSrv.getStatus();
}
```

### 2.4 验证

浏览器访问：http://euk1.com:8084/health?status=true

![image-20200915003910017](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200915003910017.png)

服务器状态

![image-20200915003946202](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200915003946202.png)



浏览器访问：http://euk1.com:8084/health?status=false

![image-20200915004011375](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200915004011375.png)

健康检查

![image-20200915004052758](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200915004052758.png)

注册中心显示服务已经`DOWN`

![image-20200915004431124](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200915004431124.png)















