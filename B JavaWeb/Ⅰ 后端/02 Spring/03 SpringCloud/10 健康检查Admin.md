# Admin健康检查

## 1 Admin服务器端

### 1.1 引入依赖

server端

```xml
<properties>
    <java.version>1.8</java.version>
    <spring-boot-admin.version>2.3.0</spring-boot-admin.version>
</properties>

<!-- Admin 服务 -->
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-server</artifactId>
</dependency>
<!-- Admin 界面 -->
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-server-ui</artifactId>
</dependency>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>de.codecentric</groupId>
            <artifactId>spring-boot-admin-dependencies</artifactId>
            <version>${spring-boot-admin.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 1.2 启动类添加`@EnableAdminServer`注解

```java
package com.yeyangshu;

import de.codecentric.boot.admin.server.config.EnableAdminServer;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@EnableAdminServer
public class AdminApplication {

	public static void main(String[] args) {
		SpringApplication.run(AdminApplication.class, args);
	}

}
```

## 2 微服务端

provider、consumer等等都可以添加client

### 2.1 pom依赖

微服务端：

```xml
<!-- Admin 服务 -->
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-client</artifactId>
    <version>2.2.1</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 2.2 配置

```properties
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always
spring.boot.admin.client.url=http://localhost:8080
```

## 3 Admin使用

访问`http://localhost:8080`可以看到具体的实例

## 4 通知

### 4.1 邮件通知

1. pom

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-mail</artifactId>
   </dependency>
   ```

2. yml

   ```yaml
   spring: 
     application: 
       name: cloud-admin
     security:
       user:
         name: root
         password: root
     # 邮件设置
     mail:
       host: smtp.qq.com
       username: 单纯QQ号
       password: xxxxxxx授权码
       properties:
         mail: 
           smpt: 
             auth: true
             starttls: 
               enable: true
               required: true
   #收件邮箱
   spring.boot.admin.notify.mail.to: 2634982208@qq.com   
   #发件邮箱
   spring.boot.admin.notify.mail.from: xxxxxxx@qq.com   
   ```

**邮件截图：**

![image-20201012222906908](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201012222906908.png)

### 4.2 钉钉群通知

参考：https://juejin.im/post/6863645878717644808

#### 4.2.1 启动类

```java
package com.mashibing.admin;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

import de.codecentric.boot.admin.server.config.EnableAdminServer;
import de.codecentric.boot.admin.server.domain.entities.InstanceRepository;

@SpringBootApplication
@EnableAdminServer
public class AdminApplication {

    public static void main(String[] args) {
        SpringApplication.run(AdminApplication.class, args);
    }
    // 复写原来的方法
    @Bean
    public DingDingNotifier dingDingNotifier(InstanceRepository repository) {
        return new DingDingNotifier(repository);
    }
}

```

#### 4.2.2 通知类

```java
package com.mashibing.admin;

import java.util.Map;

import com.alibaba.fastjson.JSONObject;

import de.codecentric.boot.admin.server.domain.entities.Instance;
import de.codecentric.boot.admin.server.domain.entities.InstanceRepository;
import de.codecentric.boot.admin.server.domain.events.InstanceEvent;
import de.codecentric.boot.admin.server.notify.AbstractStatusChangeNotifier;
import reactor.core.publisher.Mono;

public class DingDingNotifier extends AbstractStatusChangeNotifier  {
    public DingDingNotifier(InstanceRepository repository) {
        super(repository);
    }
    @Override
    protected Mono<Void> doNotify(InstanceEvent event, Instance instance) {
        String serviceName = instance.getRegistration().getName();
        String serviceUrl = instance.getRegistration().getServiceUrl();
        String status = instance.getStatusInfo().getStatus();
        Map<String, Object> details = instance.getStatusInfo().getDetails();
        StringBuilder str = new StringBuilder();
        str.append("服务预警 : 【" + serviceName + "】");
        str.append("【服务地址】" + serviceUrl);
        str.append("【状态】" + status);
        str.append("【详情】" + JSONObject.toJSONString(details));
        return Mono.fromRunnable(() -> {
            DingDingMessageUtil.sendTextMessage(str.toString());
        });
    }
}
```

#### 4.2.3 发送工具类

```java
package com.mashibing.admin;

import java.io.InputStream;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.URL;

import com.alibaba.fastjson.JSONObject;

public class DingDingMessageUtil {
	public static String access_token = "Token";
    public static void sendTextMessage(String msg) {
        try {
            Message message = new Message();
            message.setMsgtype("text");
            message.setText(new MessageInfo(msg));
            URL url = new URL("https://oapi.dingtalk.com/robot/send?access_token=" + access_token);
            // 建立 http 连接
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.setDoOutput(true);
            conn.setDoInput(true);
            conn.setUseCaches(false);
            conn.setRequestMethod("POST");
            conn.setRequestProperty("Charset", "UTF-8");
            conn.setRequestProperty("Content-Type", "application/Json; charset=UTF-8");
            conn.connect();
            OutputStream out = conn.getOutputStream();
            String textMessage = JSONObject.toJSONString(message);
            byte[] data = textMessage.getBytes();
            out.write(data);
            out.flush();
            out.close();
            InputStream in = conn.getInputStream();
            byte[] data1 = new byte[in.available()];
            in.read(data1);
            System.out.println(new String(data1));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

#### 4.2.4 消息类

```java
package com.mashibing.admin;

public class Message {
    private String msgtype;
    private MessageInfo text;
    public String getMsgtype() {
        return msgtype;
    }
    public void setMsgtype(String msgtype) {
        this.msgtype = msgtype;
    }
    public MessageInfo getText() {
        return text;
    }
    public void setText(MessageInfo text) {
        this.text = text;
    }
}

package com.mashibing.admin;

public class MessageInfo {
    private String content;
    public MessageInfo(String content) {
        this.content = content;
    }
    public String getContent() {
        return content;
    }
    public void setContent(String content) {
        this.content = content;
    }
}
```

### 4.3 微信通知

服务号 模板消息