# s-c-a q&a

现实的微服务开发，在高并发场景一定要保证高可用。

保证高可用的三大技术：降级、限流、熔断

## Nacos 原理

![](images\屏幕截图 2024-01-10 191501.png)

> Nacos 是如何感知服务的上下线的呢？

* #### 在Nacos 1.4版本之前

  注册到nacos的服务会启动一个心跳任务，告知注册中心自己存活。然后消息的传递是用`Http`协议的post请求

* #### 在Nacos 2.x版本

  注册到nacos的服务会启动一个心跳任务，告知注册中心自己存活。但是用的是RGPC长连接进行数据交互。

> Nacos 如何实现高并发的读写？

运用到了写时复制技术，读写是分离的



![](images\屏幕截图 2024-01-10 201722.png)

1. 当客户端的应用启动时，就会构建一个实例Instance，然后就会触发一个定时的定时任务（目的是更新数据重新注册），并且调用注册接口。
2. 服务端就会处理注册请求，添加服务和实例信息，然后就会触发三个事件，**客户端客户端注册服务事件（添加客户端到注册表，就会发布服务变更事件定时向订阅者推送服务变动数据）、实例元数据事件（管理过期的服务和实例元数据）、注册实例跟踪事件（和链路追踪相关）。**

> nacos 服务端的注册表是一个双重map，服务注册信息将会保存到一个 serviceMap的ConcurrentHashMap中。



> 为了保证本地服务实例列表的动态感知，采用了Pull/Push同时运作的方式。
>
> **pull**：1在服务启动的时候首先从nacao中读取指定服务名称的实例列表，缓存到本地；与此同时，还会开启一个定时任务，每10秒轮询一次可用的服务列表
>
> **push**：通过检测心跳，心跳超时则推送服务实例信息，通过UDP协议。无需保证连接的安全性，因为还有pull机制。

Nacos注册概括来说有6个步骤：

0、服务容器负责启动，加载，运行服务提供者。

1、服务提供者在启动时，向注册中心注册自己提供的服务。

2、服务消费者在启动时，向注册中心订阅自己所需的服务。

3、注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。

4、服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。

5、服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

* #### Nacos 服务注册与订阅的完整流程

  Nacos 客户端进行服务注册有两个部分组成，一个是将服务信息注册到服务端，另一个是像服务端发送心跳包，这两个操作都是通过 NamingProxy 和服务端进行数据交互的。

  Nacos 客户端进行服务订阅时也有两部分组成，一个是不断从服务端查询可用服务实例的定时任务，另一个是不断从已变服务队列中取出服务并通知 EventListener 持有者的定时任务。



### Nacos的双重检查锁

> Nacos许多地方都用到了双重检查锁

在Nacos的`InstancesChangeNotifier` 类中，有这样一个方法：

```java
private final Map<String, ConcurrentHashSet<EventListener>> listenerMap = new ConcurrentHashMap<String, ConcurrentHashSet<EventListener>>();

private final Object lock = new Object();

public void registerListener(String groupName, String serviceName, String clusters, EventListener listener) {
    String key = ServiceInfo.getKey(NamingUtils.getGroupedName(serviceName, groupName), clusters);
    ConcurrentHashSet<EventListener> eventListeners = listenerMap.get(key);
    if (eventListeners == null) {
        synchronized (lock) {
            eventListeners = listenerMap.get(key);
            if (eventListeners == null) {
                eventListeners = new ConcurrentHashSet<EventListener>();
                listenerMap.put(key, eventListeners);
            }
        }
    }
    eventListeners.add(listener);
}
```

该方法的主要功能就是对监听器事件进行注册。其中注册的事件都存在成员变量`listenerMap`当中。`listenerMap`的数据结构是key为String，value为`ConcurrentHashSet`的Map。也就是说，一个key对应一个集合。



## OpenFeign 原理

> 在老版本的OpenFeign底层内置了Ribbon，用来完成调用时的负载均衡。OpenFeign默认将Ribbon作为负载均衡器。但由于 Netfilx 已不再维护 Ribbon，所以从 Spring Cloud 2021 .x 开始集成的OpenFeign 中以彻底丢弃 Ribbon，而是采用 Spring Cloud 自行研发的 `Spring Cloud Loadbalancer` 作为负载均衡器。 

* ### 远程调用

  feign 的远程调用底层实现默认采用的是 `JDK 的 URLConnection` ，同时还支持 `HttpClient` 与 `OkHttp`

  由于 `JDK 的URLConnection` 不支持连接池，通信效率很低，所以生产中是不会使用默认实现的。所以在 Spring Cloud OpenFeign 中直接默认是实现变成了 `HttpClient`，同时也支持 `OkHttp`。

  **单例的用`HttpClient`、非单例的用`OkHttp`**；如果要自定义 http 客户端，用`HttpClient`。

* ### spring父子容器

  > 父子容器特点
  >
  > 1. 父容器和子容器是相互隔离的，他们内部可以存在名称相同的bean
  > 2. 子容器可以访问父容器中的bean，而父容器不能访问子容器中的bean
  > 3. 调用子容器的getBean方法获取bean的时候，会沿着当前容器开始向上面的容器进行查找，直到找到对应的bean为止
  > 4. 子容器中可以通过任何注入方式注入父容器中的bean，而父容器中是无法注入子容器中的bean，原因是第2点

  #### BeanFactory的方式

  ```java
  //创建父容器parentFactory
  DefaultListableBeanFactory parentFactory = new DefaultListableBeanFactory();
  //创建一个子容器childFactory
  DefaultListableBeanFactory childFactory = new DefaultListableBeanFactory();
  //调用setParentBeanFactory指定父容器
  childFactory.setParentBeanFactory(parentFactory);
  ```

  #### ApplicationContext的方式

  ```java
  //创建父容器
  AnnotationConfigApplicationContext parentContext = new AnnotationConfigApplicationContext();
  //启动父容器
  parentContext.refresh();
  //创建子容器
  AnnotationConfigApplicationContext childContext = new AnnotationConfigApplicationContext();
  //给子容器设置父容器
  childContext.setParent(parentContext);
  //启动子容器
  childContext.refresh();
  ```

  

* ### 原理

  OpenFeign的底层用到了spring父子容器（实现**配置隔离**，当在父容器查找的伪装类不存在时，就会到子容器中找）、代理模式。

  **注册bean：**

  spring容器启动时，会通过IOC容器获取FeignContext的上下文，然后会根据这个根据这个上下文在创建`Feign.Builder`时，创建Feign服务对应的子容器，并且从子容器中获取日志工厂、编码器、解码器的bean。

  **动态代理：**

  处理解析`@FeignClient`注解包括SpirngMVC的注解，封装为`MethodHandle`包装类。遍历接口中的方法，过滤Object方法，获取到FeignClient中的接口方法，创建动态代理对应的`InvocationHandler`增强方法类，根据解析注解接口方法绑定动态代理类。

  也就是说在我们调用 @FeignClient 接口时，会被 `FeignInvocationHandler#invoke` 拦截，并在动态代理方法中执行下述逻辑

  1. 接口注解信息封装为 HTTP Request
  2. 通过 Ribbon 获取服务列表，并对服务列表进行负载均衡调用（**服务名转换为 ip+port**）
  3. 请求调用后，将返回的数据封装为 HTTP Response，继而转换为接口中的返回类型



### Ribbon的负载均衡

> Ribbon的负载均衡都是实现了**IRule**接口

主要有7种负载均衡策略：

**RoundRobinRule**： 默认轮询的方式。

**RandomRule**： 随机方式。

**WeightedResponseTimeRule**： 根据响应时间来分配权重的方式，响应的越快，分配的值越大。

**BestAvailableRule**： 选择并发量最小的方式。

**RetryRule**： 在一个配置时间段内当选择server不成功，则一直尝试使用subRule的方式选择一个可用的server。

**ZoneAvoidanceRule**： 根据性能和可用性来选择。

**AvailabilityFilteringRule**： 过滤掉那些因为一直连接失败的被标记为circuit tripped的后端server，并过滤掉那些高并发的的后端server（active connections 超过配置的阈值）。



> 除了SpringCloud以外，Dubbo它也是用来作为微服务架构落地的成熟解决方案，并且它在服务通信上比SpringCloud性能更高，这取决于它的底层实现是基于原生的TCP协议，它的定位就是一款高性能的RPC(远程过程调用)框架，所以在国内很多的企业都选择Dubbo作为微服务框架，本文章的目的是帮助同学们将Dubbo这款高性能的RPC框架集成到SpringCloud中，真正实现SpringCloud 和 Dubbo的混用。



## Dubbo 原理与集成





## Sentinel 原理

> Sentinel是阿里巴巴开源的一款分布式系统的流量控制框架，它**基于AOP和注解**，提供了流量控制、熔断降级、系统负载保护等功能，可以有效地保护系统的稳定性和可用性。本文将从源码角度分析Sentinel的实现原理和代码结构，并提供相关的代码示例。

（slot：槽）

滑动窗口？

* ### Sentinel的核心组件

  * **Entry**：表示一个请求入口，它包含了请求的上下文信息和流量控制策略，用于限制请求的流量。
  * **Resource**：表示一个请求的资源，它可以是一个URL、一个方法、一个服务等，用于标识请求的类型和范围
  * **Rule**：表示一个流控规则，包含了资源、流控模式、阈值等信息
  * **SlotChain**：表示一个拦截器链（责任链），用于对请求进行拦截和处理，包含各种拦截器和处理器。
  * **ProcessorSlot**：表示一个拦截器或处理器，用于对请求进行处理转发，可以实现流量控制、熔断降级等功能。
  * SphU：表示一个入口工具类，用于创建Entry对象和进行流量控制。
  * **Tracer**：表示一个请求追踪器，用于记录请求的上下文信息和状态。

* ### 处理流程

  * #### 准备工作

    Sentinel会为每个资源创建一个处理链条也就是**SlotChain**责任链，默认情况下它会在资源第一次被访问时候创建，之后的访问都会复用这个责任链，所以责任链有且仅有一个。

    但是可以设置`spring.cloud.sentinel.eager=true`，饥饿加载，只要服务一上线，就会加载相应的责任链。

  * #### 处理请求

    当一个请求进入系统时，通过**SphU**工具类就会创建一个**Entry**对象，该对象包括了请求的上下文信息和流控策略。

    然后就会调用请求资源对应的**SlotChain**责任链，然后会调用**ProcessorSlot**拦截器和处理器依次处理请求。

    在默认情况下**ProcessorSlot**,默认就8个，如果你想扩展，只要实现**ProcessorSlot**类，按照SPI的规定配置一下，就能使用。

    8个处理器槽依次是：

    - **NodeSelectorSlot**

      设置当前资源**对应入口**的**统计Node**。

      > 什么是统计Node？

      统计Node是一个用来统计数据的类，根据这个统计类来进行流控。

      通过Node这个统计类就知道有多少请求，成功多少个，失败多少个，qps是多少之类的。底层用到了**滑动窗口**算法。

      > 什么是对应入口？

      只不过一般一个资源就一个入口，比如一个http接口一般只能通过http访问，但是Sentinel支持多入口，你可以不用，但是Sentinel有。

      所以NodeSelectorSlot的作用就是选择资源在当前调用入口的统计Node，这样就实现了统计同一个资源在不同入口访问数据，用上面的例子解释，就可以实现分别统计通过公交和地铁访问西湖的人数。

      资源的入口可以在进入资源之前通过`ContextUtil.enter("入口名", origin)`来指定，如果不指定，那么入口名称默认就是`sentinel_default_context`。

      在SpringMVC环境底下，所有的http接口资源，默认的入口都是`sentinel_spring_web_context`

    - **ClusterBuilderSlot**

      ClusterBuilderSlot的作用跟NodeSelectorSlot其实是差不多的，也是用来选择统计Node，但是选择的Node的统计维护跟NodeSelectorSlot不一样。

      ClusterBuilderSlot会选择两个统计Node：

      - 第一个统计Node是资源的所有入口的统计数据之和，就是资源访问的总数据
      - 第二个统计Node就是统计资源调用者对资源访问数据

    - **LogSlot**

      日志

    - **StatisticSlot（统计槽）**

      前面说的NodeSelectorSlot和ClusterBuilderSlot，他们的作用就是根据资源当前的入口和调用来源来选择对应的统计Node。

      而StatisticSlot就是对这些统计Node进行实际的统计，比如加一下资源的访问线程数，资源的请求数量等等。

      责任链来到 StatisticSlot 的时候会先 fire 掉，如果请求走完后续操作通过了，我们就要去增加当前时间段通过的请求数，作为后续请求能否通过的参考

      **滑动窗口算法**的实现是在这里面的。

    - **AuthoritySlot**

      跟黑白名单有关

    - **SystemSlot**

      系统流控规则

    - **FlowSlot**

      根据预设的规则，结合前面统计出来的实时信息进行流量控制。

    - **DegradeSlot**

      负责降级熔断

      

  * #### 熔断器工作流程

    Sentinel会为每个设置的规则都创建一个熔断器，熔断器有三种状态，OPEN(打开)、HALF_OPEN(半开)、CLOSED(关闭)

    - 当处于CLOSED状态时，可以访问资源，访问之后会进行慢调用比例、异常比例、异常数的统计，一旦达到了设置的阈值，就会将熔断器的状态设置为OPEN

      

    - 当处于OPEN状态时，会去判断是否达到了熔断时间，如果没到，拒绝访问，如果到了，那么就将状态改成HALF_OPEN，然后访问资源，访问之后会对访问结果进行判断，符合规则设置的要求，直接将熔断器设置为CLOSED，关闭熔断器，不符合则还是改为OPEN状态

      

    - 当处于HALF_OPEN状态时，直接拒绝访问资源







### SPI机制

> **SPI(Service Provider Interface)**，是JDK内置的一种**服务发现机制**，可以用来启动框架扩展和替换组件，Java中SPI机制主要思想是**将装配的控制权移到程序之外**，在模块化设计中这个机制尤其重要，其核心思想就是**解耦**。

![](.\images\屏幕截图 2024-01-11 185905.png)

* #### SPI与API区别：

​		API是调用并用于实现目标的类、接口、方法等的描述；

​		SPI是扩展和实现以实现目标的类、接口、方法等的描述；

​		换句话说，API 为操作提供特定的类、方法，SPI 通过操作来符合特定的类、方法。

* #### 使用方法

  当服务的提供者提供了一种接口的实现之后，需要在classpath下的META-INF/services/目录里创建一个以服务接口命名的文件，这个文件里的内容就是这个接口的具体的实现类。当其他的程序需要这个服务的时候，就可以通过查找这个jar包（一般都是以jar包做依赖）的META-INF/services/中的配置文件，配置文件中有接口的具体实现类名，可以根据这个类名进行加载实例化，就可以使用该服务了。JDK中查找服务的实现的工具类是：`java.util.ServiceLoader`

* #### 不足

  多线程情况下使用ServiceLoader创建实例是线程不安全的，要遍历所有实现才能拿到所需要的实现。可以看看Dubbo的SPI机制。

















## Seata 原理



## 链路追踪





## Spring cloud streams 原理

















