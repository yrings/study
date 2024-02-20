# SpringCloud

## 一、简介

> 微服务开发框架
>
> springcloud 为开发人员提供了在分布式系统中快速构建一些常见模式的工具（例如配置管理、服务发现、断路器、智能路由、微代理、控制总裁、一次性令牌、全局锁、领导选举、分布式会话、集群状态）。分布式系统的协调导致了样板模式（将大问题分解为上述的小问题），使用 springcloud 的开发人员可以快速地建立实现这些模式的服务和应用系统。它们在任何分布式环境下都能很好地工作，包括开发人员自己的笔记本电脑、裸机数据中心和云计算等托管平台。

* 环境版本：SpringBoot3.xxx、JDK20、Spring Cloud Alibaba 2022.0.0.0-RC2

Spring Cloud architecture hightlights（架构高光）：

![](images\屏幕截图 2023-10-23 192811.png)

* 与 Spring Boot 的关系

  ​	Spring Boot 为 Spring Cloud 提供了代码实现环境，使用Spring Boot 将其它组件有机融合到了 Spring Cloud 的体系架构中了。所以说，Spring Cloud 是基于Spring Boot 的、微服务系统架构的一站式解决方案。

  ​	所以说 Spring Cloud 就像一个粘合剂，将各种框架来融合起来。

### spring cloud alibaba

> spring Cloud Alibaba 致力于提供微服务开发的一站式解决方案。此项目包含开发分布式应用服务的必需组件，方便开发者通过 Spring Cloud 编程模型轻松使用这些组件来开发分布式应用服务。
>
> 依托 Spring Cloud Alibaba，您只需要添加一些注解和少量配置，就可以将 Spring Cloud 应用接入阿里分布式应用解决方案，通过阿里中间件来迅速搭建分布式应用系统。
>
> Alibaba 的很多开源组件、中间件、框架，在国内外很多公司中早就被使用，并且很多已经是经过了“双11”的考验与历练的。而Spring Cloud 的使用便捷性、各组件的完美结合性。深深吸引和打动了阿里，所以阿里团队研发了 Srping Cloud Alibaba。在18年10月阿里将其捐赠给了 Spring Cloud 并开始孵化。在19年8月 正式孵化毕业成为 Spring Cloud 中的重要成员，与Spring Cloud Netflix 并驾齐驱，并在国内市场迅速火爆。

​	

### 版本兼容关系（重要）

​	由于 Spring Boot 3.0，Spring Boot 2.7~2.4 和 2.4 以下版本之间变化较大，目前企业级客户老项目相关 Spring Boot 版本仍停留在 Spring Boot 2.4 以下，为了同时满足存量用户和新用户不同需求，社区以 Spring Boot 3.0 和 2.4 分别为分界线，同时维护 2022.x、2021.x、2.2.x 三个分支迭代。

>  https://github.com/alibaba/spring-cloud-alibaba。github官网

* Spring Boot 3.x 新变化

  **（1）JDK 与Spring 的要求**

  ​	`spring Boot3.x `版本要求JDK至少是17，Spring 6.0

  **（2）Jakarta 依赖**

  ​	`Spring Boot3.x `已经将所有底层依赖从` JavaEE` 迁移到了 `akartaEE`，是基于 `JakartaEE9`并尽可能地兼容 `JakartaEE10`.

  **（3）IDea版本**

  ​	`intelliJ IDEA `从2022.2版本开始完全支持`Spring Boot3.x`与Spring6.



* JDK 免费/收费问题

  从 2019 年开始，Oracle 官宣，某些版本开始收费。

  JDK8 之前版本就是免费。

  JDK17、JDK18、JDK19、JDK20,全版本免费的，但是是二进制版本。



* 2022.x 分支

  适配 Spring Boot 3.0，Spring Cloud 2022.x 版本及以上的 Spring Cloud Alibaba 版本按从新到旧排列如下表（最新版本用*标记）： *(注意，该分支 Spring Cloud Alibaba 版本命名方式进行了调整，未来将对应 Spring Cloud 版本，前三位为 Spring Cloud 版本，最后一位为扩展版本，比如适配 Spring Cloud 2022.0.0 版本对应的 Spring Cloud Alibaba 第一个版本为：2022.0.0.0，第个二版本为：2022.0.0.1，依此类推)*

  

### 测试环境搭建

> 本例实现了消费者对提供者的调用，但并未使用到 Spring Cloud Alibaba，而是使用的 Spring 提供的RestTemplate 实现，对于 MySQL数据库的访问，使用 Spring Data JPA 作为持久层技术。不过后续 Spring Cloud Alibaba 的运行测试环境就是字啊此基础修改出来的。

资源站点：`mvnrepository.com`

下面将用到较为简单JPA来代替mybatis完成示例。

* 创建生产者（provider）

导入依赖(druid连接池依赖):

```xml
<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.20</version>
        </dependency>
```



创建bean实体类，并用JPA来进行操作

```java
@Data
@Entity
@JsonIncludeProperties({"hibernatelazyInitializer", "handler", "fieldHandle"})
public class Depart {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) // ID自动递增
    private Integer id;

    private String name;
}
```

创建操作类:

会自动创建业务无关的方法，方便服务层操作数据库

```java
public interface DepartRepository extends JpaRepository<Depart, Integer> {

}
```

service层的接口，实现略：

```java
public interface DepartService {

    boolean saveDepart(Depart depart);

    boolean removeDepartById(int id);

    boolean modifyDepart(Depart depart);

    Depart getDepartById(int id);

    List<Depart> listAllDeparts();
}
```

controller层,restFUL风格实现：

```java
@RequestMapping("/depart")
@RestController
public class DepartController {

    @Autowired
    private DepartService service;

    @PostMapping("/")
    public boolean saveHandle(@RequestBody Depart depart){
        return service.saveDepart(depart);
    }

    @DeleteMapping("/{id}")
    public boolean deleteHandle(@PathVariable("id") int id){
        return service.removeDepartById(id);
    }

    @PutMapping("/")
    public boolean updateHandle(@RequestBody Depart depart){
        return service.modifyDepart(depart);
    }

    @GetMapping("/{id}")
    public Depart getHandle(@PathVariable("id") int id) {
        return service.getDepartById(id);
    }

    @GetMapping("/")
    public List<Depart> listHandle() {
        return service.listAllDeparts();
    }
}
```

谷歌浏览器的测试插件：`Restlet Client`

`application.yml`核心配置文件：

```yaml
server:
  port: 8081
spring:
  jpa:
    # ?????Spring????????????false
    generate-ddl: true
    # ??????????SQL?????false
    show-sql: true
    # ?????????????
    hibernate:
      ddl-auto: none
  # ?????
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql:///test?useUnicode=true&useSSL=false&characterEncoding=utf8
    username: root
    password: 123

# ????
logging:
  #???????
  pattern:
    console: level-%level %msg%n
  level:
    root: info
    # ??hibernate ????????
    org.hibernate: info
    # ?????????????????
    com.demo.provider8081: debug
    # ? show-sql ?true???SQL???????
    org.hibernate.type.descriptor.sql.BasicBinder: trace
    # ? show-sql ?true???????
    org.hibernate.type.descriptor.sql.BasicExtractor: trace
```

* 创建消费者

  实体类

  ```java
  package com.demo.bean;
  
  public class Depart {
      private Integer id;
      private String name;
  }
  ```

  配置类

  ```java
  package com.demo.controller;
  
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.web.client.RestTemplate;
  
  @Configuration
  public class DepartConfig {
      @Bean
      public RestTemplate restTemplate(){
          return new RestTemplate();
      }
  }
  ```

  控制层

  ```java
  @RequestMapping("/consumer/depart")
  @RestController
  public class DepartController {
      @Autowired
      private RestTemplate template;
  
      public static final String SERVICE_PROVIDER = "http://localhost:8081/provider/depart";
  
      @PostMapping("/save")
      public boolean saveHandle(@RequestBody Depart depart){
          String url = SERVICE_PROVIDER + "/save";
           // 远程调用一个HTTP接口
          return template.postForObject(SERVICE_PROVIDER,depart,Boolean.class);
      }
  
      @DeleteMapping("/del/{id}")
      public void deleteHandle(@PathVariable("id") int id){
          String url = SERVICE_PROVIDER + "/del/" + id;
          template.delete(url);
      }
  
      @PutMapping("/update")
      public void updateHandle(@RequestBody Depart depart){
          String url = SERVICE_PROVIDER + "/update";
          template.put(url,depart);
      }
      @GetMapping("/get/{id}")
      public Depart getHandle(@PathVariable("id") int id){
          String url = SERVICE_PROVIDER + "/get/" + id;
          return template.getForObject(url,Depart.class);
      }
      @GetMapping("/list")
      public List<Depart> listHandle(){
          String url = SERVICE_PROVIDER + "/list";
          return template.getForObject(url,List.class);
      }
  }
  ```



## 二、服务注册中心

### 简介

* 场景

> 在我们搭建模拟的模型中，会出现一个问题，即单点问题。当生产者挂了之后，消费者就无法访问了，很容易我们可想到解决方法，就是弄一个集群。
>
> 那么消费者又改如果调用集群呢？如何实现负载均衡呢？
>
> 一种是用nginx反向代理，另一种就是我们现在要学的加入一个注册中心。

* 注册中心简介

  > 所有提供者将自己提供的服务的名称及自己主机详情（IP、端口、版本等）写入到另一台主机中的一个列表中，这台主机称为**服务注册中心**，而这个表称为**服务注册表**。
  >
  > 大致的流程：
  >
  > 所有消费者需要调用微服务时，其会从注册中心首先将服务注册表下载到本地，然后
  >
  > 根据消费者本地设置好的负载均衡策略选择一个服务提供者进行调用。这个过程称为**服务发现**。
  >
  > 可以充当 Spring Cloud 服务注册中心的服务器有很多，如 Zookeeper、Eureka、Consul等。 Spring Cloud Alibaba 中使用的注册中心为 Alibaba 的中间件 Nacos。



### Nacos 

> Nacos（Dynamic Naming and Configuration Service）一个更易于构建**云原生应用**的动态服务发现、配置管理和服务管理平台。



* 什么是云原生应用呢？

  简单来说就是 SaaS（Software as a Service，**软件即服务**），就是跑在 IaaS（Infrastructure as a Service，**基础设施即服务**） 、PaaS （Platform as a Service，**平台即服务**上的 SaaS。

  云原生 = 微服务 + DevOps + CD + 容器化

* Nacos 架构及概念

  **Nacos client/sidecar**，Nacos sidecar是一个多语言模块，用来给除java其它语言应用的API。

### 使用

* 下载nacos（版本为2.2.3），解压后打开`application.properties`配置文件进行配置，修改如下配置

`nacos.core.auth.enabled=true`, 开启系统

`nacos.core.auth.server.identity.key=xxx`, 可设置随意的默认值s

`nacos.core.auth.server.identity.value=xxx`,可设置随意的默认值

`nacos.core.auth.plugin.nacos.token.secret.key=xxxx`,可设置随意32位的默认值



* 启动服务器

  - 注：Nacos的运行建议至少在2C4G 60G的机器配置下运行。

  ### Linux/Unix/Mac

  启动命令(standalone代表着单机模式运行，非集群模式):

  ```
  sh startup.sh -m standalone
  ```

  如果您使用的是ubuntu系统，或者运行脚本报错提示[[符号找不到，可尝试如下运行：

  ```
  bash startup.sh -m standalone
  ```

  ### Windows

  启动命令,在Nacos的bin目录下(standalone代表着单机模式运行，非集群模式):

  ```
  startup.cmd -m standalone
  ```

* 访问管理网站

  访问 `http://localhost:8848/nacos`,账号密码都为 `nacos`;

### 将 provider 和 consumer 服务注册到Nacos

* 项目搭建

  在之间搭建好的项目中，导入相关依赖

  ```xml
  <properties>
          <java.version>21</java.version>
          <spring-cloud.version>2022.0.0</spring-cloud.version>
          <spring-cloud-alibaba.version>2022.0.0.0-RC2</spring-cloud-alibaba.version>
  </properties>
  
  ...
  <!-- nacos注册中心的依赖 -->
  		<dependency>
              <groupId>com.alibaba.cloud</groupId>
              <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
         	</dependency>
  ...
  <dependencyManagement>
          <dependencies>
              <!-- spring-cloud的依赖 -->
              <dependency>     
                  <groupId>org.springframework.cloud</groupId>
                  <artifactId>spring-cloud-dependencies</artifactId>
                  <version>${spring-cloud.version}</version>
                  <type>pom</type>
                  <scope>import</scope>
              </dependency>
              <!-- spring-cloud-alibaba的依赖 -->
              <dependency>
                  <groupId>com.alibaba.cloud</groupId>
                  <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                  <version>{spring-cloud-alibaba.version}</version>
                  <type>pom</type>
                  <scope>import</scope>
              </dependency>
          </dependencies>
      </dependencyManagement>
  ```

* 修改`application.yml`配置文件

  ```yaml
    # 设置微服务名称
    application:
      name: depart-provider
  
    cloud:
      nacos:
        discovery:
          server-addr: localhost:8848  # nacos注册中心名称
          username: nacos
          password: nacos
  ```

* 启动项目

  启动项目后，就可以在Nacos管理中心的服务列表看到服务。

* 同理，将Consumer注册到Nacos

  修改配置文件：

  ```yaml
  spring:
    application:
      name: depart-consumer
    cloud:
      nacos:
        discovery:
          server-addr:  localhost:8848
          username: nacos
          password: nacos
  ```

  修改控制层：

  ```java
     //直连方式
      //public static final String SERVICE_PROVIDER = "http://localhost:8081/provider/depart";
  
      //微服务方式
      private static final String SERVICE_PROVIDER = "http://depart-provider/provider/depart";
  
  ```

  修改配置类：

  ```java
  @Configuration
  public class DepartConfig {
  
      @LoadBalanced // 以负载均衡的方式调用
      @Bean
      public RestTemplate restTemplate(){
          return new RestTemplate();
      }
  }
  ```

* 访问consumer

  发现500；这是因为新版`spring-cloud-alibaba`已经舍弃掉了默认的负载均衡，所以要导入负载均衡依赖。

  ```xml
  		<dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-loadbalancer</artifactId>
          </dependency>
  ```

  

  

  

### 获取注册中心的微服务实例信息

```java
@Autowired
private DiscoveryClient client;


@GetMapping("discovery")
    public List<String> discoveryHandle(){
        // 获取注册中心所有服务的名称
        List<String>  service = client.getServices();
        for(String s : service){
            // 获取指定微服务名称的所有微服务实例
            List<ServiceInstance> instances = client.getInstances(s);
            for(ServiceInstance instance : instances){
                Map<String, Object> map = new HashMap<>();
                map.put("serviceName", s);
                map.put("serviceId", instance.getInstanceId());
                map.put("serviceHost", instance.getHost());
                map.put("servicePort", instance.getPort());
                map.put("url", instance.getUri());
                System.out.println(map);
            }
        }
        return service;
    }
```

### 注册表缓存

​	服务在启动后，当发生调用时会自动从 Nacos 注册中心下载注册表到本地。所以，即使 Nacos 发生宕机，会发现消费者仍然是可以调用到提供者的。只不过此时已经不能再有服务进行注册了，服务中缓存的注册列表信息无法更新。



### 临时实例与持久实例

* #### 区别

  临时实例与持久实例的实际**存储位置**与**健康检测机制不同**的。

  * **临时实例：**

    默认情况下。

    服务实例仅会注册在 Nacos 内存，不会持久化到 Nacos 磁盘。

    其健康检测机制为 Client 模式，即 Client 主动向 Server上报其健康状态。默认心跳间隔为 5 秒。15秒内 Server 为收到 Client 心跳，则会将其标记为 “不健康”状态；在 30 秒内若收到了 Client心跳，则重新恢复“健康状态”，否则该实例将从 Server 端内存清除。

  * **持久实例：**

    服务实例不仅会注册到 Nacos 内存，同时也会被持久化到 Nacos 磁盘。

    其健康检测机制为 Server 模式，即 Server 会主动去检测 Client 的健康状态，默认每 20 秒检测一次。健康检测失败后服务实例会被标记为 “不健康” 状态，但不会被清除，因为其持久化在磁盘的。

  一般情况下用得最多的是临时实例，可以更好地应对服务器短时间内增加大量流量的问题。持久实例的删除更为复杂。

  删除持久实例：。。。

* #### 使用

  设置`spring.cloud.nacos.discovery.ephemeral`属性的值，false为持久实例，true为临时实例。

  并且微服务一旦启动并注册过实例的类型，就不得更改。（微服务是靠微服务名称来作为标识的）



### 将数据持久化到MySQL

> 在单机模式时 Nacos 使用嵌入式数据库实现数据的存储，不方便观察数据的存储情况。
>
> 所以我们要使用外置的mysql数据库，而且Nacos现在只支持mysql数据库。

- 1.安装数据库，版本要求：5.6.5+

- 2.初始化mysql数据库，修改脚本文件，增加建库语句，数据库初始化脚本文件：mysql-schema.sql

  ```mysql
  CREATE DATABASE IF NOT EXISTS `nacos_config`;
  USE `nacos_config`;
  ```

- 3.修改conf/application.properties文件，增加支持mysql数据源配置（目前只支持mysql），添加mysql数据源的url、用户名和密码。

  ```properties
  spring.datasource.platform=mysql
  
  db.num=1
  db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
  db.user.0=root
  db.password.0=123
  ```

  

### Nacos 集群搭建 与 CAP

* #### 集群的搭建与启动

  在单机模式下模拟集群，用不同的端口号来标识不同的主机。

  * 创建集群文件文件

    ![](images\屏幕截图 2023-10-25 200445.png)

  * 复制集群样本文件，修改集群`cluster.conf`配置文件

    要隔开一个端口号，因为系统要调用一个端口

    ```xml
    192.168.50.114:8849
    192.168.50.114:8851
    192.168.50.114:8853
    ```

  * 修改`application.properties`的端口号

    ·`server.port=8849`

  * 启动`startup.cmd`脚本文件，启动三个

  * 连接集群

    修改`application.yml`配置文件

    ```yml
    spring:
    	nacos:
    		discovery:
    			server-addr: localhost:8849,localhost:8851,localhost:8853
    ```

* #### 集群中是如何达成一致性的呢？

  利用修改后的raft算法

* #### Nacos 的 CAP 模式

  默认情况下，Nacos Discovery 集群的数据一致性采用的是 AP 模式。但其也支持 CP 模式，需要进行转换。若要转换为 CP 的，可以提交如下 PUT 请求，完成 AP 到 CP 的转换。

  `http://localhost:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP`



### 服务隔离

* #### Nacos 的数据模型？

  Nacos 中的服务是有三元组唯一确定的： `namespace` 、`group` 与服务名称 `service`。`namespace`与`group`的作用是相同的，用于划分不同的区域范围，隔离服务。不同的是， `namespace` 的范围更大，不同的 `namespace` 中可以包含相同的 `group`。不同 `group` 中可以包含相同的`service`。 `namespace` 的默认值` public` ，group 的默认值为`DEFULT_GRPUP`。

* #### 模拟服务隔离

  启动三个 `02-provider-nacos`实例，他们提供的服务相同，不同的是 namespace、group 与port

  *  public + DEFAULT_GROUP + 8081
  * public + MY_GROUP + 8081
  * hello + MY_GROUP + 8083

  namespace和group可以在`application.yml`配置文件中设置，在IDEA模拟环境下实现如下：

  ![](images\屏幕截图 2023-10-25 213308.png)

​	，允许多实例运行，然后增加 VM option，写入实例改变的参数。

关于namespace参数，需要在 Nacos 配置管理网站中增加namespace，获取到namespace的ID，然后在服务中的参数为这个ID。

## 三、Nacos Config 配置中心

> 集群中每一台主机的配置文件都是相同的，对配置文件的更新维护就成为了一个棘手的问题。此时就出现了配置中心，将集群中每个节点的配置文件交由配置中心统一管理。相关产品很多。
>
> 例如：Spring Cloud Config、 Zookeeper、Apollo、Disconf（百度，不再维护）等。但 Spring Cloud Alibaba官方推荐使用 Nacos 作为微服务的配置中心。



### 各个产品的原理

* #### Spring Cloud Config

  配置文件放在远程库中，例如`Github`；然后还有一个`Config Server`，用来与远程库通讯的；还有一个`Config Client`表示不同的微服务。

  当运维人员更新配置文件远程库的配置文件时，还要通过一个POST请求，也称为`bus-refresh`来向`Config Client`发送请求，然后`Config Client`就会向`Config Server`发送更新配置请求；同时，也会通过**消息总线系统（Spring Cloud Bus + RabbitMQ/Kafka）**广播请求到其它的`Config Client`，其它的也会发送更新配置的请求。

  ![](images\屏幕截图 2023-10-25 215112.png)

  **存在的三大问题：**

  * 无法自动感知更新，需要人工干预
  * 存在羊群效应（“跟风”），系统性能不行
  * 系统架构过于复杂，开发与维护成本大。在`Config Client`与`Config Server`之间还要配置 Nginx集群来完成负载均衡等工作。

* #### Apollo

  配置文件是放在DBMS中的，然后当维护人员想要修改配置文件的时候，它提供了一个用户管理配置文件的前端界面`Protal`，利用它来修改配置文件。当发生修改请求时，就会通过`Meta Server`来告知`Admin Service`发送修改，`Config Service`感知到配置文件发生编发，`Config Service`就会通过`Meta Server`来告知所有的微服务。

  `Meta Server`、`Admin Service`和`Config Service`都注册在`Eureka Server`注册中心中，所以他们能够互相通讯。

  ![](images\屏幕截图 2023-10-25 220349.png)

  **存在的问题**

  * 系统架构较为复杂
  * 配置文件支持类型较少，其只支持`xml、text、properties`,不支持`json、yaml`

* #### Nacos 配置中心

  配置文件放在MySQL中，然后维护人员通过`Config Server(Nacos)`的控制台来更新配置，然后还会通知到相应更新到的相关的微服务`Config Client`。

  ![](images\屏幕截图 2023-10-25 222003.png)

  

* #### Zookeeper

  > Zookeeper  是一个分布式系统协调器，协调各个组件的一致性

  配置文件都存储在Zookeeper内部中，Zookeeper可以存储少量的数据。

  ![](images\屏幕截图 2023-10-25 222620.png)

  Zookeeper 作为配置中心，其工作原理与前面的三种都不同。其没有第三方服务器去存储配置数据，而是将配置数据存放在自己的 `Znode` 中了。`ConfigClient` 也是可以自动感知的(Watcher 监听机制)。

  `Znode`的结构是树状结构，然后节点有四种类型。

* #### 一致性问题

  配置中心中的配置数据一般都是持久化在第三方服务器中的，例如DBMS、Git远程库。由于这些配置中心 Server 中根本就不存放数据，所有他们的集群中就不存在数据不一致问题。但像 Zookeeper，其作为配置中心，配置数据是存放在自己本地的。所以该集群中的节点是存在数据一致性问题的。Zookeeper采用的集群模式是`CP`模式。

  作为注册中心：这些 Server 集群间是存在数据一致性问题的，它们采用的模式是不同的Eureka（AP）、Consul（AP）、Zookeeper（CP）、Nacos默认AP，也支持CP。



### Nacos配置中心

* #### Nacos配置中心的实例

  1. 在`public`命名空间中创建配置，将`springboot`项目中的配置文件内容写入配置保存为`depart-provider.yml`.

  2. 导入 Nacos 配置中心的依赖

     ```xml
     <dependency>
         <groupId>com.alibaba.cloud</groupId>
         <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
         <version>${latest.version}</version>
     </dependency>
     ```

  3. 配置`application.yml`配置文件

     ```yaml
     spring:
       # 设置微服务名称
       application:
         name: depart-provider
     
       cloud:
         nacos:
           config:
             server-addr: localhost:8848
             file-extension: yml
             username: nacos
             password: nacos
       config:
        import:
          - optional:nacos:${spring.application.name}.${spring.cloud.nacos.config.file-extension}
     ```

  4. 设置共享配置

     ```yaml
     spring:
       # 设置微服务名称
       application:
         name: depart-provider
       cloud:
         nacos:
           config:
             server-addr: localhost:8848
             file-extension: yml
             username: nacos
             password: nacos
             # 同一个group中的不同服务可以共享以下配置
             shared-configs[0]:
               data-id: shareconfig.yml
               refresh: true
             # 不同的group中的不同服务可以共享以下扩展配置
             extension-configs[1]:
               data-id: extconfig.yml
               refresh: true
     ```

     ​	下标越大，优先级越高。

     ​	当前服务配置、共享配置与扩展配置的加载顺序为：**共享配置、扩展配置、当前服务配置**。若在三个配置中具有相同的属性设置，但它们具有相同的值，那么，后加载的值会覆盖先加载的值。所以三类配置的优先级由低到高为：**共享配置、扩展配置、当前服务配置**。

     ​	当前服务配置可以存在于三个地方：

     ​		第一个地方：`C:\Users\...\nacos\config\fixed-localhost_8848_nacos\snapshot\DEFAULT_GROUP`,这个是系统自动生成的快照文件。

     ​		第二个地方：`C:\Users\...\nacos\config\fixed-localhost_8848_nacos\data\config-data\DEFAULT_GROUP`中，这个是我们自己主动创建的配置文件的地方。

     ​		第三个地方：Nacos Config中
     
     这三个同名文件也存在加载顺序问题，加载顺序为：本地配置文件、远程配置文件、快照配置文件。只要系统加载到了配置文件，那么后面的就不再加载。

* #### 配置动态更新

  在Service层实现类中加入`@RefreshScope`注解实现配置的动态更新。然后在参数上加上`@Value`获取对应的值

* #### 多环境选择

  > 在开发应用时，通常同一套程序会被运行在多个不同的环境。每个环境的配置不同，需要在不同环境下修改不同的配置，导致操作繁琐。此时就需要定义出不同的配置信息，在不同的环境中选择不同的配置。

  ```yaml
  spring:
    # 设置微服务名称
    application:
      name: depart-provider
    cloud:
      nacos:
        config:
          server-addr: localhost:8848
          file-extension: yml
          username: nacos
          password: nacos
    profiles:
      active: dev
    config:
     import:
       - optional:nacos:${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
  ```

* #### 配置隔离

* #### 按profile获取配置文件

  如果项目的`spring.application.name=test`，然后在配置中心的`dataId`为`test.properties`的优先级要高于`test`配置文件的优先级。除开配置`spring.cloud.nacos.config.server-addr`外，还可以配置。

  1. `spring.cloud.nacos.config.group`:默认为“DEFAULT_GROUP”
  2. `spring.cloud.nacos.config.file-extension`:默认为"properties"(文件后缀)
  3. `spring.cloud.nacos.config.profix`:默认为 ${spring.application.name}

  通过阅读源码可以发现在拉取配置时会分为三步：

  1. 拉取dataid为`${spring.cloud.nacos.config.profix}`的配置
  2. 拉取dataid为`${spring.cloud.nacos.config.profix}.${spring.cloud.nacos.config.file-extension} `的配置
  3. 拉取dataid为`${spring.cloud.nacos.config.profix}-${spring-profiles.active}.${spring.cloud.nacos.config.file-extension}`

  并且优先级依次增高。

## 四、OpenFeign 与负载均衡

> 前面消费者对于微服务的远程消费是通过  `RestTemplate` 完成的，这种方式的弊端是很明显的：
>
> * 消费者对提供者的调用无法与业务接口完全吻合。例如：原本 Service 接口中的方法是有返回值的，但经过 `RestTemplate` 相关 API 调用后没有了其返回值，最终执行是否成功用户并不清楚。
> * 代码编写不方便，不直观。提供者原本是按照业务接口提供服务的，而经过 `RestTemplate` 一转手，变成了 URL，使得程序员在编写消费者对提供者的调用代码时，变得不直接、不明了。



### OpenFeign

> 声明式 REST 客户端： Feign 通过使用` JAX-RS(Java Api eXtensions of RESTful web Services) `或 `SpringMVC` 注解的修饰方式，生成接口的动态实现。
>
> Feign，假装、伪装。OpenFeign 可以将提供者提供的 `Restful` 服务伪装为接口进行消费，消费者只需要使用 “ feign 接口 + 注解” 的方式即可直接调用提供者提供的 Restful 服务。
>
> 无需使用`RestTemplate`
>
> 【总结】
>
> * OpenFeign 只涉及 Consumer， 与Provider 无关。因为其是用于 Consumer 调用 Provider的
> * OpenFeign 仅仅就是一个伪客户端，其不会对请求做任务的处理
> * OpenFeign 是通过注解的方式实现 `RESTful` 请求的

* #### OpenFeign 与 Ribbon

  OpenFeign 具有负载均衡功能，其可以对指定的微服务采用负载均衡进行消费、访问。之前的老版本 的Spring Cloud 所集成的 OpenFeign 默认采用了Ribbon 负载均衡器。但由于 Netfilx 已不再维护 Ribbon，所以从 Spring Cloud 2021 .x 开始集成的OpenFeign 中以彻底丢弃 Ribbon，而是采用 Spring Cloud 自行研发的 `Spring Cloud Loadbalancer` 作为负载均衡器。 

  Ribbon要在RestTemplate类上加上@Ribbon注解，就会自动解析域名

* #### OpenFeign 的用法

  1. 导入依赖

     ```xml
     <dependency>
     	<groupId>org.springframework.cloud</groupId>
     	<artifactId>spring-cloud-starter-openfeign</artifactId>
     </dependency>
     ```

  2. 伪装Service层业务接口

     ```java
     //将此类标记成伪装类，并设置要消费的微服务名称
     //设置请求路径时，在老版本可以用@RequestMapping或@FeignClient设置path
     //但是在新版本只能用后者。
     @FeignClient(value = "depart-provider",path = "/provider/depart")
     @RequestMapping("/provider/depart")
     public interface DepartService {
         @PostMapping("/save")
         boolean saveDepart(@RequestBody Depart depart);
     
         @DeleteMapping("/del/{id}")
         boolean removeDepartById(@PathVariable("id") int id);
     
         @PutMapping("/update")
         boolean modifyDepart(@RequestBody Depart depart);
     
         @GetMapping("/get/{id}")
         Depart getDepartById(@PathVariable("id") int id);
     
         @GetMapping("/list")
         List<Depart> listAllDepart();
     }
     ```

  3. 修改Controller层

     ```java
     @RequestMapping("/consumer/depart")
     @RestController
     public class DepartController {
         @Autowired
         private DepartService service;
         @PostMapping("/save")
         public boolean saveHandle(@RequestBody Depart depart){
             return service.saveDepart(depart);
         }
     
         @DeleteMapping("/del/{id}")
         public boolean deleteHandle(@PathVariable("id") int id){
             return service.removeDepartById(id);
         }
     
         @PutMapping("/update")
         public boolean updateHandle(@RequestBody Depart depart){
             return service.modifyDepart(depart);
         }
         @GetMapping("/get/{id}")
         public Depart getHandle(@PathVariable("id") int id){
             return service.getDepartById(id);
         }
         @GetMapping("/list")
         public List<Depart> listHandle(){
             return service.listAllDepart();
         }
     }
     ```

  4. 最后还要在启动类上面加上`@EnableFeignClients` 开启 OpenFeign

* #### OpenFeign 的超时设置

  ```yaml
  spring:
    application:
      name: depart-consumer
    cloud:
      nacos:
        discovery:
          server-addr:  localhost:8848
          username: nacos
          password: nacos
      openfeign:
        client:
          config:
            default:
            	# 全局设置，单位是毫秒
              # 连接超时：consumer 连接上 provider 的时间间隔，其决定作用的是网络
              connect-timeout: 1
              # 读超时： consumer 发出请求到接收到 provider 的响应这段时间阈值，其决定作用的是 provider 的业务代码逻辑
              read-timeout: 1
              # 局部设置
              depart-provider:
              ....
  ```

* #### 压缩设置

  ```yaml
        compression:
          request:
            enabled: true
            mime-types: ["text/xml", "application/json","video/mp4"]
            # 默认为2048
            min-request-size: 1024
          response:
            enabled: true
  ```

* #### 远程调用的底层实现技术选型

  > feign 的远程调用底层实现默认采用的是 `JDK 的 URLConnection` ，同时还支持 `HttpClient` 与 `OkHttp`
  >
  > 由于 `JDK 的URLConnection` 不支持连接池，通信效率很低，所以生产中是不会使用默认实现的。所以在 Spring Cloud OpenFeign 中直接默认是实现变成了 `HttpClient`，同时也支持 `OkHttp`。
  >
  > 单例的用`HttpClient`、非单例的用`OkHttp`；如果要自定义 http 客户端，用`HttpClient`。

  配置说明：

* #### 负载均衡

  > - `Client`feignClient：如果 `Spring Cloud LoadBalancer` 在类路径上（即是否导入`LoadBalancer`依赖, 默认轮询策略），则使用。 如果它们都不在类路径上，则使用默认的假客户端。`FeignBlockingLoadBalancerClient`

  ##### 更换负载均衡策略：

  将轮询更改为随机

  1. 创建策略类

     ```java
     public class DepartConfig {
     
         @Bean
         public ReactorLoadBalancer<ServiceInstance> randomLoadBalancer(Environment e, LoadBalancerClientFactory factory){
             // 获取负载均衡客户端名称，即提供者服务名称；
             String name = e.getProperty(LoadBalancerClientFactory.PROPERTY_NAME);
             // 从所有provider实例中指定名称的实例列表中随机选择一个实例
             // 参数一：指定名称所有provider实例列表
             // 参数二：指定要获取的provider服务的名称
             return new RandomLoadBalancer(factory.getLazyProvider(name, ServiceInstanceListSupplier.class), name);
         }
     
     }
     ```

  2. 在启动类上配置策略

     ```java
     @LoadBalancerClients(defaultConfiguration = DepartConfig.class)
     @SpringBootApplication
     @EnableFeignClients
     public class NacosConsumerApplication 
     ```

     

  这个配置比较麻烦，并且负载均衡策略较少只有两种，之后我们用`dubbo`

  上述的通讯方式是HTTP、而在`Spring Cloud Netflix`用的协议是 `GRPC`

  而`Spring Cloud Alibab`的Dubbo用的是比较知名的`RPC`框架

## 五、Spring Cloud Gateway 

### 简介

> 什么是网关？

网关是系统唯一对外的入口，介于客户端与服务器端之间，用于对请求进行**鉴权、限流、路由、监控等功能**

* #### spring Cloud Gateway

  对于之前所学的 Nacos ，可以说是消费者在调用服务提供者时的一个中间件，用来对请求进行负载均衡。那么，在用户客户端调用消费者时，又如何实现负载均衡的调用呢？

  一种是用 Nginx；第二种就是Spring Cloud Gateway。

  > spring cloud gateway 提供了一个建立在Spring 生态系统之上的 API 网关，包括 Spring6、SpringBoot 3 、和**project Reactor**。旨在提供一种简单而有效的方法来路由到api，并为它们提供跨领域的关注点，例如：安全性、监控/度量和弹性。

  **注意：**

  > ​	Spring Cloud Gateway 基于` Spring Boot`、**Spring WebFlux和 Project Reactor**构建。 因此，在使用 Spring Cloud Gateway 时，您知道的许多熟悉的同步库（例如 Spring Data 和 Spring Security）和模式可能不适用。 如果您不熟悉这些项目，我们建议您在使用 Spring Cloud Gateway 之前先阅读他们的文档以熟悉一些新概念。
  >
  >   Spring Cloud Gateway 需要 Spring Boot 和 Spring Webflux 提供的 Netty 运行时。 它不能在传统的 Servlet 容器中工作，也不能在作为 WAR 构建时工作。

* #### Reactor 简介

  ​	Reactor 是一种完全基于 Reactive Streams 规范的、全新的库。

  **（1）响应式编程**

  ​	响应式编程，Reactive Programming ，是一种新的编程范式、编程思想。

  ​	响应式编程最早由.Net平台上的 `Reactive eXtension(Rx)`库来实现的，后来发现在java很好用，被迁移到java平台，即`RxJava`，之后又产生了 Reactive Streams 规范。

  **（2）Reactive Streams**

  ​	Reactive Streams 是响应式编程的规范，定义了响应式编程的相关接口。只要符合该规则的库，就称为 Reactive 响应式编程库。

  **（3）RxJava2**

  ​	RxJava2 是一个响应式编程库，产生于 `Reactive Streams` 规范之后。但是由于其是在 RxJava基础之上进行的开发，所以在设计时不仅遵循了 `Reactive Streams `规范，同时为了兼容 RxJava，使得 RxJava2 在使用时非常不方便。

  **（4）Reactor**

  ​	Reactor 是一种全新的响应式编程库，完全遵循 Reactive Streams 规范，有与RxJava没有任何关系，所以，其使用时非常方便、直观。

* #### Zuul

  ​	Zuul 是Netflix 的开源 API 网关。Zuul 是基于 Servlet 的，使用同步阻塞 IO，不支持长连接。Zuul 是SpringCloud 生态的一员。

  ​	Zuul2.x 使用 Netty 实现了异步非阻塞 IO，支持长连接。但其未整合到 Spring Cloud。

  

* #### 重要概念

  **（1）route 路由**

  ​	路由是网关的最基本组成，有一个路由id、一个目标地址url、一组断言工厂及一组 filter组成。若断言为 true，则请求将经由 filter 被路由到目标 url。

  **（2）predicate 断言**

  ​	断言即一个条件判断，根据当前的 http请求进行指定规则的匹配，比如说 http 请求头，请求时间等。只有当匹配上规则时，断言才为 true，此时请求才会被直接路由到目标地址（目标服务器），或先路由到某过滤器链，经过过滤器链的层层处理后，再路由到相应的目标地址（目标服务器）。

  **（3）filter 过滤器**

  ​	对请求进行处理的逻辑部分。当请求的断言为 true是时，会被路由到设置好的过滤器，以对请求或响应进行处理。例如，可以为请求添加一个请求参数，或对请求 URL 进行修改，或为响应添加 header 等。

  

### 小例题

> 用户访问 spring cloud gateway 应用，直接跳转到百度主页

* #### 配置式路由

  **（1）添加依赖**

  ```xml
  		<dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-gateway</artifactId>
          </dependency>
  ```

  并且一定要删除web依赖

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  ```

  **(2)修改配置文件**

  ```yaml
  server:
    port: 9000
  
  spring:
    cloud:
      gateway:
        routes:
          - id: my_route
            uri: https://baidu.com
            predicates:
              - Path=/baidu
  
  ```

  **（3）测试**

  访问`localhost:9000/baidu`,会跳转到`baidu.com/baidu`

  修改配置文件参数：

  `- Path=/**`

  访问 `localhost:9000 `,就会跳转到`baidu.com`

  访问`localhost:9000/baidu`,会跳转到`baidu.com/baidu`

  

* #### API 配置路由

  **（1）构建配置类**

  ```java
  @Configuration
  public class GatewayConfig {
      @Bean
      public RouteLocator bdRoudeLocator(RouteLocatorBuilder builder){
  
          return builder.routes()
                  .route("baidu_route",ps -> ps.path("/bd")
                          .uri("http://baidu.com"))
                  .route("jd_route",ps -> ps.path("/jd")
                          .uri("http://jd.com"))
                  .build();
      }
  }
  ```

* #### 配置式和API的优先级

  配置式与API式的路由的优先级是或的关系。谁匹配了就用谁。

### 路由断言工厂

* #### The After(before) Route Predicate Factory

  此谓词匹配在指定日期时间之后(之前)发生的请求

  ```yam
  spring:
    cloud:
      gateway:
        routes:
        - id: after_route
          uri: https://example.org
          predicates:
          - After=2017-01-20T17:42:47.789-07:00
  ```

* #### The Between Route Predicate Factory

  ```yaml
   predicates:
          - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
  ```

  ```java
  @Bean
  public RouteLocator betweenRoudeLocator(RouteLocatorBuilder builder){
  
          ZonedDateTime dateTime1 = LocalDateTime.now().minusDays(2)//当前时间回退3天
                  .atZone(ZoneId.systemDefault());// 将系统默认时区设置为当前时区
          ZonedDateTime dateTime2 = LocalDateTime.now().minusDays(5)//当前时间前进3天
                  .atZone(ZoneId.systemDefault());
  
          return builder.routes()
                  .route("baidu_route",ps -> ps.between(dateTime1,dateTime2)
                          .uri("http://baidu.com"))
                  .build();
  }
  ```

* #### Cookie

  > 谷歌浏览器在90版本之后为了安全就禁用了Cookie

  ```yaml
    predicates:
          - Cookie=chocolate, ch.p
  ```

* #### Header

  两个Header之间的关系是与的关系，两个都要满足（使用正则表达式）

  ```yaml
    predicates:
          - Header=X-Request-Id, \d+
          - Header=Color, gr.+
  ```

  下面是表示或的关系

  ```yaml
  - id: my_route
          uri: https://example.org
          predicates:
          - Header=Color, gr.+
  - id: my_route2
          uri: https://example.org
          predicates:
          - Header=X-Request-Id, \d+
  ```

  配置式：

  ```java
  return builder.routes()
                  .route("baidu_route",ps -> ps.header("X-Request-Id","\\d+")
                         .and()//或者.or()
                         .header("color","gr.+")
                          .uri("http://baidu.com"))
                  .build();
  ```

* #### Host

  ```yam
  predicates:
          - Host=aaa.com:9000, bbb.com:9000
  ```

  配置式：

  ```java
  @Bean
      public RouteLocator hostRoudeLocator(RouteLocatorBuilder builder){
  
          return builder.routes()
                  .route("baidu_route",ps -> ps.host("aaa.com","bbb.com")
                          .uri("http://baidu.com"))
                  .build();
      }
  ```

* #### Method

  略

* #### Path

  。。。

* #### Query

  验证请求参数

* #### Weight

  设置组权重

  ```yaml
  spring:
    cloud:
      gateway:
        routes:
        - id: weight_high
          uri: https://weighthigh.org
          predicates:
          - Weight=group1, 8
        - id: weight_low
          uri: https://weightlow.org
          predicates:
          - Weight=group1, 2
  ```

* #### RemoteAddr

  配置文件：

  ```yaml
  spring:
    cloud:
      gateway:
        routes:
        - id: remoteaddr_route
          uri: https://example.org
          predicates:
          - RemoteAddr=192.168.1.1/24
  ```

  若IP地址在 ` 192.168.1.1` 到 `192.168.1.255` 网段内，就可以通过 。

  > 什么是RemoteAddr？

  当一个客户端想要访问到服务器时，而这个`RemoteAddr`就是客户端的 IP 地址，但是更多的时候会客户端会经过一个反向代理，如Nginx；这时`RemoteAddr`地址就不是客户端的 IP 地址，而是反向代理服务器的的 IP 地址。

  例如，当在虚拟机上请求到虚拟机所在的主机上时，这时win10系统会反向代理将请求发到本地，这个 `RemoteAddr` 就是本机的 IP 地址，而不是虚拟机的IP地址。

  > 如何解决呢？

  修改远程地址的解析方式，Spring Cloud Gateway 附带了一个基于 `X-Forwarded-For `标头的非默认远程地址解析器。`RemoteAddressResolver` `XForwardedRemoteAddressResolver`。

* #### XForwarded Remote Addr

  ```yaml
  predicates:
          - XForwardedRemoteAddr=192.168.1.1/24
  ```

  此谓语允许根据 HTTP 请求头过滤请求，可以与反向代理一起使用。例如负载均衡器或 Web 应用程序防火墙，其中 仅当请求来自这些请求使用的 IP 地址的受信任列表时，才应允许该请求反向代理。

  例如，当一个客户端发出请求时，经过了一个反向代理，然后将IP地址放入到了请求头中，例如`X-Forwarded-For : 192.168.1.50`，然后又经过某些服务器，又添加到了请求头中；这时服务端只会看最后一个IP地址是否在受信列表。

### ☆自定义路由断言工厂

#### Auth 认证

* #### 需求

  ​	当请求头中携带有用户名与密码的 key-value 对，且其用户名与配置文件中Auth路由断言工厂中指定的 username 相同，密码中包含 Auth 路由断言工厂中指定的 password 时才能通过认真。

* #### 定义 Factory

   ```java
   @Component
   //类名一定要以RoutePredicateFactory为后缀，配置时以后缀前面的英文为配置名（Auth）
   public class AuthRoutePredicateFactory extends AbstractRoutePredicateFactory<AuthRoutePredicateFactory.Config> {
       //构造器
       public AuthRoutePredicateFactory() {
           super(Config.class);
       }
       //具体的断言业务
       @Override
       public Predicate<ServerWebExchange> apply(Config config) {
           return exchange -> {
               //获取到请求中的所有Header
               HttpHeaders headers = exchange.getRequest().getHeaders();
               // 获取指定的Header的值,可能包含多个值；若有多个值将会拼接起来
               List<String> passwords = headers.get(config.getUsername());
               if (passwords.contains(config.getPassword())){
                   return true;
               }
               return false;
           };
       }
       //数据封装,用来封装请求的数据，静态类
       //@Data注解自动生成相关方法（set、get方法）
       @Data
       public static class Config {
           private String username;
           private String password;
       }
       //指定配置文件的读取数据顺序
       @Override
       public List<String> shortcutFieldOrder() {
           return Arrays.asList("username", "password");
       }
   }
   ```

* #### 配置文件

  ```yaml
  server:
    port: 9000
  spring:
    cloud:
      gateway:
        routes:
          - id: my_route
            uri: http://localhost:8081
            predicates:
              - Auth=zhangsan, 123
  ```

#### Token 认证

* #### 需求

  当请求中携带有一个 token 请求参数，且参数值包含在配置文件中的Token路由断言工厂指定的token值中时才能够通过认证，访问系统。

* #### 创建factory

  ```java
  @Component
  public class TokenRoutePredicateFactory extends AbstractRoutePredicateFactory<TokenRoutePredicateFactory.Config> {
  
      public TokenRoutePredicateFactory() {
          super(Config.class);
      }
      @Override
      public Predicate<ServerWebExchange> apply(Config config) {
          return exchange -> {
              //获取请求中是所有请求参数
              MultiValueMap<String, String> params = exchange.getRequest().getQueryParams();
              List<String> values = params.get("token");
              if(values.contains(config.getToken())){
                  return true;
              }
              return false;
          };
      }
      @Override
      public List<String> shortcutFieldOrder() {
          return Collections.singletonList("token");
      }
      @Data
      public static class Config {
          private String token;
      }
  }
  ```

  

### 网关过滤器工厂

> 网关过滤器工厂（GatewayFilterFactory），允许以某种方式修改传入的HTTP请求或返回HTTP响应。其作用域是某些特定路由。SpringCloud Gateway包括很多内置的网关过滤器工厂。下面会学习较常用的几种。



* #### 修改请求头

  配置式：

  ```yaml
  spring:
    cloud:
      gateway:
        routes:
          - id: my_route
            uri: http://localhost:8080
            predicates:
              - Path=/**
            filters:
              - AddRequestHeader=X-Request-Color, blue
  ```

  API式：

  ```java
  @Bean
      public RouteLocator filterAddRequestHeaderRoudeLocator(RouteLocatorBuilder builder){
  
          return builder.routes()
                  .route("baidu_route",ps -> ps.path("/**")
                          .filters(fs -> fs.addRequestHeader("X-Request-Color", "blue"))
                          .uri("http://baidu.com"))
                  .build();
      }
  ```

  控制器获取：

  ```java
  @RestController
  @RequestMapping("/info/")
  public class ShowInfoController {
  
      @GetMapping("/header")
      public String headerHandle(HttpServletRequest request){
          Enumeration<String> headers = request.getHeaders("X-Request-Color");
          StringBuilder sb = new StringBuilder();
          while(headers.hasMoreElements()){
              sb.append(headers.nextElement());
          }
          return "X-Request-Color:" + sb.toString();
      }
  }
  ```

* #### 修改多个不存在的请求头

  ```yaml
            filters:
              - AddRequestHeadersIfNotPresent=X-Request-Color:bule, city:beijing
  ```

* #### 添加请求参数

  ```yaml
            filters:
              - AddRequestParameter=color,blue
  ```

* #### 添加响应头

  。。。

* #### CircuitBreaker

  > 当系统发生错误时，开头设置服务降级

  需要引入依赖

  ```xml
  		<dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
          </dependency>
  ```

  配置文件：

  ```yaml
            filters:
              - name: CircuitBreaker
              # 设置服务降级，降级到请求到/fb
                args:
                  name: myCircuitBreaker
                  fallbackUri: forward:/fb
  ```

  创建服务降级控制器

  ```java
  @RestController
  public class FallbackController {
  
      @RequestMapping("/fb")
      public String fallbackHandle(){
          return "this is the Gateway Fallback";
      }
  }
  ```

  当网关目标服务挂了时，就会发生服务降级

* #### PrefixPath

  > 可以给URI加上前缀。
  >
  > 什么是URI?例如localhost:9000/info/header地址中，URI是/info/header

  配置文件:

  ```yaml
        routes:
          - id: my_route
            uri: http://localhost:8080
            predicates:
              - Path=/depart/**
            filters:
              - PrefixPath=/provider
  ```

  

* #### StripPrefix

  > 删除URI的前缀

  配置文件：

  ```yaml
        routes:
          - id: my_route
            uri: http://localhost:8080
            predicates:
              - Path=/xxx/oooo/depart/**
            filters:
              - StripPrefix=2
  ```

  会去掉前两个路径

* #### RewritePath

  > 重写URI

  配置文件：

  ```yaml
        routes:
          - id: my_route
            uri: http://localhost:8080
            predicates:
              - Path=/red/##
            filters:
              - RewritePath=/red, /provider/depart
              
              # 官方例子
          routes:
            - id: rewritepath_route
              uri: https://example.org
              predicates:
              - Path=/red/**
              filters:
              - RewritePath=/red/?(?<segment>.*), /$\{segment}
              # 正则表达式，<segment>是分组名称，${segment}是应用分组
              # 但是在yaml规范中，要用$\代替$
              # 这个正则表达式的效果是将/red去掉
  ```

* #### RequestRateLimiter

  > 工厂使用实现来确定是否允许当前请求继续。如果不是，则返回状态（默认情况下）。`RequestRateLimiter``GatewayFilter``RateLimiter``HTTP 429 - Too Many Requests`

  ##### 限流算法的实现：

  **The Redis RateLimiter**

  这个采用的是**令牌桶算法（Token Bucket Algorithm）**

  ![](images\屏幕截图 2023-11-01 205812.png)

  1. 令牌桶会根据设置好的速率产生令牌。
  2. 当请求进来时，若队列没有满就会进入队列；否则就会丢弃请求
  3. 当队首单元取到令牌时，就能够被处理，然后丢弃令牌

  **漏斗算法：**

  和令牌桶算法很像，不一样的地方是，漏斗算法令牌是以恒定的速率来发放的；而令牌桶算法是只要桶内有令牌就会发放令牌。

  

  需要用到Redis，所以要配置Redis

  ```yaml
  spring:
    data:
      redis:
        host: 192.168.1.1
        port: 6379
  ```

  使用`The Redis RateLimiter`，需要导入依赖

  ```xml
  		<dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
          </dependency>
  ```

  还需要配置限流键解析器`keyResolver`，

  ```java
  //请求参数“user”作为限流键，如果没有“user”请求参数，就无法访问
  @Bean
  KeyResolver userKeyResolver() {
      return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("user"));
  }
  //主机名作为限流键
  @Bean
  KeyResolver userKeyResolver() {
      return exchange -> Mono.just(exchange.getRequest().getRemoteAddress().getHostName();
  }
  //URI作为限流键
  @Bean
  KeyResolver userKeyResolver() {
      return exchange -> Mono.just(exchange.getRequest().getPath().value();
  }
  ```

  配置文件：通过SpEL表达式

  ```yaml
            filters:
              - name: RequestRateLimiter
                args:
                    key-resolver: "#{@userKeyResolver}"
                    # 令牌桶的速率，每秒钟10个
                    redis-rate-limiter.replenishRate: 10
                    # 突发的流量增大的情况下，产生速率变为每秒钟20个
                    redis-rate-limiter.burstCapacity: 20
                    # 需要的令牌数
                    redis-rate-limiter.requestedTokens: 1
  ```

* #### 默认网关过滤工厂

  ```yaml
    cloud:
      gateway:
        default-filters:
          -name: CircuitBreaker
          args:
            name: myCircuitBreaker
            fallbackUri: forward:/fb
        routes:
          - id: my_route
            uri: http://localhost:8080
            predicates:
              - Path=/red/##
  ```

* #### 优先级问题

  `局部的filter`优先级一定要高于`默认的filter`优先级，`API式filter`的优先级要高于`配置式filter`的



### 自定义的过滤工厂

创建添加请求头过滤工厂,继承`AbstractNameValueGatewayFilterFactory`接口：

```java
@Component
public class AddHeaderGatewayFilterFactory extends AbstractNameValueGatewayFilterFactory {

    @Override
    public GatewayFilter apply(NameValueConfig config) {
        return (exchange, chain) -> {
            // 将配置文件的header封装到request了
            ServerHttpRequest request = exchange.getRequest()
                                                .mutate()
                                                .header(config.getName(), config.getValue())
                                                .build();
            ServerWebExchange changedExchange = exchange.mutate()
                                                        .request(request)
                                                        .build();
            return chain.filter(changedExchange);
        };
    }
}
```

### GatewayFilter 的 pre 与post的执行顺序

> pre filter也就是在请求到达服务器之前对请求的处理， post filter 就是服务器响应客户端请求到达之前对响应的处理的处理。

```java
@Component
@Slf4j
public class OneGatewayFilterFactory extends AbstractNameValueGatewayFilterFactory {
    @Override
    public GatewayFilter apply(NameValueConfig config) {
        return (exchange, chain) -> {
            // pre-filter，有关请求过滤器业务代码
            long start = System.currentTimeMillis();
            log.info(config.getName() + "-" + config.getValue() + "开始执行时的时间：" + start);
            // 将开始时间放入到上下文中
            exchange.getAttributes().put("startTime",start);
            return chain.filter(exchange).then(
                    // post-filter，有关过响应滤器业务代码
                    Mono.fromRunnable(() -> {
                        long startTime = (long) exchange.getAttributes().get("startTime");
                        long endTime = System.currentTimeMillis();
                        long elaspedTime = endTime - startTime;
                        log.info(config.getName() + "-" + config.getValue() + "执行完毕的时间的时间：" + endTime);

                    })
            );
        };
    }
```

```yaml
          filters:
            - One=onefilter, 111
            - Two=twofilter, 222
            - Three=three, 333
```

`pre-filter`的执行顺序会和配置文件中的顺序一样，例如这里的`OneFilter、TwoFilter、ThreeFilter` ；

` post-filter`的执行顺序就和`pre-filter`的执行顺序相反。



### 关于配置文件的变量参数

```yaml
      routes:
        - id: my_route
          uri: http://localhost:8080
          predicates:
            - Path=/provider/{bean}/get/{id}
          filters:
            - AddHeader=color, red
```

可以在过滤器中获取参数：

```
Map<String, String> uriVariables = ServerWebExchangeUtils.getUriTemplateVariables(exchange);
String bean = uriVariables.get("bean");
String id = uriVariables.get("id");
```



### 全局过滤器

> 全局过滤器 Global Filter 是应用于所有路由策略上的Filter。Spring Cloud Gateway中已经定义好了很多 GlobalFilter，但这些 GlobalFilter无需任何配置与声明，在符合应用条件时就会自动生效（当符合条件时，这些应用的类就会被spring装配到容器中）。
>
> 系统帮我们定义好了10种全局过滤器



* #### 负载均衡全局过滤器

  > 想要gateway实现负载均衡的功能，就要能够获取到想要负载均衡的所有微服务的信息进行服务发现，又因为所有的微服务都注册在注册中心，所以gateway也要配置到的注册中心中。

配置文件：

```yaml
server:
  port: 9000

spring:
  data:
    redis:
      host: 192.168.1.1
      port: 6379
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        username: nacos
        password: nacos
    gateway:
      routes:
        - id: my_route
          # 开启负载均衡过滤器，lb表示负载均衡，后面表示微服务名称
          uri: lb://depart-consumer
          predicates:
            - Path=/**
      # 开启Gateway在注册中心进行服务发现的功能，默认为false
      discovery:
        locator:
          enabled: true
      # 将503状态码转化为404
      loadbalancer:
        use404: true
  application:
    name: depart-gateway
```

> 缺省情况下，当 找不到服务实例时，将返回 a。 您可以通过设置 将网关配置为返回 。`ReactorLoadBalancer``503``404``spring.cloud.gateway.loadbalancer.use404=true`



* #### 自定义全局过滤器

  * ##### 需求

    ​	这里要实现的需求是，访问当前系统的任意模块的URL都需要是合法的 URL。这里所谓合法的 URL 指的是请求中携带了 token 请求参数。

    ​	由于是对所有请求的 URL 都要进行验证，所以这里就需要定义一个 `GlobalFilter`，可以应用到所有路由中。

  * ##### 定义 URLValidateGlobalFilter

    需要注意的是，GlobalFilter 不需要在具体的路由规则中进行注册，只需要将其生命周期交给 Spring 容器来管理即可。

     ```java
     @Component
     //实现Ordered是对过滤器进行一个排序，数值越大优先级越低
     public class UrlValidataGlobalFilter implements GlobalFilter, Ordered {
         @Override
         public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
             // 从请求中获取请求参数token 的值
             String token = exchange.getRequest().getQueryParams().getFirst("token");
             if(!StringUtils.hasText(token)){
                 // 设置响应内容；若token为，设置状态码为401，未授权
                 exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
                 return exchange.getResponse().setComplete();
             }
             return chain.filter(exchange);
         }
     
         @Override
         public int getOrder() {
             //设置为最高的优先级
             return Ordered.HIGHEST_PRECEDENCE;
         }
     }
     ```

    

### 跨域配置

* #### 跨域与同源

  为了安全，浏览器中启用了一种称为**同源策略**的安全机制，禁止从一个域名页面中请求另一个域名下的资源。

  当两个请求的**访问协议、域名与端口号**三者都相同时，才称它们是同源的。只要有一个不同，就称为跨源请求。

  例如：

  源：` http://sports.abc.com/content/kb`

  跨源（域名不同）：`http://news.abc.com/content/kb`

* #### CORS

  CORS, Cross-Origin Resource Sharing, 跨域资源共享，是一种允许当前域的资源（例如，html、js、web service）被其它域的脚本请求访问的机制。

  

* #### 跨域问题模拟

  **（1）创建测试html**

  ```html
  <!DOCTYPE html>
  <html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
      <meta charset="UTF-8">
      <title>cors</title>
      <script src="https://s3.pstatp.com/cdn/expire-1-M/jquery/3.3.1/jquery.min.js"></script>
    	<script>
      function accessDepart(){
        $.get("http://localhost:9000/provider/depart/get/3", function (data){
            alert(data.name);
        })
      }
    </script>
  </head>
  <body>
    <input type="button" value="访问Depart" onclick="accessDepart()">
  </body>
  </html>
  ```

  > https://s3.pstatp.com/cdn/expire-1-M/jquery/3.3.1/jquery.min.js是字节的一个在线jquery

  如果点击按钮访问，就会发生跨域问题。

  **（2）如何解决**

  ​	有两种解决方案

  1. 全局CORS配置

     ```yaml
     spring:
       cloud:
         gateway:
           globalcors:
             cors-configurations:
               '[/**]':
                 allowedOrigins: "*"
                 allowedMethods:
                   - GET
                   - POST
     ```

  2. 路由CORS配置

     ```java
     spring:
       cloud:
         gateway:
           routes:
          	 - id: cors_route
             	uri: https://example.org
             predicates:
             	- Path=/service/**
             metadata:
               cors
                 allowedOrigins: '*'
                 allowedMethods:
                   - GET
                   - POST
                 allowedHeaders: '*'
                 # 缓存保留时间
                 maxAge: 30
     ```



### 自定义异常处理器

* #### 自定义Web异常控制器

  ```java
  @Component
  @Order(-1)
  public class CustomErrorWebExceptionHandle extends AbstractErrorWebExceptionHandler {
  
      public CustomErrorWebExceptionHandle(ErrorAttributes errorAttributes,
                                           ApplicationContext applicationContext,
                                           ServerCodecConfigurer serverCodecConfigurer) {
          super(errorAttributes, new WebProperties.Resources(), applicationContext);
          super.setMessageWriters(serverCodecConfigurer.getWriters());
          super.setMessageReaders(serverCodecConfigurer.getReaders());
      }
  
      @Override
      protected RouterFunction<ServerResponse> getRoutingFunction(ErrorAttributes errorAttributes) {
          return RouterFunctions.route(RequestPredicates.all(), this::randerErrorRespose);
      }
  
      private Mono<ServerResponse> randerErrorRespose(ServerRequest request) {
          // 获取异常信息
          Map<String, Object> map = getErrorAttributes(request, ErrorAttributeOptions.defaults());
          //构建响应
          return ServerResponse.status(HttpStatus.NOT_FOUND)  //404 状态码
                  .contentType(MediaType.APPLICATION_JSON)    //以JSON格式显示响应
                  .body(BodyInserters.fromValue(map));        //响应体（响应内容）
  
      }
  }
  ```

* #### 自定义异常属性控制器

  ```java
  @Component
  public class CustomErrorAttributes extends DefaultErrorAttributes {
      @Override
      public Map<String, Object> getErrorAttributes(ServerRequest request, ErrorAttributeOptions options) {
          //在父类设置好的默认属性加上属性，也可以自定义
          Map<String, Object> map = super.getErrorAttributes(request, options);
          map.put("msg", "对不起，没找到你搜索的资源");
          return map;
      }
  }
  ```

  

## 六、Sentinel 流量防卫兵

### 简介

* #### Sentinel简介

  随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 是面向分布式、多语言异构化服务架构的流量治理组件，主要以流量为切入点，从**流量路由、流量控制、流量整形、熔断降级、系统自适应过载保护、热点流量防护**等多个维度来帮助开发者保障微服务的稳定性。

  ​	Sentinel 是分布式系统的防御系统。

  ​	官网：`https://sentinelguard.io/zh-cn`

  ​	GitHub官网：`https://github.com/alibaba/Sentinel`

  Sentinel 的使用可以分为两个部分:

  - 核心库（Java 客户端）：不依赖任何框架/库，能够运行于 Java 8 及以上的版本的运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持（见 [主流框架适配](https://sentinelguard.io/zh-cn/docs/open-source-framework-integrations.html)）。
  - 控制台（Dashboard）：Dashboard 主要负责管理推送规则、监控、管理机器信息等。

* #### Sentinel 控制台

  > 无论是服务熔断、服务流控、服务鉴权等，都首先会涉及到相关规则的可视化管理，即通过 Sentinel 控制台来管理相关规则。所以这里首先来学习 Sentinel 控制台。

  > Sentinel  控制台是 Sentinel 的一个轻量级开源GUI控制台，可以提供读 Sentinel 主机（Sentinel应用） 的发现及健康管理、动态配置服务流控、熔断、路由规则的配置与管理。
  >
  > 实际上就是一个是spring应用

  **（1）下载Sentinel**

  ​	下载与Spring Cloud 版本相对应的Sentinel

  **（2）启动**

  ​	运行下面指令来启动

  ```
  java -Dserver.port=8888 ^
  -Dcsp.sentinel.dashboard.server=localhost:8888 ^
  -Dproject.name=sentinel-dashboard ^
  -Dsentinel.dashboard.auth.username=sentinel ^
  -Dsentinel.dashboard.auth.password=sentinel ^
  -jar sentinel-dashboard-1.8.6.jar
  ```

  二三行命令是将本身这个程序也放到Sentinel里面来进行流量监控等操作,一般可以不设置

  四五行命令是可以设置用户名和密码

  ```
  java -Dserver.port=8888 ^
  -Dsentinel.dashboard.auth.username=sentinel ^
  -Dsentinel.dashboard.auth.password=sentinel ^
  -jar sentinel-dashboard-1.8.6.jar
  ```
  
  
  
  **（3）登陆**
  
  ​	访问`http://localhost:8888`页面进行登陆
  
  ​	用户名和密码都是`sentinel`

### 服务降级

> 什么是服务降级？

服务降级是一种增强用户体验的方式。当用户的请求由于各种原因被拒后，系统返回一个事先设定好的、用户可以接受的，但又令差强人意的结果。这种请求处理方式称为服务降级。

* #### Sentinel 式方法级降级

  **（1）定义工程**

  复制`consumer-nacos`工程, 创建`06-consumer-degrade-sentinel-method-8080`工程

  引入依赖

  ```xml
  <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
  </dependency>
  ```

  **（2）创建降级方法**

  ```java
  	@SentinelResource(fallback = "getHandleFallback")
      @GetMapping("/get/{id}")
      public Depart getHandle(@PathVariable("id") int id){
          return service.getDepartById(id);
      }
      //定义 getHandle() 方法的服务降级方法，方法名前缀要它一致
  	//可以获取Throwable 参数获取异常信息
      public Depart getHandleFallback(int id,Throwable throwable){
          Depart depart = new Depart();
          depart.setId(id);
          depart.setName("degrade-sentinel-method" + throwable.getMessage());
          return depart;
      }
  ```

* #### Sentinel 式类级降级

  **（1）创建降级类**

  创建`ControllerFallback`服务降级类，然后将所有降级方法写入到这个类中，**需要注意的是：这些方法要变成静态方法。**

  **（2）修改注解内容**

  `@SentinelResource(fallback = "getHandleFallback",fallbackClass = ControllerFallback.class)`。

  在`@SentinelResource` 注解中，要加入降级类

  

* #### Feign 式的类级降级

  **（1）配置文件**

  ```yaml
  # 开启Sentinel对Feign的支持
  feign:
    sentinel:
      enabled: true
  ```

  **（2）创建降级类，然后实现Service接口**

  ```java
  @Component
  //设置要降级路径
  @RequestMapping("/fallback/provider/depart")
  public class DepartServiceFallback implements DepartService{
      @Override
      public boolean saveDepart(Depart depart) {
          System.out.println("执行 saveDepart() 降级");
          return false;
      }
  
      @Override
      public boolean removeDepartById(int id) {
          System.out.println("执行 removeDepartById() 降级");
          return false;
      }
  
      @Override
      public boolean modifyDepart(Depart depart) {
          System.out.println("执行 modifyDepart() 降级");
          return false;
      }
  
      @Override
      public Depart getDepartById(int id) {
          System.out.println("执行 getDepartById() 降级");
          Depart depart = new Depart();
          depart.setId(id);
          depart.setName("degrade-feign");
          return depart;
      }
  
      @Override
      public List<Depart> listAllDepart() {
          System.out.println("执行 listAllDepart() 降级");
          List<Depart> list = new ArrayList<>();
          Depart depart = new Depart();
          depart.setName("list-degrade-feign" );
          list.add(depart);
          return list;
      }
  }
  ```

  **（3）在Spring Cloud Alibaba2022.0.0版本**

  需要在配置文件中配置`openfeign.lazy-attributes-resolution=true`





### 熔断规则

​	熔断规则用于完成服务熔断

> 什么是服务雪崩？

![](images\soa-1-640.png)

例如上面这个场景。

第一种情况是，如果服务 I 出现异常，会导致请求阻塞；在等待响应，在等待超时之前都处于阻塞状态，并且每个阻塞请求都会占用一个线程。如果在高并发场景下发生这种问题，就会导致这些请求会迅速用完所有线程，从而导致对于其它正常的服务无法获取系统资源而被拒绝，发生系统崩溃。

另一种相似的情况是**链式的服务雪崩**。如果服务 B 需要调用服务 E，服务 E 有需要调用 服务 I。这是如果服务 I 出现异常，就会导致这一连串的服务进入阻塞状态，更加容易导致服务雪崩。

> 什么是服务熔断呢？

**为了防止服务雪崩的发生**，在发现了对某些资源请求的响应缓慢或调用异常较多时，直接将对这些资源的请求掐断一段时间。而在这段时间内的请求将不在等待超时，而是直接返回实现设定好的降级结果。这些请求将不占用系统资源，从而避免了服务雪崩的发生。这就是服务熔断。



* #### 使用

  **配置控制台信息：**

  ```yaml
  spring:
    application:
      name: depart-consumer
    cloud:
      nacos:
        discovery:
          server-addr:  localhost:8848
          username: nacos
          password: nacos
      sentinel:
        transport:
          port: 8719
          dashboard: localhost:8888
        # 开启饥饿加载 
        eager: true
  ```

  这里这个`8719`端口号会在应用对应的机器上启动一个`Http Server`，该Server 会与 Sentinel 控制台做交互。如果Sentinel控制台添加了一个限流规则，会把规则数据 push 给这个`Http Server`，`Http Server`再将规则注册到Sentinel中。

  所以这个`Http Server `作用是用来通讯的。

  运行服务器后访问`localhost:8719/api`，然后到`json.cn`，可以获取到这个服务的各种方法。

  `spring.cloud.sentinel.eager=true`，是开启饥饿加载。正常情况下，只有当`consumer `发生了请求，才会被加载到Sentinel控制台中；设置了饥饿加载之后，只要`consumer`服务一上线，就会被加载到控制台。

* #### 在Sentinel控制台添加熔断规则

  ![](images\屏幕截图 2023-11-04 164701.png)

  * **熔断策略**：慢调用比例，即最大RT超过设定值时的请求，即称为慢调用。
  * **比例阈值**，当慢调用比例在统计时间内的比例超过这个值，就会发生熔断
  * **最小请求数**，只有当在统计时间内的请求数超过最小请求数，才会启动上面策略

  **注意：这样设置的熔断规则是存储在内存中的，重启服务器后就会消失**

  系统有默认的熔断后的降级方法，可以通过之前所学的设置的降级方法来设置降级。

  可以在`@SentinelResource(value = "resourceName")`,设置资源名称，一旦添加了这个注解，默认的降级方法就会被取消，如果没有设置有降级方法，就会报500状态码。

* #### API 方式设置熔断

  在启动类中配置：

  ```java
  @LoadBalancerClients(defaultConfiguration = DepartConfig.class)
  @SpringBootApplication
  @EnableFeignClients
  public class CircuitConsumerApplication {
  
      public static void main(String[] args) {
          SpringApplication.run(CircuitConsumerApplication.class, args);
          initDegradeRule();
      }
      private static void initDegradeRule() {
          List<DegradeRule> rules = new ArrayList<>();
          DegradeRule rule = CircuitConsumerApplication.configDegradeRule();
          rules.add(rule);
          DegradeRuleManager.loadRules(rules);
      }
      private static DegradeRule configDegradeRule() {
          DegradeRule rule = new DegradeRule();
          rule.setResource("getHandle");
          // 设置熔断策略为慢调用比例
          rule.setGrade(RuleConstant.DEGRADE_GRADE_RT);
          // 指定最大RT
          rule.setCount(200);
          // 设置比例阈值
          rule.setSlowRatioThreshold(0.2);
          // 设置熔断时长
          rule.setTimeWindow(10);
          // 设置统计时长
          rule.setStatIntervalMs(1000);
          // 设置最小请求数
          rule.setMinRequestAmount(2);
          return rule;
      }
  }
  ```

* #### 异常比例和异常数

  。。。

#### 自定义异常处理器

对于系统默认的异常处理器是`DefaultBlockExceptionHandler()`和这个类中，它实现了`BlockExceptionHandler()`接口，这个接口中有一个`handle()`方法传入`HttpServletRequest`和`HttpServletResponse`两个参数来对请求和响应进行修改。还有一个`BlockException`抽象类，它有五个子类，对应这五种服务。

* ##### 自定义通用的异常处理器（返回响应流）：

```java
@Component
public class CustomBlockExceptionHandler implements BlockExceptionHandler {
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, BlockException e) throws Exception {
        response.setStatus(429);

        PrintWriter out = response.getWriter();
        String msg = "Blocked by Sentinel -";

        // 判断异常类型
        if(e instanceof FlowException){
            msg += "Flow Exception";
        } else if(e instanceof DegradeException){
            msg += "Degrade Exception";
        } else if(e instanceof SystemBlockException){
            msg += "SystemBlock Exception";
        } else if(e instanceof ParamFlowException){
            msg += "ParamFlow Exception";
        } else if(e instanceof AuthorityException){
            msg += "Authority Exception";
            // 没权限
            response.setStatus(401);
        }
        out.print(msg);
        out.flush();
        out.close();
    }
}
```

* ##### 跳转到页面

  在资源路径下创建`META-INF/resources`路径，在里面创建HTMl文件

  ```java
  @Component
  public class CustomBlockExceptionHandler2 implements BlockExceptionHandler {
      @Override
      public void handle(HttpServletRequest request, HttpServletResponse response, BlockException e) throws Exception {
          String page = "";
  
          // 判断异常类型
          if(e instanceof FlowException){
              page = "/flow.html";
          } else if(e instanceof DegradeException){
              page = "/degrade.html";
          } else if(e instanceof SystemBlockException){
              page = "/system.html";
          } else if(e instanceof ParamFlowException){
              page = "/param.html";
          } else if(e instanceof AuthorityException){
              page = "/author.html";
              // 没权限
  
          }
          //转发
          request.getRequestDispatcher(page).forward(request,response);
      }
  }
  ```

* ##### 重定向到URL

  当系统采用前后端分离模式时，使用页面跳转是不行的，此时可以使用重定向来完成。

  ```java
  @Override
      public void handle(HttpServletRequest request, HttpServletResponse response, BlockException e) throws Exception {
          String url = "";
  
          // 判断异常类型
          if(e instanceof FlowException){
              url = "https://jd.com";
          } else if(e instanceof DegradeException){
              url = "https://jd.com";
          } else if(e instanceof SystemBlockException){
              url = "https://jd.com";
          } else if(e instanceof ParamFlowException){
              url = "https://jd.com";
          } else if(e instanceof AuthorityException){
              url = "https://jd.com";
              // 没权限
          }
          response.sendRedirect(url);
      }
  ```

### 流控规则

> 流控规则是用于完成服务流控的。那么什么是服务流控呢？

服务流控即对访问流量的控制，也称为服务限流。Sentinel实现流控的原理是监控应用流量的**QPS（Queries-per-second）**或**并发线程数**等指标，当达到指定的阈值时对再到来的请求进行控制，以避免被瞬时的流量高峰冲垮，从而保障了应用的高可用性。

下面将列举三种，配置服务流控规则的方式

* #### 动态设置流控规则

  在Sentinel控制台中设置

  ![](images\屏幕截图 2023-11-04 205356.png)

  * **单机阈值**，当一台主机每秒对资源的访问超过这个数，就会拒绝访问

  对于资源配置的`SentinelResources(blockHandler="FallbackName")`注解,可以设置服务流控的异常处理方法，但是要注意的是服务流控的异常处理方法必须要带上`BlockException`参数异常，否则就无法识别到这个方法

* #### API 设置流控规则

  与熔断规则的API设置类似，不再举例

* #### 资源实体指定流控规则

  修改资源代码，即控制层请求处理代码

  ```java
  @GetMapping("/get/{id}")
      public Depart getHandle(@PathVariable("id") int id){
          Entry entry = null;
          try {
              // 设置实体名称
              entry = SphU.entry("xxx");
              return service.getDepartById(id);
          } catch (BlockException e) {
              Depart depart = new Depart();
              depart.setId(id);
              depart.setName("degrade-sentinel-method" + e.getMessage());
              return depart;
          } finally {
              if(entry != null) {
                  // 释放实体
                  entry.exit();
              }
          }
      }
  ```

  Sentinel控制台就会出现一个名字为 `xxx` 资源，可以对这个资源进行配置规则，进而完成对这个请求处理资源的规则控制；所以也可以说这时另一种命名资源方式。

  > 那么在生产环境下如何使用呢？

  当我们在 `xxx` 修改成别的资源名称，那么，这个资源名称所设置的规则，就会同样应用在这个实体所在的方法中。类似继承一样。

* #### 并发线程数流控方案

  流量控制有两种阈值统计类型，除了 QPS 外，另一种是统计并发线程数。

  > 这两种的区别？
  >
  > QPS是有时间条件的，就是在一秒钟内请求的数量。
  >
  > 而并发线程数只允许只有设定好的线程的数量在同时执行。

  **（1）线程隔离方案**

  ​	并发线程数流控方案通常是对消费者端的配置。是为了避免由于**慢调用**而将消费者端线程耗尽的情况的发生，业内会使用线程隔离方案。一般来说可以分为两种：

  ![](images\屏幕截图 2023-11-04 203442.png)

  ##### A、线程池隔离

  **系统为不同的提供者资源设置不同的线程池来隔离业务自身之间的资源争抢**。该方案隔离性较好，但需要创建的线程池及线程数量太多，系统消耗较大。当请求线程达到后，会从线程池中获取到一个新的执行线程去完成提供者的调用。由请求线程到执行线程的上下文切换时间开销较大。特别是对低延时的调用有比较大的影响。

  ##### B、信号量隔离

  **系统为不同的提供者设置不同的计数器**。每增加一个该资源的调用请求，计数器就变化一次。当达到该计数器阈值时，再来的请求将被限流。该方式的执行线程与请求线程是同一个线程，不存在线程上下文切换的问题，更不存在很多的线程池创建与线程创建问题。也正因为请求线程与执行线程没有分离，所以，其对于提供者的调用无法实现异步，执行效率低，且对于依赖资源的执行超时控制不方便。

  **（2）Sentinel 隔离方案**

  Sentinel  并发线程数控制也属于隔离方案，但不同于以上两种隔离方式，是对以上两种方案的综合与改进，或者说更像是线程池隔离。

  **其也将请求线程与执行线程进行了分离，但 Sentinel 不负责创建和管理线程池，而仅仅是简单统计当前资源请求占用的线程数目。**如果对该资源的请求占用的线程数量超过了阈值，则可以立即拒绝再新进入的请求。

* #### 流控模式

  **（1）关联流控模式**

  ​	当关联资源的访问超过了自己设置阈值，就会则会对自己当前资源的进行限流。

  **（2）链路流控模式**

  ​	当一个资源有多钟访问路径时，可以对某一路径的访问进行限流，而其它访问路径不限流。

  将资源的uri放到入口资源后，就可以完成对这个资源的限流；例如`/provider/depart/list`, 而不会影响到`/provider/depart/all`

  然后要在配置文件中更改配置：

  `spring.cloud.sentinel.web-context-unify=false`

  默认（true）会收敛所有URL入口，即对于同一资源的请求URL到了Sentinel后会自动统一（unify）为一个URL。即Sentinel只关心被访问资源，不区分请求来源。

  设置为false后，就可以区分请求来源了

* #### 流控效果

  ##### （1）快速失败

  ​	默认的设置，会直接返回错误页面。

  ##### （2）Warm Up

  ​	当系统长期处理低水位的情况下，当流量突然增加时，直接把系统拉升到高水位可能瞬间把系统压垮。通过“冷启动“，让通过的流量缓慢增加，在一定时间内逐渐增加到阈值上限。在达到QPS高水位后，在再超出阈值设定的QPS值时，将对请求执行“快速失败”，即进行降级处理。

  ​	Sentinel 的 Warm Up 流控算法是在 Guava 的`Smooth WarmingUP`算法基础上改进的。`SmoothWarmingUP`算法继承自` SmoothRateLimiter` 算法, 即限流的算法。但是算法实现思路是不同的。

  *  `SmoothWarmingUp` 算法是通过不断缩短请求间的间隔来达到逐步提高访问量的目的的
  * Sentinel 的 Warm Up 算法是通过逐步提升 QPS 来到达提高访问量的目的的。

  ##### （3）排队等待

  ​	排队等待，也称为匀速排队。该方式会严格控制请求通过的时间间隔，让请求以均匀的速度通过。其是漏斗算法的改进。不同的是，当流量超过设定阈值时，漏斗算法会直接将再来的请求给丢弃，而排队等待算法则是将请求缓存起来，后面慢慢处理。不过，该算法目前暂不支持 QPS 超过 1000 的场景。

  ​	其适合处理**间隔性**突发流量场景（削峰填谷）。

  ​	设置超时超时时间是指存储在队列的请求的时间。

* #### 来源流控

  > 在配置流控规则时，针对来源的属性有什么用呢？

  **（1）概述**

  ​	来源流控是针对请求发出者的来源名称所进行的流控。在流控规则中可以直接指定该规则仅用于限制指定来源的请求，一条规则仅可以指定一个限流的来源。

  **（2）原理**

  ​	来源名称可以在请求参数、请求头或 Cookie 中通过 key-value 形式指定，而 sentinel 提供了一个 `RequestOriginParser`的请求解析器，sentinel 可以从该解析器中获取到请求中携带的来源，然后将流控规则应用于获取到的请求来源之上。

  ##### （3）实现

  创建自定义解析器，实现 `RequestOriginParser`，的`parseOrigin()`方法

  ```java
  @Component
  public class DepartRequestOriginParser implements RequestOriginParser {
  
      // 假设请求源标识是以请求参数形式出现
      // Sentinel 会自动根据parserOrigin()方法返回结果来判断请求源
      @Override
      public String parseOrigin(HttpServletRequest request) {
          // 获取请求参数source的值，即请求源标识
          String source = request.getParameter("source");
          // 若请求中没有携带source请求参数则默认请求源为sa
          if(!StringUtils.hasText(source)){
              source = "as";
          }
          return source;
      }
  }
  ```

  ##### （5）配置

  ​	`default`，表示所有资源。`other`表示除了小范围的来源的其它所有来源，但是其它设置要相同。

  

### 授权规则

> 授权规则是一种通过对请求来源进行甄别的鉴权规则。规则规定了哪些请求可以通过访问，而哪些请求则是被拒绝访问的。而这些请求的设置是通过黑白名单来完成的。
>
> 无论黑名单还是白名单，其实就是一种请求来源名称列表。

![](images\屏幕截图 2023-11-04 221048.png)

只要指定了白名单，其它没进白名单的就是黑名单。非黑即白。

API配置与熔断规则类似。



### 热点规则

> 上面我们所学的规则，都是对方法的限流。而热点规则是用于实现热点参数限流的规则。
>
> 热点参数限流指的是，在流控规则中指定对某方法参数的QPS限流后，当所有对该资源的请求URL中携带有指定参数的请求QPS达到了阈值，则发生限流。

创建一个简单工程：

```java
@RestController
public class DepartController {


    @SentinelResource(value = "paramHandle")
    @GetMapping("/param")
    public String paramHandle(Integer id, String name){
            return "param flow:" + id + "," + name;
    }

    //定义 getHandle() 方法的服务降级方法，方法名前缀要它一致
    public String paramHandleFallback(Integer id, String name){
        return "fallback:" + id + "," + "name";
    }

}
```

配置参数：

![](images\屏幕截图 2023-11-04 224457.png)

* 参数索引：输入要限流的热点参数，从零开始；例如在示例工程中，0表示第一个参数，即 `id`
* 统计窗口时长：这里翻译有些错误。表示超过阈值限流后，要多久能够恢复服务。
* 参数例外项：设置例外的参数值，可以单独对特殊的值设置限流阈值。

访问`localhost:8080/param?id=123`,会发生限流，如果不加`id`参数，就不会发生限流。

API方式与其它规则类似

### 系统规则

> Sentinel 系统自适应过载保护从整体维度对应用入口流量进行控制，结合应用的Load、CPU 使用率、总体平均 RT、入口 QPS、和并发线程数等几个维度的监控指标，通过自适应的流控策略，让系统的入口流量和系统的负载达到一个平衡，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

* #### 背景

  长期以来，系统保护的思路是根据**硬指标**，即系统的**负载 (load1) **来做系统过载保护。当系统负载高于某个阈值，就禁止或者减少流量的进入；当 load 开始好转，则恢复流量的进入。这个思路给我们带来了不可避免的两个问题：

  - load 是一个“结果”，如果根据 load 的情况来调节流量的通过率，那么就始终有延迟性。也就意味着通过率的任何调整，都会过一段时间才能看到效果。当前通过率是使 load 恶化的一个动作，那么也至少要过 1 秒之后才能观测到；同理，如果当前通过率调整是让 load 好转的一个动作，也需要 1 秒之后才能继续调整，这样就浪费了系统的处理能力。所以我们看到的曲线，总是会有抖动。
  - 恢复慢。想象一下这样的一个场景（真实），出现了这样一个问题，下游应用不可靠，导致应用 RT 很高，从而 load 到了一个很高的点。过了一段时间之后下游应用恢复了，应用 RT 也相应减少。这个时候，其实应该大幅度增大流量的通过率；但是由于这个时候 load 仍然很高，通过率的恢复仍然不高。

  > 什么是load1？

  load1是指在linux系统中的负载量。在linux系统中，执行命令`uptime`，会得到三个平均负载数值：分别是过去的一分钟的负载、过去的五分钟的平均负载、过去的十五分钟的平均负载；分别代表着load1、load2、load3

  **TCP BBR（拥塞控制算法）**思想给了一个很大的启发。**在系统不被拖垮的情况下，提高系统的吞吐率，而不是 load 一定要到低于某个阈值**

  

![](images\屏幕截图 2023-11-04 230302.png)

- **Load 自适应**（仅对 Linux/Unix-like 机器生效）：系统的 load1 作为启发指标，进行自适应系统保护。当系统 load1 超过设定的启发值，且系统当前的并发线程数超过估算的系统容量时才会触发系统保护（BBR 阶段）。系统容量由系统的 `maxQps * minRt` 估算得出。设定参考值一般是 `CPU cores * 2.5`。
- **CPU usage**（1.5.0+ 版本）：当系统 CPU 使用率超过阈值即触发系统保护（取值范围 0.0-1.0），比较灵敏。
- **平均 RT**：当单台机器上所有入口流量的平均 RT 达到阈值即触发系统保护，单位是毫秒。
- **并发线程数**：当单台机器上所有入口流量的并发线程数达到阈值即触发系统保护。
- **入口 QPS**：当单台机器上所有入口流量的 QPS 达到阈值即触发系统保护。



### 网关流控

> Sentinel 将网关中的**路由route和请求的URL**作为Sentinel资源来进行限流的。

* #### 概述

  ##### （1）公共适配器

  从 `Sentinel1.6.0`开始，Sentinel对于主流网关限流的实现，是通过 `Sentinel API Gateway Adapter Common` 这个公共适配器模块实现的。

  ##### （2）限流维度

  这个公共适配器模块提供了两种维度的限流：

  * Route维度：根据网关路由中指定的路由id进行路由
  * API维度：使用Sentinel提供的API自定义分组进行限流

* #### 动态设置--Route 维度

  ##### （1）创建工程

  ##### （2）导入依赖

  ```xml
  		<!-- sentinel 和 gateway 整合的依赖 -->
  		<!-- 在之前的版本，导入的是adapter依赖 ,并且还要导入配置类-->
  		<dependency>
              <groupId>com.alibaba.cloud</groupId>
              <artifactId>spring-cloud-alibaba-sentinel-gateway</artifactId>
          </dependency>
          <dependency>
              <groupId>com.alibaba.cloud</groupId>
              <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
          </dependency>
  ```

  ##### （3）配置文件

  ```yaml
  server:
    port: 9000
  
  spring:
    application:
      name: depart-gateway
  
    cloud:
      nacos:
        discovery:
          username: nacos
          server-addr: localhost:8848
          password: nacos
  
      gateway:
        discovery:
          locator:
            enabled: true
        routes:
          - id: get_route
            uri: lb://depart-consumer
            predicates:
              - Path=/consumer/depart/get/**
  
          - id: list_route
            uri: lb://depart-consumer
            predicates:
              - Path=/consumer/depart/list
      sentinel:
        transport:
          port: 8719
          dashboard: localhost:8888
        eager: true
        filter:
          enabled: false
  ```

  ##### （4）添加流控规则

  ![](images\屏幕截图 2023-11-05 161358.png)

  * **Burst size:** 突发的流量增加时，会是QPS值会增加这个数值

  发现发生限流时的界面并不友好，所以下面要学习如何替换错误页面。

  

* #### 修改阻断异常结果

  API方式的，在启动类中配置：

  ```java
  @SpringBootApplication
  public class SentinelGateway9000 {
  
      public static void main(String[] args) {
          SpringApplication.run(SentinelGateway9000.class, args);
          initBlockHandler();
      }
  
      private static void initBlockHandler() {
          // 设置阻断异常结果，设置成重定向策略，将会重定向到百度首页
          // GatewayCallbackManager.setBlockHandler(new RedirectBlockRequestHandler("https://baidu.com"));
          
          // 自定义异常结果
          GatewayCallbackManager.setBlockHandler((exchange, th) -> {
              Map<String, Object> map = new HashMap<>();
              map.put("uri", exchange.getRequest().getURI());
              map.put("msg", "访问量过大，稍后请重试");
              map.put("status", 429);
              return ServerResponse.status(HttpStatus.TOO_MANY_REQUESTS)
                      .contentType(MediaType.APPLICATION_JSON)
                      .body(BodyInserters.fromValue(map));
      }
  }
  ```

  配置文件方式的：

  ```yaml
      sentinel:
        transport:
          port: 8719
          dashboard: localhost:8888
        eager: true
        filter:
          enabled: false
        scg:
          fallback:
            # redirect: https://baidu.com
            # mode: redirect
            
            mode: response
            response-status: 429
            content-type: 'application/json'
            response-body: '{"msg":"访问量过大","status:"429"}'
  ```

  ##### 优先级：

  当代码中与配置文件中都设置了限流阻断异常结果时，代码中的优先级更高。

* #### 针对路由断言的流控

  ![](images\屏幕截图 2023-11-05 202522.png)

  ##### （1）三种匹配模式

  * 精确：请求中的某个属性值与设置的“匹配串”值完全相同时才会应用流控规则
  * 字串：请求中的某个属性值包含设置的“匹配串”值时才会应用流控规则
  * 正则：请求中的某个属性值符合设置的“匹配串”中的正则表达式

  ##### （2）Client IP

  ​	这里并不是针对提交请求的客户端的IP进行流控，而是针对网关  IP 进行流控。因为对于微服务来说，网关就相当于它们的 Client。

  ​	那么，该设置有什么用呢？生产中的 Spring Cloud Gateway 网关一定是集群，集群的每个节点服务器都会具有不同的IP。每个用户请求都会通过 Nginx 被路由到不同的网关服务器。而该配置就可以针对用户提交请求锁经过不同的网关集群节点服务器进行流控

* #### API 维度的限流

  先要在 API 管理中增加API管理。需要填写**API名称**和**匹配规则**

  然后在流控规则中选择API名称来进行管理。

  

* #### API 设置 Route 维度网关流控

  与之前API方式类似

  ```java
  @SpringBootApplication
  public class SentinelGateway9000 {
  
      public static void main(String[] args) {
          SpringApplication.run(SentinelGateway9000.class, args);
          //initBlockHandler();
          initGatewayRule();
      }
  
      private static void initGatewayRule() {
          Set<GatewayFlowRule> rules = new HashSet<>();
          GatewayFlowRule rule = SentinelGateway9000.configGatewayFlowRule();
          GatewayRuleManager.loadRules(rules);
      }
  
      private static GatewayFlowRule configGatewayFlowRule() {
          GatewayFlowRule rule = new GatewayFlowRule();
          rule.setResourceMode(SentinelGatewayConstants.RESOURCE_MODE_ROUTE_ID);
          rule.setResource("list_route");
          // 设置限流规则
          rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
          rule.setCount(3);
          rule.setIntervalSec(1);
          rule.setControlBehavior(RuleConstant.CONTROL_BEHAVIOR_DEFAULT);
          rule.setMaxQueueingTimeoutMs(500);
  
          GatewayParamFlowItem item = new GatewayParamFlowItem();
          // 设置针对的“参数属性”类型为“URL参数”
          item.setParseStrategy(SentinelGatewayConstants.PARAM_PARSE_STRATEGY_URL_PARAM);
          // 设置针对的“URL参数”名称
          item.setFieldName("name");
          // 设置匹配模式为子串
          item.setMatchStrategy(SentinelGatewayConstants.PARAM_MATCH_STRATEGY_CONTAINS);
          // 设置匹配规则
          item.setPattern("ang");
          rule.setParamItem(item);
          
          return rule;
      }
  }
  ```

  

* #### API设置 API 维度网关流控

  略
  
  

### 数据源扩展

要实现规则的持久化，有下面三种模式：

| 推送模式 | 说明                                                         | 优点                         | 缺点                                                         |
| -------- | ------------------------------------------------------------ | ---------------------------- | ------------------------------------------------------------ |
| 原始模式 | API 将规则推送至客户端并直接更新到内存中，扩展写数据源（`WritableDataSource`） | 简单                         | 不保证一致性，规则保存在内存中，修改不会保存，重启即消失。严重不建议 |
| Pull模式 | 扩展写数据源（`WritableDataSource`），客户端主动向某个规则管理中心定期轮询拉取规则，这个规则中心可以是RDBMS、文件等。**（由Sentinel DashBoard 来对本地文件的更新）** | 简单，无任何依赖；规则持久化 | 不保证一致性**（因为拉取是定时的）**；实时性不保证，拉取过于频繁会影响性能 |
| Push模式 | 扩展读数据源（`ReadableDataSource`），规则中心统一推送，客户端通过注册监听器的方式时刻监听变化，比如使用 Nacos、Zookeeper 等配置中心。这种方式有更好的实时性和一致性保证。**生产环境下一般采用 push 模式的数据源。** | 规则持久；一致性；快速       | 引入第三方依赖                                               |



#### Pull 模式

**（1）定义`FileDataSourceInit`类**

```java
public class FileDataSourceInit implements InitFunc {

    @Override
    public void init() throws Exception {
        // 在当前工程根目录下创建/rules目录
        File ruleFileDir = new File(System.getProperty("user.dir") + "/rules");
        // 创建目录
        if(!ruleFileDir.exists()){
            ruleFileDir.mkdirs();
        }
        readWriteRulerFileFlow(ruleFileDir.getPath());
        readWriteRulerFileDegrade(ruleFileDir.getPath());
        readWriteRulerFileAuthority(ruleFileDir.getPath());
        readWriteRulerFileParamFlow(ruleFileDir.getPath());
        readWriteRulerFileSystem(ruleFileDir.getPath());

    }

    //流控规则
    private void readWriteRulerFileFlow(String ruleFilePath) throws IOException {
        String ruleFile = ruleFilePath + "/flow-rule.json";
        createRuleFile(ruleFile);

        ReadableDataSource<String, List<FlowRule>> ds = new FileRefreshableDataSource<>(
                ruleFile, source -> JSON.parseObject(source, new TypeReference<List<FlowRule>>() {})
        );
        // 将可读数据源注册至 FlowRuleManager.
        FlowRuleManager.register2Property(ds.getProperty());

        WritableDataSource<List<FlowRule>> wds = new FileWritableDataSource<>(ruleFile, this::encodeJson);
        // 将可写数据源注册至 transport 模块的 WritableDataSourceRegistry 中.
        // 这样收到控制台推送的规则时，Sentinel 会先更新到内存，然后将规则写入到文件中.
        WritableDataSourceRegistry.registerFlowDataSource(wds);
    }

    // 降级规则
    private void readWriteRulerFileDegrade(String ruleFilePath) throws IOException {
        String ruleFile = ruleFilePath + "/degrade-rule.json";
        createRuleFile(ruleFile);

        ReadableDataSource<String, List<DegradeRule>> ds = new FileRefreshableDataSource<>(
                ruleFile, source -> JSON.parseObject(source, new TypeReference<List<DegradeRule>>() {})
        );
        // 将可读数据源注册至 FlowRuleManager.
        DegradeRuleManager.register2Property(ds.getProperty());

        WritableDataSource<List<DegradeRule>> wds = new FileWritableDataSource<>(ruleFile, this::encodeJson);
        // 将可写数据源注册至 transport 模块的 WritableDataSourceRegistry 中.
        // 这样收到控制台推送的规则时，Sentinel 会先更新到内存，然后将规则写入到文件中.
        WritableDataSourceRegistry.registerDegradeDataSource(wds);
    }

    // Authority
    private void readWriteRulerFileAuthority(String ruleFilePath) throws IOException {
        String ruleFile = ruleFilePath + "/Authority-rule.json";
        createRuleFile(ruleFile);

        ReadableDataSource<String, List<AuthorityRule>> ds = new FileRefreshableDataSource<>(
                ruleFile, source -> JSON.parseObject(source, new TypeReference<List<AuthorityRule>>() {})
        );
        // 将可读数据源注册至 FlowRuleManager.
        AuthorityRuleManager.register2Property(ds.getProperty());

        WritableDataSource<List<AuthorityRule>> wds = new FileWritableDataSource<>(ruleFile, this::encodeJson);
        // 将可写数据源注册至 transport 模块的 WritableDataSourceRegistry 中.
        // 这样收到控制台推送的规则时，Sentinel 会先更新到内存，然后将规则写入到文件中.
        WritableDataSourceRegistry.registerAuthorityDataSource(wds);
    }
    // System
    private void readWriteRulerFileSystem(String ruleFilePath) throws IOException {
        String ruleFile = ruleFilePath + "/System-rule.json";
        createRuleFile(ruleFile);

        ReadableDataSource<String, List<SystemRule>> ds = new FileRefreshableDataSource<>(
                ruleFile, source -> JSON.parseObject(source, new TypeReference<List<SystemRule>>() {})
        );
        // 将可读数据源注册至 FlowRuleManager.
        SystemRuleManager.register2Property(ds.getProperty());

        WritableDataSource<List<SystemRule>> wds = new FileWritableDataSource<>(ruleFile, this::encodeJson);
        // 将可写数据源注册至 transport 模块的 WritableDataSourceRegistry 中.
        // 这样收到控制台推送的规则时，Sentinel 会先更新到内存，然后将规则写入到文件中.
        WritableDataSourceRegistry.registerSystemDataSource(wds);
    }

    // ParamFlow
    private void readWriteRulerFileParamFlow(String ruleFilePath) throws IOException {
        String ruleFile = ruleFilePath + "/param-rule.json";
        createRuleFile(ruleFile);

        ReadableDataSource<String, List<ParamFlowRule>> ds = new FileRefreshableDataSource<>(
                ruleFile, source -> JSON.parseObject(source, new TypeReference<List<ParamFlowRule>>() {})
        );
        // 将可读数据源注册至 FlowRuleManager.
        ParamFlowRuleManager.register2Property(ds.getProperty());

        WritableDataSource<List<ParamFlowRule>> wds = new FileWritableDataSource<>(ruleFile, this::encodeJson);
        // 将可写数据源注册至 transport 模块的 WritableDataSourceRegistry 中.
        // 这样收到控制台推送的规则时，Sentinel 会先更新到内存，然后将规则写入到文件中.
        ModifyParamFlowRulesCommandHandler.setWritableDataSource(wds);
    }
    
    private void createRuleFile(String ruleFile) throws IOException {
        File file = new File(ruleFile);
        // 创建文件
        if(!file.exists()){
            file.createNewFile();
        }
    }

    private <T> String encodeJson(T t) {
        // 将对象转换成json串
        return JSON.toJSONString(t);
    }
}
```



**（2）定义 SPI 接口文件**

​	Pull模式的 Datasource 扩展采用的是 **SPI （Service Provider Interface）服务发现机制**。按照 SPI 规范，需要在`/resources`目录下新建 `META-INF/services` 目录，然后在该目录下创建一个文本文件。文件名为扩展类实现接口的全限定性接口名，文件内容则为该接口的 实现类的全限定性类名。



#### Push 模式持久化



* ##### 引入第三方依赖

  ```xml
  		<dependency>
              <groupId>com.alibaba.csp</groupId>
              <artifactId>sentinel-datasource-nacos</artifactId>
          </dependency>
  ```

* ##### 修改配置文件

  ```yaml
  spring:
    application:
      name: depart-consumer
    cloud:
      nacos:
        discovery:
          server-addr:  localhost:8848
          username: nacos
          password: nacos
      sentinel:
        transport:
          port: 8719
          dashboard: localhost:8888
        eager: true
        datasource:
          flows:   # 这个名字可以随便起
            nacos:
              server-addr: localhost:8848
              rule-type: flow
              data-id: ${spring.application.name}-flow-rules
              data-type: json
              username: nacos
              password: nacos
          degrades:
            nacos:
              server-addr: localhost:8848
              rule-type: degrade
              data-id: ${spring.application.name}-degrade-rules
              data-type: json
              username: nacos
              password: nacos
  ```

  

* #### Nacos 配置中心

  配置flow规则：

  ```json
  [
      {
          "clusterMode":false,
          "controlBehavior":0,
          "count":500,
          "grade":1,
          "limitApp":"default",
          "maxQueueingTimeMs":20000,
          "refResource":"/provider/depart/all",
          "resource":"getHandle",
          "strategy":0,
          "warmUpPeriodSec":5
      }
  ]
  ```



**用这种方式进行持久化只能在 Nacos 中修改配置文件，若在 Sentinel 控制台中修改文件，不会保存在 Nacos 配置中心中。**

**想要实现在 Sentinel 控制台的修改也能实现保存，就要修改 Sentinel 的原码，对 Sentinel 进行改造哦。**



#### 改造 Sentinel控制台源码

**（1）下载源码并打开源码文件**

​	找到`Sentinel-dashboard`文件下，在这个目录下的`pom.xml`文件中找到Nacos数据源依赖

```xml
		<dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
            <scope>test</scope>
        </dependency>
```

​	去掉`<scpoe>`,标签；因为有这个标签，依赖就不会打包到文件里

**（2）**

​	在test中找到`/test/java/com.alibaba.csp.sentinel.dashboard/rule/nacos`文件，将整个文件复制到类路径的 `rule` 目录中。

**（3）在修改 NacosConfig 配置类**

```java
@Configuration
public class NacosConfig {

    @Value("${nacos.addr}")
    private String addr;

    @Value("${nacos.username}")
    private String username;

    @Value("${nacos.password}")
    private String password;

    @Bean
    public Converter<List<FlowRuleEntity>, String> flowRuleEntityEncoder() {
        return JSON::toJSONString;
    }

    @Bean
    public Converter<String, List<FlowRuleEntity>> flowRuleEntityDecoder() {
        return s -> JSON.parseArray(s, FlowRuleEntity.class);
    }

    @Bean
    public ConfigService nacosConfigService() throws Exception {
        Properties properties = new Properties();
        properties.put(PropertyKeyConst.SERVER_ADDR, addr);
        properties.put(PropertyKeyConst.USERNAME, username);
        properties.put(PropertyKeyConst.PASSWORD, password);
        return ConfigFactory.createConfigService(properties);
    }
}
```

**（4）更换控制类**

​	原来的流控制类是`FlowControllerV1`， 我们要更换到`FlowControllerV2`，我们需要将我们刚复制的nacos文件中的 provider和publisher注入到V2类中。

![](images\屏幕截图 2023-11-08 142839.png)

**（5）更改页面**

​	找到`src/main/webapp/resources/app/scripts/directives/sidebar/sidebar.html/`

找到标签：

```html
		<li ui-sref-active="active" ng-if="!entry.isGateway">
            <a ui-sref="dashboard.flowV1({app: entry.app})">
              <i class="glyphicon glyphicon-filter"></i>&nbsp;&nbsp;流控规则</a>
          </li>
```

将`dashboard.flowV1` 改成 `dashboard.flow`;

 另一种方法就是将这个标签上面的注释的标签加入。

下面我们选择第二种方法。

**（6）将 `sentinel-dashboard `打包**

![](images\屏幕截图 2023-11-08 143520.png)

打包成功后，在target目录下找到`sentinel-dashboard.jar`文件

**（7）Nacos 适配的 dataId 和 groupId 约定**

* groupId：SENTINEL_GROUP
* 流控规则 dataId：`{appName}-flow-rules`

在 Nacos 配置中心配置，并且还要在服务配置文件里修改引用规则的`GroupId`



现在这种修改源码，只能对熔断规则进行更改其它规则并没有改变，这种方法只是用来**测试**的，所以要要用别的方法改造。



#### 生产目的的改造

> 我们可以根据官方给出的 FlowRule 的案例，自己创建出其它的规则类

* ##### NacosConfig 配置类

  里面有三个方法：`flowRuleEntityEncoder()`、`flowRuleEntityDecoder()`、`nacosConfigService()`.

  * `nacosConfigService()`: 调用这个类会生成一个`ConfigService`类，即配置中心，所以我们在这个方法中配置信息。

    然后有一个配置工厂类`ConfigFactory`，可以用来生成`ConfigService`类，传入一个属性类

  * `flowRuleEntityEncoder()`：生成一个转换器实例：将流控规则实体列表转换为JSON串

  * `flowRuleEntityDecoder()`：生成一个转换器实例：将JSON串转换为流控规则实体列表

  在这个类中只实现了熔断规则的转化器实例，所以我们要自己创建出其它规则的转化器。

  ```java
  	@Bean
      public Converter<List<FlowRuleEntity>, String> flowRuleEntityEncoder() {
          return JSON::toJSONString;
      }
      // 生成一个转换器实例：将JSON串转换为流控规则实体列表
      @Bean
      public Converter<String, List<FlowRuleEntity>> flowRuleEntityDecoder() {
          return s -> JSON.parseArray(s, FlowRuleEntity.class);
      }
  
      // 授权规则 authorityRule
      // 生成一个转换器实例：将流控规则实体列表转换为JSON串
      @Bean
      public Converter<List<AuthorityRuleEntity>, String> authorityRuleEntityEncoder() {
          return JSON::toJSONString;
      }
      // 生成一个转换器实例：将JSON串转换为流控规则实体列表
      @Bean
      public Converter<String, List<AuthorityRuleEntity>> authorityRuleEntityDecoder() {
          return s -> JSON.parseArray(s, AuthorityRuleEntity.class);
      }
  
      // 系统规则 systemRule
      // 生成一个转换器实例：将流控规则实体列表转换为JSON串
      @Bean
      public Converter<List<SystemRuleEntity>, String> systemRuleEntityEncoder() {
          return JSON::toJSONString;
      }
      // 生成一个转换器实例：将JSON串转换为流控规则实体列表
      @Bean
      public Converter<String, List<SystemRuleEntity>> systemRuleEntityDecoder() {
          return s -> JSON.parseArray(s, SystemRuleEntity.class);
      }
  
      // 热点参数流控规则 paramFlowRule
      // 生成一个转换器实例：将流控规则实体列表转换为JSON串
      @Bean
      public Converter<List<ParamFlowRuleEntity>, String> paramFlowRuleEntityEncoder() {
          return JSON::toJSONString;
      }
      // 生成一个转换器实例：将JSON串转换为流控规则实体列表
      @Bean
      public Converter<String, List<ParamFlowRuleEntity>> paramFlowRuleEntityDecoder() {
          return s -> JSON.parseArray(s, ParamFlowRuleEntity.class);
      }
  ```

* ##### `FlowRuleNacosProvider` 和`FlowRuleNacosPublisher`读写类

  `FlowRuleNacosProvider`类的作用是用来读取 Nacos 配置中的内容的；而`FlowRuleNacosPublisher`的作用是将 `Sentinel`控制台的修改发布到 Nacos 配置中心中。所以称为 **读写类**。

  * `FlowRuleNacosProvider`：实现了` DynamicRuleProvider<T>`接口,这个接口只有一个方法，就是`getRules()`,会返回指定的泛型，也就是不同规则实体列表。

    ```java
    @Component("flowRuleNacosProvider")
    public class FlowRuleNacosProvider implements DynamicRuleProvider<List<FlowRuleEntity>> {
    
        @Autowired
        private ConfigService configService;
        @Autowired
        private Converter<String, List<FlowRuleEntity>> converter;
    
        @Override
        public List<FlowRuleEntity> getRules(String appName) throws Exception {
            // 从 Nacos 配置中心读取SENTINEL_GROUP分组中指定名称的配置文件的内容，该内容是JSON串
            String rules = configService.getConfig(appName + NacosConfigUtil.FLOW_DATA_ID_POSTFIX,
                NacosConfigUtil.GROUP_ID, 3000);
            if (StringUtil.isEmpty(rules)) {
                return new ArrayList<>();
            }
            // 将JSON串转换为规则实例列表
            return converter.convert(rules);
        }
    }
    ```

  * `FlowRuleNacosPublisher`：实现了`DynamicRulePublisher<T>`，只有一个方法`publish()`，将规则发布到指定的微服务

    ```java
    @Component("flowRuleNacosPublisher")
    public class FlowRuleNacosPublisher implements DynamicRulePublisher<List<FlowRuleEntity>> {
    
        @Autowired
        private ConfigService configService;
        // 将规则实体列表转化为JSON串
        @Autowired
        private Converter<List<FlowRuleEntity>, String> converter;
    
        @Override
        public void publish(String app, List<FlowRuleEntity> rules) throws Exception {
            AssertUtil.notEmpty(app, "app name cannot be empty");
            if (rules == null) {
                return;
            }
            // 将Sentinel Dashboard内存库中指定服务的规则实体列表持久化到Nacos配置中心的指定配置文件中
            configService.publishConfig(app + NacosConfigUtil.FLOW_DATA_ID_POSTFIX,
                NacosConfigUtil.GROUP_ID, converter.convert(rules));
        }
    }
    ```

    在`NacosConfigUtil`配置工具类中的`GROUP_ID`默认参数只定义了两种，`-flow-rules`和`-param-rules`；所以需要我们自己定义。

    ```java
    public static final String FLOW_DATA_ID_POSTFIX = "-flow-rules";
        public static final String PARAM_FLOW_DATA_ID_POSTFIX = "-param-rules";
        public static final String AUTHORITY_FLOW_DATA_ID_POSTFIX = "-authority-rules";
        public static final String SYSTEM_FLOW_DATA_ID_POSTFIX = "-system-rules";
        public static final String DEGRADE_FLOW_DATA_ID_POSTFIX = "-degrade-rules";
    ```

  修改不同的规则类名即可。

* ##### 修改处理器`FlowControllerV2`类

  根据官方给出的`V2`修改案例，来创建其它规则的处理器。

  * 查看`FlowControllerV2`。这个类中有几个关键的属性：

    `InMemoryRuleRepositoryAdapter<T> repository`，即内存的规则仓库。里面有三个map集合来存储不同的数据。然后又很多方法可以对内存规则库进行增删改查。

    ```java
    	// 规则内存库，与当前Sentinel Dashboard 有心跳的所有的机器包含的所有实体规则。
        // key：机器名，value是一个内置map。表示该机器包含的所有规则实体
        private Map<MachineInfo, Map<Long, T>> machineRules = new ConcurrentHashMap<>(16);
    
        // 规则内存库：当前 Sentinel Dashboard 所包含的所有规则实体
        // key:规则实体id，value：规则实体
        private Map<Long, T> allRules = new ConcurrentHashMap<>(16);
    
        // 规则内存库：当前 Sentinel Dashboard 所包含的所有服务的所有规则实体
        // key:服务名称，value是一种内置map，也就是说是上面那个map。表示这个服务所包含的所有规则实体，也就是说是上面那个map
    	private Map<String, Map<Long, T>> appRules = new ConcurrentHashMap<>(16);
    ```

  * 在控制类中注入之前修改后的读写类

    ```java
    	@Autowired
        @Qualifier("flowRuleNacosProvider")
        private DynamicRuleProvider<List<FlowRuleEntity>> ruleProvider;
        @Autowired
        @Qualifier("flowRuleNacosPublisher")
        private DynamicRulePublisher<List<FlowRuleEntity>> rulePublisher;
    ```

  * 控制类源码

    ```java
    @RestController
    @RequestMapping(value = "/v2/flow")
    public class FlowControllerV2 {
    
        private final Logger logger = LoggerFactory.getLogger(FlowControllerV2.class);
    
        @Autowired
        private InMemoryRuleRepositoryAdapter<FlowRuleEntity> repository;
    
        @Autowired
        @Qualifier("flowRuleNacosProvider")
        private DynamicRuleProvider<List<FlowRuleEntity>> ruleProvider;
        @Autowired
        @Qualifier("flowRuleNacosPublisher")
        private DynamicRulePublisher<List<FlowRuleEntity>> rulePublisher;
    
    
        // 当在Sentinel Dashboard 侧边栏上互动时，就会调用这个方法，就会将对应的服务名传入
        @GetMapping("/rules")
        @AuthAction(PrivilegeType.READ_RULE)
        public Result<List<FlowRuleEntity>> apiQueryMachineRules(@RequestParam String app) {
    
            if (StringUtil.isEmpty(app)) {
                return Result.ofFail(-1, "app can't be null or empty");
            }
            try {
                // 根据服务名称从Nacos配置中心获取对应的配置文件信息
                List<FlowRuleEntity> rules = ruleProvider.getRules(app);
                // 集群相关
                if (rules != null && !rules.isEmpty()) {
                    for (FlowRuleEntity entity : rules) {
                        entity.setApp(app);
                        if (entity.getClusterConfig() != null && entity.getClusterConfig().getFlowId() != null) {
                            entity.setId(entity.getClusterConfig().getFlowId());
                        }
                    }
                }
                // 将添加过集群的信息的规则实体列表写入内存库
                rules = repository.saveAll(rules);
                return Result.ofSuccess(rules);
            } catch (Throwable throwable) {
                logger.error("Error when querying flow rules", throwable);
                return Result.ofThrowable(-1, throwable);
            }
        }
    
        // 逻辑判断
        private <R> Result<R> checkEntityInternal(FlowRuleEntity entity) {
            if (entity == null) {
                return Result.ofFail(-1, "invalid body");
            }
            if (StringUtil.isBlank(entity.getApp())) {
                return Result.ofFail(-1, "app can't be null or empty");
            }
            if (StringUtil.isBlank(entity.getLimitApp())) {
                return Result.ofFail(-1, "limitApp can't be null or empty");
            }
            if (StringUtil.isBlank(entity.getResource())) {
                return Result.ofFail(-1, "resource can't be null or empty");
            }
            if (entity.getGrade() == null) {
                return Result.ofFail(-1, "grade can't be null");
            }
            if (entity.getGrade() != 0 && entity.getGrade() != 1) {
                return Result.ofFail(-1, "grade must be 0 or 1, but " + entity.getGrade() + " got");
            }
            if (entity.getCount() == null || entity.getCount() < 0) {
                return Result.ofFail(-1, "count should be at lease zero");
            }
            if (entity.getStrategy() == null) {
                return Result.ofFail(-1, "strategy can't be null");
            }
            if (entity.getStrategy() != 0 && StringUtil.isBlank(entity.getRefResource())) {
                return Result.ofFail(-1, "refResource can't be null or empty when strategy!=0");
            }
            if (entity.getControlBehavior() == null) {
                return Result.ofFail(-1, "controlBehavior can't be null");
            }
            int controlBehavior = entity.getControlBehavior();
            if (controlBehavior == 1 && entity.getWarmUpPeriodSec() == null) {
                return Result.ofFail(-1, "warmUpPeriodSec can't be null when controlBehavior==1");
            }
            if (controlBehavior == 2 && entity.getMaxQueueingTimeMs() == null) {
                return Result.ofFail(-1, "maxQueueingTimeMs can't be null when controlBehavior==2");
            }
            if (entity.isClusterMode() && entity.getClusterConfig() == null) {
                return Result.ofFail(-1, "cluster config should be valid");
            }
            return null;
        }
    
    
        // Sentinel Dashboard中创建新的流控规则时会自动提交post请求，会将表单内容自动封装为一个实体
        @PostMapping("/rule")
        @AuthAction(value = AuthService.PrivilegeType.WRITE_RULE)
        public Result<FlowRuleEntity> apiAddFlowRule(@RequestBody FlowRuleEntity entity) {
    
            Result<FlowRuleEntity> checkResult = checkEntityInternal(entity);
            if (checkResult != null) {
                return checkResult;
            }
            // 对新增的规则实体进一步完善
            entity.setId(null);
            Date date = new Date();
            entity.setGmtCreate(date);
            entity.setGmtModified(date);
            entity.setLimitApp(entity.getLimitApp().trim());
            entity.setResource(entity.getResource().trim());
            try {
                // 写入到内存库
                entity = repository.save(entity);
                // 写入到配置中心
                publishRules(entity.getApp());
            } catch (Throwable throwable) {
                logger.error("Failed to add flow rule", throwable);
                return Result.ofThrowable(-1, throwable);
            }
            return Result.ofSuccess(entity);
        }
    
    
        @PutMapping("/rule/{id}")
        @AuthAction(AuthService.PrivilegeType.WRITE_RULE)
        public Result<FlowRuleEntity> apiUpdateFlowRule(@PathVariable("id") Long id,
                                                        @RequestBody FlowRuleEntity entity) {
            if (id == null || id <= 0) {
                return Result.ofFail(-1, "Invalid id");
            }
            FlowRuleEntity oldEntity = repository.findById(id);
            if (oldEntity == null) {
                return Result.ofFail(-1, "id " + id + " does not exist");
            }
            if (entity == null) {
                return Result.ofFail(-1, "invalid body");
            }
    
            entity.setApp(oldEntity.getApp());
            entity.setIp(oldEntity.getIp());
            entity.setPort(oldEntity.getPort());
            Result<FlowRuleEntity> checkResult = checkEntityInternal(entity);
            if (checkResult != null) {
                return checkResult;
            }
    
            entity.setId(id);
            Date date = new Date();
            entity.setGmtCreate(oldEntity.getGmtCreate());
            entity.setGmtModified(date);
            try {
                entity = repository.save(entity);
                if (entity == null) {
                    return Result.ofFail(-1, "save entity fail");
                }
                publishRules(oldEntity.getApp());
            } catch (Throwable throwable) {
                logger.error("Failed to update flow rule", throwable);
                return Result.ofThrowable(-1, throwable);
            }
            return Result.ofSuccess(entity);
        }
    
    
        @DeleteMapping("/rule/{id}")
        @AuthAction(PrivilegeType.DELETE_RULE)
        public Result<Long> apiDeleteRule(@PathVariable("id") Long id) {
            if (id == null || id <= 0) {
                return Result.ofFail(-1, "Invalid id");
            }
            FlowRuleEntity oldEntity = repository.findById(id);
            if (oldEntity == null) {
                return Result.ofSuccess(null);
            }
    
            try {
                repository.delete(id);
                publishRules(oldEntity.getApp());
            } catch (Exception e) {
                return Result.ofFail(-1, e.getMessage());
            }
            return Result.ofSuccess(id);
        }
    
        private void publishRules(/*@NonNull*/ String app) throws Exception {
            // 从内存库中查找相应的规则
            List<FlowRuleEntity> rules = repository.findAllByApp(app);
            rulePublisher.publish(app, rules);
        }
    }
    ```

* ##### 修改页面

  ```html
  	<li ui-sref-active="active" ng-if="entry.appType==0">
              <a ui-sref="dashboard.flow({app: entry.app})">
                <i class="glyphicon glyphicon-filter"></i>&nbsp;&nbsp;流控规则-Nacos</a>
            </li>
  ```

  将`dashboard.flowV1`修改成`dashboard.flow`





## 七、分布式解决方案-Seata

### 7.1 Seata概述

* #### 分布式事务简介

  ​	对于分布式事务，通俗地说就是，一次操作有若干个分支操作组成，这些分支操作分属不同应用，分布在不同服务器上。分布式事务需要保证这些分支操作要么全部成功，要么全部失败。

  ​	能够实现分布式事务解决方案很多，但阿里 Seata 则是一个非常成熟的分布式事务产品，Spring Cloud Alibaba 推荐使用 Seata 作为分布式事务解决方案。
  
* #### 什么是 Seata

  Seata 是一款开源的分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务。Seata 将为用户提供了 AT、TCC、SAGA 和 XA 事务模式，为用户打造一站式的分布式解决方案

  ![](images\屏幕截图 2023-11-09 162848.png)

  当事务开始一次消费时，就会通知到所有的分支服务并把事务ID传递，然后分支就会把执行的结果交给TC，当都成功了，就会让分支全部的真正的提交。

  这个提交有**2PC（两阶段提交）、3PC（三阶段提交）**。

* #### 事务结构

  ![](images\屏幕截图 2023-11-09 162012.png)

  术语：

  **TC（Transaction Coordinator）- 事务协调者：维护全局和分支事务的状态，驱动全局事务提交或回滚。**

  **TM（Transaction Manager）- 事务管理器：定义全局事务范围：开始全局事务、提交或回滚全局事务。**

  **RM（Resource Manager）- 资源管理器：管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。**

### 7.2 分布式事务模式

* #### 概述

  事务模式的分类有四种，**AT模式、TCC模式、XA模式、Sage模式**；前面三种是2PC的提交模式、最后一种是3PC提交模式。用的最多的就是 AT 模式。

* #### XA模式

  基于支持 XA 事务的数据库

  XA（UniX TransAction）是一种分布式事务解决方案，一种分布式事务处理模式，是基于 **XA 协议**的。

  XA 协议：有 Tuxedo（Transaction for Unix has been Extended for Distributed for Distributed Operation，分布式操作扩展之后的 Unix 事务系统）首先提出的，并交给 X/Open 组织，**作为资源管理器与事务管理器的接口标准**。

  ![](images\屏幕截图 2023-11-09 164854.png)

  **要注意的几个点：1. TC只负责收集执行结果，判断工作还是会交由TM来进行。2. 第一阶段的写操作，是会将数据写到数据库的。**

  存在的问题：

  * 回滚日志（当第一阶段提交时，会产生一个回滚日志，作用是回滚事务恢复数据）无法自动清理，需要手动清理。

  * 多线程下对同一个RM中的数据进行修改，存在 ABA 问题。

    > 什么是ABA问题？
    >
    > 当一个事务A对一个数据a进行修改变成了b；在很短的时间内，事务B又将这个数据修改成了c；然后事务A发生了回滚，这个值又变成了a。
    >
    > 如果某个线程需要重复读某个内存地址，并以内存地址的值变化作为该值是否变化的唯一判定依据；那么这个线程就会认为这个变量在两次读取时间间隔内**没有发生任何变化**；

  为了解决上述的问题，就出现了**AT模式**。

* #### AT模式

  基于支持本地 ACID 事务的关系型数据库

  AT，Automatic Transaction。 AT 模式是Seata默认的分布式事务模型，是由 XA 模式演变而来的，通过**全局锁**对 XA 模式中的问题（ABA问题）进行了改进，并实现了回滚日志的自动清理。

  

  **与XA模式流程区别：**

  在XA模式中，当有全局事务发起，就会通过TC通知到各个分支事务，这时分支事务的注册就会由TC来完成的。

  而在AT模式中，分支事务的注册是由自己生成的一个事务ID，并且是在第一阶段成功之后才会注册。

  

  

  **存在的问题：**

  * 不支持 NoSQL
  * prepare 阶段、commit/rollback 阶段及回滚日志清理过程，完全“自动化”，无法实现定制化过程。

  有为了解决这些问题，又又出现了**TCC模式**。

* #### TCC模式

  TCC，Try Confirm/Cancel，同样也是2PC的，其与AT的重要区别是，支持将自定义的分支事务纳入到全局事务管理中，即可以实现定制化的日志清理与回滚过程。当然，该模式对业务逻辑的侵入性是较大的。

  TCC的prepart、commit、rollbakc逻辑都是自定义的，而AT中的都是自动的，执行系统定义好的逻辑。

* #### 2PC 的先天缺陷

  2PC 最大的特点是简单（原理简单、实现简单）。

  由于在微服务第一阶段`Prepare`阶段执行完后，会在等待TM的判断结果，即会进入**阻塞状态**。如果在同一个分布式事务中的分支的执行时间差别很大，就会导致操作响应时间过长。

  又如果当TC服务挂掉了以后，就会一直在阻塞状态。这也是**中心化问题**。



* #### Sage 模式

  Saga模式是SEATA提供的**长事务解决方案**，在Saga模式中，业务流程中每个参与者都提交本地事务，当出现某一个参与者失败则补偿前面已经成功的参与者，一阶段正向服务和二阶段补偿服务都由业务开发实现。

  对于架构复杂，且业务流程较多的系统，一般不适合使用2PC的分布式事务模式。

  ![](images\屏幕截图 2023-11-09 173226.png)

  对于一个流程很长的复杂业务，其会包含很多的子流程（事务）。每个流程都让它们真实提交它们真正的执行结果。只有当前子流程执行成功后才能执行下一个子流程。若整个流程中所有子流程全部成功，则全部成功；只要有一个子流程子流程，其执行结果可采用两种补偿方式：

  * 向后恢复：对于其前面所有成功的子流程，其执行结果全部撤销
  * 向前恢复：重试失败的子流程直到其成功。当然这个前提是要确保每个分支事务都能够成功。

  >  如何保证不发生脏读？
  >
  > 当子流程在提交了执行结果后，可根据业务场景为业务逻辑加锁或为资源加锁。

  

* #### 3PC 与 2PC 模式的区别

  1. **Sage模式的所有分支事务是串行执行的，而2PC的这是并行执行的。**
  2. Sage 模式没有TC，其是通过子流程之间的消息传递来完成全局事务管理的。



### 7.3 测试环境搭建

* #### 需求

  ![](images\屏幕截图 2023-11-09 162848.png)

  按照官方的 Seata 架构示例先搭建一下后进行 Seata 用法测试的代码环境。

  这里要定义`Stock(id,goodId,total)、Order(id,userId,goodsId,goodsPrice,conut)订单类`，Account与 Business等至少四个工程。

  Business 消费 Stock 与 Order 服务， Order 同时也消费 Account 服务。另外，Stock、Order 与 Account 分别连接着 mysql1、mysql2 与 mysql3三个DBMS，且均采用 JPA 作为 ORM。



* #### 创建 `07-ec-stock-8081`工程

  创建实体类：

  ```java
  @Data
  @Entity
  @JsonIgnoreProperties(value={"hibernateLazyInitializer","handler","fieldHandler"})
  public class Stock {
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY) // ID自动递增
      private Integer id;
  
      private Integer goodsId;// 商品id
  
      private Integer total;  // 库存量
  }
  ```

  创建控制类：

  ```java
  @RestController
  public class StockController {
      @Autowired
      private StockService service;
      // 减库存
      @PutMapping("/stock/deduct")
      public Stock updateHandle(@RequestParam("goodsId") Integer goodsId,
                                  @RequestParam("count") Integer count){
          return service.deductStock(goodsId, count);
      }
  }
  ```

  service层：

  ```java
  @Service
  public class StockServiceImpl implements StockService {
  
      @Autowired
      private StockRepository repository;
      @Override
      public Stock deductStock(Integer goodsId, Integer count) {
          Stock stock = repository.findStocksByGoodsId(goodsId);
          if(stock == null || stock.getTotal() < count){// 库存不足
              return null;
          }
          stock.setTotal(stock.getTotal() - count);
          return repository.save(stock);
      }
  }
  ```

  修改配置文件：

  ```yaml
  server:
    port: 8081
  spring:
    # 微服务名称
    application:
      name: ec-stock
    jpa:
      generate-ddl: true
      show-sql: true
      hibernate:
        ddl-auto: none
    datasource:
      type: com.alibaba.druid.pool.DruidDataSource
      driver-class-name: com.mysql.cj.jdbc.Driver
      # 修改url，mysql1地址在host文件中配置过了
      url: jdbc:mysql://mysql1:3306/test?useUnicode=true&useSSL=false&characterEncoding=utf8
      username: root
      password: 123
    cloud:
      nacos:
        discovery:
          server-addr: localhost:8848  # nacos注册中心名称
          username: nacos
          password: nacos
  ```

* #### 创建 Account 工程

  。。。

* #### 创建 Order 工程

  。。。

* #### 创建 common 工程

  在里面创建三个之前创建的实体类，开启`OpenFeign`;

  。。。

* #### Order 工程调用 Account

  在 Order 工程中需要引入 common 工厂的依赖，就可以使用之前定义的类。

* #### 创建 Business 工程

  

### 7.4 Seata-Server 安装与配置

根据 `Spring-cloud-alibaba` 的版本说明，下载相对应的 Seata 版本。

下载二进制后，打开conf文件中的`application.yml`文件（在老一些的版本配置文件是有两个的）。

`Seata-Server` 网页程序的默认端口号是`7091`,服务的默认端口号是这个端口号+1000 

找到`seata.store.mode: <mode> `，这个属性是配置事务管理服务数据的存储方式，支持三种存储方式 。



#### 存储模式的 Seata-Server

* file模式：会将相关数据存储在本地文件中，一般用于 Seata Server 的单机测试。
* db模式：存储在数据库，一般用于生产环境下的Seata server 集群部署，生产环境下使用最多的模式。
* redis模式：存储在 redis，一般用于生产环境下的 Seata Server 集群部署。性能略高于 db模式。

在配置文件目录中，有一个`application.example.yml`模板配置文件，找到相应的配置模板复制到配置文件中。

file：

```yaml
store:
    # support: file 、 db 、 redis
    mode: file
    file:
      dir: sessionStore
      max-branch-session-size: 16384
      max-global-session-size: 512
      file-write-buffer-cache-size: 16384
      session-reload-read-size: 100
      flush-disk-mode: async
```

db:

1. 要配置注册中和配置中心，都用Nacos 

2. 配置弄好后，还要在数据库创建表；

3. 在`script\server\db\mysql.sql`中有官方的建表语句
4. 在Nacos配置中心创建配置文件，在`script\config-center\config.txt`有官方模板。

### 7.5 AT模式下的Seata Client

**AT模式的回滚操作是基于在各个微服务的数据库中创建的一个`undo_log`日志表进行的，底层实现了自动清理。**

所以要在每个业务库中都要创建一个日志表。而对于要创建表的SQL语句，可以下载对应版本的**源码文件**的目录`script\client\at\db\mysql.sql`文件中有。

#### 7.5.1 实现过程

**添加依赖：**

> seata-server的版本要和seata-client的版本要一致。

```xml
           <dependency>
                <groupId>io.seata</groupId>
                <artifactId>seata-spring-boot-starter</artifactId>
               <version>[与seata-server版本相同]</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
                <exclusions>
                    <exclusion>
                        <groupId>io.seata</groupId>
                        <artifactId>seata-spring-boot-starter</artifactId>
                    </exclusion>
                </exclusions>
            </dependency>
```

**启动**

当`seata-server`启动成功后会自动注册到`SEATA_GROUP`组内。不用配置事务，新版本默认已经配置好了。

**配置事务管理器（TM）**

在要实现分布式事务的`controller`类处理器方法上加上`@GlobalTransactional`注解

### 7.6 AT模式工作过程

> 两阶段提交协议的演变：
>
> - 一阶段：业务数据和回滚日志记录在同一个本地事务中提交，释放本地锁和连接资源。
> - 二阶段：
>   - 提交异步化，非常快速地完成。
>   - 回滚通过一阶段的回滚日志进行反向补偿。

#### 写隔离

两个全局事务`tx1`和`tx2`。同时操作同一个数据段。

1. `tx1`首先获取**本地锁**，更新数据。获取**全局锁**，成功就提交本地事务，失败就会重试等待。`tx2`同`tx1`操作。**这是第一阶段提交。**
2. 若`tx1`全局提交，释放全局锁，`tx2`获取到全局锁，提交本地事务，之后同`tx1`。
3. 若`tx1`第二阶段异常，发生回滚，此时`tx1`要重新获取本地锁，进行反向补偿的更新操作，实现分支回滚。若`tx2`仍重试等待，因为拿着本地锁，`tx1`无法获取本地锁进行反向补偿，故要等待`tx2`等待超时

#### 读隔离

在数据库本地事务隔离级别 **读已提交（Read Committed）** 或以上的基础上，Seata（AT 模式）的默认全局隔离级别是 **读未提交（Read Uncommitted）** 。

如果应用在特定场景下，必需要求全局的 **读已提交** ，目前 Seata 的方式是通过 SELECT FOR UPDATE 语句的代理

> * 普通的读操作的执行需要获取到该记录的本地读锁
> * for update 的读操作的执行需要同时获取该记录的本地读锁与全局读锁
> * 读锁是共享锁
> * 一条记录上只要有读锁或写锁，就不能写或读

SELECT FOR UPDATE 语句的执行会申请 **全局锁** ，如果 **全局锁** 被其他事务持有，则释放本地锁（回滚 SELECT FOR UPDATE 语句的本地执行）并重试。这个过程中，查询是被 block 住的，直到 **全局锁** 拿到，即读取的相关数据是 **已提交** 的，才返回。

出于总体性能上的考虑，Seata 目前的方案并没有对所有 SELECT 语句都进行代理，仅针对 FOR UPDATE 的 SELECT 语句。



**所以对数据隔离有要求的读请求，要在加上`SELECT FOR UPDATE`**



### 7.7 TCC模式下的Seata Client



## 八 调用链跟踪

> 随着分布式系统规模的越来越大，各微服务间的调用关系也变得越来越复杂。一般情况下，一个客户端发起的请求在后端系统中会经过许多不同的微服务调用才能最终完成处理，而这些调用的过程形成了一个复杂的分布式服务调用链路。
>
> 那么调用链追踪系统就是为了能够快速地发现定位问题，判断故障影响范围。
>
> Spring Cloud Alibaba 会使用SkyWalking作为调用链追踪系统
>
> 国内的分布式追踪系统：Hydra(京东)、CAT（大众点评）、WatchMan（新浪）、Micoscope（唯品会）、Eagleeye（淘宝、为开源）。国际上使用得最多的Zipkin（Twitter）及Skywalking



### 7.1 SkyWalking 简介

> 其是通过在被监测应用中插入**探针**，以无侵入的方式自动手机所需指标，并自动推送到OAP系统平台。OAP会将收集到的数据存储到指定的存储介质 Storage。UI 系统通过调用OAP提供的接口，可以实现对相应数据的查询。

SkyWalking系统整体由四部分构成:。

**Agent**:探针，无侵入收集，是被插入到被监测应用中的一个进程。其会将收集到的监控指标自动推送给OAP系统。

**OAP**:Observability Analysis Platform，可观测性分析平台，其包含一个收集器Collector,能够将来自于Agent的数据收集并存储到相应的存储介质Storage。

**Storage**: 数据中心。用于存储由OAP系统收集到的链路数据，支持H2、MySQL、ElagticSearch等。默认使用的是H2（测试使用)，推荐使用ElasticSearch(生产使用)。

**UI**:一个独立运行的可视化 web平台，其通过调用oAP系统提供的接口，可以对存储在 Storage中的数据进行多维度查询。

### 7.2 trace 与span

* **trace（轨迹）：**跟踪单元是从客户端锁发起的请求抵达被跟踪的系统的边界开始，到被跟踪系统向客户返回响应为止的过程，这个过程称为一个trace。
* **span（跨度）：**每个 trace 中会调用若干个服务，为了记录调用了哪些服务，以及每次调用所消耗的时间等信息，在每次调用服务时，埋入一个调用记录，这样两个调用记录之间的区域称为一个span。一个 trace 由若干个有序的 span 组成
* **segment（片段）：**一个 trace 的一个片段，可以由多个 span 组成。

为了标识。跟踪系统一般会为每一个trace、span和segement标识一个id。







































## 八、消息系统整合 Spring Cloud Stream



















