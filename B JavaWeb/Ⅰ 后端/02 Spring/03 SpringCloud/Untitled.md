# Eureka-Server

pom

```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
```



## 1 application.properties

```
# 安全认证
spring.security.user.name=yeyangshu
spring.security.user.password=1
```

## 2 application.yml

注释所有

## 3 config

```java
package com.yeyangshu.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

/**
 * 关闭security防止跨域攻击配置类
 *
 */
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // TODO Auto-generated method stub
        http.csrf().disable();
        super.configure(http);
    }

}
```

# Eureka-Provider

web项目

## 1 pom

```

```

## 2 application.properties

```
eureka.client.serviceUrl.defaultZone=http://yeyangshu:1@euk1.com:7001/eureka/
```

## 3 MainController

```
    @Value("${server.port}")
    String port;

    @GetMapping("/hi")
    public String getHi() {
        return "hi, my post is " + port;
    }

```

## 4 EurekaProviderApplication

```
//    @Bean
//    public IRule myRule() {
//        return new RandomRule();
//    }
```



# Eureka-Consumer

## 1 pom

```

```

## 2 application.properties

```
#负载均衡策略
provider.ribbon.NFLoadBalancerRuleClassName=com.netflix.loadbalancer.RandomRule
```

## 3 MainController2

```
package com.yeyangshu.controller;

import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.loadbalancer.LoadBalancerClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import java.util.List;
import java.util.Random;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 负载均衡测试 controller
 */
@RestController
public class MainController2 {

    @Autowired
    DiscoveryClient discoveryClient;

    @Autowired
    LoadBalancerClient loadBalancerClient;

    @Autowired
    RestTemplate restTemplate;

    /**
     * provider 负载均衡
     *
     * 默认：ZoneAvoidanceRule（区域权衡策略）：复合判断Server所在区域的性能和Server的可用性，轮询选择服务器。
     * @return
     */
    @GetMapping("/client6")
    public Object client6() {
        ServiceInstance instance = loadBalancerClient.choose("provider");
        String url = "http://" + instance.getHost() + ":" + instance.getPort() + "/hi";
        System.out.println(instance.getHost() + instance.getPort());
        return restTemplate.getForObject(url, String.class);
    }

    /**
     * 自定义负载均衡算法
     * @return
     */
    @GetMapping("/client7")
    public Object client7() {
        List<ServiceInstance> instances = discoveryClient.getInstances("provider");

        /* 自定义算法 */
        // 例：随机算法
        int nextInt = new Random().nextInt(instances.size());
        AtomicInteger atomicInteger = new AtomicInteger();

        ServiceInstance instance = instances.get(nextInt);
        String url = "http://" + instance.getHost() + ":" + instance.getPort() + "/hi";
        return restTemplate.getForObject(url, String.class);
    }

    /**
     * 自动处理 url
     * 在restTemplate添加 @LoadBalanced 注解
     * @return
     */
    @GetMapping("/client9")
    public Object client9() {
        // 自动处理URL
        String url ="http://provider/hi";
        String respStr = restTemplate.getForObject(url, String.class);
        System.out.println(respStr);
        return respStr;
    }

}

```

## 4 EurekaConsumerApplication

```
    @Bean
    @LoadBalanced
    RestTemplate getRestTemplate() {

        RestTemplate restTemplate = new RestTemplate();
//        restTemplate.getInterceptors().add(new LoggingClientHttpRequestInterceptor());
        return restTemplate;
    }
```