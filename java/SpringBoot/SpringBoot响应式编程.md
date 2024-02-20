# SpringBoot响应式编程

![](image\屏幕截图 2024-01-28 225702.png)

传统的编程方式，对于cpu来说是**同步阻塞的**，也就是当有很多请求过来到系统中时，系统会将请求分配给cpu来处理。

而响应式编程是当请求过来到系统时，会将大量请求放到缓存区中，让cpu开始工作，只要cpu处于空闲状态，缓存区就会分配任务给cpu执行。

## 一、前置知识

### 1.1 Lambda

> Lambda表达式是java 8引入的一个重要特性
>
> Lambda表达式可以被视为**匿名函数**
>
> 允许在**需要函数**的地方可以更简洁的方式定义

**语法：**

```java
(parameters) -> expression
(parameters) -> {statements;}
```

**案例：**

```java
interface MyInterface{
    int sum(int i, int j);
}

public class Lamda {
    public static void main(String[] args) {
        // 完整写法
        MyInterface myInterface = (int i, int j) -> {
            return i*i + j*j;
        }
        // 简化写法，参数类型可以不写，
        /* 
        变量名可以自定义,
        只有一个参数可以省略括号
        方法只有一句话可以省略中括号
        */
        MyInterface myInterface = (i, j) -> {
            return i*i + j*j;
        }
    }
}
```

这种实现接口的方式叫做**函数式接口**。

**接口中只有一个未实现的方法，可以为其它的方法这只默认实现，就可以实现函数式接口。**

可以在接口上加上`@FunctionalInterface`，检查注解，会自动判断是否符合函数式接口。



实际使用例子：

```java
public static void main(String[] args){
    // var是高版本jdk中的语法，变量声明的细节写在后面
    var names = new ArrayList<String>();
    names.add("Alice");
    names.add("Bily");
    
    Collections.sort(names, new Comparator<String>() {
       @Override
        public int compare(String o1, String o2){
            return o2.compareTo(o1);
        }
    });
    
    Collections.sort(names, (o1, o2) ->o2.compartTO(o2));
    //[类]::[方法]，表示引用类中的实例方法
    Collections.sort(names, String::compareTo);
    
    
    //创建线程例子
    new Thread(
        new Runnable(){
            @Override
            public void run(){
                System.out.println("hhh");
            }
        }
    ).start();
    
    new Tread(()-> System.out.println("hhh")).start();
}
```

### 1.2 Function函数式接口

> Function是JDK内置的函数式接口的顶层定义。在`java.util.function`

函数式接口的出入参定义：

1. 有入参，无出参 **[消费者 consumer]** **function.accept()**

   ```java
   public interface BiConsumer<T,U>{
       void accept(T t, U,u);
   }
   -------------------------------
   BiConsumer<String, String> function = (a,b)->{
       System.out.println("a=" + a + ";b=" + b);
   }
   function.accept("1", "2");
   ```

2. 有入参，有出参  **[多功能函数 function]** **function.apply()**

   ```java
   public interface Function<T,R>{
       R apply(T t);
   }
   ------------------
   Function<String,Integer> function = (String x) -> Integer.parseInt(x);
   System.out.println(function.apply("2"));
   ```

3. 无入参，无出参

   ```java
   Runnable  runnable = () -> System.out.println("created");
   new Thread(runnable).start();
   ```

4. 无入参，有出参 **[提供者 supplier]** **get()**

   ```java
   public interface Supplier<T>{
       T get();
   }
   -------------------------
   Supplier<String> supplier = () -> UUID.randomUUID.toString();
   System.out.println(supplier.get());
   ```

**断言 Predicate：**

```java
Predicate<Integer> even = (t) -> t%2 == 0;
System.out.println(even.test(2));
even.test(2);//正向判断
even.negate().test(2);//反向判断
```

**案例：**

```java
public static void main(String[] args) {
    //1. 提供者：自定义数据提供者
    Supplier<String> supplier = ()->"42";
    //2. 断言：验证是否是一个数字
    Predicate<String> isNumber = str -> str.matches("-?\\d+(\\.\\d+)?");
    //3. 转换器：把字符串转换为数据；类::实例方法
    Function<String, Integer> change = Integer::parseInt;
    //4.消费者：打印数字
    Consumer<Integer> consumer = integer -> {
        if(integer%2 == 0){
            System.out.println("偶数:" + integer);
        } else {
            System.out.println("奇数:" + integer);
        }
    };
    //串在一起，实现判断字符串是奇数还是偶数
    if( isNumber.test(supplier.get()) ){
        consumer.accept(change.apply(supplier.get()));
    } else {
        System.out.println("非法的数字");
    }
}
```

这时我们可以把这些方法封装到一起，只要传入提供者、断言、函数、消费者



### 1.3 StreamAPI

> 最佳实战：以后凡是写for循环处理数据的统一全部用StreamAPI替换
>
> 声明式处理集合数据，包括：筛选、转化、组合等

![](image\屏幕截图 2024-01-30 155435.png)

> 重要概念

流的三大部分：

**Sream Pipelin：**流管道、流水线

**Intermediate Operations：**中间操作

**Terminal Operations：**终止操作

流管道的组成：一个数据源、零或者多个中间操作、一个终止操作。

**案例：**

```java
public static void main(String[] args) {
    //1.跳出最大偶数
    List<Integer> list = List.of(1,2,3,4,5,6,7,8,9,10,11);
    //(1) 首先把数据封装到刘，要到数据流
    //(2) 定义流式操作
    //(3) 获取最终结果
    list.stream()
        .filter(ele -> {
            System.out.println("正在filter：" + ele);
            ele%2 == 0;
        })
        .max(Integer::compareTo)
        .ifPresent(System.out::println);
}
--------------
filter是中间操作、max是终止操作
```

> **流的特性：**
>
> 1）流是lazy的，懒加载的，没有使用终止操作方法就不会调用

```java
//创建流
Stream<Integer> stream = Stream.of(1,2,3);
Stream<Integer> concat = Stream.concat(Stream.of(2,3,4),stream);
Stream<Object> build = Stream.builder().add("11").add("22"),build();
// 从集合中获取流
List<Integer> integers = List.of(1,2);
Stream<Integer> stream1 = integers.stream();

Set<Integer> integers1 = Set.of(1,2);
integers1.stream();

Map<Object,Object> of = Map.of();
of.keySet().stream();
of.values().stream();
```



> 流是并发还是不并发？和for有啥区别？

默认的是不并发的，但是可以设置`parallel()`中间操作，使其并发

```java
long count = Stream.of(1,2,3,4,5)
    .parallel() //intermediate 中间操作 并发流
    .filter(i -> {
        ....
    })
    .count();
```

有状态数据将产生并发安全问题，即流数据操作有关函数外的数据。

#### 1.3.1 常见的中间操作

* ##### filter

  过滤，根据规则筛选指定元素

* ##### map

  映射：一 一映射，a变成b，获得新流

  * mapToInt、mapToLong、mapToDouble

  ```java
  List<Person> list = List.of(
  new Person("王 五","男",16)
  ......);
  list.stream()
      .filter(person -> person.age > 18)
      .map(person -> person.getName()) //只拿到姓名
      .forEach(System.out::println);
  ```

* ##### flatMap

  一对多映射，散页

  继上面的例子

  ```java
  list.stream()
      .filter(person -> person.age > 18)
      .map(person -> person.getName()) //只拿到姓名
      .flatMap(ele -> {
          String[] s = ele.split(" ");
          return Arrays.stream(s);
      })
      .forEach(System.out::println);
  ```

* ##### distinct

  去重

* ##### sorted

  排序

  ```java
  list.stream()
      .filter(person -> person.age > 18)
      .map(person -> person.getName()) //只拿到姓名
      .flatMap(ele -> {
          String[] s = ele.split(" ");
          return Arrays.stream(s);
      })
      .distinct()
      .sorted(String::compareTo)
      .forEach(System.out::println);
  ```

* ##### peek

  中间操作来查看流元素

* ##### limit

  限制元素

* ##### takeWhile

  当满足条件就继续走下一个元素，当不满足条件直接结束流操作



#### 1.3.2 终止操作

* ##### collect

  收集操作，可以选择分组

  ```java
  Map<String,List<Person>> collect = list.stream()
      .filter(s -> s.age > 15)
      .collect(Collectors.groupingBy(t -> t.gender));
  ```

  
