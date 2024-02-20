# 第七章 Redis缓存（实战）



```xml
<!--jedis 依赖-->
 <dependency>
 <groupId>redis.clients</groupId>
 <artifactId>jedis</artifactId>
 <version>4.2.0</version>
 </dependency>
```



```java
Jedis jedis = new Jedis("redis", 6379);
jedis.set("name","张三");
jedis.mset("age","23","depart","市场部");
```



```java
public void text01(){
	try(Jedis jedis = jedisPool.getResource()){//会自动将资源返回到连接池
    
	}
}
```



## springboot整合Redis缓存

在idea连接mysql数据库：

![](image\屏幕截图 2023-10-08 153516.png)



```xml
    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/webapp</directory>
                <targetPath>META-INF/resources</targetPath>
                <includes>
                    <include>**/*.*</include>
                </includes>
            </resource>
        </resources>
    </build>
```

注册资源文件路径，将`src/main/java`路径下在xml文件都给注册为资源文件。





```xml
<!-- spring boot与Redis的整合依赖 -->
<dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```





> spring Boot  中可以直接使用Jedis实现对Redis的操作，但一般不这样用，而是使用Redis操作模板RedisTemplate 类的实例来操作的 Redis、
>
> RedisTemplate 类是一个对Redis 进行操作的模板类。该模板类中具有很多方法，这些方法很多与Redis操作命令同名或类似。还有两种方法boundXxxOps(k) 与 opsForXxx（）。前者会绑定key，然后之后的操作就可以不输入key来操作。



![](image\屏幕截图 2023-10-08 191131.png)



修改findTurnover()

```java
@Autowired
private RedisTemplate<Object,Object> rt;

@Override
public Double findTurnover(){
    
    //获取指定key的操作对象
    BoundValueOperations<Object,Object> ops = rt.boundValueOps("turnover");
    //从缓存中读取指定key的value
    Object turnover = ops.get();
    if(turnover == null){
        //如果不存在到数据库中获取
        Date date = new Date();
    	SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
    	turnover = dao.selectTurnover(sdf.format(date));
        
        //将查询结果放入缓存
        ops.set(turnover, 10, TimeUnit.SECONDS);
    }
    return (Double)turnover;
}
```



总结

* 在POM中导入依赖

* 在配置文件中注册Redis连接信息与缓存信息

* 需要缓存到Redis中的实体类必须要序列化

* Spring Boot启动类中要添加@EnableCaching注解开启缓存

* 要将查询到的数据添加到缓存需要在方法上添加@Cacheable注解

  :`@Cacheable(value = "pc",key = "'product_'+#name")`

* 对数据进行写操作的方法上添加@CacheEvict注解使方法调用时清除缓存

  ：`@CacheEvict(value = "pc", allEntries = true)`

* 对于需要添加过期时间的缓存要编写具体代码来实现



## 高并发问题

### 缓存穿透

​	**当用户访问的数据既不在缓存也不在数据库中时，就会导致每个用户查询都会“穿透” 缓存“直抵”数据库。这种情况就称为缓存穿透**。

当高度发的访问请求到达时，缓存穿透不 仅增加了响应时间，而且还会引发对 DBMS 的高并发查询，这种高并发查询很可能会导致 DBMS 的崩溃。 

缓存穿透产生的主要原因有两个：

一是在数据库中没有相应的查询结果，二是查询结果为空时，不对查询结果进行缓存。所以，针对以上两点，解决方案也有两个：

1. 对非法请求进行限制 
2. 对结果为空的查询给出默认值

### 缓存击穿

上面的findTurnover()方法就是很好的例子，**因为每次访问网页时都要调用这个方法，并且过期时间很短，当缓存刚好过期时，有大量的访问网页的请求时，就会导致缓存击穿。**可以使用“双重检测锁”。

```java
@Autowired
private RedisTemplate<Object,Object> rt;

@Override
public Double findTurnover(){
    
    //获取指定key的操作对象
    BoundValueOperations<Object,Object> ops = rt.boundValueOps("turnover");
    //从缓存中读取指定key的value
    Object turnover = ops.get();
    if(turnover == null){
        synchronized(this){
            turnover = ops.get();
            if(turnover == null){
                //如果不存在到数据库中获取
        		Date date = new Date();
    			SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
    			turnover = dao.selectTurnover(sdf.format(date));
            }
        }
        
        //将查询结果放入缓存
        ops.set(turnover, 10, TimeUnit.SECONDS);
    }
    return (Double)turnover;
}
```

### 缓存雪崩

​	对于缓存中的数据，很多都是有过期时间的。若大量缓存的过期时间在同一很短的时间 段内几乎同时到达，那么在高并发访问场景下就可能会引发对 DBMS 的高并发查询，而这将 可能直接导致 DBMS 的崩溃。这种情况称为缓存雪崩。 

对于缓存雪崩没有很直接的解决方案，最好的解决方案就是预防，即提前规划好缓存的 过期时间。要么就是让缓存永久有效，当 DB 中数据发生变化时清除相应的缓存。如果 DBMS 采用的是分布式部署，则将热点数据均匀分布在不同数据库节点中，将可能到来的访问负载均衡开来

### 数据库缓存双写不一致问题

> 以上三种情况都是针对**高并发读场景**中可能会出现的问题，而数据库缓存双写不一致问 题，则是在**高并发写场景**下可能会出现的问题。 

对于数据库缓存双写不一致问题，以下两种场景下均有可能会发生：

**修改DB更新缓存场景**

对于具有缓存 warmup 功能的系统，DBMS 中常用数据的变更，都会引发缓存中相关数 据的更新。在高并发写请求场景下，若多个请求要对 DBMS 中同一个数据进行修改，修改后 还需要更新缓存中相关数据，那么就有可能会出现缓存与数据库中数据不一致的情况。

![](image\屏幕截图 2023-12-20 184415.png)

**修改DB删除缓存场景**

​	在很多系统中是没有缓存 warmup 功能的，为了保持缓存与数据库数据的一致性，一般 都是在对数据库执行了写操作后，就会删除相应缓存。 

​	在高并发读写请求场景下，若这些请求对 DBMS 中同一个数据的操作既包含写也包含读， 且修改后还要删除缓存中相关数据，那么就有可能会出现缓存与数据库中数据不一致的情况。

![](image\屏幕截图 2023-12-20 184426.png)

解决方案：

（1）延迟双删

​	**延迟双删方案是专门针对于“修改 DB 删除缓存”**场景的解决方案。但该方案并不能彻 底解决数据不一致的状况，其只可能降低发生数据不一致的概率。 

​	**延迟双删方案是指，在写操作完毕后会立即执行一次缓存的删除操作，然后再停上一段 时间（一般为几秒）后再进行一次删除。而两次删除中间的间隔时长，要大于一次缓存写操作的时长**

（2）队列

**☆（3）分布式锁**

​	使用队列的串行化虽然可以解决数据库与缓存中数据不一致，但系统失去了并发性，降 低了性能。使用分布式锁可以在不影响并发性的前提下，协调各处理线程间的关系，使数据 库与缓存中的数据达成一致性。

 只需要对数据库中的这个共享数据的访问通过分布式锁来协调对其的操作访问即可

（4）binlog异步删除

* 实现思路，低耦合的解决方案是使用canal。canal伪装成mysql的从机，监听主机mysql的二进制文件，当数据发送变化时发送给MQ。最终消费进行删除。
