# SpringMVC

## 1、SpringMVC简介

### 1.1 什么是MVC

MVC是一种软件架构的思想，将软件按照模型、视图、控制器来划分 

M：Model，模型层，指工程中的JavaBean，作用是处理数据 

JavaBean分为两类： 

* 一类称为实体类Bean：专门存储业务数据的，如 Student、User 等 

* 一类称为业务处理 Bean：指 Service 或 Dao 对象，专门用于处理业务逻辑和数据访问。 

V：View，视图层，指工程中的html或jsp等页面，作用是与用户进行交互，展示数据 

C：Controller，控制层，指工程中的servlet，作用是接收请求和响应浏览器 

MVC的工作流程： 

用户通过View视图发送请求到服务器，在服务器中请求被Controller接收，Controller 调用相应的Model层处理请求，处理完毕将结果返回到Controller，Controller再根据请求处理的结果 找到相应的View视图，渲染数据后最终响应给浏览器

### 1.2 什么是SpringMVC

> SpringMVC是Spring的一个后续产品，是Spring的一个子项目
>
> SpringMVCs Spring为表述层开发提供的一整套完备的解决方案
>
> 三层架构：
>
> 表述层：前台页面和后台servlet
>
> 业务逻辑层：
>
> 数据访问层：

### 1.3 SpringMVC的特点

* Spring 家族原生产品，与 IOC 容器等基础设施无缝对接 

* 基于原生的Servlet，通过了功能强大的**前端控制器DispatcherServlet**，对请求和响应进行统一 处理 

* 表述层各细分领域需要解决的问题全方位覆盖，提供全面解决方案 

* 代码清新简洁，大幅度提升开发效率 

* 内部组件化程度高，可插拔式组件即插即用，想要什么功能配置相应组件即可 

* 性能卓著，尤其适合现代大型、超大型互联网项目要求

## 2、入门案例

### 2.1 创建maven工程

引入依赖

```xml
<dependencies>
	<!-- SpringMVC -->
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-webmvc</artifactId>
		<version>5.3.1</version>
	</dependency>
    
	<!-- 日志 -->
	<dependency>
		<groupId>ch.qos.logback</groupId>
		<artifactId>logback-classic</artifactId>
		<version>1.2.3</version>
	</dependency>
    
	<!-- ServletAPI -->
	<dependency>
		<groupId>javax.servlet</groupId>
		<artifactId>javax.servlet-api</artifactId>
		<version>3.1.0</version>
		<scope>provided</scope>
	</dependency>
    
	<!-- Spring5和Thymeleaf整合包 -->
	<dependency>
		<groupId>org.thymeleaf</groupId>
		<artifactId>thymeleaf-spring5</artifactId>
		<version>3.0.12.RELEASE</version>
	</dependency>
</dependencies>
```

**注：由于 Maven 的传递性，我们不必将所有需要的包全部配置依赖，而是配置最顶端的依赖，其他靠 传递性导入。**

### 2.2 配置web.xml

①默认配置方式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

        <!--配置SpringMVC的前端控制器DispatcherServlet，对浏览器发送的请求统一进行处理
        SpringMVC的配置文件默认的位置和名称：
        位置：WEB-INF下
        名称：<servlet-name>的值+ -servlet.xml
        例如：当前的SpringMVC-servlet.xml
     -->
    <servlet>
        <servlet-name>SpringMVC</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>SpringMVC</servlet-name>
        <!--
            设置springMVC的核心控制器所能处理的请求的请求路径
            /所匹配的请求可以是/login或.html或.js或.css方式的请求路径，但是/不能匹配.jsp请求路径的请求
            /*:匹配浏览器向服务器发送的所有请求，包括.jsp,但是jsp的请求是要tomcat所配置的jspServlet来处理的
            *.do是后缀匹配
        -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

②扩展配置方式

```xml
```



### 2.3 创建请求控制器

> 因为前端控制器对浏览器发送的请求进行了统一的处理，但是具体的请求会有不同的过程，所以要创建处理具体请求的类即请求控制器
>
> 请求控制器中每一个处理方法成为控制器方法
>
> 因为SpringMVC的控制器有一个POJO(普通的java类)担任，因此需要通过@Controller注解将其标识为一个控制层组件，交给Spring的IoC容器管理，此时SpringMVC才能够识别控制器的存在

```java
@Controller
public class HelloController {
    
}
```



### 2.4 创建springMVC的配置文件

```xml
<!-- 自动扫描包 -->
<context:component-scan base-package="com.atguigu.mvc.controller"/>

<!-- 配置Thymeleaf视图解析器 -->
<bean id="viewResolver"
class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
	<property name="order" value="1"/>
	<property name="characterEncoding" value="UTF-8"/>
	<property name="templateEngine">
		<bean class="org.thymeleaf.spring5.SpringTemplateEngine">
			<property name="templateResolver">
				<bean
class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
                    <!--
                            逻辑视图 = 物理视图 - 前缀 - 后缀
                            物理视图：/WEB-INF/templates/index.html
                            逻辑视图：index
                        -->
					<!-- 视图前缀 -->
					<property name="prefix" value="/WEB-INF/templates/"/>
					<!-- 视图后缀 -->
					<property name="suffix" value=".html"/>
					<property name="templateMode" value="HTML5"/>
					<property name="characterEncoding" value="UTF-8" />
				</bean>
			</property>
		</bean>
	</property>
</bean>

<!--
	处理静态资源，例如html、js、css、jpg
	若只设置该标签，则只能访问静态资源，其他请求则无法访问
	此时必须设置<mvc:annotation-driven/>解决问题
-->
<mvc:default-servlet-handler/>

<!-- 开启mvc注解驱动 -->
<mvc:annotation-driven>
	<mvc:message-converters>
		<!-- 处理响应中文内容乱码 -->
		<bean
class="org.springframework.http.converter.StringHttpMessageConverter">
			<property name="defaultCharset" value="UTF-8" />
			<property name="supportedMediaTypes">
				<list>
					<value>text/html</value>
					<value>application/json</value>
				</list>
			</property>
		</bean>
	</mvc:message-converters>
</mvc:annotation-driven>
```

### 2.5 测试

部署tomcat

Application context 应用上下文路径：

作用是用来访问某一个具体的工程的

![屏幕截图 2023-08-19 090119](D:\data\java笔记\markdown笔记\SSM\images\屏幕截图 2023-08-19 090119.png)



On ‘Update’ action: Redeploy

当点击刷新时会重新部署工程

On frame deactivation: Update classes and resources

当IDEA窗口失去焦点时会更新类和资源

![屏幕截图 2023-08-19 090604](D:\data\java笔记\markdown笔记\SSM\images\屏幕截图 2023-08-19 090604.png)



在Controller层来处理首页资源：

```java
@Controller
public class HelloController {

    //来处理首页资源
    //在服务器端"/"开头表示绝对路径，被服务器解析成localhost:8080/上下文路径
    @RequestMapping("/")
    public String protal(){
        //将逻辑视图返回
        return "index";
    }
    @RequestMapping("/hello")
    public String hello(){
        return "success";
    }
}
```

html：

```html
!DOCTYPE html>
<!--引入thymeleaf约束-->
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>首页</title>
</head>
<body>
<h1>index.html</h1>
<a th:href="@{/hello}">测试SpringMVC</a>
<!--
<a href="/hello">
如果不用thymeleaf提供标签，浏览器会将“/“解析成localhost:8080没有上下文路径
-->
</body>
</html>
```

### 2.6 总结

1. 浏览器发送请求，若请求地址符合在web.xml中配置的前端控制器的url-pattern，该请求就会被前端控制器 DispatcherServlet处理。

   ```xml
   	<servlet>
           <servlet-name>SpringMVC</servlet-name>
           <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
       </servlet>
       <servlet-mapping>
           <servlet-name>SpringMVC</servlet-name>
           <url-pattern>/</url-pattern>
       </servlet-mapping>
   ```

2. 前端控制器会读取SpringMVC的核心配置文件，通过扫描组件找到控制器， 将请求地址和控制器中@RequestMapping注解的value属性值进行匹配，若匹配成功，该注解所标识的 控制器方法就是处理请求的方法。

   ```xml
   <!--配置SpringMVC的前端控制器DispatcherServlet，对浏览器发送的请求统一进行处理
           SpringMVC的配置文件默认的位置和名称：
           位置：WEB-INF下
           名称：<servlet-name>的值+ -servlet.xml
           例如：当前的SpringMVC-servlet.xml
        -->
   ```

3. 处理请求的方法需要返回一个字符串类型的视图名称，该视图名称会 被视图解析器解析，加上前缀和后缀组成视图的路径，通过Thymeleaf对视图进行渲染，最终转发到视 图所对应页面

### 2.7 扩展

* **如何将SpringMVC配置文件置入到resource目录下**

在web.xml中配置servlet标签中的<init-param>属性

```xml
<servlet>
	<servlet-name>SpringMVC</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 设置springmvc配置文件的位置和名称 -->
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:springMVC.xml</param-value>
	</init-param>
    <!-- 
		由于当DispatcherServlet默认初始化是在第一次访问的时候
		而DispatcherServlet初始化的东西很多，为了优化初始化时间
		将DispatcherServlet的初始化时间提前到服务器启动时
	-->
    <load-on-startup>1</load-on-startup>
</servlet>
```

springMVC.xml是指定的配置文件，之后在resource路径下创建同名文件即可

> pom.xml变成橙色，右键选择 add as project maven



## 3、@RequestMapping注解

### 3.1@RequestMapping注解的功能

> 作用就是将请求和处理请求的控制器方法关联起来，建立映射关系。
>
> 当SpringMVC前端控制器接收到指定的请求，就会找到在映射关系中对应的控制器方法来处理这个请求

### 3.2@RequestMapping的位置

>可以标识到一个方法上，设置映射请求请求路径的具体信息
>
>也可以标识到一个类上，设置映射请求请求路径的初始信息
>
>然后一个方法的位置信息就等于类上的注解信息加上方法上的注解信息

### 3.3values属性

> values属性是字符数组类型的，说明可以写入多个请求路径，即一个控制器可以处理多个请求路径

```xml
@RequestMapping({"/hello","/abc"})
```

### 3.4method属性

> method是设置请求方法，若不符合设置的请求方法，则不会处理
>
> 即通过请求方式匹配请求
>
> method属性值是RequestMethod[],枚举数组



> 注： 
>
> 1、对于处理指定请求方式的控制器方法，SpringMVC中提供了@RequestMapping的派生注解 
>
> 处理get请求的映射-->@GetMapping 
>
> 处理post请求的映射-->@PostMapping 
>
> 处理put请求的映射-->@PutMapping 
>
> 处理delete请求的映射-->@DeleteMapping 
>
> 2、常用的请求方式有get，post，put，delete 
>
> 但是目前浏览器只支持get和post，若在form表单提交时，为method设置了其他请求方式的字符 串（put或delete），则按照默认的请求方式get处理 
>
> 若要发送put和delete请求，则需要通过spring提供的过滤器HiddenHttpMethodFilter，在 RESTful部分会讲到

### 3.5params属性和header属性

> @RequestMapping注解的params属性通过请求的请求参数匹配请求映射 @RequestMapping注解的params属性是一个字符串类型的数组，可以通过四种表达式设置请求参数 和请求映射的匹配关系 
>
> "param"：要求请求映射所匹配的请求必须携带param请求参数
>
> "!param"：要求请求映射所匹配的请求必须不能携带param请求参数 
>
> "param=value"：要求请求映射所匹配的请求必须携带param请求参数且param=value 
>
> "param!=value"：要求请求映射所匹配的请求必须携带param请求参数但是param!=value

```html
<a th:href="@{/test(username='admin',password=123456)">测试@RequestMapping的
params属性-->/test</a><br>
```

headers同理，略。。

### 3.6 SpringMVC支持ant风格的路径

？：表示任意的单个字符 

*：表示任意的0个或多个字符 

**：表示任意层数的任意目录 

注意：在使用 ** 时，只能使用 /**/xxx的方式



### 3.7 支持路径中的占位符☆

> 原始方式：/deleteUser?id=1 
>
> rest方式：/user/delete/1
>
> SpringMVC路径中的占位符常用于RESTful风格中，当请求路径中将某些数据通过路径的方式传输到服 务器中，就可以在相应的@RequestMapping注解的value属性中通过占位符{xxx}表示传输的数据，在 通过@PathVariable注解，将占位符所表示的数据赋值给控制器方法的形参

```html
<a th:href="@{/testRest/1/admin}">测试路径中的占位符-->/testRest</a><br>
```

控制器映射方法：

```java
RequestMapping("/testRest/{id}/{username}")
public String testRest(@PathVariable("id") String id, @PathVariable("username")
String username){
System.out.println("id:"+id+",username:"+username);
return "success";
}
//最终输出的内容为-->id:1,username:admin
```

## 4、获取请求参数

### 4.1通过ServletAPI获取

> 当DispatcherServlet前端控制器当根据请求路径匹配到相应的处理器方法时，会根据方法的所需参数类型传入相应的数据。例如request、response等

```java
@RequestMapping("/param/servletAPI")
    public String getParamByservletAPI(HttpServletRequest request){
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        System.out.println(username +  ":"  + password);
        return "success";
    }
```

### 4.2通过控制器方法的形参获取

> 在控制器方法的形参位置，设置和请求参数同名的形参，当浏览器发送请求，匹配到请求映射时，在 DispatcherServlet中就会将请求参数赋值给相应的形参

```java
@RequestMapping("/param")
public String getParam(String username,String password){
        System.out.println("username:" + username + ",password:" + password);
        return "success";
    }
```

> 注： 
>
> 若请求所传输的请求参数中有多个同名的请求参数，此时可以在控制器方法的形参中设置字符串 数组或者字符串类型的形参接收此请求参数 
>
> 若使用字符串数组类型的形参，此参数的数组中包含了每一个数据 
>
> 若使用字符串类型的形参，此参数的值为每个数据中间使用逗号拼接的结果

### 4.3@RequestParam、@RequestHeader和@CookieValue注解

> @RequestParam是将请求参数和控制器方法的形参创建映射关系 
>
> @RequestParam注解一共有三个属性： 
>
> value：指定为形参赋值的请求参数的参数名 
>
> required：设置是否必须传输此请求参数，默认值为true 若设置为true时，则当前请求必须传输value所指定的请求参数，若没有传输该请求参数，且没有设置 defaultValue属性，则页面报错400：Required String parameter 'xxx' is not present；若设置为 false，则当前请求不是必须传输value所指定的请求参数，若没有传输，则注解所标识的形参的值为 null 
>
> defaultValue：不管required属性值为true或false，当value所指定的请求参数没有传输或传输的值 为""时，则使用默认值为形参赋值

@RequestHeader和@CookieValue的用法同理



```java
@RequestMapping("/param")
    public String getParam(
            @RequestParam("userName") String username, String password,
            @RequestHeader("referer") String referer,
            @CookieValue("JSESSIONID") String jsessionid
    ){
        System.out.println("username:" + username + ",password:" + password);
        System.out.println("referer:" + referer);
        System.out.println("jsessionid:" + jsessionid);
        return "success";
    }
```

> 当处理器方法获取第一次session对象时，会创建一个cookie，键为JSESSIONID，值是随机生成的一串数码。
>
> HttpSession session = request.getSession();

### 4.4通过pojo获取请求参数

> 可以在控制器方法的形参位置设置一个实体类类型的形参，此时若浏览器传输的请求参数的参数名和实 体类中的属性名一致，那么请求参数就会为此属性赋值

```java
@RequestMapping("/testpojo")
public String testPOJO(User user){
System.out.println(user);
return "success";
}
//最终结果-->User{id=null, username='张三', password='123', age=23, sex='男',
email='123@qq.com'}

```

### 4.5 解决获取请求参数乱码问题

> 解决获取请求参数的乱码问题，可以使用SpringMVC提供的编码过滤器CharacterEncodingFilter，但是 必须在web.xml中进行注册
>
> 注：在设置编码前若获取了参数则设置的编码将没有任何作用，所以不能通过SpringMVC的ServletAPI获取的request对象来设置编码类型。



> post方法获取中文参数的都会有乱码
>
> tomcat7.0用get方法获取中文参数会有乱码，可以通过设置tomcat中server.xml的解决
>
> tomcat8.5的版本get方法不会出现乱码

```xml
<!--配置springMVC的编码过滤器-->
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
    <!--  设置初始参数forceEncoding可以将响应的编码类型也一起设置  -->
    <init-param>
      <param-name>forceEncoding</param-name>
      <param-value>true</param-value>
    </init-param>
</filter>
```



## 5、域对象共享数据

> 在JSP中有四个与对象：
>
> pageContext:当前页面范围
>
> request:一次请求
>
> session:一个会话范围（直到关闭浏览器）
>
> application:整个web工程服务
>
> 而thymeleaf只有后三个

### 5.1使用ServletAPI向request与对象共享数据

```java
@RequestMapping("/testServletAPI")
public String testServletAPI(HttpServletRequest request){
    request.setAttribute("testScope","hello.servletAPI");
    return "success";
}
```

### 5.2使用ModelAndView向request域对象共享数据

> 需要将方法返回类型设置成ModelAndView，底层中数据都会被封装到ModelAndView中

处理器方法：

```java
@RequestMapping("test/mav")
    public ModelAndView testMAV(){
        /*
        * ModelAndView包含Model和View的功能
        * Model：向请求域中共享数据
        * View：设置逻辑视图实现页面跳转
        * */
        ModelAndView mav = new ModelAndView();
        mav.addObject("testRequestScope","hello,ModelAndView");
        mav.setViewName("success");
        return mav;
    }
```

html：

```html
<body>
<h1>success.html</h1>
<p th:text="${testRequestScope}"></p>
</body>
```

### 5.3使用Model、ModelMap、map向请求域共享数据

Model:

```java
@RequestMapping("/testModel")
public String testModel(Model model){
	model.addAttribute("testScope", "hello,Model");
	return "success";
}
```

Map:

```java
@RequestMapping("/testMap")
public String testMap(Map<String, Object> map){
	map.put("testScope", "hello,Map");
	return "success";
}
```

ModelMap:

```java
@RequestMapping("/testModelMap")
public String testModelMap(ModelMap modelMap){
	modelMap.addAttribute("testScope", "hello,ModelMap");
	return "success";
}
```

三者的关系：

Model、ModelMap、Map类型的参数其实本质上都是 BindingAwareModelMap 类型的

> public interface Model{} 
>
> public class ModelMap extends LinkedHashMap {} 
>
> public class ExtendedModelMap extends ModelMap implements Model {} 
>
> public class BindingAwareModelMap extends ExtendedModelMap {}

### 5.4向session和application域共享数据

session（客户端关闭浏览器时失效）:

```java
@RequestMapping("/testSession")
public String testSession(HttpSession session){
	session.setAttribute("testSessionScope", "hello,session");
	return "success";
}
```

![屏幕截图 2023-08-20 192311](D:\data\java笔记\markdown笔记\SSM\images\屏幕截图 2023-08-20 192311.png)

通过勾选Preser sessions across restarts and redeploys来设置重启服务器时不清空session

**session的钝化和活化:**

服务器关闭了，但是session中的数据会被钝化（保存）到服务器的work目录下；当服务器再次启动时session中的数据就会被活化（加载）到session中。所以如果要在session里面共享一个实体类的对象，则这个对象要实现序列化。

>什么是序列化操作？
>指将对象以流的形式从内存写出到磁盘
>什么是反序列化操作？
>将本地磁盘中的对象以流的形式还原成内存中的对象

application（服务器关闭时失效）:

```java
@RequestMapping("/testApplication")
public String testApplication(HttpSession session){
	ServletContext application = session.getServletContext();
	application.setAttribute("testApplicationScope", "hello,application");
	return "success";
}
```

## 6、SpringMVC的视图

> 在SpringMVC中的视图是View接口，视图的作用渲染数据，将模型Model中的数据展示给用户
>
> SpringMVC视图的种类很多，默认有转发视图和重定向视图。
>
> 当工程中有JSP并且引用了JSTL标签，视图会自动从默认的转发视图InternalResourceView或者默认的重定向视图RedirectView转化为JSTLView
>
> 若使用的视图技术为Thymeleaf，在SpringMVC的配置文件中配置了Thymeleaf的视图解析器，由此视 图解析器解析之后所得到的是ThymeleafView

### 6.1 ThymeleafView

> 当控制器方法中所设置的视图名称没有任何前缀时，此时的视图名称会被SpringMVC配置文件中所配置 的视图解析器解析，视图名称拼接视图前缀和视图 后缀所得到的最终路径，会通过转发的方式实现跳转

### 6.2 转发视图

> SpringMVC中默认的转发视图是InternalResourceView 
>
> SpringMVC中创建转发视图的情况： 
>
> 当控制器方法中所设置的视图名称以"forward:"为前缀时，创建InternalResourceView视图，此时的视 图名称不会被SpringMVC配置文件中所配置的视图解析器解析，而是会将前缀"forward:"去掉，剩余部 分作为最终路径通过转发的方式实现跳转 
>
> 例如"forward:/"，"forward:/employee"

```java
@RequestMapping("/testForward")
public String testForward(){
	return "forward:/testHello";
}
```

> InternalResourceView和ThymeleafView都是转发，但是前者用得不多
>
> 由于当页面运用到Thymeleaf语法，需要用ThymeleafView转发并解析
>
> 当用jsp来作为视图技术，需要在SpringMVC配置文件中配置不同的视图解析器

### 6.3 重定向视图

> 当控制器方法中所设置的视图名称以"redirect:"为前缀时，创建RedirectView视图，此时的视图名称不 会被SpringMVC配置文件中所配置的视图解析器解析，而是会将前缀"redirect:"去掉，剩余部分作为最 终路径通过重定向的方式实现跳转 
>
> 例如"redirect:/"，"redirect:/employee"
>
> 注： 
>
> 重定向视图在解析时，会先将redirect:前缀去掉，然后会判断剩余部分是否以/开头，若是则会自 动拼接上下文路径

```java
@RequestMapping("/test/view/redirect")
public String testRedirectView(){
    return "redirect:/test/view/thymeleaf";
}
```



### 6.4 视图控制器view-controller

> 控制器方法中，仅仅用来实现页面跳转，即只需要设置视图名称时，可以将处理器方法在SpringMVC配置文件中使用view-controller标签进行表示

```xml
<!--开启MVC的注解驱动-->
<mvc:annotation-driven/>
<!--
    视图控制器：为当前的请求直接设置视图名称实现页面跳转
若设置视图控制器，则只有视图控制器所设置的请求会被处理，其它请求将全部404
此时必须再配置一个标签开启注解驱动
-->
<mvc:view-controller path="/" view-name="index"></mvc:view-controller>
```



## 7、RESTful

### 7.1RESTful简介

> REST：Representational State Transfer，表现层资源状态转移。
>
> RESTful是一种风格，也是一种看待服务器的一种方式，把服务器全部内容都看成资源，即一切皆资源。
>
> 以名称的为核心标识一个资源，并添加到URL中，即URLURI既是资源的名称，也是资源在Web上的地址，一个 资源可以由一个或多个URI来标识。

### 7.2 RESTful的实现

> 具体说，就是 HTTP 协议里面，四个表示操作方式的动词：GET、POST、PUT、DELETE。
>
>  它们分别对应四种基本操作：GET 用来获取资源，POST 用来新建资源，PUT 用来更新资源，DELETE 用来删除资源。 
>
> REST 风格提倡 URL 地址使用统一的风格设计，从前到后各个单词使用斜杠分开，不使用问号键值对方 式携带请求参数，而是将要发送给服务器的数据作为 URL 地址的一部分，以保证整体风格的一致性。
>
> 用同一种URL路径来标识访问同一个资源，而用不同的请求方式表示对资源的操作

| 操作     | 传统方式         | REST风格                |
| -------- | ---------------- | ----------------------- |
| 查询操作 | getUserById?id=1 | user/1-->get请求方式    |
| 保存操作 | savaUser         | user-->post请求方式     |
| 删除操作 | deleteUser?id=1  | user/1-->delete请求方式 |
| 更新操作 | updateUser       | user-->put请求方式      |

#### 设置注释快捷键：

setting->Editor->LiveTemplates(现存模板)

![屏幕截图 2023-08-20 230333](D:\data\java笔记\markdown笔记\SSM\images\屏幕截图 2023-08-20 230333.png)

右上角添加快捷键组，在里面添加快捷键；

Template设置模板内容

最下面蓝色的Define/Change是设置生效的语言

Edit variables设置变量的参数

Expand with设置生效按键

### 7.3HiddenHttpMethodFilter

> 由于浏览器只支持发送get和post方式的请求，那么该如何发送put和delete请求呢？
>
>  SpringMVC 提供了 HiddenHttpMethodFilter 帮助我们将 POST 请求转换为 DELETE 或 PUT 请求 



> HiddenHttpMethodFilter 处理put和delete请求的条件： 
>
> a>当前请求的请求方式必须为post 
>
> b>当前请求必须传输请求参数 _ method
>
> 满足以上条件，HiddenHttpMethodFilter 过滤器就会将当前请求的请求方式转换为请求参数  _ method的值，因此请求参数_ method的值才是最终的请求方式

```html
<form th:action="@{/user}" method="post">
    <input type="hidden" name="_method" value="put">
    <input type="submit" value="修改用户信息">
</form>
```

```xml
<!--设置处理请求方式的过滤器-->
  <filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
  </filter>

  <filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```

### 7.4 案例

* **功能清单**

| 功能               | URL地址     | 请求方式 |
| ------------------ | ----------- | -------- |
| 访问首页           | /           | GET      |
| 查询全部数据       | /employee   | GET      |
| 删除               | /employee   | DELETE   |
| 跳转到添加数据页面 | /toAdd      | GET      |
| 执行保存           | /employee   | POST     |
| 跳转到更新数据页面 | /employee/2 | GET      |
| 执行更新           | /employee   | PUT      |



* 处理静态资源

  ```xml
  <!--
      配置默认的servlet处理静态资源
      当前工程的web.xml配置的前端控制器DispatcherServlet的url-pattern是/
      tomcat的web.xml配置的DefaultServlet的url-pattern也是/
      此时，浏览器分发送的请求会优先被DispatcherServlet进行处理，但它无法处理静态资源
      若配置了<mvc:default-servlet-handler/>和<mvc:annotation-driven/>
      则浏览器请求先被DispatcherServlet处理，无法处理再有DefaultServlet处理
      -->
      <mvc:default-servlet-handler/>
  
      <!--开启MVC的注解驱动-->
      <mvc:annotation-driven/>
      <!--
  ```

## 8、SpringMVC处理ajax请求

> 导入axios.min.js和vue.js的静态资源

首页html：

```xml
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>首页</title>
</head>
<body>

<div id="app">
    <h1>index.html</h1>
    <input type="button" value="测试springMVC处理ajax" @click="testAjax()">
</div>

<script type="text/javascript" th:src="@{/js/vue.js}"></script>
<script type="text/javascript" th:src="@{/js/axios.min.js}"></script>
<script type="text/javascript">
    /**
     * axios({
     * url:"",//请求路径
     * method:"",//请求方式
     * //以name=value&name=value的方式发送的请求参数
     * //不管是get还是post都会被拼接到请求地址后
     * //服务器可以通过request.getParameter()获取
     * params:{},
     *
     * //以json格式发送的请求参数，会被保存到请求报文的请求体中，所以一定要用post
     * //原生方法：利用处理json格式的jar包获取
     * data:{}
     * }).then(response=>{
     *  //服务器中的数据被封装到response.data中
     *  console.log(response.data);
     *  });
     */
    var vue = new Vue({
        /*设置一个挂载容器*/
        el:"#app",
        methods:{
            testAjax(){
                axios.post(
                    "/SpringMVC/test/ajax?id=1001",
                    {username:"admin",password:"123456"}
                ).then(response=>{
                    console.log(response.data);
                });
            }
        }
    });
</script>
</body>
</html>
```

### 8.1 @RequestBody处理json格式请求

> 使用@RequestBody注解将json根式的请求对象参数转换为java对象
>      * a>导入jackson的依赖
>           * b>在springMVC配置文件中设置< mvc:annotation-driven/>
>           * c>在处理请求控制器方法的形参位置，
>                * 直接设置json格式的请求参数要转换的java类型的形参用@RequestBody标识

* 初步使用

```java
@Controller
public class TestControllerAjax {

    @RequestMapping("/test/ajax")
    public void testAjax(Integer id, @RequestBody String requestBody, HttpServletResponse response) throws IOException {
        System.out.println("requestBody:"+ requestBody);
        System.out.println("id:"+ id);
        response.getWriter().write("hello,axios");
    }
}
//requestBody:{"username":"admin","password":"123456"}
//id:1001
```



```java
/**
     *
     * @param response
     * @throws IOException
     * 使用@RequestBody注解将json根式的请求对象参数转换为java对象
     * a>导入jackson的依赖
     * b>在springMVC配置文件中设置<mvc:annotation-driven/>
     * c>在处理请求控制器方法的形参位置，
     * 直接设置json格式的请求参数要转换的java类型的形参用@RequestBody标识
     */
    @RequestMapping("/test/RequestBody/json")
    public void testRquestBody(@RequestBody User user, HttpServletResponse response) throws IOException {
        System.out.println(user);
        response.getWriter().write("hello RequestBody");
    }

//将json格式的数据转换为map集合
@RequestMapping("/test/RequestBody/json")
public void testRequestBody(@RequestBody Map<String, Object> map,
HttpServletResponse response) throws IOException {
	System.out.println(map);
	//{username=admin, password=123456}
	response.getWriter().print("hello,axios");
}

```

### 8.2@ResponseBody返回json格式数据

> 在处理器上面加上这个注解，会使该方法的返回值作为响应报文的响应体响应到浏览器中
>
> 
>
> 使用@ResponseBody注解将json根式的请求对象参数转换为java对象
>
> a>导入jackson的依赖
>
> b>在soringMVC配置文件中设置< mvc:annotation-driven/>
>
> c>需要将需要返回的java对象作为处理器方法的返回值，然后将用@ResponseBody标识
>
> 即可自动将java对象转换为json格式响应到浏览器

```java
/**
*常用的java对象转换为json的结果：
*实体类、map-->json对象
*list-->json数组
*/
@RequestMapping("/test/ResponseBody")
@ResponseBody
public User testResponseBody(){
        User user = new User(1001,"admin","admin",23,"男");
        return user;
}
```

### 8.3@RestController注解

> @RestController注解是springMVC提供的一个复合注解，标识在控制器的类上，将相当于为类添加了@Controller注解，并且为其中的每个方法添加了@ResponseBody注解



## 9、文件上传和下载

> IDEA中Ctrl + 鼠标左键可以查看源码
>
> 若控制器方法没有设置返回值，即为void时，会将请求路径作为逻辑视图返回到前端控制器

### 9.1文件下载

> ResponseEntity用于控制器方法的返回值类型，该控制器方法的返回值就是响应到浏览器的响应报文，即就只要把文件数据封装到ResponseEntity中即可实现文件下载

```java
@RequestMapping("/test/down")
    public ResponseEntity<byte[]> testResponseEntity(HttpSession session) throws IOException {
        //获取ServletContext对象
        ServletContext servletContext = session.getServletContext();
        //获取服务器中文件的真实路径
        //getRealPath方法会将字符串中的内容拼接到工程的真实路径后
        String realPath = servletContext.getRealPath("images");
        //当不知道文件之间的分隔符用什么的时候（在不同的系统的文件分隔符可能不同）使用这个方法来获取文件路径
        realPath = realPath + File.separator + "1.png";
        //创建输入流
        InputStream is = new FileInputStream(realPath);
        //创建字节数组
        //is.available()是获取目标文件的字节数
        byte[] bytes = new byte[is.available()];
        //将流读到字节数组中
        is.read(bytes);
        //创建HttpHeaders对象设置响应头信息
        MultiValueMap<String, String> headers = new HttpHeaders();
        //设置要下载方式以及下载文件的默认名字
        //attachment以附件的形式
        headers.add("Content-Disposition", "attachment;filename=1.png");
        //设置响应状态码
        HttpStatus statusCode = HttpStatus.OK;
        //创建ResponseEntity对象
        ResponseEntity<byte[]> responseEntity = new ResponseEntity<>(bytes, headers,
                statusCode);
        //关闭输入流
        is.close();
        return responseEntity;
    }
```

### 9.2文件上传

> 文件上传要求：
>
> 1. form表单的请求方式必须问post
> 2. form表单必须设置属性enctype=“multipart/form-data”
>
> 在SpringMVC中将上传的文件封装到了MultipartFile对象中，名字要与前端中文件域的name属性相对应以匹配

在springMVC配置文件中，配置文件上传解析器

但是在配置bean单例（默认）时会在jar包未被导入时就加载，会出现类找不到的情况

所以要在maven项目中提前导入相关jar包

```xml
<!-- https://mvnrepository.com/artifact/commons-fileupload/commons-fileupload -->
    <dependency>
      <groupId>commons-fileupload</groupId>
      <artifactId>commons-fileupload</artifactId>
      <version>1.3.1</version>
    </dependency>
```

```xml
<!-- 配置文件上传解析器-->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        
    </bean>
```

控制器方法：

```java
@RequestMapping("/test/up")
    public String testUp(MultipartFile photo, HttpSession session) throws IOException {
        //获取上传文件的文件名
        String fileName = photo.getOriginalFilename();
        //获取文件后缀
        String hzName = fileName.substring(fileName.lastIndexOf("."));
        //获取UUID
        String uuid = UUID.randomUUID().toString();
        //拼接成新的文件名
        fileName = uuid + hzName;
        //获取ServletContext对象然后获取工程真实路径
        ServletContext context = session.getServletContext();
        String photoPath = context.getRealPath("photo");
        //创建photoPath对应的文件对象
        File file = new File(photoPath);
        if(!file.exists()){
            file.mkdir();
        }
        String finalPath = photoPath + File.separator + fileName;
        //保存上传文件
        //但是如果第一次创建“photo”目录，并且文件名有中文会出现错误
        photo.transferTo(new File(finalPath));
        return "success";
    }
```

### 9.3 上传多个文件



## 10、拦截器 interceptor

> 拦截器和过滤器的区别：
>
> 浏览器发送请求访问目标资源后，先由过滤器执行完以后再交由DispatcherServlet控制器处理。
>
> DispatcherServlet又根据注解来匹配控制器方法，而拦截器的三个方法就是分布在控制器方法执行前后进行的。（前、后、视图控制器执行完后）

### 10.1拦截器的配置

> SpringMVC中的拦截器用于拦截控制器方法的执行 
>
> SpringMVC中的拦截器需要实现HandlerInterceptor

SpringMVC的拦截器必须在SpringMVC的配置文件中进行配置：

```xml
<bean class="com.atguigu.interceptor.FirstInterceptor"></bean>
<!-- 下面这种方式要加上注解和调整组件扫描的路径-->
<ref bean="firstInterceptor"></ref>
<!-- 以上两种配置方式都是对DispatcherServlet所处理的所有的请求进行拦截 -->
<mvc:interceptor>
	<!-- 用"/*"只会拦截所有只有一层路径的请求，所有要用"/**"-->
	<mvc:mapping path="/**"/>
	<mvc:exclude-mapping path="/testRequestEntity"/>
	<ref bean="firstInterceptor"></ref>
</mvc:interceptor>
<!--
以上配置方式可以通过ref或bean标签设置拦截器，通过mvc:mapping设置需要拦截的请求，通过
mvc:exclude-mapping设置需要排除的请求，即不需要拦截的请求
-->
```

> SpringMVC中的拦截器有三个抽象方法： 
>
> **拦截器需要实现HandlerInterceptor，并实现preHandle方法**
>
> preHandle：控制器方法执行之前执行preHandle()，其boolean类型的返回值表示是否拦截或放行，返 回true为放行，即调用控制器方法；返回false表示拦截，即不调用控制器方法 
>
> postHandle：控制器方法执行之后执行postHandle() 
>
> afterCompletion：处理完视图和模型数据，渲染视图完毕之后执行afterCompletion()

### 10.2 多个拦截器的执行顺序

> ①若每个拦截器的preHandle()都返回true 
>
> 此时多个拦截器的执行顺序和拦截器在SpringMVC的配置文件的配置顺序有关： preHandle()会按照配置的顺序执行，而postHandle()和afterCompletion()会按照配置的反序执行 
>
> ②若某个拦截器的preHandle()返回了false 
>
> preHandle()返回false和它之前的拦截器的preHandle()都会执行，postHandle()都不执行，返回false 的拦截器之前的拦截器的afterCompletion()会执行



## 11、异常处理器

> SpringMVC提供了一个处理控制器方法执行过程中所出现的异常的接口：HandlerExceptionResolver 
>
> HandlerExceptionResolver接口的实现类有：DefaultHandlerExceptionResolver和 SimpleMappingExceptionResolver 
>
> SpringMVC提供了自定义的异常处理器SimpleMappingExceptionResolver

### 11.1 异常处理的使用

**基于xml配置的异常处理：**

```xml
<bean
class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
	<property name="exceptionMappings">
		<props>
	<!--
properties的键表示处理器方法执行过程中出现的异常
properties的值表示若出现指定异常时，设置一个新的视图名称，跳转到指定页面
-->
			<prop key="java.lang.ArithmeticException">error</prop>
	</props>
	</property>
<!--
exceptionAttribute属性设置一个属性名，将出现的异常信息在请求域中进行共享,可以在前端中使用
-->
	<property name="exceptionAttribute" value="ex"></property>
</bean>

```

**基于注解的异常处理：**

```java
//@ControllerAdvice将当前类标识为异常处理的组件
@ControllerAdvice
public class ExceptionController {
	//@ExceptionHandler用于设置所标识方法处理的异常
	@ExceptionHandler(ArithmeticException.class)
	//ex表示当前请求处理中出现的异常对象
	public String handleArithmeticException(Exception ex, Model model){
		model.addAttribute("ex", ex);
		return "error";
	}
}
```

## 12、注解配置SpringMVC

### 12.1 创建初始化类，代替web.xml

> 在Servlet3.0环境中，容器会在类路径中查找实现javax.servlet.ServletContainerInitializer接口的类， 如果找到的话就用它来配置Servlet容器。 
>
> Spring提供了这个接口的实现，名为 SpringServletContainerInitializer，这个类反过来又会查找实现WebApplicationInitializer的类并将配 置的任务交给它们来完成。Spring3.2引入了一个便利的WebApplicationInitializer基础实现，名为 AbstractAnnotationConfigDispatcherServletInitializer，当我们的类扩展了 AbstractAnnotationConfigDispatcherServletInitializer并将其部署到Servlet3.0容器的时候，容器会自 动发现它，并用它来配置Servlet上下文。

```java
//代替web.xml
public class WebInit extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    //设置配置类来代替Spring的配置文件
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{SpringConfig.class};
    }
    @Override
    //设置配置类来代替SpringMVC的配置文件
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }
    @Override
    //设置SpringMVC的前端控制器Dispa的 url-pattern
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
    @Override
    //设置当前的过滤器
    protected Filter[] getServletFilters() {
        //创建编码过滤器
        CharacterEncodingFilter characterEncodingFilter = new CharacterEncodingFilter();
        characterEncodingFilter.setEncoding("UTF-8");
        characterEncodingFilter.setForceEncoding(true);
        //创建请求处理方式的过滤器
        HiddenHttpMethodFilter hiddenHttpMethodFilter = new HiddenHttpMethodFilter();
        return new Filter[]{characterEncodingFilter,hiddenHttpMethodFilter};
    }
}
```

### 12.2 创建SpringConfig代替Spring配置文件



### 12.3 创建WebConfig代替SpringMVC配置文件

```java
/**
 * 扫描组件，视图解析器，默认的servlet，mvc的注解驱动，
 * 视图控制器，文件上传解析器，拦截器，异常解析器
 */
//将这类标识问配置类，代替springMVC的配置文件
@Configuration
@ComponentScan("com.demo.controller")
//开启mvc的注解驱动
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
    @Override
    //默认的servlet处理静态资源
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
    @Override
    //配置视图控制器
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("index");
    }
    @Bean
    //@Bean注解可以将标识的方法的返回值作为bean进行管理，bean的id为方法名
    public CommonsMultipartResolver multipartResolver(){
        return new CommonsMultipartResolver();
    }
    @Override
    //设置拦截器
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new FirstInterceptor()).addPathPatterns("/**");
    }
    @Override
    //配置异常解析器
    public void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        SimpleMappingExceptionResolver exceptionResolver = new SimpleMappingExceptionResolver();
        Properties prop = new Properties();
        prop.setProperty("java.lang.AritmeticException","error");
        exceptionResolver.setExceptionMappings(prop);
        exceptionResolver.setExceptionAttribute("ex");
        resolvers.add(exceptionResolver);
    }
    @Bean
    //配置生成模板解析器
    public ITemplateResolver templateResolver() {
        WebApplicationContext webApplicationContext =
                ContextLoader.getCurrentWebApplicationContext();
    // ServletContextTemplateResolver需要一个ServletContext作为构造参数，可通过WebApplicationContext 的方法获得
        assert webApplicationContext != null;
        ServletContextTemplateResolver templateResolver = new
                ServletContextTemplateResolver(
                webApplicationContext.getServletContext());
        templateResolver.setPrefix("/WEB-INF/templates/");
        templateResolver.setSuffix(".html");
        templateResolver.setCharacterEncoding("UTF-8");
        templateResolver.setTemplateMode(TemplateMode.HTML);
        return templateResolver;
    }
    //生成模板引擎并为模板引擎注入模板解析器
    @Bean
    public SpringTemplateEngine templateEngine(ITemplateResolver templateResolver) {
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(templateResolver);
        return templateEngine;
    }
    //生成视图解析器并未解析器注入模板引擎
    @Bean
    public ViewResolver viewResolver(SpringTemplateEngine templateEngine) {
        ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
        viewResolver.setCharacterEncoding("UTF-8");
        viewResolver.setTemplateEngine(templateEngine);
        return viewResolver;
    }
}
```

## 13、 SpringMVC执行流程

## 14、SSM整合

> Spring提供了监听器ContextLoaderListener，实现ServletContextListener接口，可监听 ServletContext的状态，在web服务器的启动，读取Spring的配置文件，创建Spring的IOC容器。web 应用中必须在web.xml中配置

```xml
<listener>
	<!--
	配置Spring的监听器，在服务器启动时加载Spring的配置文件
	Spring配置文件默认位置和名称：/WEB-INF/applicationContext.xml
	可通过上下文参数自定义Spring配置文件的位置和名称
	-->
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
	<!--自定义Spring配置文件的位置和名称-->
<context-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>classpath:spring.xml</param-value>
</context-param>
```

### 14.1准备工作

* 引入依赖

  ```xml
    <properties>
      <spring.version>5.3.1</spring.version>
    </properties>
  
    <dependencies>
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
      </dependency>
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>${spring.version}</version>
      </dependency>
      <!--springmvc-->
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>${spring.version}</version>
      </dependency>
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>${spring.version}</version>
      </dependency>
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>${spring.version}</version>
      </dependency>
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aspects</artifactId>
        <version>${spring.version}</version>
      </dependency>
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${spring.version}</version>
      </dependency>
      <!-- Mybatis核心 -->
      <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.7</version>
      </dependency>
      <!--mybatis和spring的整合包-->
      <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>2.0.6</version>
      </dependency>
      <!-- 连接池 -->
      <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.0.9</version>
      </dependency>
      <!-- junit测试 -->
      <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
      </dependency>
      <!-- MySQL驱动 -->
      <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.16</version>
      </dependency>
      <!-- log4j日志是一个日志的门面 -->
      <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
      </dependency>
      <!-- https://mvnrepository.com/artifact/com.github.pagehelper/pagehelper -->
       <!-- 分页插件 -->
      <dependency>
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper</artifactId>
        <version>5.2.0</version>
      </dependency>
      <!-- lo4gj日志的实现 -->
      <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.3</version>
      </dependency>
      <!-- ServletAPI -->
      <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
        <scope>provided</scope>
      </dependency>
      <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.12.1</version>
      </dependency>
      <dependency>
        <groupId>commons-fileupload</groupId>
        <artifactId>commons-fileupload</artifactId>
        <version>1.3.1</version>
      </dependency>
      <!-- Spring5和Thymeleaf整合包 -->
      <dependency>
        <groupId>org.thymeleaf</groupId>
        <artifactId>thymeleaf-spring5</artifactId>
        <version>3.0.12.RELEASE</version>
      </dependency>
    </dependencies>
  ```

  

* 创建表

  ```mysql
  CREATE TABLE `t_emp` (
  `emp_id` int(11) NOT NULL AUTO_INCREMENT,
  `emp_name` varchar(20) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `sex` char(1) DEFAULT NULL,
  `email` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`emp_id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8
  ```

  

* 配置web.xml文件

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
           version="4.0">
    <!--配置Spring的编码过滤器-->
    <filter>
      <filter-name>CharacterEncodingFilter</filter-name>
      <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
      <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
      </init-param>
      <init-param>
        <param-name>forceEncoding</param-name>
        <param-value>true</param-value>
      </init-param>
    </filter>
    <filter-mapping>
      <filter-name>CharacterEncodingFilter</filter-name>
      <url-pattern>/*</url-pattern>
    </filter-mapping>
    <!--配置处理请求方式的过滤器-->
    <filter>
      <filter-name>HiddenHttpMethodFilter</filter-name>
      <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
    </filter>
    <filter-mapping>
      <filter-name>HiddenHttpMethodFilter</filter-name>
      <url-pattern>/*</url-pattern>
    </filter-mapping>
  
    <!--配置前端控制器-->
    <servlet>
      <servlet-name>SpringMVC</servlet-name>
      <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
      <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc.xml</param-value>
      </init-param>
      <!--设置前端控制器初始化时间-->
      <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
      <servlet-name>SpringMVC</servlet-name>
      <url-pattern>/</url-pattern>
    </servlet-mapping>
  
    <!--配置spring的监听器，在服务器启动时加载Spring的配置文件-->
    <listener>
      <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
  
    <!--设置Spring配置文件自定义的位置和名称-->
    <context-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:spring.xml</param-value>
    </context-param>
  </web-app>
  ```

  

* 配置SpringMVC.xml文件

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:mvc="http://www.springframework.org/schema/mvc"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">
  
      <!--扫描控制层组件-->
      <context:component-scan base-package="com.demo.ssm.controller"></context:component-scan>
  
      <!-- 配置Thymeleaf视图解析器 -->
      <bean id="viewResolver"
            class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
          <property name="order" value="1"/>
          <property name="characterEncoding" value="UTF-8"/>
          <property name="templateEngine">
              <bean class="org.thymeleaf.spring5.SpringTemplateEngine">
                  <property name="templateResolver">
                      <bean
                              class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
                          <!--
                                  逻辑视图 = 物理视图 - 前缀 - 后缀
                                  物理视图：/WEB-INF/templates/index.html
                                  逻辑视图：index
                              -->
                          <!-- 视图前缀 -->
                          <property name="prefix" value="/WEB-INF/templates/"/>
                          <!-- 视图后缀 -->
                          <property name="suffix" value=".html"/>
                          <property name="templateMode" value="HTML5"/>
                          <property name="characterEncoding" value="UTF-8" />
                      </bean>
                  </property>
              </bean>
          </property>
      </bean>
      <!--
      当前工程的web.xml配置的前端控制器DispatcherServlet的url-pattern是/
      tomcat的web.xml配置的DefaultServlet的url-pattern也是/
      此时，浏览器分发送的请求会优先被DispatcherServlet进行处理，但它无法处理静态资源
      若配置了<mvc:default-servlet-handler/>和<mvc:annotation-driven/>
      则浏览器请求先被DispatcherServlet处理，无法处理再有DefaultServlet处理
      -->
      <!--配置默认的servlet处理静态资源-->
      <mvc:default-servlet-handler/>
  
      <!--开启MVC的注解驱动-->
      <mvc:annotation-driven/>
  
      <!--
      视图控制器：为当前的请求直接设置视图名称实现页面跳转
      若设置视图控制器，则只有视图控制器所设置的请求会被处理，其它请求将全部404
      此时必须再配置一个标签开启注解驱动
      -->
      <mvc:view-controller path="/" view-name="index"></mvc:view-controller>
  
      <!-- 配置文件上传解析器-->
      <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
      </bean>
  
      <mvc:interceptors>
          <!--<bean class="com.demo.interceptor.FirstInterceptor"></bean>-->
          <!--<ref bean="firstInterceptor"></ref>-->
          <mvc:interceptor>
              <mvc:mapping path="/*"/>
              <mvc:exclude-mapping path="/abc"/>
              <ref bean="firstInterceptor"></ref>
          </mvc:interceptor>
      </mvc:interceptors>
  
  
  </beans>
  ```

* 配置spring.xml文件

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
      
      <!--扫描组件，除了控制层-->
      <context:component-scan base-package="com.demo.ssm">
          <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
      </context:component-scan>
  
      <!--引入jdcb.properties文件-->
      <context:property-placeholder location="classpath:jdbc.properties"></context:property-placeholder>
  
      <!--配置数据源-->
      <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
          <property name="driverClassName" value="${jdbc.driver}"></property>
          <property name="url" value="${jdbc.url}"></property>
          <property name="username" value="${jdbc.username}"></property>
          <property name="password" value="${jdbc.password}"></property>
      </bean>
      
      <!--配置事务管理器-->
      <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
          <property name="dataSource" ref="dataSource"></property>
      </bean>
      <!--开启事务注解驱动-->
      <!--
          将使用注解@Transactional标识的方法或类中的所有的方法进行事务管理
      -->
      <tx:annotation-driven transaction-manager="transactionManager"/>
      
      <!--配置SqlSessionFactoryBean,可以直接在Spring的IOC中获取SqlSessionFactory对象-->
      <bean class="org.mybatis.spring.SqlSessionFactoryBean">
          <!--设置当前mybatis核心配置文件的路径-->
          <property name="configLocation" value="classpath:mybatis-config.xml"></property>
          <!--设置数据源，就不用mybatis核心配置文件中设置-->
          <property name="dataSource" ref="dataSource"></property>
          <!--设置类型别名所对应的包-->
          <property name="typeAliasesPackage" value="com.demo.ssm.pojo"></property>
          <!--用来设置映射文件的路径，只有映射文件的包和mapper接口的包不一致时才要设置-->
          <!--<property name="mapperLocations" value="classpath:mappers/*.xml"></property>-->
          <!--可以在这个地方配置数据源、映射等配置信息.用这种方法就不用引入mybatis核心配置文件-->
      </bean>
      
      <!--
      配置mapper接口的扫描
      可以把设置的包下的所有的mapper接口，
      通过SqlSessionFactoryBean创建出来的sqlSession，
      来获取所有mapper接口的代理实现类对象，
      并且交由IoC容器管理
  	但是映射文件和mapper接口的包要一致
  	然后就可以在Service层直接自动装配mapper接口对象了
      -->
      <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
          <property name="basePackage" value="com.demo.ssm.mapper"></property>
      </bean>
  </beans>
  ```

* mybatis核心配置文件

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE configuration
          PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <configuration>
  
      <settings>
          <!--将下划线映射成驼峰-->
          <setting name="mapUnderscoreToCamelCase" value="true"/>
      </settings>
  
      <!--配置分页插件-->
      <plugins>
          <plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
      </plugins>
  </configuration>
  ```

* 配置log4j.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
  <log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">
      <appender name="STDOUT" class="org.apache.log4j.ConsoleAppender">
          <param name="Encoding" value="UTF-8" />
          <layout class="org.apache.log4j.PatternLayout">
              <param name="ConversionPattern" value="%-5p %d{MM-dd HH:mm:ss,SSS}
  %m (%F:%L) \n" />
          </layout>
      </appender>
      <logger name="java.sql">
          <level value="debug" />
      </logger>
      <logger name="org.apache.ibatis">
          <level value="info" />
      </logger>
      <root>
          <level value="debug" />
          <appender-ref ref="STDOUT" />
      </root>
  </log4j:configuration>
  ```

### 14.2 实现查询所有信息

功能列表：

```
/**
 * RESTful风格：
 * 查询所有员工信息--> /employee-->get
 * 根据id查询员工信息-->/employee/1-->get
 * 跳转到添加页面-->/to/add-->
 * 修改员工信息-->/employee-->post
 * 删除员工信息/employee/1-->delete
 */
```

前端页面：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>员工列表</title>
</head>
<body>
<table>
    <tr>
        <!--colspan是合并列-->
        <th colspan="6">员工列表</th>
    </tr>
    <tr>
        <th>流水号</th>
        <th>员工姓名</th>
        <th>年龄</th>
        <th>性别</th>
        <th>邮箱</th>
        <th>操作</th>
    </tr>
     <tr th:each="employee,status:${list}">
         <td th:text="${status.count}"></td>
         <td th:text="${employee.empName}"></td>
         <td th:text="${employee.age}"></td>
         <td th:text="${employee.gender}"></td>
         <td th:text="${employee.email}"></td>
         <td>
             <a href="">删除</a>
             <a href="">修改</a>
         </td>
     </tr>
</table>
</body>
</html>
```



控制器方法：

```java
@RequestMapping(value = "/employee",method = RequestMethod.GET)
    public String getAllEmployee(Model model){
        //查询所有的员工信息
        List<Employee> list = employeeService.getAllEmployee();
        //将员工信息在请求域中共享
        model.addAttribute("list",list);
        //跳转到employee_list.html
        return "employee_list";
    }
```

###  实现分页功能：

前端页面：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>员工列表</title>
</head>
<body>
<table>
    <tr>
        <!--colspan是合并列-->
        <th colspan="6">员工列表</th>
    </tr>
    <tr>
        <th>流水号</th>
        <th>员工姓名</th>
        <th>年龄</th>
        <th>性别</th>
        <th>邮箱</th>
        <th>操作</th>
    </tr>
     <tr th:each="employee,status:${page.list}">
         <td th:text="${status.count}"></td>
         <td th:text="${employee.empName}"></td>
         <td th:text="${employee.age}"></td>
         <td th:text="${employee.gender}"></td>
         <td th:text="${employee.email}"></td>
         <td>
             <a href="">删除</a>
             <a href="">修改</a>
         </td>
     </tr>
</table>
<div style="text-align: center;">
    <a th:if="${page.hasPreviousPage}" th:href="@{/employee/page/1}">首页</a>
    <a th:if="${page.hasPreviousPage}" th:href="@{'/employee/page/' + ${page.prePage}}">上一页</a>
    <span th:each="num : ${page.navigatepageNums}">
        <a th:if="${page.pageNum == num}"  style="color:red;" th:href="@{'/employee/page/' + ${num}}" th:text="'[' + ${num} + ']'"></a>
        <a th:if="${page.pageNum != num}" th:href="@{'/employee/page/' + ${num}}" th:text="${num}"></a>
    </span>
    <a th:if="${page.hasNextPage}" th:href="@{'/employee/page/' + ${page.nextPage}}">下一页</a>
    <a th:if="${page.hasNextPage}" th:href="@{'/employee/page/' + ${page.pages}}">末页</a>
</div>
</body>
</html>
```



Controller层代码：

```java
@RequestMapping(value = "/employee/page/{pageNum}",method = RequestMethod.GET)
    public String getEmployeePage(@PathVariable("pageNum") Integer pageNum,Model model){
        //获取员工的分页信息
        PageInfo<Employee> page = employeeService.getAllEmployeePage(pageNum);
        //将分页信息共享到请求域中
        model.addAttribute("page",page);
        return "employee_list";
    }
```

Service层代码：

```java
@Override
    public PageInfo<Employee> getAllEmployeePage(Integer pageNum) {
        //开启分页功能
        PageHelper.startPage(pageNum,4);
        //查询所有的员工信息
        List<Employee> list = employeeMapper.getAllEmployee();
        //获取分页相关数据
        PageInfo<Employee> page = new PageInfo<Employee>(list,5);

        return page;
    }
```
