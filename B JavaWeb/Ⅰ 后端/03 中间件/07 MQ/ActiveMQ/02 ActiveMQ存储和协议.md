# ActiveMQ

![img](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/activemq_logo_white_vertical.png)

官方网站

http://activemq.apache.org/



## 1 Broker

ActiveMQ 5.0 的二进制发布包中bin目录中包含一个名为activemq的脚本，直接运行这个脚本就可以启动一个broker。 此外也可以通过Broker Configuration URI或Broker XBean URI对broker进行配置，以下是一些命令行参数的例子：

| Example                                                      | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| activemq                                                     | Runs a broker using the default  'xbean:activemq.xml' as the broker configuration file. |
| activemq xbean:myconfig.xml                                  | Runs a broker using the file myconfig.xml as the  broker configuration file that is located in the classpath. |
| activemq xbean:file:./conf/broker1.xml                       | Runs a broker using the file broker1.xml as the  broker configuration file that is located in the relative file path  ./conf/broker1.xml |
| activemq xbean:file:C:/ActiveMQ/conf/broker2.xml             | Runs a broker using the file broker2.xml as the  broker configuration file that is located in the absolute file path  C:/ActiveMQ/conf/broker2.xml |
| activemq broker:(tcp://localhost:61616,  tcp://localhost:5000)?useJmx=true | Runs a broker with two transport connectors and  JMX enabled. |
| activemq broker:(tcp://localhost:61616,  network:tcp://localhost:5000)?persistent=false | Runs a broker with 1 transport connector and 1  network connector with persistence disabled. |

## 2 存储

### 2.1 KahaDB存储

KahaDB是默认的持久化策略，所有消息顺序添加到一个日志文件中，同时另外有一个索引文件记录指向这些日志的存储地址，还有一个事务日志用于消息回复操作。是一个专门针对消息持久化的解决方案,它对典型的消息使用模式进行了优化。

在data/kahadb这个目录下，会生成四个文件，来完成消息持久化 
1.db.data 它是消息的索引文件，本质上是B-Tree（B树），使用B-Tree作为索引指向db-*.log里面存储的消息 
2.db.redo 用来进行消息恢复 *

3.db-.log 存储消息内容。新的数据以APPEND的方式追加到日志文件末尾。属于顺序写入，因此消息存储是比较 快的。默认是32M，达到阀值会自动递增 
4.lock文件 锁，写入当前获得kahadb读写权限的broker ，用于在集群环境下的竞争处理

```xml
<persistenceAdapter>
    <!--directory:保存数据的目录;journalMaxFileLength:保存消息的文件大小 --> 	          <kahaDBdirectory="${activemq.data}/kahadb"journalMaxFileLength="16mb"/> </persistenceAdapter>
```

特性：

1、日志形式存储消息；

2、消息索引以 B-Tree 结构存储，可以快速更新；

3、 完全支持 JMS 事务；

4、支持多种恢复机制kahadb 可以限制每个数据文件的大小。不代表总计数据容量。 

### 2.2 AMQ 方式

只适用于 5.3 版本之前。 AMQ 也是一个文件型数据库，消息信息最终是存储在文件中。内存中也会有缓存数据。 

```xml
<persistenceAdapter>
    <!--directory:保存数据的目录 ;maxFileLength:保存消息的文件大小 --> 				<amqPersistenceAdapterdirectory="${activemq.data}/amq"maxFileLength="32mb"/> </persistenceAdapter>
```

 性能高于 JDBC，写入消息时，会将消息写入日志文件，由于是顺序追加写，性能很高。

 为了提升性能，创建消息主键索引，并且提供缓存机制，进一步提升性能。

每个日志文件的 大小都是有限制的（默认 32m，可自行配置） 。 

当超过这个大小，系统会重新建立一个文件。

当所有的消息都消费完成，系统会删除这 个文件或者归档。 

主要的缺点是 AMQ Message 会为每一个 Destination 创建一个索引，如果使用了大量的 Queue，索引文件的大小会占用很多磁盘空间。 

而且由于索引巨大，一旦 Broker（ActiveMQ 应用实例）崩溃，重建索引的速度会非常 慢。 

虽然 AMQ 性能略高于 Kaha DB 方式，但是由于其重建索引时间过长，而且索引文件 占用磁盘空间过大，所以已经不推荐使用。

### 2.3 JDBC存储 

使用JDBC持久化方式，数据库默认会创建3个表，每个表的作用如下： 

- activemq_msgs：queue和topic的消息都存在这个表中 

- activemq_acks：存储持久订阅的信息和最后一个持久订阅接收的消息ID 

- activemq_lock：跟kahadb的lock文件类似，确保数据库在某一时刻只有一个broker在访问

ActiveMQ 将数据持久化到数据库中。 

不指定具体的数据库。 可以使用任意的数据库 中。 

本环节中使用 MySQL 数据库。 下述文件为 activemq.xml 配置文件部分内容。 

 首先定义一个 mysql-ds 的 MySQL 数据源，然后在 persistenceAdapter 节点中配置 jdbcPersistenceAdapter 并且引用刚才定义的数据源。

dataSource 指定持久化数据库的 bean，createTablesOnStartup 是否在启动的时候创建数 据表，默认值是 true，这样每次启动都会去创建数据表了，一般是第一次启动的时候设置为 true，之后改成 false。 

#### 2.3.1 配置方式

**Beans中添加**

```xml
<bean id="mysql-ds" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close"> 

<property name="driverClassName" value="com.mysql.jdbc.Driver"/> 
<property name="url" value="jdbc:mysql://localhost/activemq?relaxAutoCommit=true"/> 
<property name="username" value="activemq"/>
<property name="password" value="activemq"/>
<property name="maxActive" value="200"/>
<property name="poolPreparedStatements" value="true"/> 

</bean>
```

**修改persistenceAdapter**

```xml
<persistenceAdapter>
    <!-- <kahaDB directory="${activemq.data}/kahadb"/> -->
    <jdbcPersistenceAdapter dataSource="#mysql-ds" createTablesOnStartup="true" /> 
</persistenceAdapter>
```

依赖jar包

commons-dbcp commons-pool mysql-connector-java

#### 2.3.2 表字段解释

**activemq_acks**：用于存储订阅关系。如果是持久化Topic，订阅者和服务器的订阅关系在这个表保存。
主要的数据库字段如下：

```
container：消息的destination 
sub_dest：如果是使用static集群，这个字段会有集群其他系统的信息 
client_id：每个订阅者都必须有一个唯一的客户端id用以区分 
sub_name：订阅者名称 
selector：选择器，可以选择只消费满足条件的消息。条件可以用自定义属性实现，可支持多属性and和or操作 
last_acked_id：记录消费过的消息的id。
```

2：**activemq_lock**：在集群环境中才有用，只有一个Broker可以获得消息，称为Master Broker，其他的只能作为备份等待Master Broker不可用，才可能成为下一个Master Broker。这个表用于记录哪个Broker是当前的Master Broker。

3：**activemq_msgs**：用于存储消息，Queue和Topic都存储在这个表中。
主要的数据库字段如下：

```
id：自增的数据库主键 
container：消息的destination 
msgid_prod：消息发送者客户端的主键 
msg_seq：是发送消息的顺序，msgid_prod+msg_seq可以组成jms的messageid 
expiration：消息的过期时间，存储的是从1970-01-01到现在的毫秒数 
msg：消息本体的java序列化对象的二进制数据 
priority：优先级，从0-9，数值越大优先级越高 
xid:用于存储订阅关系。如果是持久化topic，订阅者和服务器的订阅关系在这个表保存。
```

### 2.4 LevelDB存储 

LevelDB持久化性能高于KahaDB，虽然目前默认的持久化方式仍然是KahaDB。并且，在ActiveMQ 5.9版本提供 了基于LevelDB和Zookeeper的数据复制方式，用于Master-slave方式的首选数据复制方案。 但是在ActiveMQ官网对LevelDB的表述：LevelDB官方建议使用以及不再支持，推荐使用的是KahaDB 


### 2.5 Memory 消息存储

顾名思义，基于内存的消息存储，就是消息存储在内存中。persistent=”false”,表示不设置持 久化存储，直接存储到内存中 
在broker标签处设置。

### 2.6 JDBC Message store with ActiveMQ Journal 

这种方式克服了JDBC Store的不足，JDBC存储每次消息过来，都需要去写库和读库。 ActiveMQ Journal，使用延迟存储数据到数据库，当消息来到时先缓存到文件中，延迟后才写到数据库中。

当消费者的消费速度能够及时跟上生产者消息的生产速度时，journal文件能够大大减少需要写入到DB中的消息。 举个例子，生产者生产了1000条消息，这1000条消息会保存到journal文件，如果消费者的消费速度很快的情况 下，在journal文件还没有同步到DB之前，消费者已经消费了90%的以上的消息，那么这个时候只需要同步剩余的 10%的消息到DB。 如果消费者的消费速度很慢，这个时候journal文件可以使消息以批量方式写到DB。 

## 3 协议

完整支持的协议

http://activemq.apache.org/configuring-version-5-transports.html



ActiveMQ支持的client-broker通讯协议有：TCP、NIO、UDP、SSL、Http(s)、VM。

### 3.1 Transmission Control Protocol (TCP) 

1：这是默认的Broker配置，TCP的Client监听端口是61616。
2：在网络传输数据前，必须要序列化数据，消息是通过一个叫wire protocol的来序列化成字节流。默认情况下，ActiveMQ把wire protocol叫做OpenWire，它的目的是促使网络上的效率和数据快速交互。
3：TCP连接的URI形式：tcp://hostname:port?key=value&key=value，加粗部分是必须的
4：TCP传输的优点：
(1)TCP协议传输可靠性高，稳定性强
(2)高效性：字节流方式传递，效率很高
(3)有效性、可用性：应用广泛，支持任何平台

```xml
<transportConnector name="openwire" uri="tcp://0.0.0.0:61616?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
```

### 3.2 New I/O API Protocol（NIO） 

1：NIO协议和TCP协议类似，但NIO更侧重于底层的访问操作。它允许开发人员对同一资源可有更多的client调用和服务端有更多的负载。 
2：适合使用NIO协议的场景：
(1)可能有大量的Client去链接到Broker上一般情况下，大量的Client去链接Broker是被操作系统的线程数所限制的。因此，NIO的实现比TCP需要更少的线程去运行，所以建议使用NIO协议
(2)可能对于Broker有一个很迟钝的网络传输NIO比TCP提供更好的性能
3：NIO连接的URI形式：nio://hostname:port?key=value
4：Transport Connector配置示例： 

```xml
<transportConnectors>
　　<transportConnector
　　　　name="tcp"
　　　　uri="tcp://localhost:61616?trace=true" />
　　<transportConnector
　　　　name="nio"
　　　　uri="nio://localhost:61618?trace=true" />
</transportConnectors>
```


上面的配置，示范了一个TCP协议监听61616端口，一个NIO协议监听61618端口 

### 3.3 User Datagram Protocol（UDP)

1：UDP和TCP的区别
(1)TCP是一个原始流的传递协议，意味着数据包是有保证的，换句话说，数据包是不会被复制和丢失的。UDP，另一方面，它是不会保证数据包的传递的
(2)TCP也是一个稳定可靠的数据包传递协议，意味着数据在传递的过程中不会被丢失。这样确保了在发送和接收之间能够可靠的传递。相反，UDP仅仅是一个链接协议，所以它没有可靠性之说
2：从上面可以得出：TCP是被用在稳定可靠的场景中使用的；UDP通常用在快速数据传递和不怕数据丢失的场景中，还有ActiveMQ通过防火墙时，只能用UDP
3：UDP连接的URI形式：udp://hostname:port?key=value
4：Transport Connector配置示例： 

```xml
<transportConnectors>
    <transportConnector
        name="udp"
        uri="udp://localhost:61618?trace=true" />
</transportConnectors>
```

### 3.4 Secure Sockets Layer Protocol (SSL) 

1：连接的URI形式：ssl://hostname:port?key=value
2：Transport Connector配置示例： 

```xml
<transportConnectors>
    <transportConnector name="ssl" uri="ssl://localhost:61617?trace=true"/>
</transportConnectors>
```

### 3.5 Hypertext Transfer Protocol (HTTP/HTTPS) 

1：像web和email等服务需要通过防火墙来访问的，Http可以使用这种场合
2：连接的URI形式：http://hostname:port?key=value或者https://hostname:port?key=value
3：Transport Connector配置示例：

```xml
<transportConnectors>
    <transportConnector name="http" uri="http://localhost:8080?trace=true" />
</transportConnectors>
```

### 3.6 VM Protocol（VM） 

1、VM transport允许在VM内部通信，从而避免了网络传输的开销。这时候采用的连 接不是socket连接，而是直接的方法调用。 

2、第一个创建VM连接的客户会启动一个embed VM broker，接下来所有使用相同的 broker name的VM连接都会使用这个broker。当这个broker上所有的连接都关闭 的时候，这个broker也会自动关闭。 

3、连接的URI形式：vm://brokerName?key=value 

4、Java中嵌入的方式： vm:broker:(tcp://localhost:6000)?brokerName=embeddedbroker&persistent=fal se ， 定义了一个嵌入的broker名称为embededbroker以及配置了一个 tcptransprotconnector在监听端口6000上 

5、使用一个加载一个配置文件来启动broker vm://localhost?brokerConfig=xbean:activemq.xml

