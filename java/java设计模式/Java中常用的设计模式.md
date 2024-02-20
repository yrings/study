# Java中常用的设计模式

> **设计模式（Design pattern）是一套被反复使用、多数人知晓的、经过分类编目的、代码设计经验的总结。**使用设计模式是为了**可重用代码、让代码更容易被他人理解、保证代码可靠性**。

## 一、设计模式的六大原则

**1、开闭原则（Open Close Principle）**

开闭原则就是说对***扩展开放，对修改关闭\***。在程序需要进行拓展的时候，不能去修改原有的代码，实现一个热插拔的效果。所以一句话概括就是：为了使程序的扩展性好，易于维护和升级。想要达到这样的效果，我们需要使用接口和抽象类，后面的具体设计中我们会提到这点。

**2、里氏代换原则（Liskov Substitution Principle）**

里氏代换原则(Liskov Substitution Principle LSP)面向对象设计的基本原则之一。 里氏代换原则中说，任何基类可以出现的地方，子类一定可以出现。 LSP是继承复用的基石，只有当衍生类可以替换掉基类，软件单位的功能不受到影响时，基类才能真正被复用，而衍生类也能够在基类的基础上增加新的行为。里氏代换原则是对“开-闭”原则的补充。实现“开-闭”原则的关键步骤就是抽象化。而基类与子类的继承关系就是抽象化的具体实现，所以里氏代换原则是对实现抽象化的具体步骤的规范。—— From Baidu 百科

***\*3、依赖倒转原则（Dependence Inversion Principle）\****

这个是开闭原则的基础，具体内容：针对接口编程，依赖于抽象而不依赖于具体。

***\*4、接口隔离原则（Interface Segregation Principle）\****

这个原则的意思是：使用多个隔离的接口，比使用单个接口要好。还是一个降低类之间的耦合度的意思，从这儿我们看出，其实设计模式就是一个软件的设计思想，从大型软件架构出发，为了升级和维护方便。所以上文中多次出现：降低依赖，降低耦合。

***\*5、迪米特法则（最少知道原则）（Demeter Principle）\****

为什么叫最少知道原则，就是说：一个实体应当尽量少的与其他实体之间发生相互作用，使得系统功能模块相对独立。

***\*6、合成复用原则（Composite Reuse Principle）\****

原则是尽量使用合成/聚合的方式，而不是使用继承。



## 二、设计模式的三大类

总体来说设计模式分为三大类：

**创建型模式(5种)：工厂方法模式、抽象工厂模式、单例模式、建造者模式、原型模式。**

**结构型模式(7种)：适配器模式、装饰器模式、代理模式、外观模式、桥接模式、组合模式、享元模式。**

**行为型模式(11种)：策略模式、模板方法模式、观察者模式、迭代子模式、责任链模式、命令模式、备忘录模式、状态模式、访问者模式、中介者模式、解释器模式。**

并发型模式和线程池模式



## 观察者模式

>  定义了对象之间的一对多的依赖，这样一来，当一个对象改变时，它的所有的依赖者都会收到通知并自动更新。 

例如微信公众号的推送消息功能：

一个服务号可以看作是一个主题，用户就是观察者。

1. 服务号就是主题，业务就是推送消息
2. 观察者只需要订阅主题，只要有新的消息就会送来
3. 当不想要此主题消息时，取消订阅
4. 只要服务号还在，就会一直有人订阅

**自己实现一个观察者模式：**

1. 定义主题接口，定义观察者接口。
2. 实现这两个接口，然后根据业务需求编写代码

**利用JDK自带的接口：**

主题实现`Observable`接口，观察者实现`Observer`接口



## 工厂模式

* ### 静态工厂模式

  > 这个很简单，就是简单是类+静态方法，例如工具类

* ### 简单工厂模式

  > 定义：通过专门定义一个类来负责创建其他类的实例，被创建的实例通常都具有共同的父类。

  可以用买包子来举例说明这个模式，你开了一个小店，有很多不同类型的包子；也就是说你要创建很多不同的类。

  定义包子的抽象共同父类：

  ```java
  public abstract class BaoZi{
      protected String name;
      
      /**
  	 * 准备工作
  	 */
  	public void prepare()
  	{
  		System.out.println("揉面-完成准备工作");
  	}
   
  	/**
  	 * 使用你们的专用袋-包装
  	 */
  	public void pack()
  	{
  		System.out.println("包子-专用袋-包装");
  	}
  	/**
  	 * 秘制设备-烘烤2分钟
  	 */
  	public void fire()
  	{
  		System.out.println("包子-专用设备-烘烤");
  	}
      
  }
  ```

  定义各种包子的子类：

  ```java
  public class RouBaoZi extends BaoZi{
      public RouBaoZi(){
          this.name = "肉包子";
      }
  }
  ```

  .....

  然后你要在店内销售，就要一个统一的出口。也就是创建对象的方法类。

  定义`BaoZiStore` 类

  ```java
  public class BaoZiStore{
      
      public BaoZi sellBaoZi(String type){
          BaoZi baoZi = null;
          if(type.equals("rouBaoZi")){
              baoZi = new RouBaoZi();
          }
          .......
          baoZi.prepare();
          baoZi.pack();
          baoZi.fire();
          return baoZi;
      }
  }
  ```

  这种方式弊端也很明显，当要增加或删除包子的类型时，要修改sellBaoZi方法，耦合度高。这时就需要简单对象工厂来了。

   ```java
    public class SimpleBaoZiFactroy
   {
   	public RouJiaMo createBaoZi(String type)
   	{
   		BaoZi baoZi = null;
   		if(type.equals("rouBaoZi")){
               baoZi = new RouBaoZi();
           }
           ....
   		return baoZi;
   	}
    
   }
   ```

  然后修改`BaoZiStroe` 类

  ```java
  package com.zhy.pattern.factory.a;
   
  public class RoujiaMoStore
  {
  	private SimpleRouJiaMoFactroy factroy;
   
  	public RoujiaMoStore(SimpleRouJiaMoFactroy factroy)
  	{
  		this.factroy = factroy;
  	}
   
  	/**
  	 * 根据传入类型卖不同的肉夹馍
  	 * 
  	 * @param type
  	 * @return
  	 */
  	public RouJiaMo sellRouJiaMo(String type)
  	{
  		RouJiaMo rouJiaMo = factroy.createRouJiaMo(type);
  		rouJiaMo.prepare();
  		rouJiaMo.fire();
  		rouJiaMo.pack();
  		return rouJiaMo;
  	}
   
  }
  ```

* ### 工厂方法模式

  > 定义：定义一个创建对象的接口，但由子类决定要实例化的类是哪一个。工厂方法模式把类实例化的过程推迟到子类。
  >
  > **工厂方法模式，创建一个工厂接口和创建多个工厂实现类，一旦需要增加新的功能，直接增加新的工厂类就可以了，不需要修改之前的代码。有利于代码的维护和扩展。**

  工厂接口：

  ```java
  public interface PhoneFactory {
      public Phone create();
  }
  ```

  小米手机工厂：

  ```java
  public class MPhoneFactory implements PhoneFactory{
      @Override
      public Phone create() {
          return new MPhone();
      }
  }
  ```

  苹果手机工厂：

  ```java
  public class IPhoneFactory implements PhoneFactory{
      @Override
      public Phone create() {
          return new IPhone();
      }
  }
  ```

  测试类：

  ```java
  public class Test {
      public static void main(String[] args) {
          // 生产小米手机
          PhoneFactory factory1 = new MPhoneFactory();
          factory1.create().call();
  
          // 生产苹果手机
          PhoneFactory factory2 = new IPhoneFactory();
          factory2.create().call();
      }
  }
  ```

* ### 抽象工厂模式

  > 定义：提供一个接口，用于创建相关的或依赖对象的家族，而不需要明确指定具体类
  >
  > 抽象工厂模式可以将简单工厂模式和工厂方法模式进行整合。
  > 从设计层面看，抽象工厂模式就是对简单工厂模式的改进(或者称为进一步的抽象)。
  > 将工厂抽象成两层，抽象工厂 和 具体实现的工厂子类。程序员可以根据创建对象类型使用对应的工厂子类。这样将单个的简单工厂类变成了工厂集合， 更利于代码的维护和扩展。



## 单例设计模式

>  单例模式主要是为了避免因为创建了多个实例造成资源的浪费，且多个实例由于多次调用容易导致结果出现错误，而**使用单例模式能够保证整个应用中有且只有一个实例**。
>
> -  定义：只需要三步就可以保证对象的唯一性 
>   - (1) 不允许其他程序用new对象
>   - (2) 在该类中创建对象
>   - (3) 对外提供一个可以让其他程序获取该对象的方法



* ### 实现方法

  #### 饿汉式：

  由于是在静态代码块中的创建对象，在类加载的时候就完成了实例化。

  ```java
  public class Singleton{
   
  	private static Singleton instance = null;
  	
  	static {
  		instance = new Singleton();
  	}
   
  	private Singleton() {};
   
  	public static Singleton getInstance() {
  		return instance;
  	}
  }
  -------------访问方式--------------
  Singleton instance = Singleton.getInstance();
  ```

  #### 懒汉式（双重校验锁）：

  ```java
  public class Singleton {
  	/**
  	 * 懒汉式变种，属于懒汉式中最好的写法，保证了：延迟加载和线程安全
  	 */
  	private volatile static Singleton instance=null;
  	
  	private Singleton() {};
  	
  	public static Singleton getInstance(){
  		 if (instance == null) {  
  	          synchronized (Singleton.class) {  
  	              if (instance == null) {  
  	            	  instance = new Singleton();  
  	              }  
  	          }  
  	      }  
  	      return instance;  
  	}
  }
  -----------访问方式--------------
  Singleton instance = Singleton.getInstance();
  ```

  #### 内部类：

  ```java
  public class Singleton{
  	
  	private Singleton() {};
  	
  	private static class SingletonHolder{
  		private static Singleton instance=new Singleton();
  	} 
  	
  	public static Singleton getInstance(){
  		return SingletonHolder.instance;
  	}
  }
  ```

  #### 枚举：

  ```java
  public enum SingletonEnum {
  	
  	 instance; 
  	 
  	 private SingletonEnum() {}
  	 
  	 public void method(){
  	 }
  }
  ------------访问方式-------------
  SingletonEnum.instance.method();
  ```



## 策略模式

>  策略模式：定义了算法族，分别封装起来，让它们之间可相互替换，此模式让算法的变化独立于使用算法的客户。
>
> - 定义了一组算法（业务规则）
> - 封装了每个算法
> - 这族的算法可互换代替

* ### 组成

  - **抽象策略角色**： 策略类，通常由一个接口或者抽象类实现
  - **具体策略角色**：包装了相关的算法和行为
  - **环境角色**：持有一个策略类的引用，最终给客户端调用

* ### 应用场景

  1. 多个类只区别在表现行为不同，可以使用Strategy模式，在运行时动态选择具体要执行的行为

  2. 需要在不同情况下使用不同的策略(算法)，或者策略还可能在未来用其它方式来实现

  3. 对客户隐藏具体策略(算法)的实现细节，彼此完全独立

  4. 一个类定义了多种行为，并且这些行为在类的操作中以多个条件语句的形式出现

[Java设计模式-策略模式（优化过多if/switch语句） - 掘金 (juejin.cn)](https://juejin.cn/post/7023351087230877704)

* ### 使用

  1. 定义一个抽象策略类

  2. 定义多个具体策略类

  3. 用策略工厂来创建策略类

     > 如何获取策略类？

     1. 用 `if-else` 语句来获取策略类
     2. 用`map`集合存储策略类
     3. 在抽象策略类中定义注册方法，然后在具体策略类中实现注册到map集合中。
     4. 用注解加反射的方式获取，自动装配



## 适配器模式

>  定义：将一个类的接口转换成客户期望的另一个接口，适配器让原本接口不兼容的类可以相互合作。这个定义还好，说适配器的功能就是把一个接口转成另一个接口。



* ### 类适配器模式

  > 通过继承特性来实现适配器功能

  源对象：

  ```java
  /**
   * 两孔插座
   */
  public class TwoHoleSocket {
      public void twoHole() {
          System.out.println("插入两孔插座");
      }
  }
  ```

  目标对象：

  ```java
  public interface ThreeHoleSocket {
      void threeHole();
  }
  ```

  适配器：

  ```java
  /**
   * 适配器：可以同时使用两孔和三孔插座
   */
  public class Adapter extends TwoHoleSocket implements ThreeHoleSocket{
      @Override
      public void threeHole() {
          System.out.println("插入三孔插座");
      }
  
      public static void main(String[] args) {
          Adapter adapter = new Adapter();
          adapter.twoHole();//使用两孔插座
          adapter.threeHole();//使用三孔插座
      }
  }
  ```

* ### 对象适配器模式

  ```java
  /**
   * 适配器
   */
  public class Adapter implements ThreeHoleSocket{
      //通过组合持有两孔插座的对象，内部引用两孔插座来适配
      private TwoHoleSocket twoHoleSocket;
  
      public Adapter(TwoHoleSocket twoHoleSocket) {
          this.twoHoleSocket = twoHoleSocket;
      }
  
      public void twoHole() {
          twoHoleSocket.twoHole();
      }
  
      @Override
      public void threeHole() {
          System.out.println("插入三孔插座");
      }
  
      public static void main(String[] args) {
          Adapter adapter = new Adapter(new TwoHoleSocket());
          adapter.twoHole();//两孔插座
          adapter.threeHole();//三孔插座
      }
  }
  ```

* ### 接口适配器模式

  > 接口适配器模式也称作缺省适配模式，就是有时候一个接口的方法太多，**我只想用其中的一两个，不想为其他方法提供实现，就可以通过一个抽象类为这个接口的所有方法，提供空实现，如果想用哪个方法，再提供一个子类继承这个抽象类，覆盖父类某个方法即可；**

  抽象接口：

  ```java
  /**
   * 墙上插座
   */
  public interface HallSocket {
      //两孔插口
      void twoHole();
      //三孔插口
      void threeHole();
      //usb插口
      void usbHole();
  }
  ```

  缺省类实现：

  ```java
  /**
   * 抽象类：提供缺省实现
   */
  public class AbstractAdapter implements HallSocket{
      @Override
      public void twoHole() {
      }
      @Override
      public void threeHole() {
      }
      @Override
      public void usbHole() {
      }
  }
  ```

  目标子类：

  ```java
  public class UsbSocket extends AbstractAdapter{
      @Override
      public void usbHole() {
          System.out.println("usb插口");
      }
  
      //测试
      public static void main(String[] args) {
          UsbSocket usbSocket = new UsbSocket();
          usbSocket.usbHole();
      }
  }
  ```



## 命令模式

>  定义：将“请求”封装成对象，以便使用不同的请求、队列或者日志来参数化其他对象。命令模式也支持可撤销的操作。(简化: 将请求封装成对象，将动作请求者和动作执行者解耦。)



* ### 命令模式的结构和说明

![](image\屏幕截图 2023-12-27 174557.png)

* **Command** 命令接口，声明执行方法。
* **ConcreteCommand** 命令接口的实现对象，是“虚”的实现，通常会持有接收者，通过调用接收者的方法，来完成命令要执行的操作。
* **Receiver** 接收者，真正执行命令的对象。
* **Invoker** 调用者，通常会持有命令，相当于使用命令模式的入口。

## 装饰者模式

>  装饰者模式：若要扩展功能，装饰者提供了比集成更有弹性的替代方案，动态地将责任附加到对象上。

1. **定义抽象组件**（Component）：抽象组件是一个接口或者抽象类，定义了被装饰对象和装饰对象共同实现的方法。在Java中，可以使用接口或者抽象类来实现抽象组件。
2. **定义具体组件**（ConcreteComponent）：具体组件是实现抽象组件接口或抽象类的一个具体对象。具体组件是被装饰的对象。
3. **定义装饰者抽象类**（Decorator）：装饰者抽象类继承了抽象组件，并持有一个抽象组件的引用。装饰者抽象类中定义了一个构造函数，用于传入被装饰对象，并且在其中调用抽象组件的方法。
4. **定义具体装饰者类**（ConcreteDecorator）：具体装饰者类继承了装饰者抽象类，并在其中添加新的行为或者功能。具体装饰者类可以通过组合其他具体装饰者类，实现多个装饰者的嵌套。



```java
// 定义抽象组件
interface Component {
    void operation();
}

// 定义具体组件
class ConcreteComponent implements Component {
    public void operation() {
        System.out.println("具体组件的操作");
    }
}

// 定义装饰者抽象类
class Decorator implements Component {
    private Component component;

    public Decorator(Component component) {
        this.component = component;
    }

    public void operation() {
        component.operation();
    }
}

// 定义具体装饰者类
class ConcreteDecoratorA extends Decorator {
    public ConcreteDecoratorA(Component component) {
        super(component);
    }

    public void operation() {
        super.operation();
        addedBehavior();
    }

    public void addedBehavior() {
        System.out.println("具体装饰者A的操作");
    }
}

// 定义具体装饰者类
class ConcreteDecoratorB extends Decorator {
    public ConcreteDecoratorB(Component component) {
        super(component);
    }

    public void operation() {
        super.operation();
        addedBehavior();
    }

    public void addedBehavior() {
        System.out.println("具体装饰者B的操作");
    }
}

// 测试代码
public class DecoratorPatternDemo {
    public static void main(String[] args) {
        Component component = new ConcreteComponent();
        Component decoratorA = new ConcreteDecoratorA(component);
        Component decoratorB = new ConcreteDecoratorB(decoratorA);
        decoratorB.operation();
    }
}
```





## 外观模式

>  定义：提供一个统一的接口，用来访问子系统中的一群接口，外观定义了一个高层的接口，让子系统更容易使用。**其实就是为了方便客户的使用，把一群操作，封装成一个方法。**



## 模板方法模式

>  定义：定义了一个算法的骨架，而将一些步骤延迟到子类中，模版方法使得子类可以在不改变算法结构的情况下，重新定义算法的步骤。



## 状态模式

>  定义：允许对象在内部状态改变时改变它的行为，对象看起来好像修改了它的类。



## 建造者模式

>  建造模式是对象的创建模式。建造模式可以将一个产品的内部表象（internal representation）与产品的生产过程分割开来，从而可以使一个建造过程生成具有不同的内部表象的产品对象。

* ### 模式结构

  ![](image\屏幕截图 2023-12-27 180747.png)

  **抽象建造者类**（Builder）：这个接口规定要实现复杂对象的那些部分的创建，并不涉及具体的部件对象的创建。

  **具体建造者类**（ConcreteBuilder）：实现Builder接口，完成复杂产品的各个部件的具体创建方法。在构造过程完成后，提供产品的实例。

  **产品类**（Product）：要创建的复杂对象。

  **指挥者类**（Director）：调用具体建造者来创建复杂对象的各个部分，在指导者中不涉及具体产品的信息，只负责保证对象各部分完整创建或按某种顺序创建。



* ### 使用

  以构建一个自行车为例子，然后结构有车架和座位。

  ```java
  public class Bike {
      private String frame;
      private String seat;
   
      public String getFrame() {
          return frame;
      }
   
      public void setFrame(String frame) {
          this.frame = frame;
      }
   
      public String getSeat() {
          return seat;
      }
   
      public void setSeat(String seat) {
          this.seat = seat;
      }
  }
   
   
  public abstract class Builder {
      protected Bike bike=new Bike();
   
      public abstract void buildFrame();
      public abstract void buildSeat();
   
      public abstract Bike createBike();
   
  }
   
  public class MobileBuilder extends Builder{
      @Override
      public void buildFrame() {
          bike.setFrame("碳纤维车架");
      }
   
      @Override
      public void buildSeat() {
          bike.setSeat("真皮");
      }
   
      @Override
      public Bike createBike() {
   
          return bike;
      }
  }
  public class OfoBuilder extends Builder{
      @Override
      public void buildFrame() {
          bike.setFrame("铝合金车架");
      }
   
      @Override
      public void buildSeat() {
          bike.setSeat("橡胶");
      }
   
      @Override
      public Bike createBike() {
          return bike;
      }
  }
   
  public class Director {
      private Builder builder;
   
      public Director(Builder builder) {
          this.builder = builder;
      }
      public Bike construct(){
          builder.buildFrame();
          builder.buildSeat();
          return builder.createBike();
      }
  }
   
  public class Client {
      public static void main(String[] args) {
          //创建指挥者对象
          Director director = new Director(new MobileBuilder());
          //让指挥者组装自行车
          Bike bike = director.construct();
          System.out.println(bike.getFrame());
          System.out.println(bike.getSeat());
      }
  }
  ```

  







## 原型模式

>  原型模式是用于创建重复的对象，同时又能保证性能。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。



* ### 模式结构

  **Prototype**(抽象原型类)
  抽象原型类是定义具有克隆自己的方法的接口,是所有具体原型类的公共父类,可以是抽象类,也可以是接口。

  **ConcretePrototype**(具体原型类)
  具体原型类实现具体的克隆方法,在克隆方法中返回自己的一个克隆对象。

  **Client**(客户类)
  客户类让一个原型克隆自身,从而创建一个新的对象。在客户类中只需要直接实例化或通过工厂方法等方式创建一个对象,再通过调用该对象的克隆方法复制得到多个相同的对象。

* ### 使用

  使用浅拷贝和深拷贝`clone()`函数



## 代理模式

> 代理模式是一种比较好理解的设计模式。简单来说就是 **我们使用代理对象来代替对真实对象(real object)的访问，这样就可以在不修改原目标对象的前提下，提供额外的功能操作，扩展目标对象的功能。**
>
> **代理模式的主要作用是扩展目标对象的功能，比如说在目标对象的某个方法执行前后你可以增加一些自定义的操作。**
>
> 举个例子：新娘找来了自己的姨妈来代替自己处理新郎的提问，新娘收到的提问都是经过姨妈处理过滤之后的。姨妈在这里就可以看作是代理你的代理对象，代理的行为（方法）是接收和回复新郎的提问。

* ### 静态代理

  > **静态代理中，我们对目标对象的每个方法的增强都是手动完成的（\*后面会具体演示代码\*），非常不灵活（\*比如接口一旦新增加方法，目标对象和代理对象都要进行修改\*）且麻烦(\*需要对每个目标类都单独写一个代理类\*）。** 实际应用场景非常非常少，日常开发几乎看不到使用静态代理的场景。
  >
  > 上面我们是从实现和应用角度来说的静态代理，从 JVM 层面来说， **静态代理在编译时就将接口、实现类、代理类这些都变成了一个个实际的 class 文件。
  >
  > 编译时代理

  静态代理实现步骤:

  1. 定义一个接口及其实现类；
  2. 创建一个代理类同样实现这个接口
  3. 将目标对象注入进代理类，然后在代理类的对应方法调用目标类中的对应方法。这样的话，我们就可以通过代理类屏蔽对目标对象的访问，并且可以在目标方法执行前后做一些自己想做的事情。

* ### 动态代理

  > 相比于静态代理来说，动态代理更加灵活。我们不需要针对每个目标类都单独创建一个代理类，并且也不需要我们必须实现接口，我们可以直接代理实现类( *CGLIB 动态代理机制*)。
  >
  > **从 JVM 角度来说，动态代理是在运行时动态生成类字节码，并加载到 JVM 中的。**
  >
  > 说到动态代理，Spring AOP、RPC 框架应该是两个不得不提的，它们的实现都依赖了动态代理。
  >
  > **动态代理在我们日常开发中使用的相对较少，但是在框架中的几乎是必用的一门技术。学会了动态代理之后，对于我们理解和学习各种框架的原理也非常有帮助。**
  >
  > 就 Java 来说，动态代理的实现方式有很多种，比如 **JDK 动态代理**、**CGLIB 动态代理**等等

  #### 基于JDK动态代理类的实现

  **在 Java 动态代理机制中 `InvocationHandler` 接口和 `Proxy` 类是核心。**

  ```java
  public class ProxyFactory {
  
      //目标对象
      private  Object target;
  
      public ProxyFactory(Object target){
          this.target = target;
      }
  
      //返回代理对象
      public Object getProxy() {
          /*
          * Proxy.newProxyInstance()方法
          * 有三个参数
          *   第一个参数：ClassLoader：加载动态生成代理类的加载器
          *   第二个参数：Class[] interfaces：目录对象实现的所有接口的class类型数组
          *   第三个参数：InvocationHandler：设置代理对象实现目标对象方法的过程
           */
  
          //第一个参数：ClassLoader：加载动态生成代理类的加载器
          ClassLoader classLoader = target.getClass().getClassLoader();
          //第二个参数：Class[] interfaces：目录对象实现的所有接口的class类型数组
          Class<?>[] interfaces = target.getClass().getInterfaces();
          //第三个参数：InvocationHandler：设置代理对象实现目标对象方法的过程
          InvocationHandler invocationHandler = new InvocationHandler() {
  
              //第一个参数：proxy的代理对象
              //第二个参数：需要重写目标对象的方法
              //第三个参数：method对象所需要的参数
              @Override
              public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  
                  System.out.println("[动态代理][日志] "+method.getName()+"，参数："+ Arrays.toString(args));
                  Object result = method.invoke(target,args);
                  System.out.println("[动态代理][日志] "+method.getName()+"，结果："+ result);
                  return result;
              }
          };
          return Proxy.newProxyInstance(classLoader,interfaces,invocationHandler);
      }
  }
  ```

  1. 定义一个接口及其实现类；

  2. 自定义 `InvocationHandler` 并重写`invoke`方法，在 `invoke` 方法中我们会调用原生方法（被代理类的方法）并自定义一些处理逻辑；

  3. 通过 `Proxy.newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h)` 方法创建代理对象。

  #### 原理：

  在`newProxyInstance`方法中会以克隆的方式拿到接口。

  #### CGLIB 动态代理机制

  > **JDK 动态代理有一个最致命的问题是其只能代理实现了接口的类。**
  >
  > **为了解决这个问题，我们可以用 CGLIB 动态代理机制来避免。**
  >
  > [CGLIBopen in new window](https://github.com/cglib/cglib)(*Code Generation Library*)是一个基于[ASMopen in new window](http://www.baeldung.com/java-asm)的字节码生成库，它允许我们在运行时对字节码进行修改和动态生成。CGLIB 通过继承方式实现代理。很多知名的开源框架都使用到了[CGLIBopen in new window](https://github.com/cglib/cglib)， 例如 Spring 中的 AOP 模块中：如果目标对象实现了接口，则默认采用 JDK 动态代理，否则采用 CGLIB 动态代理。

  **CGLIB 动态代理是通过生成一个被代理类的子类来拦截被代理类的方法调用，因此不能代理声明为 final 类型的类和方法。**

  **在 CGLIB 动态代理机制中 `MethodInterceptor` 接口和 `Enhancer` 类是核心。**

  你需要自定义 `MethodInterceptor` 并重写 `intercept` 方法，`intercept` 用于拦截增强被代理类的方法

  #### 使用：

  导入依赖

  ```xml
  <dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.3.0</version>
  </dependency>
  ```

  目标类

  ```java
  public class AliSmsService {
      public String send(String message) {
          System.out.println("send message:" + message);
          return message;
      }
  }
  ```

  自定义拦截器

  ```java
  import net.sf.cglib.proxy.MethodInterceptor;
  import net.sf.cglib.proxy.MethodProxy;
  
  import java.lang.reflect.Method;
  
  /**
   * 自定义MethodInterceptor
   */
  public class DebugMethodInterceptor implements MethodInterceptor {
  
  
      /**
       * @param o           被代理的对象（需要增强的对象）
       * @param method      被拦截的方法（需要增强的方法）
       * @param args        方法入参
       * @param methodProxy 用于调用原始方法
       */
      @Override
      public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
          //调用方法之前，我们可以添加自己的操作
          System.out.println("before method " + method.getName());
          Object object = methodProxy.invokeSuper(o, args);
          //调用方法之后，我们同样可以添加自己的操作
          System.out.println("after method " + method.getName());
          return object;
      }
  }
  ```

  获取代理类

  ```java
  import net.sf.cglib.proxy.Enhancer;
  
  public class CglibProxyFactory {
  
      public static Object getProxy(Class<?> clazz) {
          // 创建动态代理增强类
          Enhancer enhancer = new Enhancer();
          // 设置类加载器
          enhancer.setClassLoader(clazz.getClassLoader());
          // 设置被代理类
          enhancer.setSuperclass(clazz);
          // 设置方法拦截器
          enhancer.setCallback(new DebugMethodInterceptor());
          // 创建代理类
          return enhancer.create();
      }
  }
  ```

  使用

  ```java
  AliSmsService aliSmsService = (AliSmsService) CglibProxyFactory.getProxy(AliSmsService.class);
  aliSmsService.send("java");
  ```

  #### 

  
