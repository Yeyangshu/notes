# Active MQ 整合SpringBoot

### 1 pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.3.BUILD-SNAPSHOT</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.mashibing.arika</groupId>
	<artifactId>mq</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>mq</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
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
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-activemq</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		
		
		<dependency>
		    <groupId>org.messaginghub</groupId>
		    <artifactId>pooled-jms</artifactId>
		</dependency>
				
		<dependency>
	            <groupId>org.apache.commons</groupId>
	            <artifactId>commons-pool2</artifactId>
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

### 2 application.yml

```yaml
server:
  port: 80
  
spring:
  activemq:
    broker-url: tcp://localhost:61616
    user: admin
    password: admin
    
    pool:
      enabled: true
      # 连接池最大连接数
      max-connections: 5
      # 空闲的连接过期时间，默认为30秒
      idle-timeout: 0
    packages:
      trust-all: true
  # 开启支持发布订阅模型，activemq默认只支持点对点
  jms:
    pub-sub-domain: true
```

### 3 Config配置类

用于生产ConnectionFactory

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jms.annotation.EnableJms;
import org.springframework.jms.config.DefaultJmsListenerContainerFactory;
import org.springframework.jms.config.JmsListenerContainerFactory;

import javax.jms.ConnectionFactory;

/**
 * @author yeyangshu
 * @version 1.0
 * @date 2020/11/8 23:08
 */
@Configuration
@EnableJms
public class ActiveMqConfig {

    /**
     * Topic模式的ListenerContainer
     *
     * @param activeMQConnectionFactory
     * @return
     */
    @Bean
    public JmsListenerContainerFactory<?> jmsListenerContainerTopic(ConnectionFactory activeMQConnectionFactory) {
        DefaultJmsListenerContainerFactory bean = new DefaultJmsListenerContainerFactory();
        bean.setPubSubDomain(true);
        bean.setConnectionFactory(activeMQConnectionFactory);
        return bean;
    }

    /**
     * Queue模式的ListenerContainer
     *
     * @param activeMQConnectionFactory
     * @return
     */
    @Bean
    public JmsListenerContainerFactory<?> jmsListenerContainerQueue(ConnectionFactory activeMQConnectionFactory) {
        DefaultJmsListenerContainerFactory bean = new DefaultJmsListenerContainerFactory();
        bean.setConnectionFactory(activeMQConnectionFactory);
        return bean;
    }
}
```

### 4 发送者

```java
public class ProducerService {

    /**
     * 继承JmsTemplate
     */
    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;

    /**
     * 包含mq原始的api
     */
    @Autowired
    private JmsTemplate jmsTemplate;

    /**
     * jmsMessagingTemplate可以发送任何数据，不需要转换类型，封装好了
     *
     * @param destination
     * @param message
     */
    public void sendObject(String destination, String message) {
        System.out.println("发送简单的数据");
        jmsMessagingTemplate.convertAndSend(destination, message);
    }

    /**
     * 发送列表
     *
     * @param destination
     * @param message
     */
    public void sendList(String destination, String message) {
        System.out.println("发送列表");
        ArrayList<String> list = new ArrayList<>();
        list.add("1");
        list.add("2");
        // 设置为队列
        jmsMessagingTemplate.convertAndSend(new ActiveMQQueue(destination), list);
        // 设置为P/S
        jmsMessagingTemplate.convertAndSend(new ActiveMQTopic(destination), list);
    }

    /**
     * 使用JmsTemplate原始api，可以自定义参数
     *
     * @param destination
     * @param message
     */
    public void sendByJmsTemplate(String destination, String message) {
        jmsTemplate.send(destination, new MessageCreator() {
            @Override
            public Message createMessage(Session session) throws JMSException {
                TextMessage textMessage = session.createTextMessage("使用JmsTemplate原始api");
                textMessage.setIntProperty("name", 1);
                return textMessage;
            }
        });
    }

    /**
     * 最原始的factory，自定义配置
     *
     * @param destination
     * @param msg
     */
    public void sendUseFactory(String destination, String msg) {
        System.out.println("send...");
        ActiveMQQueue queue = new ActiveMQQueue(destination);
        jmsMessagingTemplate.afterPropertiesSet();

        ConnectionFactory factory = jmsMessagingTemplate.getConnectionFactory();

        try {
            Connection connection = factory.createConnection();
            connection.start();

            Session session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);
            Queue queue2 = session.createQueue(destination);

            MessageProducer producer = session.createProducer(queue2);
            TextMessage message = session.createTextMessage("hahaha");
            producer.send(message);
        } catch (JMSException e) {
            e.printStackTrace();
        }
        jmsMessagingTemplate.convertAndSend(queue, msg);
    }
    
}
```



### 5 接收者

```java
public class ConsumerService {

    @JmsListener(destination = "user",containerFactory = "jmsListenerContainerQueue")
    public void receiveStringQueue(String msg) {
        System.out.println("接收到消息...." + msg);
    }

    @JmsListener(destination = "user",containerFactory = "jmsListenerContainerTopic")
    public void receiveStringTopic(String msg) {
        System.out.println("接收到消息...." + msg);
    }

}
```

