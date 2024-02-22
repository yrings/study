# kafka

## kafka概述

![](images/image-20210525181028436.png)

> ### 消息中间件对比                              

| 特性       | ActiveMQ                               | RabbitMQ                   | RocketMQ                 | Kafka                                    |
| ---------- | -------------------------------------- | -------------------------- | ------------------------ | ---------------------------------------- |
| 开发语言   | java                                   | erlang                     | java                     | scala                                    |
| 单机吞吐量 | 万级                                   | 万级                       | 10万级                   | 100万级                                  |
| 时效性     | ms                                     | us                         | ms                       | ms级以内                                 |
| 可用性     | 高（主从）                             | 高（主从）                 | 非常高（分布式）         | 非常高（分布式）                         |
| 功能特性   | 成熟的产品、较全的文档、各种协议支持好 | 并发能力强、性能好、延迟低 | MQ功能比较完善，扩展性佳 | 只支持主要的MQ功能，主要应用于大数据领域 |

消息中间件对比-选择建议

| **消息中间件** | **建议**                                                     |
| -------------- | ------------------------------------------------------------ |
| Kafka          | 追求高吞吐量，适合产生大量数据的互联网服务的数据收集业务     |
| RocketMQ       | 可靠性要求很高的金融互联网领域,稳定性高，经历了多次阿里双11考验 |
| RabbitMQ       | 性能较好，社区活跃度高，数据量没有那么大，优先选择功能比较完备的RabbitMQ |

> ### **kafka介绍**

Kafka 是一个分布式流媒体平台,类似于消息队列或企业消息传递系统。kafka官网：http://kafka.apache.org/  

![](images\image-20210525181100793.png)

其组件：

- producer：发布消息的对象称之为主题生产者（Kafka topic producer）

- topic：Kafka将消息分门别类，每一类的消息称之为一个主题（Topic）

- consumer：订阅消息并处理发布的消息的对象称之为主题消费者（consumers）

- broker：已发布的消息保存在一组服务器中，称之为Kafka集群。集群中的每一个服务器都是一个代理（Broker）。 消费者可以订阅一个或多个主题（topic），并从Broker拉数据，从而消费这些已发布的消息。

## kafka安装配置

> Kafka对于zookeeper是强依赖，保存kafka相关的节点数据，所以安装Kafka之前必须先安装zookeeper

- Docker安装zookeeper

下载镜像：

```shell
docker pull zookeeper:3.4.14
```

创建容器

```shell
docker run -d --name zookeeper -p 2181:2181 zookeeper:3.4.14
```

- Docker安装kafka

下载镜像：

```shell
docker pull wurstmeister/kafka:2.12-2.3.1
```

创建容器

```shell
docker run -d --name kafka \
--env KAFKA_ADVERTISED_HOST_NAME=192.168.200.130 \
--env KAFKA_ZOOKEEPER_CONNECT=192.168.200.130:2181 \
--env KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.200.130:9092 \
--env KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 \
--env KAFKA_HEAP_OPTS="-Xmx256M -Xms256M" \
--net=host wurstmeister/kafka:2.12-2.3.1
```

配置使用kafka服务端

```shell
# 进入kafka服务端控制台
docker exec -it [容器id] /bin/bash
# 进入kafka安装文件夹 
cd /opt/kafka/bin
# 增删改查topic
kafka-topics.sh --create --topic example --zookeeper 192.168.30.100:2181 --replication-factor 1 --partitions 1

--zookeeper 表示ZK地址，可以传递多个，用逗号分隔 IP:PORT,IP:PORT,IP:PORT/kafka
--replication-factor 表示副本数量，这里的数量是包含Leader副本和Follower副本，副本数量不能超过代理数量
--partitions 表示主题的分区数量，必须传递该参数。Kafka的生产者和消费者采用多线程并行对主题的消息进行处理，每个线程处理一个分区，分区越多吞吐量就会越大，但是分区越多也意味着需要打开更多的文件句柄数量，这样也会带来一些开销。

# 查看topic
kafka-topics.sh --zookeeper 192.168.30.100:2181 --list
# 删除topic
kafka-topics.sh --zookeeper 192.168.30.100:2181 --delete --topic example
# 获取topic详情 
kafka-topics.sh --zookeeper 192.168.30.100:2181 --describe --topic yrings-topic

# 彻底删除topic
# 进入zookeeper容器
docker exec -it [容器名或容器id] /bin/bash
# 进入zookeeper客户端，不同的zookeeper版本进入方式可能不同
./bin/zkCli.sh
# 查看topics数据
ls /brokers/topics
# 找到要删除的topic，执行命令：rmr /brokers/topics/【topic name】即可，此时topic被彻底删除。
rmr /brokers/topics/topic名称

#修改指定topic的分区数
kafka-topics.sh --zookeeper 192.168.30.100:2181 -alter --partitions 1 --topic yrings-topic
```





## kafka入门案例

### 1)java

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>${spring.kafka.version}</version>
</dependency>
```

生产者：

```java
public class ProducerQuickStart {
    public static void main(String[] args) {
        // 1. kafka的配置信息
        Properties properties = new Properties();
        // kafka的连接地址
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.30.100:9092");
        //  重试次数
        properties.put(ProducerConfig.RETRIES_CONFIG, 5);
        // 消息key的序列化器
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
        // 消息value的序列化器
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
        // 2. 生产者对象
        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(properties);
        // 封装发送的消息
        ProducerRecord<String, String> record = new ProducerRecord<String, String>("yrings-topic","100001","hello kafka");
        // 发送消息
        for (int i = 0; i < 10; i++) {
            producer.send(record);
        }
        // 关闭通道,必须关闭否则消息发送不成功
        producer.close();

    }
}
```

消费者：

```java
public class ConsumerQuickStart {
    public static void main(String[] args) {
        // 1. 添加kafka配置信息
        Properties properties = new Properties();
        // kafka连接地址
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.30.100:9092");
        // 消费者组
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "group2");
        // 消息的反序列化器
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");

        // 2. 消费者对象
        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(properties);
        // 3. 订阅主题
        consumer.subscribe(Collections.singletonList("yrings-topic"));
        // 当前线程一直监听
        while(true) {
            // 获取信息
            ConsumerRecords<String, String> consumerRecords = consumer.poll(Duration.ofMillis(1000));
            for (ConsumerRecord<String, String> record : consumerRecords) {
                System.out.println("message-key:"+record.key() + ", message-value:" + record.value());
            }
        }
    }
}
```



### 2)go

#### kafka-go

```
# 安装kafka-go包
go get github.com/segmentio/kafka-go
```

`kafka-go` 提供了两套与Kafka交互的API。

- 低级别（ low-level）：基于与 Kafka 服务器的原始网络连接实现。
- 高级别（high-level）：对于常用读写操作封装了一套更易用的API。

通常建议直接使用高级别的交互API。

生产者：

```go
func WriteByWriter() {
	w := &kafka.Writer{
		Addr:                   kafka.TCP("192.168.30.100:9092"),
		Topic:                  "yrings-topic",
		Balancer:               &kafka.LeastBytes{}, //指定分区的balancer模式为最小字节分布
		RequiredAcks:           kafka.RequireAll,    //ack 模式
		Async:                  true,                //异步
		AllowAutoTopicCreation: true,                // 自动创建topic
	}
	err := w.WriteMessages(context.Background(),
		kafka.Message{
			Key:   []byte("Key-A"),
			Value: []byte("hello world! A"),
		},
		kafka.Message{
			Key:   []byte("Key-B"),
			Value: []byte("hello world! B"),
		},
		kafka.Message{
			Key:   []byte("Key-C"),
			Value: []byte("hello world! C"),
		},
	)
	if err != nil {
		log.Fatal("failed to write message:", err)
	}
	if err := w.Close(); err != nil {
		log.Fatal("failed to close writer:", err)
	}
}
```

消费者：

```go
func ReadByReaderGroup() {
	r := kafka.NewReader(kafka.ReaderConfig{
		Brokers:   []string{"192.168.30.100:9092"},
		GroupID:   "group2",
		Topic:     "yrings-topic",
		Partition: 0,
		MaxBytes:  10e6, // 10MB
	})
	for {
		m, err := r.ReadMessage(context.Background())
		if err != nil {
			break
		}
		fmt.Printf("partition:%d message at offset %d: %s=%s\n", m.Partition, m.Offset, string(m.Key), string(m.Value))
	}
	// 程序退出前关闭Reader
	if err := r.Close(); err != nil {
		log.Fatal("failed to close reader:", err)
	}
}
```





## kafka高可用设计

### 1)集群

![image-20210530223101568](F:\study_project\黑马头条\day6\讲义\kafka及异步通知文章上下架.assets\image-20210530223101568.png)

- Kafka 的服务器端由被称为 Broker 的服务进程构成，即一个 Kafka 集群由多个 Broker 组成

- 这样如果集群中某一台机器宕机，其他机器上的 Broker 也依然能够对外提供服务。这其实就是 Kafka 提供高可用的手段之一

### 2)备份机制(Replication）

![image-20210530223218580](F:\study_project\黑马头条\day6\讲义\kafka及异步通知文章上下架.assets\image-20210530223218580.png)

Kafka 中消息的备份又叫做 副本（Replica）

Kafka 定义了两类副本：

- 领导者副本（Leader Replica）

- 追随者副本（Follower Replica）

**同步方式**

![image-20210530223316815](F:\study_project\黑马头条\day6\讲义\kafka及异步通知文章上下架.assets\image-20210530223316815.png)

ISR（in-sync replica）需要同步复制保存的follower



如果leader失效后，需要选出新的leader，选举的原则如下：

第一：选举时优先从ISR中选定，因为这个列表中follower的数据是与leader同步的

第二：如果ISR列表中的follower都不行了，就只能从其他follower中选取



极端情况，就是所有副本都失效了，这时有两种方案

第一：等待ISR中的一个活过来，选为Leader，数据可靠，但活过来的时间不确定

第二：选择第一个活过来的Replication，不一定是ISR中的，选为leader，以最快速度恢复可用性，但数据不一定完整

## kafka生产者详解 

### 1)发送类型

- 同步发送

  使用send()方法发送，它会返回一个Future对象，调用get()方法进行等待，就可以知道消息是否发送成功

```java
RecordMetadata recordMetadata = producer.send(kvProducerRecord).get();
System.out.println(recordMetadata.offset());
```

- 异步发送

  调用send()方法，并指定一个回调函数，服务器在返回响应时调用函数

```java
//异步消息发送
producer.send(kvProducerRecord, new Callback() {
    @Override
    public void onCompletion(RecordMetadata recordMetadata, Exception e) {
        if(e != null){
            System.out.println("记录异常信息到日志表中");
        }
        System.out.println(recordMetadata.offset());
    }
});
```

### 2)参数详解

- ack

![image-20210530224302935](F:\study_project\黑马头条\day6\讲义\kafka及异步通知文章上下架.assets\image-20210530224302935.png)

代码的配置方式：

```java
//ack配置  消息确认机制
prop.put(ProducerConfig.ACKS_CONFIG,"all");
```

参数的选择说明

| **确认机制**     | **说明**                                                     |
| ---------------- | ------------------------------------------------------------ |
| acks=0           | 生产者在成功写入消息之前不会等待任何来自服务器的响应,消息有丢失的风险，但是速度最快 |
| acks=1（默认值） | 只要集群首领节点收到消息，生产者就会收到一个来自服务器的成功响应 |
| acks=all         | 只有当所有参与赋值的节点全部收到消息时，生产者才会收到一个来自服务器的成功响应 |

- retries

![image-20210530224406689](F:\study_project\黑马头条\day6\讲义\kafka及异步通知文章上下架.assets\image-20210530224406689.png)

生产者从服务器收到的错误有可能是临时性错误，在这种情况下，retries参数的值决定了生产者可以重发消息的次数，如果达到这个次数，生产者会放弃重试返回错误，默认情况下，生产者会在每次重试之间等待100ms

代码中配置方式：

```java
//重试次数
prop.put(ProducerConfig.RETRIES_CONFIG,10);
```

- 消息压缩

默认情况下， 消息发送时不会被压缩。

代码中配置方式：

```java
//数据压缩
prop.put(ProducerConfig.COMPRESSION_TYPE_CONFIG,"lz4");
```

| **压缩算法** | **说明**                                                     |
| ------------ | ------------------------------------------------------------ |
| snappy       | 占用较少的  CPU，  却能提供较好的性能和相当可观的压缩比， 如果看重性能和网络带宽，建议采用 |
| lz4          | 占用较少的 CPU， 压缩和解压缩速度较快，压缩比也很客观        |
| gzip         | 占用较多的  CPU，但会提供更高的压缩比，网络带宽有限，可以使用这种算法 |

使用压缩可以降低网络传输开销和存储开销，而这往往是向 Kafka 发送消息的瓶颈所在。

## kafka消费者详解

### 1)消费者组

![image-20210530224706747](F:\study_project\黑马头条\day6\讲义\kafka及异步通知文章上下架.assets\image-20210530224706747.png)

- 消费者组（Consumer Group） ：指的就是由一个或多个消费者组成的群体

- 一个发布在Topic上消息被分发给此消费者组中的一个消费者

  - 所有的消费者都在一个组中，那么这就变成了queue模型

  - 所有的消费者都在不同的组中，那么就完全变成了发布-订阅模型

### 2)消息有序性

应用场景：

- 即时消息中的单对单聊天和群聊，保证发送方消息发送顺序与接收方的顺序一致

- 充值转账两个渠道在同一个时间进行余额变更，短信通知必须要有顺序

![image-20210530224903891](F:\study_project\黑马头条\day6\讲义\kafka及异步通知文章上下架.assets\image-20210530224903891.png)

topic分区中消息只能由消费者组中的唯一一个消费者处理，所以消息肯定是按照先后顺序进行处理的。但是它也仅仅是保证Topic的一个分区顺序处理，不能保证跨分区的消息先后处理顺序。 所以，如果你想要顺序的处理Topic的所有消息，那就只提供一个分区。

### 3)提交和偏移量

kafka不会像其他JMS队列那样需要得到消费者的确认，消费者可以使用kafka来追踪消息在分区的位置（偏移量）

消费者会往一个叫做**_consumer_offset**的特殊主题发送消息，消息里包含了每个分区的偏移量。如果消费者发生崩溃或有新的消费者加入群组，就会触发再均衡

![image-20210530225021266](F:\study_project\黑马头条\day6\讲义\kafka及异步通知文章上下架.assets\image-20210530225021266.png)

正常的情况

![image-20210530224959350](F:\study_project\黑马头条\day6\讲义\kafka及异步通知文章上下架.assets\image-20210530224959350.png)

如果消费者2挂掉以后，会发生再均衡，消费者2负责的分区会被其他消费者进行消费

再均衡后不可避免会出现一些问题

问题一：

![image-20210530225215337](F:\study_project\黑马头条\day6\讲义\kafka及异步通知文章上下架.assets\image-20210530225215337.png)

如果提交偏移量小于客户端处理的最后一个消息的偏移量，那么处于两个偏移量之间的消息就会被重复处理。



问题二：

![image-20210530225239897](F:\study_project\黑马头条\day6\讲义\kafka及异步通知文章上下架.assets\image-20210530225239897.png)

如果提交的偏移量大于客户端的最后一个消息的偏移量，那么处于两个偏移量之间的消息将会丢失。



如果想要解决这些问题，还要知道目前kafka提交偏移量的方式：

提交偏移量的方式有两种，分别是自动提交偏移量和手动提交

- 自动提交偏移量

当enable.auto.commit被设置为true，提交方式就是让消费者自动提交偏移量，每隔5秒消费者会自动把从poll()方法接收的最大偏移量提交上去

- 手动提交 ，当enable.auto.commit被设置为false可以有以下三种提交方式

  - 提交当前偏移量（同步提交）

  - 异步提交

  - 同步和异步组合提交



1.提交当前偏移量（同步提交）

把`enable.auto.commit`设置为false,让应用程序决定何时提交偏移量。使用commitSync()提交偏移量，commitSync()将会提交poll返回的最新的偏移量，所以在处理完所有记录后要确保调用了commitSync()方法。否则还是会有消息丢失的风险。

只要没有发生不可恢复的错误，commitSync()方法会一直尝试直至提交成功，如果提交失败也可以记录到错误日志里。

```java
while (true){
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
    for (ConsumerRecord<String, String> record : records) {
        System.out.println(record.value());
        System.out.println(record.key());
        try {
            consumer.commitSync();//同步提交当前最新的偏移量
        }catch (CommitFailedException e){
            System.out.println("记录提交失败的异常："+e);
        }

    }
}
```

2.异步提交

手动提交有一个缺点，那就是当发起提交调用时应用会阻塞。当然我们可以减少手动提交的频率，但这个会增加消息重复的概率（和自动提交一样）。另外一个解决办法是，使用异步提交的API。

```java
while (true){
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
    for (ConsumerRecord<String, String> record : records) {
        System.out.println(record.value());
        System.out.println(record.key());
    }
    consumer.commitAsync(new OffsetCommitCallback() {
        @Override
        public void onComplete(Map<TopicPartition, OffsetAndMetadata> map, Exception e) {
            if(e!=null){
                System.out.println("记录错误的提交偏移量："+ map+",异常信息"+e);
            }
        }
    });
}
```

3.同步和异步组合提交

异步提交也有个缺点，那就是如果服务器返回提交失败，异步提交不会进行重试。相比较起来，同步提交会进行重试直到成功或者最后抛出异常给应用。异步提交没有实现重试是因为，如果同时存在多个异步提交，进行重试可能会导致位移覆盖。

举个例子，假如我们发起了一个异步提交commitA，此时的提交位移为2000，随后又发起了一个异步提交commitB且位移为3000；commitA提交失败但commitB提交成功，此时commitA进行重试并成功的话，会将实际上将已经提交的位移从3000回滚到2000，导致消息重复消费。

```java
try {
    while (true){
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
        for (ConsumerRecord<String, String> record : records) {
            System.out.println(record.value());
            System.out.println(record.key());
        }
        consumer.commitAsync();
    }
}catch (Exception e){+
    e.printStackTrace();
    System.out.println("记录错误信息："+e);
}finally {
    try {
        consumer.commitSync();
    }finally {
        consumer.close();
    }
}
```

## kafka分区概念

> partition（分区）是kafka的一个核心概念，kafka将1个topic分成了一个或者多个分区，每个分区在物理上对应一个目录，分区目录下存储的是该分区的日志段(segment)，包括日志的数据文件和两个索引文件。然后每个分区又对应一个或多个副本，由一个ISR列表来维护

**一个分区只能由一个消费者组中的一个消费者来进行消费**，当一个topic只有



## springboot集成kafka

### 1)入门

1.导入spring-kafka依赖信息

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- kafkfa -->
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.apache.kafka</groupId>
                <artifactId>kafka-clients</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka-clients</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
    </dependency>
</dependencies>
```

2.在resources下创建文件application.yml

```yaml
server:
  port: 9991
spring:
  application:
    name: kafka-demo
  kafka:
    bootstrap-servers: 192.168.200.130:9092
    producer:
      retries: 10
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    consumer:
      group-id: ${spring.application.name}-test
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
```

3.消息生产者

```java
package com.heima.kafka.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @Autowired
    private KafkaTemplate<String,String> kafkaTemplate;

    @GetMapping("/hello")
    public String hello(){
        kafkaTemplate.send("itcast-topic","黑马程序员");
        return "ok";
    }
}
```

4.消息消费者

```java
package com.heima.kafka.listener;

import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;

@Component
public class HelloListener {

    @KafkaListener(topics = "itcast-topic")
    public void onMessage(String message){
        if(!StringUtils.isEmpty(message)){
            System.out.println(message);
        }

    }
}
```

### 2)传递消息为对象

目前springboot整合后的kafka，因为序列化器是StringSerializer，这个时候如果需要传递对象可以有两种方式

方式一：可以自定义序列化器，对象类型众多，这种方式通用性不强，本章节不介绍

方式二：可以把要传递的对象进行转json字符串，接收消息后再转为对象即可，本项目采用这种方式

- 发送消息

```java
@GetMapping("/hello")
public String hello(){
    User user = new User();
    user.setUsername("xiaowang");
    user.setAge(18);

    kafkaTemplate.send("user-topic", JSON.toJSONString(user));

    return "ok";
}
```

- 接收消息

```java
package com.heima.kafka.listener;

import com.alibaba.fastjson.JSON;
import com.heima.kafka.pojo.User;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;

@Component
public class HelloListener {

    @KafkaListener(topics = "user-topic")
    public void onMessage(String message){
        if(!StringUtils.isEmpty(message)){
            User user = JSON.parseObject(message, User.class);
            System.out.println(user);
        }

    }
}
```

### 

## Go集成kafka

首先是**[IBM/saram](https://link.zhihu.com/?target=https%3A//github.com/IBM/sarama)**（这个库已经由Shopify转给了IBM），之前我写过一篇使用**[sarama操作Kafka](https://link.zhihu.com/?target=https%3A//www.liwenzhou.com/posts/Go/kafka)**的教程，相较于sarama， **[kafka-go](https://link.zhihu.com/?target=https%3A//github.com/segmentio/kafka-go)** 更简单、更易用。

**[segmentio/kafka-go](https://link.zhihu.com/?target=https%3A//github.com/segmentio/kafka-go)** 是纯Go实现，提供了与kafka交互的低级别和高级别两套API，同时也支持Context。

此外社区中另一个比较常用的**[confluentinc/confluent-kafka-go](https://link.zhihu.com/?target=https%3A//github.com/confluentinc/confluent-kafka-go)**，它是一个基于cgo的**[librdkafka](https://link.zhihu.com/?target=https%3A//github.com/edenhill/librdkafka)**包装，在项目中使用它会引入对C库的依赖。

### **管理提交间隔**

### **显式提交**

`kafka-go` 也支持显式提交。当需要显式提交时不要调用 `ReadMessage`，而是调用 `FetchMessage`获取消息，然后调用 `CommitMessages` 显式提交。

```go
ctx := context.Background()
for {
    // 获取消息
    m, err := r.FetchMessage(ctx)
    if err != nil {
        break
    }
    // 处理消息
    fmt.Printf("message at topic/partition/offset %v/%v/%v: %s = %s\n", m.Topic, m.Partition, m.Offset, string(m.Key), string(m.Value))
    // 显式提交
    if err := r.CommitMessages(ctx, m); err != nil {
        log.Fatal("failed to commit messages:", err)
    }
}
```

在消费者组中提交消息时，具有给定主题/分区的最大偏移量的消息确定该分区的提交偏移量的值。例如，如果通过调用 `FetchMessage` 获取了单个分区的偏移量为 1、2 和 3 的消息，则使用偏移量为3的消息调用 `CommitMessages` 也将导致该分区的偏移量为 1 和 2 的消息被提交。

默认情况下，调用`CommitMessages`将同步向Kafka提交偏移量。为了提高性能，可以在ReaderConfig中设置CommitInterval来定期向Kafka提交偏移。

```text
// 创建一个reader从 topic-A 消费消息
r := kafka.NewReader(kafka.ReaderConfig{
    Brokers:        []string{"localhost:9092", "localhost:9093", "localhost:9094"},
    GroupID:        "consumer-group-id",
    Topic:          "topic-A",
    MaxBytes:       10e6, // 10MB
    CommitInterval: time.Second, // 每秒刷新一次提交给 Kafka
})
```

