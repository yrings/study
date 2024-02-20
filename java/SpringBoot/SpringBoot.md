# SpringBoot

> Spring程序的缺点很明显，无论是使用XML、注解、Java配置类还是混合使用，都会让人觉得麻烦，复杂。
>
> 在早期对于单体项目只用配置一次没有完全体现出来，但是现在微服务项目的流行即将单个项目根据服务功能拆分为多个微服务单体项目，这就要写太多次配置。
>
> 为了解决这种问题SpringBoot由此而生。
>
> SpringBoot帮我们简单、快速地创建一个独立的、生成级别的Spring应用。

## 1、SpringBoot准备快速入门

* 版本要求：

  maven 3.6.3+，Tomcat 10.0+，Servlet 9.0+，JDK 17+

* 配置maven的pom.xml文件

  创建一个空项目，在项目中创建一个新模型

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  
      <modelVersion>4.0.0</modelVersion>
  
      <!--spring-boot-->
      <parent>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-parent</artifactId>
          <version>3.0.5</version>
      </parent>
  
      <groupId>com.demo</groupId>
      <artifactId>springboot-base-quick-01</artifactId>
      <version>1.0-SNAPSHOT</version>
  
      <!--导入对应的启动器即可-->
      <dependencies>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
          </dependency>
      </dependencies>
  </project>

创建启动类：

```java
@SpringBootApplication //启动类
public class Main {
    public static void main(String[] args) {
        //自动创建ioc容器，启动tomcat服务器
        //创建IOC容器，开启tomcat服务器
        SpringApplication.run(Main.class,args);
    }
}
```

创建控制层组件：

```java
@RestController
@RequestMapping("hello")
public class HelloController {

    @GetMapping("boot")
    public String hello(){
        return "hello springboot!!";
    }
}
```

问题：

* 为什么不用在依赖中声明版本？

  > 在导入的父工程中又导入了父工程，它里面声明有版本

* （starter）启动器是什么？

  > 例如导入的spring-boot-starter-web依赖是一个整合包，包含有web的所有依赖；
  >
  > 它是一组预定义的依赖项集合。自动装配在应用程序启动时自动装配所需的组件和功能。
  >
  > 配置文件会根据需要自动装配。

* @SpringBootApplication注解的作用

  > 作用：
  >
  > 1、自身就是一个配置类
  >
  > 2、自动加载配置文件，@EnableAutoConfiguration 自动加载其它配置类
  >
  > 3、@ComponentScan扫描包注解，默认扫描当前类，子包的类

## 2、SpringBoot 配置文件

### 2.1 统一配置管理

> SpringBoot 工程下，进行统一的配置管理。你想设置的任何参数（端口号、项目根路径、数据库连接信息等等）都集中到一个固定的位置和命名的配置文件下（application.properties 或 application.yml）中
>
> 配置文件应该放在Spring Boot工程的`src/main/resources`目录下。这是因为`src/main/resources`目录是Spring Boot默认的类路径（classpath），配置文件会自动加载可供应用程序访问。
>
> 1、所以我们要使用固定的key配置一些参数
>
> 可以通过官方文档查询：
>
> （(https://docs.spring.io/spring-boot/docs/3.0.10/reference/html/application-properties.html#appendix.application-properties）
>
> 2、也可以自定义key，然后在项目中用@Value(${key})去读取数据
>
> ```java
> @Value(${自定义的key})
> private String name;
> ```

### 2.2 YAML配置文件的使用
