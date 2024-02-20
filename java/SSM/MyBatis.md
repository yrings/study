# MyBatis

## 1、MyBatic简介

封装的是JDBC，所实现的功能是连接数据库和操作数据库中的数据

是一个基于java的持久层框架

**特性**：

1）定制化SQL、存储过程以及高级映射的优秀的持久层框架

2）避免了几乎所有的JDBC代码和手动设置参数以及获取结果集

3）可以用简单的XML或注解用于配置和原始映射，将接口和java的POJO（Plain Ordinary Java Object，普通的java对象）映射成数据库中的记录

4）是一个带着的的ORM（Object Relation Mapping）框架

MyBatis下载地址：https://github.com/mybatis/mybatis-3

可查看文档，可以到文档中找到约束



**对比**：

* JDBC ：
  * SQL 夹杂在Java代码中耦合度高，导致硬编码内伤 
  * 维护不易且实际开发需求中 SQL 有变化，频繁修改的情况多见 代码冗长，开发效率低 

* Hibernate（全自动持久层框架） 和 JPA ：
  * 操作简便，开发效率高 
  * 程序中的长难复杂 SQL 需要绕过框架 
  * 内部自动生产的 SQL，不容易做特殊优化 
  * 基于全映射的全自动框架，大量字段的 POJO 进行部分映射时比较困难。 
  * 反射操作太多，导致数据库性能下降 
* MyBatis 
  * 轻量级，性能出色 
  * SQL 和 Java 编码分开，功能边界清晰。Java代码专注业务、SQL语句专注数据 
  * 开发效率稍逊于HIbernate，但是完全能够接受



## 2、 搭建Mybatis

1.创建一个空的项目

2.在结构里面选择JDK的版本

3.在setting中的Build、Execution、Deployment中的maven中设置使用的maven





### **引入依赖**

```xml
<dependencies>
<!-- Mybatis核心 -->
	<dependency>
		<groupId>org.mybatis</groupId>
		<artifactId>mybatis</artifactId>
		<version>3.5.7</version>
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
</dependencies>

```

### 创建Mybatis的核心配置文件

> 习惯上命名为mybatis-config.xml，这个文件名仅仅只是建议，并非强制要求。将来整合Spring 之后，这个配置文件可以省略，所以大家操作时可以直接复制、粘贴。
>
> 核心配置文件主要用于配置连接数据库的环境以及MyBatis的全局配置信息 核心配置文件存放的位置是src/main/resources目录下

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
	PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
	"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<!--设置连接数据库的环境-->
	<environments default="development">
		<environment id="development">
			<transactionManager type="JDBC"/>
			<dataSource type="POOLED">
				<property name="driver" value="com.mysql.cj.jdbc.Driver"/>
				<property name="url" value="jdbc:mysql://localhost:3306/ssm?
serverTimezone=UTC"/>
				<property name="username" value="root"/>
				<property name="password" value="123456"/>
			</dataSource>
		</environment>
	</environments>
	<!--引入映射文件-->
	<mappers>
		<package name="mappers/UserMapper.xml"/>
	</mappers>
</configuration>
```

**(IDEA中的注解快捷键的ctrl+shift+/)**

### 创建Mapper接口

```java
public interface UserMapper {
    
   int inserUser();
    /*
    *
    */
}
```

### 创建MyBatis的映射文件

相关概念：**ORM**（Object Relationship Mapping）对象关系映射

* 对象：java的实体类
* 关系：关系型数据库
* 映射：两者之间的对应关系

> 1、映射文件的命名规则： 表所对应的实体类的类名+Mapper.xml 
>
> 例如：表t_user，映射的实体类为User，所对应的映射文件为UserMapper.xml
>
> 因此一个映射文件对应一个实体类，对应一张表的操作 
>
> MyBatis映射文件用于编写SQL，访问以及操作表中的数据 
>
> MyBatis映射文件存放的位置是src/main/resources/mappers目录下 
>
> 2、 MyBatis中可以面向接口操作数据，要保证两个一致： 
>
> a>mapper接口的全类名和映射文件的命名空间（namespace）保持一致 
>
> b>mapper接口中方法的方法名和映射文件中编写SQL的标签的id属性保持一致

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--约束文档-->
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--配置映射文件的详细信息，namespace属性是配置这个映射里面操作的针对具体Dao的各种方法-->
<mapper namespace="com.dao.CharContentDao">
<!--    id属性是指定Dao中的哪一个具体的方法-->
<!--    resultType属性是指定返回值类型,即查询的数据要转换的java类型-->
<!--    resultMap属性是自定义映射，处理多对一或一对多的映射关系-->
<!--    parameterType属性指定的参数类型-->
    <insert id="addNewcontent" parameterType="com.pojo.CharContent">
        insert into charcontent (content,user,flag) values(#{content},#{username},1)
    </insert>
    <delete id="deleteContent" parameterType="com.pojo.CharContent">
        delete from charcontent where id = #{id}
    </delete>
    <update id="logicDelete" parameterType="com.pojo.CharContent">
        update charcontent set flag = 0 where id = #{id}
    </update>
    <update id="recoverContent" parameterType="com.pojo.CharContent">
        update  charcontent set flag = 1 where  id =#{id}
    </update>
    <select id="findAllLogicDelete" resultType="com.pojo.CharContent">
        select * from charcontent where flag = 0
    </select>
</mapper>
```

###  功能实现

```java
//读取MyBatis的核心配置文件
InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
//创建SqlSessionFactoryBuilder对象
SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new
SqlSessionFactoryBuilder();
//通过核心配置文件所对应的字节输入流创建工厂类SqlSessionFactory，生产SqlSession对象
SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(is);

//创建SqlSession对象，此时通过SqlSession对象所操作的sql都必须手动提交或回滚事务
//SqlSession sqlSession = sqlSessionFactory.openSession();

//创建SqlSession对象，此时通过SqlSession对象所操作的sql都会自动提交
SqlSession sqlSession = sqlSessionFactory.openSession(true);

//通过代理模式创建UserMapper接口的代理实现类对象
UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
//调用UserMapper接口中的方法，就可以根据UserMapper的全类名匹配元素文件，通过调用的方法名匹配
//映射文件中的SQL标签，并执行标签中的SQL语句
int result = userMapper.insertUser();
//sqlSession.commit();
System.out.println("结果："+result);
```

### **封装优化**

方法的调用是先通过接口的全类名来找到映射文件，再通过接口中的方法找到sql语句

可以直接提供sql以及的唯一标识找到sql并执行，唯一标识是namespace.sqlId

```java
int result = sqlSession.insert("com.demo.mybatis.mapper.UserMapper.insertUser");
```

设置自动提交事务

```java
//创建SqlSession对象，此时通过SqlSession对象所操作的sql都会自动提交
SqlSession sqlSession = sqlSessionFactory.openSession(true);
```



## 3、加入log4j日志功能

引入依赖

```xml
<!-- log4j日志 -->
<dependency>
	<groupId>log4j</groupId>
	<artifactId>log4j</artifactId>
	<version>1.2.17</version>
</dependency>
```

加入配置文件

> log4j的配置文件名为log4j.xml，存放的位置是src/main/resources目录下

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



## 4、MyBatis核心配置文件详解

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!--
        MyBatis核心配置文件中的标签必须要按照指定的顺序配置
        properties?,settings?,typeAliases?,typeHandlers?,objectFactory?,
        objectWrapperFactory?,reflectorFactory?,plugins?,environments?,
        databaseIdProvider?,mappers?
    -->

    <!--引入properties文件，此后就可以在当前文件中使用${key}的方式来获取value-->
    <properties resource="jdbc.properties"></properties>

    <!--typeAliases：设置类型的别名-->
    <typeAliases>
        <!--type设置的类型，alias是设置别名，
        如果不设置alias，就会有一个默认的别名,即类名，且不区分大小写-->
        <typeAlias type="com.demo.mybatis.pojo.User" alias="abc"></typeAlias>
        <!--通过包设置类型别名，指定的包下的类都会有默认的类型别名-->
        <package name="com.demo.mybatis.pojo"/>
    </typeAliases>

    <!--设置连接数据库的环境-->
    <!--default是默认使用的环境id-->
    <environments default="development">
        <!--id是唯一标识不能重复-->
        <environment id="development">
            <!--
                transactionManager：设置事务管理器
                属性：
                type：设置事务管理的方式
                type=“JDBC|MANAGED”
                JDBC：表示使用JDBC中原生的事务管理器
                MANAGED：被管理，例如Spring
            -->
            <transactionManager type="JDBC"/>
            <!--
                dataSource：设置数据源
                属性：
                type：设置数据源的类型
                type=“POOLED|UNPOOLED|JNDI”
                POOLED：表示使用数据库连接池，作用是管理连接，当第二次创立连接时，可以直接到数据库连接池中获取
                UNPOOLED:表示不使用数据库连接池，每次获取连接都是要获取和创建
                JNDI：表示使用上下文中的数据
            -->
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <!--引入映射文件-->
    <mappers>
        <!--<package name="mappers/UserMapper.xml"/>-->
        <!--
            以包的方式引入映射文件,但是必须满足两个条件
            1.mapper接口和映射文件所在的包必须一致
            2.mapper接口的名字和映射文件的名字必须一致
        -->
        <package name="com.demo.mybatis.mapper"/>
    </mappers>
</configuration>
```

选择resource Bundle，创建jdbc. properies文件

```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc.mysql://localhost:3306/ssm?userTimezone=UTC
jdbc.username=root
jdbc.password=123456
```

在resource中创建映射文件的包,要和对应接口在java的包路径一致。

编译以后会在同一个目录

创建时要用/而不是.

例如：com/demo/mybatis/mapper

> 右键目录选择Make Directory as可以将目录设置成根目录、资源目录等

### 在IDEA中创建MyBatis核心配置文件

在setting中找到Editor下的File and Code Templates

![](images\屏幕截图 2023-08-11 033721.png)

## 5、MyBatis获取参数值的两种方式

> MyBatis获取参数值的两种方式：${}和#{}
>
> ${}的本质就是字符串拼接，#{}的本质就是占位符赋值
>
> ${}使用字符串拼接的方式拼接sql，若为字符串类型或日期类型的字段进行赋值
>
> 时，需要手动加单引号；
>
> 但是#{}使用占位符赋值的方式拼接sql，此时为字符
>
> 串类型或日期类型的字段进行赋值时， 可以自动添加单引号

### 单个字面量类型的参数

#{name}和${name}占位符里面的name内容可以是任意的

但是${}中不能是数值

### 多个字面量类型的参数

MyBatis会自动的将传入参数放到一个Map集合中，有两种方式

一是arg0...为键

二是param1...为键

```xml
<select id="checkLogin" resultType="User">
	select * from t_user where username = #{arg0} and password = #{arg1}
</select>

<select id="checkLogin" resultType="User">
	select * from t_user where username = #{param1} and password = #{param2}
</select>

```

### map集合

可以自己创建一个Map类型传入，然后通过key获取值



### 实体类类型的参数

通过属性名即可获取参数



### 使用@Param标识参数

```java
User checkLoginByParam(@Param("username")String username,@Param("password")Stirng password);
```



@Param中的只为value属性赋值可以省略value

当设置后就会以value值为键放到Map集合中，就会代替arg0...

但是param1还在

## 6、MyBatis的各种查询功能

### 查询单个数据

```java
/**
* 查询用户的总记录数
* @return
* 在MyBatis中，对于Java中常用的类型都设置了类型别名
* 例如： java.lang.Integer-->int|integer
* 例如： int-->_int|_integer
* 例如： Map-->map,List-->list
*/
int getCount();
```

```xml
<!--int getCount();-->
<select id="getCount" resultType="_integer">
select count(id) from t_user
</select>
```

### 查询出一条Map集合

```java
/**
* 根据用户id查询用户信息为map集合
* @param id
* @return
*/
Map<String, Object> getUserToMap(@Param("id") int id);
```

 ```xml
 <!--Map<String, Object> getUserToMap(@Param("id") int id);-->
 <!--结果： {password=123456, sex=男 , id=1, age=23, username=admin}-->
 <select id="getUserToMap" resultType="map">
 select * from t_user where id = #{id}
 </select>
 ```

**若在数据库中对应的字段为null，是不会放到Map集合中的**

### 查询多条数据为map集合

①方法一

```java
/**
* 查询所有用户信息为map集合
* @return
* 将表中的数据以map集合的方式查询，一条数据对应一个map；若有多条数据，就会产生多个map集合，此
时可以将这些map放在一个list集合中获取
*/
List<Map<String, Object>> getAllUserToMap();

```

```xml
<!--Map<String, Object> getAllUserToMap();-->
<select id="getAllUserToMap" resultType="map">
select * from t_user
</select>
```

②方法二

```java
/**
* 查询所有用户信息为map集合
* @return
* 将表中的数据以map集合的方式查询，一条数据对应一个map；若有多条数据，就会产生多个map集合，并
且最终要以一个map的方式返回数据，此时需要通过@MapKey注解设置map集合的键，相当于创建了一个大的Map集合，值是每条数据所对应的，设置的键是查询出来的字段id为键
map集合
*/
@MapKey("id")
Map<String, Object> getAllUserToMap();
```

```xml
<!--Map<String, Object> getAllUserToMap();-->
<!--
{
1={password=123456, sex=男, id=1, age=23, username=admin},
2={password=123456, sex=男, id=2, age=23, username=张三},
3={password=123456, sex=男, id=3, age=23, username=张三}
}
-->
<select id="getAllUserToMap" resultType="map">
select * from t_user
</select>
```

## 7、特殊SQL的执行

### 7.1 模糊查询

```java
/**
* 测试模糊查询
* @param mohu
* @return
*/
List<User> testMohu(@Param("mohu") String mohu);
```

```xml
<!--List<User> testMohu(@Param("mohu") String mohu);-->
<select id="testMohu" resultType="User">
<!-- 三种方式 -->
<!--select * from t_user where username like '%${mohu}%'-->
<!--select * from t_user where username like concat('%',#{mohu},'%')-->
select * from t_user where username like "%"#{mohu}"%"
</select>
```

### 7.2批量删除

> 实现SQL：DELETE FROM t_user WHERE id IN (1,2,3)

```java
//传入“1,2,3”字符串
void deleteMoreUser(@Param("ids")String ids);
```



```xml
<delete id="deleteMoreUser">
    <!-- 错误示范：delete from t_user where id in(#{ids}) 
		#{}会自动加上单引号，变成('1,2,3')
	-->
    delete from t_user where id in(${ids})
</delete>
```

### 7.3动态设置表名

```java
List<User> getUserList(@Param("tablename") String tablename);
```



```xml
<select id="getUserList">
    <!-- 错误示范：select * from #{tablename} 
		#{}会自动加上单引号
	-->
    select * from ${tablename} 
</select>
```



### 7.4添加功能获取自增主键

> 场景模拟： 
>
> t_clazz(clazz_id,clazz_name) 
>
> t_student(student_id,student_name,clazz_id) 
>
> 1、添加班级信息 
>
> 2、获取新添加的班级的id 
>
> 3、为班级分配学生，即将某学的班级id修改为新添加的班级的id
>
> 这时要获取更加**准确**地班级的自增id，就要用到这个功能。
>
> 这个功能在JDBC中有，mybatis对其进行了封装

JDBC中的实现：

```java
Class.forName("");
Connection connection = DriverManager.getConnection("","","");
String sql = "insert into t_user values()";
//设置是否可以获取自动自增的主键
PreparadStatement ps = connection.preparaeStatement(sql,1);
ps.executeUpdate();
ResultSet resultSet = ps.getGeneratedKeys();
resultSet.next();
int id = resultSet.getInt(1);
```

添加用户信息并获取自动自增的主键：

```java
void insertUser(User user);
```

首先设置useGeneratedKeys属性为true，可以获取到自动自增主键。

keyProperty属性的含义是将获取到的主键值传入到指定的实体类中的属性上。

```xml
<insert id="insetUser" useGeneratedKeys="true" keyProperty="id">
    insert into t_user values(null,#{username},#{password},#{age},#{gender},#{email})
</insert>
```



## 8、自定义映射resultMap

### 8.1 resultMap处理字段和属性的映射关系

> 若字段名和实体类中的属性名不一致，可以用resultMap来处理

获取字段名和实体类中属性名不一样的属性：

第一种方式：设置别名,和属性名保持一致

```xml
<select id="getEmpByEmpId" resultType="Emp">
    select emp_id empId,emp_name empName,age,gender from
    emp_id = #{empId}
</select>
```

第二种方式：当字段符合MySQL的要求使用_,而属性符合java的要求使用驼峰

此时可以再MyBatis的核心配置文件中设置一个全局配置，可以自动将下划线映射为驼峰

在核心配置文件中：

```xml
<settings>
    <setting name="mapUnderscoreToCamelCase" value="true">
</settings>
```

第三种方式：使用resultMap来处理

```xml
<resultMap id="empResultMap" type="Emp">
    <!-- 
		id处理主键
		collection处理一对多映射关系
		association处理多对一映射关系
		result处理普通关系
	-->
    <id column="emp_id" property="empId"></id>
    <result column="emp_name" property="empName"></result>
</resultMap>
<select id="getEmpByEmpId" resultMap="empResultMap">
    select emp_id empId,emp_name empName,age,gender from
    emp_id = #{empId}
</select>
```

### 8.2多对一映射处理

> 在员工和部门的关系表中，在实体类中如何体现员工部门的多对一关系？
>
> 在员工类中加入一个属性即部门类。



获取员工以及对应的部门信息：（两表联查）

```xml
<select id="getEmpAndDeptByEmpId" resultType="Emp">
    select 
    * 
    from t_emp 
    left join t_dept 
    on t_emp.dept_id = t_dept.dept_id 
    where t_emp.emp_id = #{empId}
</select>
```

第一种方式：级联方式

```xml
<resultMap id="empAndDeptResultMap" type="Emp">
    <id column="emp_id" property="empId"></id>
    <result column="emp_name" property="empName"></result>
    <result column="age" property="age"></result>
    <result column="gender" property="gender"></result>
    <result column="dept_id" property="dept.deptId"></result>
    <result column="dept_name" property="dept.deptName"></result>
</resultMap>
<select id="getEmpAndDeptByEmpId" resultMap="empAndDeptResultMap">
    select 
    * 
    from t_emp 
    left join t_dept 
    on t_emp.dept_id = t_dept.dept_id 
    where t_emp.emp_id = #{empId}
</select>
```

第二种方式：association标签，处理实体类类型的属性(可以看出层级关系)

```xml
<resultMap id="empAndDeptResultMap" type="Emp">
    <id column="emp_id" property="empId"></id>
    <result column="emp_name" property="empName"></result>
    <result column="age" property="age"></result>
    <result column="gender" property="gender"></result>
    
    <!-- 
		property:设置需要处理映射关系的属性的属性名
		javaType：设置处理的属性的类型
	-->
    <association property="dept" javaType="Dept">
        <id column="dept_id" property="deptId"></id>
        <result column="dept_name" property="deptName"></result>
    </association>
        
</resultMap>
<select id="getEmpAndDeptByEmpId" resultMap="empAndDeptResultMap">
    select 
    * 
    from t_emp 
    left join t_dept 
    on t_emp.dept_id = t_dept.dept_id 
    where t_emp.emp_id = #{empId}
</select>
```

第三种方式：分步查询

Emp类的映射：

```java
//第一步
Emp getEmpAndDeptByStepOne(@Param("empId") Integer empId);
```

```xml
<resultMap id="empAndDeptResultMap" type="Emp">
    <id column="emp_id" property="empId"></id>
    <result column="emp_name" property="empName"></result>
    <result column="age" property="age"></result>
    <result column="gender" property="gender"></result>
    
    <!-- 
		property:设置需要处理映射关系的属性的属性名
		select:设置要执行下一个语句的唯一标识：全类名+方法名
		column:将查询出来的字段作为下一步查询语句的条件
	-->
    <association property="dept" 
                 select="com.demo.mybatis.mapper.DeptMapper.getEmpAndDeptByStepTwo" 
                 column="dept_id"></association>
    
</resultMap>
<select id="getEmpAndDeptByStepOne" resultMap="empAndDeptResultMap">
    select 
    * 
    from t_emp 
    where emp_id = #{empId}
</select>
```

Dept类的映射：

```java
//第二步
Dept getEmpAndDeptByStepTwo(@Param("deptId") Integer deptId);
```

```xml
<select id="getEmpAndDeptByStepTwo" resultType="Dept">
    select 
    * 
    from t_dept 
    where dept_id = #{deptId}
</select>
```

**分步查询的优点**：

可以实现延迟加载，优化内存

#### 延迟加载

> 可以实现延迟加载，优化内存
>
> 但是必须在核心配置文件中设置全局配置信息： 
>
> lazyLoadingEnabled：延迟加载的全局开关。当开启时，所有关联对象都会延迟加载 
>
> aggressiveLazyLoading：当开启时，任何方法的调用都会加载该对象的所有属性。否则，每个属 性会按需加载 此时就可以实现按需加载，获取的数据是什么，就只会执行相应的sql。
>
> 此时可通过association和 collection中的fetchType属性设置当前的分步查询是否使用延迟加载， fetchType="lazy(延迟加载)|eager(立即加载)"



### 8.3 一对多映射处理

>处理Dept实体类中，List<Emp>集合类型的属性

查询部门已经部门中所有的员工信息：

```java
Dept getDeptAndEmpByDeptId(@Param("deptId") Integer deptId);
```

第一种方式：collection属性,处理集合类型的属性

```xml
<resultMap id="deptAndEmpResultMap" Type="Dept">
    <id column="dept_id" property="deptId"></id>
    <result column="dept_name" property="deptName"></result>
    
    <!-- 
		ofType:设置集合类型的属性中存储的数据的类型
	-->
    <collection property = "emps" ofType="Emp">
        <id column="emp_id" property="empName"></id>
        <result column="emp_name" property="empName"></result>
    	<result column="age" property="age"></result>
    	<result column="gender" property="gender"></result>
    </collection>
       
</resultMap> 

<select id="getDeptAndEmpByDeptId" resultMap="deptAndEmpResultMap">
    select * 
    from t_dept 
    left join t_emp 
    on t_dept.dept_id = t_emp.dept_id
    where t_dept.dept_id = #{deptId}
</select>
```

第二种方式：分步查询

第一步：

```java
Dept getDeptAndEmpByStepOne(@Param("deptId") Integer deptId);
```

```xml
<resultMap id="deptAndEmpResultMap" Type="Dept">
    <id column="dept_id" property="deptId"></id>
    <result column="dept_name" property="deptName"></result>
    <collection property="emps"
                select="com.demo.mybatis.mapper.Emp.Mapper.getDpetAndEmpByStepTwo"
                column="dept_id"
                ></collection>
</resultMap>

<select id="getDeptAndEmpByStepOne" resultMap="deptAndEmpResultMap">
    select * from t_dept where dept_id = #{deptId}
</select>
```

第二步：

```java
List<Emp> getDpetAndEmpByStepTwo(@Param("deptId") Integer deptId);
```

```xml
<select id="getDpetAndEmpByStepTwo" resultType="Emp">
    select * from t_emp where dpet_id = #{deptId}
</select>
```



## 9、动态SQL

> MyBatis框架的动态SQL技术,解决拼接SQL语句字符串时的痛点问题
>
> 能够很好地控制SQL语句条件的个数

### 9.1 if、where、trim标签

根据条件查询员工信息：

```java
List<Emp> getEmpByCondition(Emp emp);
```

由于当所有条件不成立或者第一个条件不成立时，sql语句会出错即多一个where或者多一个and

第一种方法：要在where加上一个 **1=1**恒成立条件

```xml
<!-- 
	test:设置判断语句
-->
<select id="getEmpByCondition" resultType="Emp">
	select * from t_emp where 1=1
    <if test="empName != null and empName != ''">
    	and emp_name = #{empName}
    </if>
    <if test="age != null and age != ''">
        and age = #{age}
    </if>
    <if test="gender != null and gender != '' ">
        and gender = #{gender}
    </if>
</select>
```

第二种方法：使用where标签

a.若where标签中有条件成立，会自动生成where关键字

b.会自动将where标签中的内容前多余的and删除，但是无法去掉后面多余的and

c.若where标签中没有任何一个条件成立，则where标签就没有任何功能

```xml
<!-- 
	test:设置判断语句
-->
<select id="getEmpByCondition" resultType="Emp">
	select * from t_emp  
    <where>
    	<if test="empName != null and empName != ''">
    		emp_name = #{empName}
    	</if>
    	<if test="age != null and age != ''">
        	and age = #{age}
    	</if>
    	<if test="gender != null and gender != '' ">
        	and gender = #{gender}
    	</if>
    </where>
</select>
```

第三种方法：使用trim标签

有四个属性：

prefix、suffix：在标签中内容前面或后面添加指定内容

prefixOverrides、suffixOverrrides：在标签中内容前面或后面去掉指定内容

```xml
<select id="getEmpByCondition" resultType="Emp">
	select * from t_emp where 1=1
    <trim prefix="where" suffixOverrides="and">
        <if test="empName != null and empName != ''">
    		emp_name = #{empName} and
    	</if>
    	<if test="age != null and age != ''">
        	age = #{age} and
    	</if>
    	<if test="gender != null and gender != '' ">
        	gender = #{gender}
    	</if>
    </trim>
</select>
```

### 9.2 choose、when、otherwise标签

> 相当于java中的if。。。else if 。。。else
>
> when：相当于else if，
>
> otherw：相当于else

```xml
<select id="getEmpByChoose" resultType="Emp">
    select * from t_emp
    <where>
        <choose>
            <when test="empName != null and empName != ''">
                emp_name= #{empName}
            </when>
            <when test="age != null and age != ''">
                age= #{age}
            </when>
            <when test="gender != null and gender != ''">
                gender = #{gender}
            </when>
        </choose>
    </where>
</select>
```

### 9.3  foreach标签

> 可以实现批量添加和批量删除

批量添加：

```java
void insertMoreEmp(@Param("emps") List<Emp> emps);
//如果不设置@Param，MyBatis会将emps放入Map集合中以list为键
```

```xml
<insert id="insertMoreEmp">
    insert into t_emp values
    <foreach collection="emps" item="emp" separator=",">
    	(null,#{emp.empName},#{emp.age},#{emp.gender},null)
    </foreach>
</insert>
```

foreach标签：

collection：设置要循环的集合

item：设置集合中每一项的名字

separator：设置分隔符

批量删除：

```java
void deleteMoreEmp(@Param("empIds")Integer[] empIds);
//如果不设置@Param，会将empIds放入Map集合中以array为键
```

```xml
<delete id="deleteMoreEmp">
     delete from t_emp where emp_id in 
     <foreach collection="empIds" item="empId" separator="," open="(" close=")">
         #{empId}
     </foreach>
</delete>

<!-- 第二种方式-->
<delete id="deleteMoreEmp">
     delete from t_emp where 
     <foreach collection="empIds" item="empId" separator="or">
         emp_id = #{empId}
     </foreach>
</delete>
```

open:以指定的字符开始

close:以指定的字符结束

### 9.4 sql标签

> 设置sql片段，可以用<include></include>引用

```xml
<sql id="empColumn">
    emp_id,emp_name,age,gender,dept_id
</sql>

<select id="getEmpByCondition" resultType="Emp">
    select <include refid="empColumn"></include> from t_emp where 1=1
    <if test="empName != null and empName != ''">
    	and emp_name = #{empName}
    </if>
    <if test="age != null and age != ''">
        and age = #{age}
    </if>
    <if test="gender != null and gender != '' ">
        and gender = #{gender}
    </if>
</select>
```

## 10、MyBatis的缓存

### 10.1 MyBatis的一级缓存

> 是默认开启的，一级缓存是sqlsession级别的，通过同一个SqlSession查询的数据会被缓存，下次查询相同的数据，就 会从缓存中直接获取，不会从数据库重新访问

**使一级缓存失效的四种情况**： 

1) 不同的SqlSession对应不同的一级缓存 

2) 同一个SqlSession但是查询条件不同 

3) 同一个SqlSession两次查询期间执行了任何一次增删改操作

4) 同一个SqlSession两次查询期间手动清空了缓存

   ```java
   sqlSeesion.clearCache();
   ```

### 10.2 MyBatis的二级级缓存

>二级缓存是SqlSessionFactory级别，通过同一个SqlSessionFactory创建的SqlSession查询的结果会被缓存；此后若再次执行相同的查询语句，结果就会从缓存中获取

**二级缓存开启的条件**： 

a>在核心配置文件中，设置全局配置属性cacheEnabled="true"，默认为true，不需要设置 

b>在映射文件中设置标签<cache/>

**c>二级缓存必须在SqlSession关闭或提交之后有效 **

**d>查询的数据所转换的实体类类型必须实现序列化的接口 **

```java
public class Emp implements Serializable{
    ....
}
```

**使二级缓存失效的情况**： 

两次查询之间执行了任意的增删改，会使一级和二级缓存同时失效

### 10.3 二级缓存的相关配置

在mapper配置文件中添加的cache标签可以设置一些属性： 

**①eviction**属性：缓存回收策略，默认的是 LRU。 

LRU（Least Recently Used） – 最近最少使用的：移除最长时间不被使用的对象。 

FIFO（First in First out） – 先进先出：按对象进入缓存的顺序来移除它们。 

SOFT – 软引用：移除基于垃圾回收器状态和软引用规则的对象。

WEAK – 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象。 

**②flushInterval**属性：刷新间隔，单位毫秒 默认情况是不设置，也就是没有刷新间隔，缓存仅仅调用语句时刷新 

**③size**属性：引用数目，正整数 代表缓存最多可以存储多少个对象，太大容易导致内存溢出 

**④readOnly**属性：只读， true/false 

true：只读缓存；会给所有调用者返回缓存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势。 

false：读写缓存；会返回缓存对象的拷贝（通过序列化）。这会慢一些，但是安全，因此默认是 false。

### 10.4MyBatis缓存查询的顺序

先查询二级缓存，因为二级缓存中可能会有其他程序已经查出来的数据，可以拿来直接使用。 

如果二级缓存没有命中，再查询一级缓存 

如果一级缓存也没有命中，则查询数据库 

SqlSession关闭之后，一级缓存中的数据会写入二级缓存

### 10.5整合第三方缓存EHCache

略。。。



## 11、MyBatis的逆向工程

> 正向工程：先创建java实体类，由框架负责根据实体类生成数据库表。Hibernate是支持正向工程的
>
> 逆向工程：先创建数据库表，由框架负责根据数据库表，反向生成以下资源：
>
> * java实体类
> * Mapper接口
> * Mapper映射文件

### 11.1实现步骤：

**①添加依赖和插件**

```xml
<!-- 依赖MyBatis核心包 -->
<dependencies>
	<dependency>
		<groupId>org.mybatis</groupId>
		<artifactId>mybatis</artifactId>
		<version>3.5.7</version>
	</dependency>
	<!-- junit测试 -->
	<dependency>
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
		<version>4.12</version>
		<scope>test</scope>
	</dependency>
    <!-- mysql驱动-->
    <dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>8.0.16</version>
	</dependency>
    
	<!-- log4j日志 -->
	<dependency>
		<groupId>log4j</groupId>
		<artifactId>log4j</artifactId>
		<version>1.2.17</version>
	</dependency>
	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>8.0.16</version>
	</dependency>
</dependencies>

<!-- 控制Maven在构建过程中相关配置 -->
<build>
    
	<!-- 构建过程中用到的插件 -->
	<plugins>
		<!-- 具体插件，逆向工程的操作是以构建过程中插件形式出现的 -->
		<plugin>
			<groupId>org.mybatis.generator</groupId>
			<artifactId>mybatis-generator-maven-plugin</artifactId>
			<version>1.3.0</version>
            
		<!-- 插件的依赖 -->
		<dependencies>
			<!-- 逆向工程的核心依赖 -->
			<dependency>
				<groupId>org.mybatis.generator</groupId>
				<artifactId>mybatis-generator-core</artifactId>
				<version>1.3.2</version>
			</dependency>
            
			<!-- MySQL驱动 -->
			<dependency>
				<groupId>mysql</groupId>
				<artifactId>mysql-connector-java</artifactId>
				<version>8.0.16</version>
			</dependency>
		</dependencies>
		</plugin>
	</plugins>
</build>
```

**②创建MyBatis核心配置文件**

**③建逆向工程的配置文件**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
	PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
	"http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
	<!--
		targetRuntime: 执行生成的逆向工程的版本
			MyBatis3Simple: 生成基本的CRUD（清新简洁版）
			MyBatis3: 生成带条件的CRUD（奢华尊享版）
	-->
	<context id="DB2Tables" targetRuntime="MyBatis3">
		<!-- 数据库的连接信息 -->
		<jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                    	connectionURL="jdbc:mysql://localhost:3306/mybatis?serverTimezone=UTC"
						userId="root"
						password="123456">
		</jdbcConnection>
        
		<!-- javaBean的生成策略-->
		<javaModelGenerator targetPackage="com.demo.mybatis.pojo"
targetProject=".\src\main\java">
			<property name="enableSubPackages" value="true" />
			<property name="trimStrings" value="true" />
		</javaModelGenerator>
        
		<!-- SQL映射文件的生成策略 -->
		<sqlMapGenerator targetPackage="com.demo.mybatis.mapper"
targetProject=".\src\main\resources">
			<property name="enableSubPackages" value="true" />
		</sqlMapGenerator>
        
		<!-- Mapper接口的生成策略 -->
		<javaClientGenerator type="XMLMAPPER"
targetPackage="com.atguigu.mybatis.mapper" targetProject=".\src\main\java">
		<property name="enableSubPackages" value="true" />
		</javaClientGenerator>
		<!-- 逆向分析的表 -->
		<!-- tableName设置为*号，可以对应所有表，此时不写domainObjectName -->
		<!-- domainObjectName属性指定生成出来的实体类的类名 -->
		<table tableName="t_emp" domainObjectName="Emp"/>
		<table tableName="t_dept" domainObjectName="Dept"/>
	</context>
</generatorConfiguration>

```

**④用插件执行**

Plugins/mybatis-generator/mybatis-generator:generate

### 11.2 QBC查询

```java
@Test
public void testMBG(){
	try {
		InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
		SqlSessionFactory sqlSessionFactory = new
		SqlSessionFactoryBuilder().build(is);
		SqlSession sqlSession = sqlSessionFactory.openSession(true);
		EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);
		//查询所有数据
		/*List<Emp> list = mapper.selectByExample(null);
		list.forEach(emp -> System.out.println(emp));*/
        
		//根据条件查询
		/*EmpExample example = new EmpExample();
		example.createCriteria().andEmpNameEqualTo("张
		三").andAgeGreaterThanOrEqualTo(20);
		example.or().andDidIsNotNull();//上面的条件和本行的条件会以or连接
		List<Emp> list = mapper.selectByExample(example);
		list.forEach(emp -> System.out.println(emp));*/
        
		mapper.updateByPrimaryKeySelective(new
		Emp(1,"admin",22,null,"456@qq.com",3));
	} catch (IOException e) {
		e.printStackTrace();
	}
}
```

## 12、分页插件

> 在sql语句中实现分页要知道index、pageSize等数据
>
> index：当前页的起始索引,index = (pageNum - 1) * pageSize
>
> pageSize：每页显示的条数
>
> pageNum：当前页的页码
>
> count:总记录数
>
> totalPage：总页数，
>
> totalPage = count / pageSize
>
> if(count % pageSize != 0) {
>
> ​	totalPage += 1;
>
> }
>
> pageSize=4，pageNum=1，index=0 -----limit 0,4
>
> pageSize=4，pageNum=3，index=8 -----limit 8,4
>
> pageSize=4，pageNum=6，index=20 -----limit 20,4

### 12.1分页插件的使用步骤

**①添加依赖**

```xml
<dependency>
	<groupId>com.github.pagehelper</groupId>
	<artifactId>pagehelper</artifactId>
	<version>5.2.0</version>
</dependency>
```

**②配置分页插件**

在MyBatis的核心配置问价那种配置插件

```xml
<plugins>
	<!--设置分页插件-->
	<plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
</plugins>

```

> interceptor:拦截器

### 12.2分页插件的使用

```java
//查询功能之前开启分页功能
Page<Object> page = PageHelper.startPage(1,4);
//page对象中会存储大部分分页相关的数据
//可以将方法的返回值设置为Page<Object>
List<Emp> list = mapper.selectByExample(null);
//查询功能之后可以获取分页相关的所有数据
PageInfo<Emp> pageInfo = new PageInfo(list,5);
//5代表的导航分页的页码数
/*
PageInfo{
pageNum=8, pageSize=4, size=2, startRow=29, endRow=30, total=30, pages=8,
list=Page{count=true, pageNum=8, pageSize=4, startRow=28, endRow=32, total=30,
pages=8, reasonable=false, pageSizeZero=false},
prePage=7, nextPage=0, isFirstPage=false, isLastPage=true, hasPreviousPage=true,
hasNextPage=false, navigatePages=5, navigateFirstPage=4, navigateLastPage=8,
navigatepageNums=[4, 5, 6, 7, 8]
}
pageNum：当前页的页码
pageSize：每页显示的条数
size：当前页显示的真实条数
total：总记录数
pages：总页数
prePage：上一页的页码
nextPage：下一页的页码
isFirstPage/isLastPage：是否为第一页/最后一页
hasPreviousPage/hasNextPage：是否存在上一页/下一页
navigatePages：导航分页的页码数
navigatepageNums：导航分页的页码，[1,2,3,4,5]
*/
list.foreach(System.out::println);
```

