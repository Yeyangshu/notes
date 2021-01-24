# Kafka高级特性

- 消息拦截器
- 偏移量控制
- 幂等性&事物控制

## 1 Offset控制

Kafka 消费者默认对于未订阅的 topic 的 offset 的时候，也就是系统并没有存储该消费者的消费分区的记录信息，默认Kafka 消费者的默认首次消费策略：latest

```properties
auto.offset.reset=latest
```

消费策略一共有三种：

- earliest：自动将偏移量重置为最早的偏移量
- latest：自动将偏移量重置为最新的偏移量
- none：如果未找到消费者组的先前偏移量，则向消费者抛出异常

### 1.1 自动提交

Kafka 消费者在消费数据的时候默认会定期的提交消费的偏移量，这样就可以保证所有的消息至少可以被消费者消费 1 次，用户可以通过以下两个参数配置：

```properties
# 默认
enable.auto.commit = true
# 默认
auto.commit.interval.ms = 5000
```

如果用户需要自己管理 offset 的自动提交，可以关闭 offset 的自动提交，手动管理 offset 提交的偏移量，注意用户提交的 offset 偏移量永远都要比本次消费的偏移量 +1，因为提交的 offset 是 kafka 消费者下一次抓取数据的位置。

## 2 Acks & Retries

Kafka生产者在发送完一个的消息之后，要求Broker在规定的额时间Ack应答，如果没有在规定时间内应答，Kafka生产者会尝试n次重新发送消息。

```properties
# 默认
acks=1
```

Ack应答共有三种：

- acks=1

  Leader会将Record写到其本地日志中，但会在不等待所有Follower的完全确认的情况下做出响应。在这种情况下，如果Leader在确认记录后立即失败，但在Follower复制记录之前失败，则记录将丢失。

- acks=0

  生产者根本不会等待服务器的任何确认。该记录将立即添加到套接字缓冲区中并视为已发送。在这种情况下，不能保证服务器已收到记录。

- acks=all

  这意味着Leader将等待全套同步副本确认记录。这保证了只要至少一个同步副本仍处于活动状态，记录就不会丢失。这是最有力的保证。这等效于acks = -1设置。

如果生产者在规定的时间内，并没有得到Kafka的Leader的Ack应答，Kafka可以开启reties机制。

```properties
# 默认
request.timeout.ms = 30000
# 默认
retries = 2147483647
```

## 幂等性

HTTP/1.1中对幂等性的定义是：一次和多次请求某一个资源对于资源本身应该具有同样的结果（网络超时等问题除外）。也就是说，其任意多次执行对资源本身所产生的影响均与一次执行的影响相同。



Methods can also have the property of “idempotence” in that (aside from error or expiration issues) the side-effects of N > 0 identical requests is the same as for a single request.



Kafka在0.11.0.0版本支持增加了对幂等的支持。幂等是针对生产者角度的特性。幂等可以保证上生产者发送的消息，不会丢失，而且不会重复。实现幂等的关键点就是服务端可以区分请求是否重复，过滤掉重复的请求。要区分请求是否重复的有两点：



**唯一标识**：要想区分请求是否重复，请求中就得有唯一标识。例如支付请求中，订单号就是唯一标识



**记录下已处理过的请求标识**：光有唯一标识还不够，还需要记录下那些请求是已经处理过的，这样当收到新的请求时，用新请求中的标识和处理记录进行比较，如果处理记录中有相同的标识，说明是重复记录，拒绝掉。

幂等又称为exactly once。要停止多次处理消息，必须仅将其持久化到Kafka Topic中仅仅一次。在初始化期间，kafka会给生产者生成一个唯一的ID称为Producer ID或PID。



PID和序列号与消息捆绑在一起，然后发送给Broker。由于序列号从零开始并且单调递增，因此，仅当消息的序列号比该PID / TopicPartition对中最后提交的消息正好大1时，Broker才会接受该消息。如果不是这种情况，则Broker认定是生产者重新发送该消息。





enable.idempotence= false 默认



注意:在使用幂等性的时候，要求必须开启retries=true和acks=all

## 事务控制

Kafka的幂等性，只能保证一条记录的在分区发送的原子性，但是如果要保证多条记录（多分区）之间的完整性，这个时候就需要开启kafk的事务操作。



在Kafka0.11.0.0除了引入的幂等性的概念，同时也引入了事务的概念。通常Kafka的事务分为 **生产者事务****Only**、**消费者****&****生产者事务**。一般来说默认消费者消费的消息的级别是read_uncommited数据，这有可能读取到事务失败的数据，所有在开启生产者事务之后，需要用户设置消费者的事务隔离级别。



isolation.level = read_uncommitted 默认



该选项有两个值read_committed|read_uncommitted，如果开始事务控制，消费端必须将事务的隔离级别设置为read_committed



开启的生产者事务的时候，只需要指定transactional.id属性即可，一旦开启了事务，默认生产者就已经开启了幂等性。但是要求"transactional.id"的取值必须是唯一的，同一时刻只能有一个"transactional.id"存储在，其他的将会被关闭。