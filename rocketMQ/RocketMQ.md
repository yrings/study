# RocketMQ

## 简介

> RocketMQ是阿里巴巴2016年MQ中间件，使用Java语言开发，RocketMQ 是一款开源的**分布式消息系统**，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。同时，广泛应用于多个领域，包括异步通信解耦、企业解决方案、金融支付、电信、电子商务、快递物流、广告营销、社交、即时通信、移动应用、手游、视频、物联网、车联网等。
>
> 具有以下特点：
>
> \1.  能够保证严格的消息顺序
>
> \2.  提供丰富的消息拉取模式
>
> \3.  高效的订阅者水平扩展能力
>
> \4.  实时的消息订阅机制
>
> \5.  亿级消息堆积能力

* ### MQ的作用

  **异步：**

  **削峰限流：**

  例如，在一个高并发的场景下，有很多的请求要访问一个controller，也就是访问服务器，这时如果每秒有1w请求进来，但是tomcat的线程池只有500个线程处理能力，无法被处理的请求就会报错503（服务不可用）。

  这时就可以用到MQ，然后创建一个新的控制器，作为类似“前台服务员”，用来“接待”请求，将请求放入到MQ中去，然后再来一个控制器去MQ获取请求信息，然后处理请求返回处理结果到MQ中，最后再又“前台”去返回结果。

  **解耦合：**



* ### MQ定义

  是指利用**高效可靠的消息传递机制进行与平台无关（跨平台）的数据交流**，并基于数据通信来进行分布式系统的集成。

  **跨平台：MQ能够用不同的语言来接收和处理，比如用java语言接收请求，用golang语言来处理请求。**

  大致流程

  发送者把消息发给消息服务器[MQ]，消息服务器把消息存放在若干**队列**/**主题**中，在合适的时候，消息服务器会把消息转发给接受者。在这个过程中，发送和接受是异步的,也就是发送无需等待，发送者和接受者的生命周期也没有必然关系在发布pub/订阅sub模式下，也可以完成一对多的通信，可以让一个消息有多个接受者[微信订阅号就是这样的]

  

  在四种MQ软件中**（ActiveMQ、RabbitMQ、RocketMQ、kafka）**，吞吐量最大的是**kafka**和**RocketMQ**，都是10万级别的；时效性最好的是**RabbitMQ**，是us级别的

  

  ### 各种MQ产品

  ![](image\屏幕截图 2024-01-22 202415.png)

  

* ### RocketMQ的结构组成

  **Producer**：消息的发送者，生产者；举例：发件人

  **Consumer**：消息接收者，消费者；举例：收件人

  **Broker**：暂存和传输消息的通道；举例：快递

  **NameServer**：管理Broker；举例：各个快递公司的管理机构 **相当于broker**的注册中心，保留了**broker**的信息

  **Queue**：队列，消息存放的位置，一个**Broker**中可以有多个队列

  **Topic**：主题，消息的分类

  **ProducerGroup**：生产者组 

  **ConsumerGroup**：消费者组，多个消费者组可以同时消费一个主题的消息

  **消息发送的流程是，Producer**询问**NameServer**，**NameServer**分配一个broker **然后Consumer**也要询问**NameServer**，得到一个具体的**broker**，然后消费消息

  

  > 是如何实现高可用的集群？

  Broker中有很多主题，主题里面又有很多的队列

  利用到了**NameServer**，它是一个路由注册中心，所有是broker都要将自己的注册信息例如端口号、ip地址来存入**NameServer**里面。然后发送者和接收者都是先要与它连接，然后分配到可用的broker。这是**写的高可用（负载均衡）**一部分。

  **读的高可用**是利用主从机实现的。
  
  读写高可用类似Redis主从集合？
  
  

## 使用

* **下载二进制文件，然后上传到服务器或虚拟机中**

* **使用linux命令创建和解压：**

  ```
  mkdir roketmq
  mv rocketmq-all-4.9.2-bin-release.zip ./roketmq
  yum -y install unzip
  unzip rocketmq-all-4.9.2-bin-release.zip
  ```

* **进入到解压目录**

  Benchmark：包含一些性能测试的脚本；

  Bin：可执行文件目录；

  Conf：配置文件目录；

  Lib：第三方依赖；

  LICENSE：授权信息;

  NOTICE：版本公告；

* **配置环境变量**

   进入到环境变量文件

  ```
  vim /etc/profile
  ```

  在文件末尾添加

  ```
  export NAMESRV_ADDR=阿里云公网 IP:9876
  ```

  刷新环境变量(但是只能在当前终端有效)

  ```
  source /etc/profile
  ```

* **修改nameServer的运行脚本的配置参数**

  进入bin目录下，修改`runserver.sh`文件，将71行和76行的Xmx 和 Xmx等比例改小一些。

  ![](image\屏幕截图 2023-12-29 150810.png)

* **修改broker的运行脚本**

  进入bin目录下，修改`runbroker.sh`文件,修改67行

* **修改broker的配置文件**

  进入conf目录下，修改`broker.conf`文件

  ```
  brokerClusterName = DefaultCluster
  brokerName = broker-a
  brokerId = 0
  deleteWhen = 04
  fileReservedTime = 48
  brokerRole = ASYNC_MASTER
  flushDiskType = ASYNC_FLUSH
  
  namesrvAddr=localhost:9876
  autoCreateTopicEnable=true
  brokerIP1=阿里云公网IP
  ```

  **添加参数解释**

  **namesrvAddr：nameSrv地址**可以写localhost因为nameSrv和broker在一个服务器

  **autoCreateTopicEnable**：自动创建主题，不然需要手动创建出来

  **brokerIP1**：broker也需要一个公网ip，如果不指定，那么是阿里云的内网地址，我们再本地无法连接使用

* **启动**

  后台启动nameSrv,然后日子输出

  ```
  nohup sh bin/mqnamesrv > ../logs/namesrv.log &
  ```

  后台启动broker

  ```
  nohup sh bin/mqbroker -c conf/broker.conf > ./logs/broker.log &
  ```

  查看java进程，第二种只要安装了JDK就能使用

  ```
  ps -ef |grep java
  jps -l
  ```

* **启动可视化工具**

  Rocketmq 控制台可以可视化MQ的消息发送！

  旧版本源码是在rocketmq-external里的rocketmq-console，新版本已经单独拆分成dashboard

  运行

  ```
  nohup java -jar ./rocketmq-dashboard-1.0.0.jar --server.prot=8001 --rocketmq.config.namesrvAddr=127.0.0.1:9876 > ./rocketmq-4.9.3/logs/dashboard.log &
  ```

  命令拓展:--server.port指定运行的端口

  --rocketmq.config.namesrvAddr=127.0.0.1:9876 指定namesrv地址

  然后访问`localhost:8001`

  > 如何看成linux系统的内存够不够，空闲内存？
  >
  > 以m为单位显示内存使用情况
  >
  > ```
  > free -m
  > ```



## 快速入门

> RocketMQ提供了发送多种发送消息的模式，例如同步消息，异步消息，顺序消息，延迟消息，事务消息等，我们一一学习

我们先搞清楚消息发送和监听的流程，然后我们在开始敲代码

* ### 消息生产者

1. 创建消息生产者producer，并制定生产者组名  

2. 指定Nameserver地址  
3. 启动producer  
4. 创建消息对象，指定主题Topic、Tag和消息体等  
5. 发送消息  
6. 关闭生产者producer  

* ### 消息消费者

1. 创建消费者consumer，制定消费者组名  
2. 指定Nameserver地址 
3. 创建监听订阅主题Topic和Tag等 
4. 处理消息  
5. 启动消费者consumer  



* ### 使用原生的api

  #### 导入依赖

  ```xml
  	<dependency>
          <groupId>org.apache.rocketmq</groupId>
          <artifactId>rocketmq-client</artifactId>
          <version>4.9.2</version>
      </dependency>
  ```

  #### 测试生产者

  ```java
  /**
   * 测试生产者
   *
   * @throws Exception
   */
  @Test
  public void testProducer() throws Exception {
      // 创建默认的生产者(制定一个组名)
      DefaultMQProducer producer = new DefaultMQProducer("test-producer-group");
      // 设置nameServer地址
      producer.setNamesrvAddr("localhost:9876");
      // 启动实例
      producer.start();
      for (int i = 0; i < 10; i++) {
          // 创建消息
          // 第一个参数：主题的名字
          // 第二个参数：消息内容
          Message msg = new Message("TopicTest", ("Hello RocketMQ " + i).getBytes());
          SendResult send = producer.send(msg);
          System.out.println(send.getSendStatus);
      }
      // 关闭实例
      producer.shutdown();
  }
  --------------测试结果-----------
  SEND_OK
  ```

  #### 测试消费者

  ```java
     /**
       * 测试消费者
       *
       * @throws Exception
       */
      @Test
      public void testConsumer() throws Exception {
          // 创建默认消费者组
          DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("test-consumer-group");
          // 设置nameServer地址
          consumer.setNamesrvAddr("localhost:9876");
          // 订阅一个主题来消费   *表示没有过滤参数 表示这个主题的任何消息
          consumer.subscribe("TopicTest", "*");
          // 注册一个消费监听 MessageListenerConcurrently 是多线程消费，默认20个线程，可以参看consumer.setConsumeThreadMax()
          consumer.registerMessageListener(new MessageListenerConcurrently() {
              @Override
              public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
                                                              ConsumeConcurrentlyContext context) {
                  System.out.println(Thread.currentThread().getName() + "----" + msgs);
                  // 返回消费的状态 如果是CONSUME_SUCCESS 则成功，若为RECONSUME_LATER则该条消息会被重回队列，重新被投递
                  // 重试的时间为messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
                  // 也就是第一次1s 第二次5s 第三次10s  ....  如果重试了18次 那么这个消息就会被终止发送给消费者
  //                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
                  return ConsumeConcurrentlyStatus.RECONSUME_LATER;
              }
          });
          // 这个start一定要写在registerMessageListener下面
          consumer.start();
          // 挂起当前的jvm，让监听一直执行
          System.in.read();
      }
  
  ```

  #### 问题

  > 为什么创建默认的消费者和生产者都要指定一个组名？

  对于生产者组没有过多的要求，但是**消费者内的消费者订阅关系必须保持一致。**对于不同组的消费者订阅的同一主题会对每个组都发一份消息（即一个组一份）；然后对于同组组内的消费者，消息处理策略有两种**一是负载均衡模式，二是广播模式**。

  > 一个主题默认会有四个队列，对于负载均衡模式，消费者是怎么样选择的呢？

  生产者发送消息的策略是**轮询策略**。

  **对于一个消费者组内的消费者它们与队列是有对应关系的**。如果有两个消费者然后有四个队列，则每个消费者都对应处理固定的两个队列的消息，如果增加消费者，则就会进行 `reBalance`重平衡。如果有五个消费者，则会有一个消费者永远得不到消息。

  所以 `队列的数量 >= 消费者组的消费者数量` 

### 消费进度管理

> RocketMQ 的消费进度管理。

在服务端，定义队列中最早的一条消息位点为最小消息位点（MinOffset）；最新的一条消息的位点为最大消息位点（MaxOffset）。队列理论上是无限的，由于服务端物理节点存储空间有限，会滚动删除队列中存储最早的消息。所有这两个位点会一直递增变化。

因为有很多的消费者分组，所以如果当一个分组消费完最早消息后就删除消息，就会导致其它分组就消费不到该消息。

所以在服务端会**基于各个分组的消费情况，维护一个位点，也就是最近消费的位点，消费位点**。如果该记录被删除，就去读取服务端的`MinOffset`位点服务。



## 消费模式

MQ的消费模式可以大致分为两种，一种是推Push，一种是拉Pull。

Push是服务端【MQ】主动推送消息给客户端，优点是及时性较好，但如果客户端没有做好流控，一旦服务端推送大量消息到客户端时，就会导致客户端消息堆积甚至崩溃。

Pull是客户端需要主动到服务端取数据，优点是客户端可以依据自己的消费能力进行消费，但拉取的频率也需要用户自己控制，拉取频繁容易造成服务端和客户端的压力，拉取间隔长又容易造成消费不及时。

**Push模式也是基于pull模式的，只能客户端内部封装了api，一般场景下，上游消息生产量小或者均速的时候，选择push模式。在特殊场景下，例如电商大促，抢优惠券等场景可以选择pull模式**



## 消息传输模式

* ### RocketMQ 发送同步消息

  上面的快速入门就是发送同步消息，发送过后会有一个返回值，也就是mq服务器接收到消息后返回的一个确认，这种方式非常安全，但是性能上并没有这么高，而且在mq集群中，也是要等到所有的从机都复制了消息以后才会返回，所以针对重要的消息可以选择这种方式

* ### RocketMQ 发送异步消息

  > 会以异步的回调方式将发送结果返回

  生产者：

  ```java
  @Test
  public void testAsyncProducer() throws Exception {
      // 创建默认的生产者
      DefaultMQProducer producer = new DefaultMQProducer("test-group");
      // 设置nameServer地址
      producer.setNamesrvAddr("localhost:9876");
      // 启动实例
      producer.start();
      Message msg = new Message("TopicTest", ("异步消息").getBytes());
      producer.send(msg, new SendCallback() {
          @Override
          public void onSuccess(SendResult sendResult) {
              System.out.println("发送成功");
          }
          @Override
          public void onException(Throwable e) {
              System.out.println("发送失败");
          }
      });
      System.out.println("看看谁先执行");
      // 挂起jvm 因为回调是异步的不然测试不出来
      System.in.read();
      // 关闭实例
      producer.shutdown();
  }
  ```

* ### RocketMQ发送单向消息

  > 这种方式主要用在不关心发送结果的场景，这种方式吞吐量很大，但是存在消息丢失的风险，例如日志信息的发送。
  >
  > 对于日志的处理：
  >
  > 在开发环境下，日志输出在控制台中
  >
  > 在生产环境下，日志输出在文件；但是输出在文件里不好排除，所以可输出在数据库中，例如noSQL的ES（elasticsearch）。我们可以搭建一个日志服务器，然后通过MQ来进行信息传递。
  >
  > 还要进行冷热分离，将近一个月的数据放到固态硬盘（SSD），将冷数据放入到机械硬盘（HDD）

  生产者

  ```java
  @Test
  public void testOnewayProducer() throws Exception {
      // 创建默认的生产者
      DefaultMQProducer producer = new DefaultMQProducer("test-group");
      // 设置nameServer地址
      producer.setNamesrvAddr("localhost:9876");
      // 启动实例
      producer.start();
      Message msg = new Message("TopicTest", ("单向消息").getBytes());
      // 发送单向消息
      producer.sendOneway(msg);
      // 关闭实例
      producer.shutdown();
  }
  ```

* ### RocketMQ 发送延迟消息

  > 消息放入MQ后，过一段时间才会被监听到，然后消费
  >
  > 比如下订单业务，提交了一个订单就可以发送一个延时消息，30min后去检查这个订单的状态，如果还是未付款就取消订单释放库存。

  生产者：

  ```java
  @Test
  public void testDelayProducer() throws Exception {
      // 创建默认的生产者
      DefaultMQProducer producer = new DefaultMQProducer("test-group");
      // 设置nameServer地址
      producer.setNamesrvAddr("localhost:9876");
      // 启动实例
      producer.start();
      Message msg = new Message("TopicTest", ("延迟消息").getBytes());
      // 给这个消息设定一个延迟等级
      // messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
      msg.setDelayTimeLevel(3);
      // 发送单向消息
      producer.send(msg);
      // 打印时间
      System.out.println(new Date());
      // 关闭实例
      producer.shutdown();
  }
  ```

  这里注意的是RocketMQ不支持任意时间的延时

  只支持以下几个固定的延时等级，等级1就对应1s，以此类推，最高支持2h延迟

  `private String messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h";`但是可以去到配置文件中配置。

* ### RocketMQ 发送批量消息

  > Rocketmq可以一次性发送一组消息，那么这一组消息会被当做一个消息消费

  ```java
  @Test
  public void testBatchProducer() throws Exception {
      // 创建默认的生产者
      DefaultMQProducer producer = new DefaultMQProducer("test-group");
      // 设置nameServer地址
      producer.setNamesrvAddr("localhost:9876");
      // 启动实例
      producer.start();
      List<Message> msgs = Arrays.asList(
              new Message("TopicTest", "我是一组消息的A消息".getBytes()),
              new Message("TopicTest", "我是一组消息的B消息".getBytes()),
              new Message("TopicTest", "我是一组消息的C消息".getBytes())
  
      );
      SendResult send = producer.send(msgs);
      System.out.println(send);
      // 关闭实例
      producer.shutdown();
  }
  ```

* ### RocketMQ 发送顺序消息

  > 问题引入：由于在一个主题中默认会有四个队列，也就是多个队列；又消费者消费模式默认是并发的，就是不同线程处理不同队列中的数据，这样就没法保证消息的顺序处理，会出现一些问题。
  >
  > 处理办法是：让需要排序的消息强制放到同一队列中去，保证局部有序。然后让消费者运用单线程模式消费消息。

  生产者：(省略了一个订单类的定义)

  ```java
  @Test
  public void testOrderlyProducer() throws Exception {
      // 创建默认的生产者
      DefaultMQProducer producer = new DefaultMQProducer("test-group");
      // 设置nameServer地址
      producer.setNamesrvAddr("localhost:9876");
      // 启动实例
      producer.start();
      List<Order> orderList = Arrays.asList(
              new Order(1, 111, 59D, new Date(), "下订单"),
              new Order(2, 111, 59D, new Date(), "物流"),
              new Order(3, 111, 59D, new Date(), "签收"),
              new Order(4, 112, 89D, new Date(), "下订单"),
              new Order(5, 112, 89D, new Date(), "物流"),
              new Order(6, 112, 89D, new Date(), "拒收")
      );
      // 循环集合开始发送
      orderList.forEach(order -> {
          Message message = new Message("TopicTest", order.toString().getBytes());
          try {
              // 发送的时候 相同的订单号选择同一个队列
              producer.send(message, new MessageQueueSelector() {
                  @Override
                  public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {
                      // 当前主题有多少个队列
                      int queueNumber = mqs.size();
                      // 这个arg就是后面传入的 order.getOrderNumber()
                      Integer i = (Integer) arg;
                      // 用这个值去%队列的个数得到一个队列
                      int index = i % queueNumber;
                      // 返回选择的这个队列即可 ，那么相同的订单号 就会被放在相同的队列里 实现FIFO了
                      return mqs.get(index);
                  }
              }, order.getOrderNumber());
          } catch (Exception e) {
              System.out.println("发送异常");
          }
      });
      // 关闭实例
      producer.shutdown();
  }
  ```

  消费者：

  ```java
  @Test
  public void testOrderlyConsumer() throws Exception {
      // 创建默认消费者组
      DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer-group");
      // 设置nameServer地址
      consumer.setNamesrvAddr("localhost:9876");
      // 订阅一个主题来消费   *表示没有过滤参数 表示这个主题的任何消息
      consumer.subscribe("TopicTest", "*");
      // 注册一个消费监听 MessageListenerOrderly 是顺序消费 单线程消费
      consumer.registerMessageListener(new MessageListenerOrderly() {
          @Override
          public ConsumeOrderlyStatus consumeMessage(List<MessageExt> msgs, ConsumeOrderlyContext context) {
              MessageExt messageExt = msgs.get(0);
              System.out.println(new String(messageExt.getBody()));
              return ConsumeOrderlyStatus.SUCCESS;
          }
      });
      consumer.start();
      System.in.read();
  }
  ```

  `ConsumerOrderlyStatus.SUSPEND_CURRENT_QUEUE_A_MOMENT`；

  这个状态码表示将这个消息所在的队列挂起，然后等一段时间再次消费，不会消费这个队列后面的消息。

  > 消费失败后的处理？

  对于并发模式，多线程的消费，它会重试16次；超过16次就会把它放到别的队列中去,即死信队列。

  对于顺序模式，单线程的，它会无限重试Integer.Max_Value。

  顺序消息消费失败后，会自动进行消息重试，不进入重试队列。会造成消费阻塞，因此使用顺序消息的时候要及时监控消费失败的情况，防止发生阻塞。



## RocketMQ发送带标签的消息，消息过滤

> Rocketmq提供消息过滤功能，通过tag或者key进行区分
>
> 我们往一个主题里面发送消息的时候，根据业务逻辑，可能需要区分，比如带有tagA标签的被A消费，带有tagB标签的被B消费，还有在事务监听的类里面，只要是事务消息都要走同一个监听，我们也需要通过过滤才区别对待

生产者：

```java
 Message msg = new Message("TopicTest","tagA", "我是一个带标记的消息".getBytes());
```

消费者：

```java
// 订阅一个主题来消费   表达式，默认是*,支持"tagA || tagB || tagC" 这样或者的写法 只要是符合任何一个标签都可以消费
consumer.subscribe("TopicTest", "tagA || tagB || tagC");
```



## RocketMQ 中消息的 Key

> 在rocketmq中的消息，默认会有一个messageId当做消息的唯一标识，我们也可以给消息携带一个key，用作唯一标识或者业务标识，包括在控制面板查询的时候也可以使用messageId或者key来进行查询

生产者：

这个`key`是业务参数，我们自身要确保唯一，之后为了查阅和去重

```java
Message msg = new Message("TopicTest","tagA","key", "我是一个带标记和key的消息".getBytes());
```



## ☆RocketMQ 消息重复消费问题（去重）

> 为什么会出现重复消费问题呢？
>
> 发送方需要给消息携带一个唯一标记（自己业务控制），消费者方需要控制信息的**幂等性**
>
> 什么是幂等性？
>
> 多次操作产生的影响均和第一次操作产生的影响相同。普通的新增操作是非幂等的，修改操作根据情况判别是不是幂等的；查询和删除一般是幂等操作。

* **第一种情况是消费者扩容（reBalance重平衡）**

在**CLUSTERING(负载均衡)** 模式下，同一个 `consumerGroup` 下，也有可能发生消息重复消费问题。由于当有新的消费者加入或者消费者被删除了，都会发生`ReBalance`重平衡,来从新负载均衡，重新分配队列。如果一个队列在被消费过程中发生了这种请求，并且消费者没有及时把位点消息提交给 `broker`。这时新分配到这个队列的消费者又会在消费一次最早消息。

**那么如果在CLUSTERING**（负载均衡）模式下，并且在同一个消费者组中，不希望一条消息被重复消费，改怎么办呢？我们可以想到去重操作，找到消息唯一的标识，可以是`msgId`也可以是你自定义的唯一的key，这样就可以去重了

* **第二种请求是生产者重复投递了**

* ### 解决方案

  #### 第一种解决方案

  使用去重方案解决，例如将消息的唯一标识存起来，然后每次消费之前先判断是否存在这个唯一标识，如果存在则不消费，如果不存在则消费，并且消费以后将这个标记保存。

  * > 当在一个表中添加非主键的唯一索引，如果有两个相同的插入语句被执行，就会发生`Duplicate entry 值 for key 唯一索引字段`；这种情况就是幂等操作。

    所以对于插入操作，可以不用在业务代码中对key进行存在判断，可以创建一个`去重表`，去重表将key作为唯一索引，然后进行插入操作，如果插入成功说明不存在，如果插入失败说明key已存在。

  #### 第二种解决方案

  想法很好，但是消息的体量是非常大的，可能在生产环境中会到达上千万甚至上亿条，那么我们该如何选择一个容器来保存所有消息的标识，并且又可以快速的判断是否存在呢？

  我们可以选择布隆过滤器(BloomFilter)
  
  **布隆过滤器（Bloom Filter**）是1970年由布隆提出的。它实际上是一个很长的二进制向量和一系列随机映射函数。布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都比一般的算法要好的多，缺点是有一定的误识别率和删除困难。
  
  **在hutool的工具中我们可以直接使用，当然你自己使用redis的bitmap类型手写一个也是可以的**
  
  依赖

  ```xml
  <dependency>
      <groupId>cn.hutool</groupId>
      <artifactId>hutool-all</artifactId>
      <version>5.7.11</version>
  </dependency>
  ```
  
  消费者：
  
  ```java
  /**
   * 在boot项目中可以使用@Bean在整个容器中放置一个单利对象
   */
  public static BitMapBloomFilter bloomFilter = new BitMapBloomFilter(100);
  
  @Test
  public void testRepeatConsumer() throws Exception {
      // 创建默认消费者组
      DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer-group");
      consumer.setMessageModel(MessageModel.BROADCASTING);
      // 设置nameServer地址
      consumer.setNamesrvAddr("localhost:9876");
      // 订阅一个主题来消费   表达式，默认是*
      consumer.subscribe("TopicTest", "*");
      // 注册一个消费监听 MessageListenerConcurrently是并发消费
      // 默认是20个线程一起消费，可以参看 consumer.setConsumeThreadMax()
      consumer.registerMessageListener(new MessageListenerConcurrently() {
          @Override
          public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
                                                          ConsumeConcurrentlyContext context) {
              // 拿到消息的key
              MessageExt messageExt = msgs.get(0);
              String keys = messageExt.getKeys();
              // 判断是否存在布隆过滤器中
              if (bloomFilter.contains(keys)) {
                  // 直接返回了 不往下处理业务
                  return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
              }
              // 这个处理业务，然后放入过滤器中
              // do sth...
              bloomFilter.add(keys);
              return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
          }
      });
      consumer.start();
      System.in.read();
  }
  ```
  
  #### 第三种解决方案
  
  > 用redis的(setnx)数据结构

## RocketMQ 重试机制

* ### 生产者重试

  ```java
  // 失败的情况重发3次
  
  `producer.setRetryTimesWhenSendFailed(3);
  
  // 消息在1S内没有发送成功，就会重试`
  
  producer.send(msg, 1000);
  ```

* ### 消费者重试

  > 关于消费者重试问题，它默认重试16次，即十六个级别。但是可以自定义设置重试次数。延迟等级可以在生产者中设置。

  在消费者放`return ConsumeConcurrentlyStatus.RECONSUME_LATER;`后就会执行重试

  上图代码中说明了，我们再实际生产过程中，一般重试3-5次，如果还没有消费成功，则可以把消息签收了，通知人工等处理

  ```java
  /**
   * 测试消费者
   *
   * @throws Exception
   */
  @Test
  public void testConsumer() throws Exception {
      // 创建默认消费者组
      DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer-group");
      // 设置nameServer地址
      consumer.setNamesrvAddr("localhost:9876");
      // 订阅一个主题来消费   *表示没有过滤参数 表示这个主题的任何消息
      consumer.subscribe("TopicTest", "*");
      // 注册一个消费监听
      consumer.registerMessageListener(new MessageListenerConcurrently() {
          @Override
          public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
                                                          ConsumeConcurrentlyContext context) {
              try {
                  // 这里执行消费的代码
                  System.out.println(Thread.currentThread().getName() + "----" + msgs);
                  // 这里制造一个错误
                  int i = 10 / 0;
              } catch (Exception e) {
                  // 出现问题 判断重试的次数
                  MessageExt messageExt = msgs.get(0);
                  // 获取重试的次数 失败一次消息中的失败次数会累加一次
                  int reconsumeTimes = messageExt.getReconsumeTimes();
                  if (reconsumeTimes >= 3) {
                      // 则把消息确认了，可以将这条消息记录到日志或者数据库 通知人工处理
                      return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
                  } else {
                      return ConsumeConcurrentlyStatus.RECONSUME_LATER;
                  }
              }
              return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
          }
      }
  ```



## RocketMQ 死信消息

> 消费失败后的处理？

如果是广播模式，就会直接将消息丢弃掉。

对于并发模式，多线程的消费，将消息返回Broker（修改topic为重试队列）然后放入延时队列,如果到达了调度时间，就会修改Topic和queueId，发送回到原始队列等待消费者拉取，它会重试16次。

超过16次就会把它放到别的主题中去，即**死信主题（Dead-Letter Message）**。

对于顺序模式，单线程的，它会无限重试Integer.Max_Value。

顺序消息消费失败后，会自动进行消息重试，不进入重试队列。会造成消费阻塞，因此使用顺序消息的时候要及时监控消费失败的情况，防止发生阻塞。

> 死信队列是死信Topic下分区数唯一的单独队列。如果产生了死信消息，那对应的`ConsumerGroup`的死信Topic名称为`%DLQ%ConsumerGroupName`，死信队列的消息将不会再被消费。可以利用RocketMQ Admin工具或者RocketMQ Dashboard上查询到对应死信消息的信息。我们也可以去监听死信队列，然后进行自己的业务上的逻辑





## SpringBoot 集成 RockeMQ

* ### 导入依赖

  ```java
  <!-- rocketmq的依赖 -->
          <dependency>
              <groupId>org.apache.rocketmq</groupId>
              <artifactId>rocketmq-spring-boot-starter</artifactId>
              <version>2.0.2</version>
          </dependency>
  ```

* ### 生产者

  #### 配置application.yaml文件

  ```yaml
  spring:
      application:
          name: rocketmq-producer
  rocketmq:
      name-server: 127.0.0.1:9876
      producer: 
      	group: boot-producer-group
      	max-message-size: ...(默认4MB)
  ```

  #### 使用方法

  ##### 注入操作模板类

  ```java
  @Autowired
  private RocketMQTemplate rocketMQTemplate;
  ```

* ### 消费者

  #### 配置application.yaml文件

  ```
  ```

  #### 使用方法

  ```java
  	 /**
     * 创建一个简单消息的监听
     * 1.类上添加注解@Component和@RocketMQMessageListener
     *      @RocketMQMessageListener(topic = "powernode", consumerGroup = "powernode-group")
     *      topic指定消费的主题，consumerGroup指定消费组,一个主题可以有多个消费者组,一个消息可以被多个不同的组的消费者都消费
     * 2.实现RocketMQListener接口，注意泛型的使用，可以为具体的类型，如果想拿到消息
     * 的其他参数可以写成MessageExt
     */
    @Component
    @RocketMQMessageListener(topic = "powernode", consumerGroup = "powernode-group",messageModel = MessageModel.CLUSTERING)
    public class SimpleMsgListener implements RocketMQListener<String> {
    
        /**
         * 消费消息的方法
         *
         * @param message
         */
        @Override
        public void onMessage(String message) {
            System.out.println(message);
        }
    }
  ```

  如果没有报错，就签收了，如果报错了，就是拒收了，就会重试。也不用挂起了。

* ### 发送延迟消息

  ```java
  Message<String> msg = MessageBuilder.withPayLoad("我是一个延迟消息").build();
  rocketMQTemplate.synSend("Topic",msg,3000,3);
  ```

* ### 发送带key的消息

  ```java
  /**
   * 发送一个带key的消息,我们使用事务消息 打断点查看消息头
   *
   * @throws Exception
   */
  @Test
  public void testKeyMsg() throws Exception {
      // 发送一个key为spring的事务消息
      Message<String> message = MessageBuilder.withPayload("我是一个带key的消息")
              .setHeader(RocketMQHeaders.KEYS, "spring")
              .build();
      rocketMQTemplate.sendMessageInTransaction("powernode", message, "我是一个带key的消息");
  }
  ```

  

* ### 发送带tag的消息

  ```java
  /**
   * 发送一个带tag的消息
   *
   * @throws Exception
   */
  @Test
  public void testTagMsg() throws Exception {
      // 发送一个tag为java的数据
      rocketMQTemplate.syncSend("powernode-tag:java", "我是一个带tag的消息");
  
  ```



## 什么情况下会出现堆积

\1. 生产太快了 

生产方可以做业务限流

增加消费者数量,但是消费者数量<=队列数量,适当的设置最大的消费线程数量(根据`IO(2n)/CPU(n+1))`

动态扩容队列数量,从而增加消费者数量

\2. 消费者消费出现问题

排查消费者程序的问题

## 如何确保消息不丢失？

\1. 生产者使用同步发送模式 ，收到mq的返回确认以后 顺便往自己的数据库里面写

`msgId status(0) time`

\2. 消费者消费以后 修改数据这条消息的状态 = 1

\3. 写一个定时任务 间隔两天去查询数据 如果有status = 0 and time < day-2

\4. 将mq的刷盘机制设置为同步刷盘

> MQ持久化消息分为两种：同步刷盘和异步刷盘。默认情况是异步刷盘，Broker收到消息后会先存到cache里然后立马通知Producer说消息我收到且存储成功了，你可以继续你的业务逻辑了，然后Broker起个线程异步的去持久化到磁盘中，但是Broker还没持久化到磁盘就宕机的话，消息就丢失了。同步刷盘的话是收到消息存到cache后并不会通知Producer说消息已经ok了，而是会等到持久化到磁盘中后才会通知Producer说消息完事了。这也保障了消息不会丢失，但是性能不如异步高。看业务场景取舍。

\5. 使用集群模式 ，搞主备模式，将消息持久化在不同的硬件上

\6. 可以开启mq的trace机制，消息跟踪机制

1.在`broker.conf`中开启消息追踪

`traceTopicEnable=true`

2.重启broker即可

3.生产者配置文件开启消息轨迹

`enable-msg-trace: true`

\1.  消费者开启消息轨迹功能，可以给单独的某一个消费者开启

`enableMsgTrace = true`

 

 

在rocketmq的面板中可以查看消息轨迹

默认会将消息轨迹的数据存在 `RMQ_SYS_TRACE_TOPIC` 主题里面

## 安全

\1. 开启acl的控制 在broker.conf中开启aclEnable=true

\2. 配置账号密码 修改plain_acl.yml 

\3. 修改控制面板的配置文件 放开52/53行 把49行改为true 上传到服务器的jar包平级目录下即可



## RocketMQ设计

* ### Broker 底层原理

  ![](image\屏幕截图 2023-12-31 182314.png)

  1. 当有生产者的消息进来时，它会把消息放到一个叫 **CommitLog** 容器中去，这个容器存储的是消息实体，并且将消息标记为两类一种是已经被写入到磁盘中的，一种是未被写入到磁盘中的。并且有不同的刷盘方式，一种是异步的，一种是同步的。
  2. 然后会根据消息的主题，将消息的信息、消息在 **CommitLog** 中的位置放入到队列中去，而不是消息全部信息。这个过程是异步的，异步分配逻辑地址

* ### 消息架构

  #### 消息索引文件

  消息索引文件，能够快速查找到消息，存储的结构是HashMap。





## 秒杀场景

> 秒杀的场景是在很短的时间内，要处理大量的请求。实现高并发。我们做秒杀就要尽量减少接口的处理时间。
>
> 并发：多个任务再同一时间段内执行
>
> 并行：多核cpu上，多个任务再同一时刻执行
>
> tomcat的默认最大线程数为200，利用`apache-jmeter` 压测工具进行压力测试。打开`bin\jmeter.bat`文件打开图形化界面。假设正常的tomcat的QPS在2000到4000左右。
>
> 但是在实际高并发场景中这远远不够，所以就用到了集群，用**nginx**进行负载均衡多个tomcat，但是假设nginx的QPS也只有5w/s左右，还是不够。这时就要用到了更高**硬件负载均衡（Lvs/F5）**来负载均衡nginx集群。
>
> 但是对于更高要求的QPS，例如100w/s，就可以分区域来分流量。原理就是用域名--DNS轮询策略。一个域名可以对应很多ip地址，然后根据请求的IP地址的区域访问不同的ip地址。

> 如何优化接口的响应时间？

流程：查询数据库->数据库循环处理->调用其它服务->操作缓存->操作数据库

1. 对操作异步进行，
2. 减少IO（统一查，统一写）
3. 能return就return
4. 加锁粒度尽可能小
5. 事务控制粒度尽可能小

实际场景真的要做秒杀，是需要前端配合进行分流的。例如添加一个滑块验证码，还能进行人机校验。

![](image\屏幕截图 2024-01-09 174208.png)

















