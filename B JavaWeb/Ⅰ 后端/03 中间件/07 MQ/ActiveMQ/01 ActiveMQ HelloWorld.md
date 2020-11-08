## HelloWorld

## 1 下载

http://activemq.apache.org/

## 2 安装启动

解压后直接执行

`bin/win64/activemq.bat`

## 3 web控制台

http://localhost:8161/

通过8161端口访问

## 4 修改访问端口

修改 ActiveMQ 配置文件:/usr/local/activemq/conf/jetty.xml

**jettyport节点**

配置文件修改完毕，保存并重新启动 ActiveMQ 服务。

## 5 开发

### 5.1 maven

```xml
<!-- https://mvnrepository.com/artifact/org.apache.activemq/activemq-all -->
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-all</artifactId>
    <version>5.15.11</version>
</dependency>
```

### 5.2 Sender

```java

package com.mashibing.mq;

import javax.jms.Connection;
import javax.jms.DeliveryMode;
import javax.jms.MessageProducer;
import javax.jms.Queue;
import javax.jms.Session;
import javax.jms.TextMessage;

import org.apache.activemq.ActiveMQConnectionFactory;

/**
 * 消息发送
 */
public class Sender {

    public static void main(String[] args) throws Exception {
        // 1.获取连接工厂
        ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory(
                "admin",
                "admin",
                "tcp://localhost:61616"
        );
        // 2.获取一个向ActiveMQ的连接
        Connection connection = connectionFactory.createConnection();
        // 3.获取session
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4.查找目的地，获取destination，消费端也会从这个目的地取消息
        Queue queue = session.createQueue("user");
        // 5.1消息创建者
        MessageProducer producer = session.createProducer(queue);
        // 5.2创建消息
        TextMessage textMessage = session.createTextMessage("Hello World");
        // 5.3向目的地写入消息
        producer.send(textMessage);
        // 6.关闭连接
        connection.close();
        System.out.println("System exit....");
    }
}
```

### 5.3 Receiver

```java
package com.mashibing.mq;

import javax.jms.Connection;
import javax.jms.Destination;
import javax.jms.MessageConsumer;
import javax.jms.Session;
import javax.jms.TextMessage;

import org.apache.activemq.ActiveMQConnectionFactory;

/**
 * 消息接收
 */
public class Receiver {

    public static void main(String[] args) throws Exception {

        // 1.获取连接工厂
        ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory(
                "admin",
                "admin",
                "tcp://localhost:61616"
        );
        // 2.获取一个向ActiveMQ的连接
        Connection connection = connectionFactory.createConnection();
        connection.start();
        // 3.获取session
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 4.找目的地，获取destination，消费端会从这个目的地取消息
        Destination queue = session.createQueue("user");
        // 5.获取消息
        MessageConsumer consumer = session.createConsumer(queue);
        TextMessage message = (TextMessage) consumer.receive();
        System.out.println("message:" + message.getText());
    }
}

```

# 