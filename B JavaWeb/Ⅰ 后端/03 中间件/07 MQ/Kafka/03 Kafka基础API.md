# Kafka基础API

## 1 Topic基本操作 DML管理

Java代码：

```java
package com.yeyangshu.dml;

import org.apache.kafka.clients.KafkaClient;
import org.apache.kafka.clients.admin.*;

import java.io.IOException;
import java.util.*;
import java.util.concurrent.ExecutionException;

/**
 * @author yeyangshu
 * @version 1.0
 * @date 2020/11/24 0:23
 */
public class KafkaTopicDML {

    public static void main(String[] args) throws IOException, ExecutionException, InterruptedException {
        // 1 配置kafka属性
        Properties properties = new Properties();
        properties.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "node02:9092,node03:9092,node04:9092");

        // 2 创建Kafka admin
        KafkaAdminClient adminClient = (KafkaAdminClient) KafkaAdminClient.create(properties);

        // 3 创建Topic
        // 异步创建Topic
        // adminClient.createTopics(Arrays.asList(new NewTopic("topic02", 3, (short) 3)));
        // 同步创建Topic
        // CreateTopicsResult createTopic03 = adminClient.createTopics(Collections.singletonList(new NewTopic("topic03", 3, (short) 3)));
        // createTopic03.all().get();

        // 4 查看Topic列表
        ListTopicsResult topicsResult = adminClient.listTopics();
        Set<String> names = topicsResult.names().get();
        for (String name : names) {
            System.out.println(name);
        }

        // 5 查看Topic详情
        DescribeTopicsResult describeTopicsResult = adminClient.describeTopics(Collections.singleton("topic01"));
        Map<String, TopicDescription> topicDescriptionMap = describeTopicsResult.all().get();
        for (Map.Entry<String, TopicDescription> entry : topicDescriptionMap.entrySet()) {
            System.out.println(entry.getKey() + "\t" + entry.getValue());
        }

        // 6 删除Topic
        // 同步删除，异步删除同 同步创建Topic
        // DeleteTopicsResult deleteTopicsResult = adminClient.deleteTopics(Arrays.asList("topic02", "topic03"));
        // deleteTopicsResult.all().get();

        // 7 关闭AdminClient
        adminClient.close();
    }
}
```

打印日志：

```
INFO 2020-11-24 01:01:03 org.apache.kafka.clients.admin.AdminClientConfig - AdminClientConfig values: 
	bootstrap.servers = [node02:9092, node03:9092, node04:9092]
	client.dns.lookup = default
	client.id = 
	connections.max.idle.ms = 300000
	metadata.max.age.ms = 300000
	metric.reporters = []
	metrics.num.samples = 2
	metrics.recording.level = INFO
	metrics.sample.window.ms = 30000
	receive.buffer.bytes = 65536
	reconnect.backoff.max.ms = 1000
	reconnect.backoff.ms = 50
	request.timeout.ms = 120000
	retries = 5
	retry.backoff.ms = 100
	sasl.client.callback.handler.class = null
	sasl.jaas.config = null
	sasl.kerberos.kinit.cmd = /usr/bin/kinit
	sasl.kerberos.min.time.before.relogin = 60000
	sasl.kerberos.service.name = null
	sasl.kerberos.ticket.renew.jitter = 0.05
	sasl.kerberos.ticket.renew.window.factor = 0.8
	sasl.login.callback.handler.class = null
	sasl.login.class = null
	sasl.login.refresh.buffer.seconds = 300
	sasl.login.refresh.min.period.seconds = 60
	sasl.login.refresh.window.factor = 0.8
	sasl.login.refresh.window.jitter = 0.05
	sasl.mechanism = GSSAPI
	security.protocol = PLAINTEXT
	send.buffer.bytes = 131072
	ssl.cipher.suites = null
	ssl.enabled.protocols = [TLSv1.2, TLSv1.1, TLSv1]
	ssl.endpoint.identification.algorithm = https
	ssl.key.password = null
	ssl.keymanager.algorithm = SunX509
	ssl.keystore.location = null
	ssl.keystore.password = null
	ssl.keystore.type = JKS
	ssl.protocol = TLS
	ssl.provider = null
	ssl.secure.random.implementation = null
	ssl.trustmanager.algorithm = PKIX
	ssl.truststore.location = null
	ssl.truststore.password = null
	ssl.truststore.type = JKS

INFO 2020-11-24 01:01:05 org.apache.kafka.common.utils.AppInfoParser - Kafka version: 2.2.0
INFO 2020-11-24 01:01:05 org.apache.kafka.common.utils.AppInfoParser - Kafka commitId: 05fcfde8f69b0349
topic01
# Topic信息
topic01	(name=topic01, internal=false, partitions=(partition=0, leader=node04:9092 (id: 0 rack: null), replicas=node04:9092 (id: 0 rack: null), isr=node04:9092 (id: 0 rack: null)),(partition=1, leader=node04:9092 (id: 0 rack: null), replicas=node04:9092 (id: 0 rack: null), isr=node04:9092 (id: 0 rack: null)),(partition=2, leader=node04:9092 (id: 0 rack: null), replicas=node04:9092 (id: 0 rack: null), isr=node04:9092 (id: 0 rack: null)))

Process finished with exit code 0

```

## 2 生产者

```java
package com.yeyangshu.quickstart;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;
import java.util.stream.IntStream;

/**
 * 创建一个生产者
 *
 * @author yeyangshu
 * @version 1.0
 * @date 2020/12/10 23:39
 */
public class Producer {
    public static void main(String[] args) {
        // 1 配置属性信息
        Properties properties = new Properties();
        // bootstrap.servers，服务器参数
        properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "node02:9092,node03:9092,node04:9092");
        // key、value序列化
        properties.setProperty(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.setProperty(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        // 2 创建Kafka producer
        KafkaProducer<String, String> producer = new KafkaProducer<>(properties);

        // 3 生产者生产消息
        IntStream.range(0, 10)
                .forEach(i -> {
                    ProducerRecord<String, String> record = new ProducerRecord<>("topic01", "key" + i, "value" + i);
                    // 发送消息给服务器
                    producer.send(record);
                });

        // 4 关闭生产者
        producer.close();
    }
}

```

## 3 消费者 sub/assign

指定分组

```java
package com.yeyangshu.quickstart;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.time.Duration;
import java.util.Iterator;
import java.util.Properties;
import java.util.regex.Pattern;

/**
 * 创建一个消费者，指定分组
 *
 * @author yeyangshu
 * @version 1.0
 * @date 2020/12/10 23:54
 */
public class Consumer {
    public static void main(String[] args) {
        // 1 配置属性信息
        Properties properties = new Properties();
        // bootstrap.servers，服务器参数
        properties.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "node02:9092,node03:9092,node04:9092");
        // key、value反序列化
        properties.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        // group.id，分组
        properties.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "g1");

        // 2 创建Kafka consumer
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);

        // 3 消费者订阅topics
        consumer.subscribe(Pattern.compile("^topic.*"));

        while (true) {
            ConsumerRecords<String, String> consumerRecords = consumer.poll(Duration.ofSeconds(1));
            if (!consumerRecords.isEmpty()) {
                Iterator<ConsumerRecord<String, String>> recordIterator = consumerRecords.iterator();
                while (recordIterator.hasNext()) {
                    ConsumerRecord<String, String> record = recordIterator.next();
                    String topic = record.topic();
                    int partition = record.partition();
                    long offset = record.offset();
                    String key = record.key();
                    String value = record.value();
                    long timestamp = record.timestamp();
                    System.out.println(topic + "\t" + partition + "\t" + key + "\t" + value + timestamp);
                }
            }
        }
    }
}
```

订阅

```java
package com.yeyangshu.quickstart;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.TopicPartition;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.time.Duration;
import java.util.Arrays;
import java.util.Iterator;
import java.util.List;
import java.util.Properties;

/**
 * 创建一个消费者，手动指定分区
 *
 * @author yeyangshu
 * @version 1.0
 * @date 2020/12/10 23:54
 */
public class Consumer2 {
    public static void main(String[] args) {
        // 1 配置属性信息
        Properties properties = new Properties();
        // bootstrap.servers，服务器参数
        properties.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "node02:9092,node03:9092,node04:9092");
        // key、value反序列化
        properties.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

        // 2 创建Kafka consumer
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);

        // 3 消费者订阅topics，手动指定消费分区，失去组管理特性
        List<TopicPartition> partitions = Arrays.asList(new TopicPartition("topic01", 0));
        consumer.assign(partitions);
        // 指定消费分区的位置
        // consumer.seekToBeginning(partitions);
        // 0分区，1位置
        consumer.seek(new TopicPartition("topic01", 0), 1);

        while (true) {
            ConsumerRecords<String, String> consumerRecords = consumer.poll(Duration.ofSeconds(1));
            if (!consumerRecords.isEmpty()) {
                Iterator<ConsumerRecord<String, String>> recordIterator = consumerRecords.iterator();
                while (recordIterator.hasNext()) {
                    ConsumerRecord<String, String> record = recordIterator.next();
                    String topic = record.topic();
                    int partition = record.partition();
                    long offset = record.offset();
                    String key = record.key();
                    String value = record.value();
                    long timestamp = record.timestamp();
                    System.out.println(topic + "\t" + partition + "\t" + key + "\t" + value + timestamp);
                }
            }
        }
    }
}
```



## 4 自定义分区

```java

```

## 5 序列化

## 6 拦截器