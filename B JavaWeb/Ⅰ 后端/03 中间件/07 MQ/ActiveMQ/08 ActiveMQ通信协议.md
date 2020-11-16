# 通信协议

## NIO配置

默认配置为tcp，使用的是bio

```xml
 <transportConnector name="openwire" uri="tcp://0.0.0.0:61616?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
```

http://activemq.apache.org/configuring-version-5-transports

Nio是基于TCP的

客户端使用连接时也应使用nio

```java
ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory(
    "admin",
    "admin",
    "nio://localhost:61617"
);
```

Auto + Nio 自动适配协议

```
<transportConnector name="auto+nio" uri="auto+nio://localhost:5671"/>
```

