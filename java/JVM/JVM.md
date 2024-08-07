## JVM

> JVM是运行在操作系统之上的，它与硬件没有直接的交互



## JVM的架构模型

java编译器输入的指令流基本是一种**基于栈的指令集架构**，另一种指令集架构则是**基于寄存器的指令集架构**。



字节码文件对象进入JVM，先被**类加载器子系统Class Loader SubSystem**初始化，把相关数据存储在**运行时数据区Runtime Data Area**，最后被**执行引擎Execution Engine**所执行。



### JVM生命周期

**虚拟机的启动**



Java虚拟机的启动是通过引导类加载器（bootstrap class loader）创建一个初始类（initial class）来完成的，这个类是由虚拟机的具体实现指定的。



**虚拟机的执行**



- 一个运行中的Java虚拟机有着一个清晰的任务：执行Java程序。

- 程序开始执行时他才运行，程序结束时他就停止。

- 执行一个所谓的Java程序的时候，真真正正在执行的是一个叫做Java虚拟机的进程。



**虚拟机的退出**



有如下的几种情况：



- 程序正常执行结束

- 程序在执行过程中遇到了异常或错误而异常终止

- 由于操作系统用现错误而导致Java虚拟机进程终止

- 某线程调用Runtime类或system类的exit方法，或Runtime类的halt方法，并且Java安全管理器也允许这次exit或halt操作。

- 除此之外，JNI（Java Native Interface）规范描述了用JNI Invocation API来加载或卸载 Java虚拟机时，Java虚拟机的退出情况。



## 三大虚拟机

### HotSpot VM（用的最多）

### JRockit（专注于服务器应用）

### IBM的J9

### Graal VM

跨语言的全栈虚拟机



## 第二章 类加载器子系统

### 2.1 内存结构概述

- Class文件

- 类加载子系统

- 运行时数据区 

- - 方法区

- - 堆

- - 程序计数器

- - 虚拟机栈

- - 本地方法栈

 

- 执行引擎

- 本地方法接口

- 本地方法库

![](image\屏幕截图 2024-01-14 133447.png)

![](image\屏幕截图 2024-01-14 133534.png)

### 概述

```java
public class HelloLoader {

    public static void main(String[] args) {
        System.out.println("谢谢ClassLoader加载我....");
        System.out.println("你的大恩大德，我下辈子再报！");
    }
}
```

它的加载过程是怎么样的呢?

*   执行 main() 方法（静态方法）就需要先加载main方法所在类 HelloLoader
*   加载成功，则进行链接、初始化等操作。完成后调用 HelloLoader 类中的静态方法 main
*   加载失败则抛出异常

* 类加载器的过程：

### 加载阶段

1. 通过一个类的全限定名获取定义此类的二进制字节流
2. 将这个字节流所代表的的静态存储结构转换为方法区的运行时数据结构
3. **在内存中生成一个代表这个类的java.lang.Class对象（可以说成是字节码文件对象）**，作为方法区这个类的各种数据访问入口。

### 链接阶段

验证-> 准备 -> 解析

#### 准备

1. 为类变量（static变量）分配内存并且设置默认初始值
2. 用final修饰的static在编译时就会分配好了默认值，准备阶段会显示初始化
3. 注意：这里不会为实例变量分配初始化，类变量会分配在方法区中，而实例变量是会随着对象一起分配到Java堆中



### 堆

> Java 7及之前堆内存逻辑上分为三部分：新生区+养老区+永久区
>
> - Young Generation Space 新生区 Young/New 又被划分为Eden区和Survivor区
>
> - Tenure generation space 养老区 Old/Tenure
>
> - Permanent Space 永久区 Perm
>
> Java 8及之后堆内存逻辑上分为三部分：新生区+养老区+元空间
>
> - Young Generation Space 新生区 Young/New 又被划分为Eden区和Survivor区
>
> - Tenure generation space 养老区 Old/Tenure
>
> - Meta Space 元空间 Meta

![](image\屏幕截图 2024-01-14 135628.png)

* #### 年轻代与老年代

  存储在JVM中的Java对象可以被划分为两类：

  

  - 一类是生命周期较短的瞬时对象，这类对象的创建和消亡都非常迅速

  - 另外一类对象的生命周期却非常长，在某些极端的情况下还能够与JVM的生命周期保持一致

  Java堆区进一步细分的话，可以划分为年轻代（YoungGen）和老年代（oldGen）

  其中年轻代又可以划分为Eden空间、Survivor0空间和Survivor1空间（有时也叫做from区、to区）

  **几乎所有的Java对象都是在Eden区被new出来的。绝大部分的Java对象的销毁都在新生代进行了。**

  ![](image\屏幕截图 2024-01-14 135901.png)

* #### 对象分配过程

  1.  new的对象先放伊甸园区。此区有大小限制。 

  2. 当伊甸园的空间填满时，程序又需要创建对象，JVM的垃圾回收器将对伊甸园区进行垃圾回收（MinorGC），将伊甸园区中的不再被其他对象所引用的对象进行销毁。再加载新的对象放到伊甸园区 

  3. 然后将伊甸园中的剩余对象移动到幸存者0区。 

  4. 如果再次触发垃圾回收，此时上次幸存下来的放到幸存者0区的，如果没有回收，就会放到幸存者1区。 

  5. 如果再次经历垃圾回收，此时会重新放回幸存者0区，接着再去幸存者1区。 

  6. 啥时候能去养老区呢？可以设置次数。默认是15次。 

     可以设置参数：进行设置`-Xx:MaxTenuringThreshold= N`

  7.  在养老区，相对悠闲。当养老区内存不足时，再次触发GC：Major GC，进行养老区的内存清理 

  8. 若养老区执行了Major GC之后，发现依然无法进行对象的保存，就会产生OOM异常。 

  - **总结**
  - - 针对幸存者s0，s1区的总结：复制之后有交换，谁空谁是to
  - - 关于垃圾回收：频繁在新生区收集，很少在老年代收集，几乎不再永久代和元空间进行收集

### 方法区

![](image\屏幕截图 2024-01-14 142041.png)

- 方法区（Method Area）与Java堆一样，是各个线程共享的内存区域。

- 方法区在JVM启动的时候被创建，并且它的实际的物理内存空间中和Java堆区一样都可以是不连续的。

- 方法区的大小，跟堆空间一样，可以选择固定大小或者可扩展。

- 方法区的大小决定了系统可以保存多少个类，如果系统定义了太多的类，导致方法区溢出，虚拟机同样会抛出内存溢出错误：`java.lang.OutOfMemoryError: PermGen space` 或者`java.lang.OutOfMemoryError: Metaspace` 

- - 加载大量的第三方的jar包；Tomcat部署的工程过多（30~50个）；大量动态的生成反射类

- 关闭JVM就会释放这个区域的内存。

> 在jdk7及以前，习惯上把方法区，称为永久代。jdk8开始，使用元空间取代了永久代。
>
> **元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代最大的区别在于：元空间不在虚拟机设置的内存中，而是使用本地内存**

#### 如何解决这些OOM

1.  要解决OOM异常或heap space的异常，一般的手段是首先通过内存映像分析工具（如Eclipse Memory Analyzer）对dump出来的堆转储快照进行分析，重点是确认内存中的对象是否是必要的，也就是要先分清楚到底是出现了内存泄漏（Memory Leak）还是内存溢出（Memory Overflow）
2.  如果是内存泄漏，可进一步通过工具查看泄漏对象到GC Roots的引用链。于是就能找到泄漏对象是通过怎样的路径与GCRoots相关联并导致垃圾收集器无法自动回收它们的。掌握了泄漏对象的类型信息，以及GCRoots引用链的信息，就可以比较准确地定位出泄漏代码的位置。 
3.  如果不存在内存泄漏，换句话说就是内存中的对象确实都还必须存活着，那就应当检查虚拟机的堆参数（`-Xmx`与`-Xms`），与机器物理内存对比看是否还可以调大，从代码上检查是否存在某些对象生命周期过长、持有状态时间过长的情况，尝试减少程序运行期的内存消耗。



* #### 内存结构

  

  #### 类型信息

  对每个加载的类型（类class、接口interface、枚举enum、注解annotation），JVM必须在方法区中存储以下类型信息：

  1. 这个类型的完整有效名称（全名=包名.类名）

  1. 这个类型直接父类的完整有效名（对于interface或是java.lang.object，都没有父类）

  1. 这个类型的修饰符（public，abstract，final的某个子集）

  1. 这个类型直接接口的一个有序列表

  #### 域（Field）信息

  JVM必须在方法区中保存类型的所有域的相关信息以及域的声明顺序。

  

  域的相关信息包括：域名称、域类型、域修饰符（public，private，protected，static，final，volatile，transient的某个子集）

  #### 方法（Method）信息

  JVM必须保存所有方法的以下信息，同域信息一样包括声明顺序：

  

  1. 方法名称

  1. 方法的返回类型（或void）

  1. 方法参数的数量和类型（按顺序）

  1. 方法的修饰符（public，private，protected，static，final，synchronized，native，abstract的一个子集）

  1. 方法的字节码（bytecodes）、操作数栈、局部变量表及大小（abstract和native方法除外）

  1. 异常表（abstract和native方法除外） 

  - - 每个异常处理的开始位置、结束位置、代码处理在程序计数器中的偏移地址、被捕获的异常类的常量池索引

  #### non-final的类变量

  

  - 静态变量和类关联在一起，随着类的加载而加载，他们成为类数据在逻辑上的一部分

  - 类变量被类的所有实例共享，即使没有类实例时，你也可以访问它



### 垃圾回收概述及算法

> ### Stop-the-World，简称STW
>
> **1、指的是GC事件发生过程中，会产生应用程序的停顿。停顿产生时整个应用程序线程都会被暂停，没有任何响应, 有点像卡死的感觉，这个停顿称为STW。**

> GC主要关注的区域？

主要是**方法区**和**堆**中的垃圾收集



* #### 对象存活判断算法

  在堆里存放着几乎所有的Java对象实例，在GC执行垃圾回收之前，首先需要区分出内存中哪些是存活对象，哪些是已经死亡的对象。只有被标记为己经死亡的对象，GC才会在执行垃圾回收时，释放掉其所占用的内存空间，因此这个过程我们可以称为垃圾标记阶段。

  

  那么在JVM中究竟是如何标记一个死亡对象呢？简单来说，当一个对象已经不再被任何的存活对象继续引用时，就可以宣判为已经死亡。

  

  判断对象存活一般有两种方式：**引用计数算法**和**可达性分析算法**。

  #### 方法一：引用计数算法

  引用计数算法（Reference Counting）比较简单，对每个对象保存一个整型的引用计数器属性。用于记录对象被引用的情况。

  优点：实现简单，垃圾对象便于辨识；判定效率高，回收没有延迟性。

  缺点：

  - 它需要单独的字段存储计数器，这样的做法增加了存储空间的开销。

  - 每次赋值都需要更新计数器，伴随着加法和减法操作，这增加了时间开销。

  - 引用计数器有一个严重的问题，即**无法处理循环引用**的情况。这是一条致命缺陷，导致在Java的垃圾回收器中没有使用这类算法。会导致内存泄露问题。（**内存泄露 memory leak，是指程序在申请内存后，无法释放已申请的内存空间，一次内存泄露危害可以忽略，但内存泄露堆积后果很严重，无论多少内存,迟早会被占光。**）

  #### 方法二：可达性分析算法

  ##### 可达性分析算法（根搜索算法、追踪性垃圾收集）

  相对于引用计数算法而言，可达性分析算法不仅同样具备实现简单和执行高效等特点，更重要的是该算法可以有效地**解决在引用计数算法中循环引用的问题，防止内存泄漏的发生。**

  相较于引用计数算法，这里的可达性分析就是Java、C#选择的。这种类型的垃圾收集通常也叫作**追踪性垃圾收集（Tracing Garbage Collection）**

  所谓**"GCRoots”**根集合就是一组必须活跃的引用。

  ##### 基本思路

  - 可达性分析算法是以根对象集合（GCRoots）为起始点，按照从上至下的方式搜索被根对象集合所连接的**目标对象是否可达**。

  - 使用可达性分析算法后，内存中的存活对象都会被根对象集合直接或间接连接着，搜索所走过的路径称为**引用链（Reference Chain）**

  - 如果目标对象没有任何引用链相连，则是不可达的，就意味着该对象己经死亡，可以标记为垃圾对象。

  - 在可达性分析算法中，只有能够被根对象集合直接或者间接连接的对象才是存活对象。

  ##### 在java语言中，GC Roots包括以下几类元素：

  - 虚拟机栈中引用的对象 

  - - 比如：各个线程被调用的方法中使用到的参数、局部变量等。

  - 本地方法栈内JNI（通常说的本地方法）引用的对象

  - 方法区中类静态属性引用的对象 

  - - 比如：Java类的引用类型静态变量

  - 方法区中常量引用的对象 

  - - 比如：字符串常量池（String Table）里的引用

  - 所有被同步锁synchronized持有的对象

  - Java虚拟机内部的引用。 

  - - 基本数据类型对应的Class对象，一些常驻的异常对象（如：NullPointerException、OutOfMemoryError），系统类加载器。

  - 反映java虚拟机内部情况的JMXBean、JVMTI中注册的回调、本地代码缓存等。

  

  ![img](https://gitee.com/vectorx/ImageCloud/raw/master/img/20210512105351.png)

* #### 对象的finalization机制

  > Java语言提供了对象终止（finalization）机制来允许开发人员提供对象被销毁之前的自定义处理逻辑。
  >
  > 
  >
  > 当垃圾回收器发现没有引用指向一个对象，即：垃圾回收此对象之前，总会先调用这个对象的finalize()方法。
  >
  > 
  >
  > finalize() 方法允许在子类中被重写，用于在对象被回收时进行资源释放。通常在这个方法中进行一些资源释放和清理的工作，比如关闭文件、套接字和数据库连接等。
  >
  > 
  >
  > 永远不要主动调用某个对象的finalize()方法I应该交给垃圾回收机制调用。理由包括下面三点：
  >
  > - 在finalize()时可能会导致对象复活。
  >
  > - finalize()方法的执行时间是没有保障的，它完全由GC线程决定，极端情况下，若不发生GC，则finalize()方法将没有执行机会。
  >
  > - 一个糟糕的finalize()会严重影响Gc的性能。
  >
  > 
  >
  > 从功能上来说，finalize()方法与C中的析构函数比较相似，但是Java采用的是基于垃圾回收器的自动内存管理机制，所以finalize()方法在本质上不同于C中的析构函数。
  >
  > 
  >
  > 由于finalize()方法的存在，虚拟机中的对象一般处于三种可能的状态。
  >
  > - 可触及的：从根节点开始，可以到达这个对象。
  >
  > - 可复活的：对象的所有引用都被释放，但是对象有可能在finalize()中复活。
  >
  > - 不可触及的：对象的finalize()被调用，并且没有复活，那么就会进入不可触及状态。不可触及的对象不可能被复活，因为finalize()只会被调用一次。

* #### 清除阶段：标记-清除算法

  > 当成功区分出内存中存活对象和死亡对象后，GC接下来的任务就是执行垃圾回收，释放掉无用对象所占用的内存空间，以便有足够的可用内存空间为新对象分配内存。
  >
  > 
  >
  > 目前在JVM中比较常见的三种垃圾收集算法是标记一**清除算法（Mark-Sweep）、复制算法（copying）、标记-压缩算法（Mark-Compact）**

  当堆中的有效内存空间（available memory）被耗尽的时候，就会停止整个程序（也被称为stop the world），然后进行两项工作，第一项则是标记，第二项则是清除

  -  标记：Collector从引用根节点开始遍历，标记所有被引用的对象。一般是在对象的Header中记录为可达对象。 

  -  清除：Collector对堆内存从头到尾进行线性的遍历，如果发现某个对象在其Header中没有标记为可达对象，则将其回收

  ##### 缺点

  - 标记清除算法的效率不算高

  - 在进行GC的时候，需要停止整个应用程序，用户体验较差

  - 这种方式清理出来的空闲内存是不连续的，产生内碎片，需要维护一个空闲列表

  > 何为清除？

  这里所谓的清除并不是真的置空，而是把需要清除的对象地址保存在空闲的地址列表里。下次有新对象需要加载时，判断垃圾的位置空间是否够，如果够，就存放覆盖原有的地址。

* #### 清除阶段：复制算法

  ##### 核心思想

  将活着的内存空间分为两块，每次只使用其中一块，在垃圾回收时将正在使用的内存中的存活对象复制到未被使用的内存块中，之后清除正在使用的内存块中的所有对象，交换两个内存的角色，最后完成垃圾回收

  ##### 优点

  - 没有标记和清除过程，实现简单，运行高效

  - 复制过去以后保证空间的连续性，不会出现“碎片”问题。

  ##### 缺点

  - 此算法的缺点也是很明显的，就是需要两倍的内存空间。

  - 对于G1这种分拆成为大量region的GC，复制而不是移动，意味着GC需要维护region之间对象引用关系，不管是内存占用或者时间开销也不小

* #### 清除阶段：标记-压缩（整理）算法

  1.  第一阶段和标记清除算法一样，从根节点开始标记所有被引用对象 
  2. 第二阶段将所有的存活对象压缩到内存的一端，按顺序排放。 
  3. 之后，清理边界外所有的空间。 

  标记-压缩算法的最终效果等同于标记-清除算法执行完成后，再进行一次内存碎片整理，因此，也可以把它称为标记-清除-压缩（Mark-Sweep-Compact）算法。

  

  二者的本质差异在于标记-清除算法是一种非移动式的回收算法，标记-压缩是移动式的。是否移动回收后的存活对象是一项优缺点并存的风险决策。可以看到，标记的存活对象将会被整理，按照内存地址依次排列，而未被标记的内存会被清理掉。如此一来，当我们需要给新对象分配内存时，JVM只需要持有一个内存的起始地址即可，这比维护一个空闲列表显然少了许多开销。

  

  #### 指针碰撞（Bump the Pointer）

  

  如果内存空间以规整和有序的方式分布，即已用和未用的内存都各自一边，彼此之间维系着一个记录下一次分配起始点的标记指针，当为新对象分配内存时，只需要通过修改指针的偏移量将新对象分配在第一个空闲内存位置上，这种分配方式就叫做指针碰撞（Bump tHe Pointer）。

  ##### 优点

  - 消除了标记-清除算法当中，内存区域分散的缺点，我们需要给新对象分配内存时，JVM只需要持有一个内存的起始地址即可。

  - 消除了复制算法当中，内存减半的高额代价。

  ##### 缺点

  - 从效率上来说，标记-整理算法要低于复制算法。

  - 移动对象的同时，如果对象被其他对象引用，则还需要调整引用的地址

  - 移动过程中，需要全程暂停用户应用程序。即：STW

  | Mark-Sweep         | Mark-Compact     | Copying                               |
  | ------------------ | ---------------- | ------------------------------------- |
  | 中等               | 最慢             | 最快                                  |
  | 少（但会堆积碎片） | 少（不堆积碎片） | 通常需要活对象的2倍空间（不堆积碎片） |
  | 否                 | 是               | 是                                    |

* #### 分代收集算法

  前面所有这些算法中，并没有一种算法可以完全替代其他算法，它们都具有自己独特的优势和特点。分代收集算法应运而生。

  分代收集算法，是基于这样一个事实：不同的对象的生命周期是不一样的。因此，不同生命周期的对象可以采取不同的收集方式，以便提高回收效率。一般是把Java堆分为新生代和老年代，这样就可以根据各个年代的特点使用不同的回收算法，以提高垃圾回收的效率。

  在Java程序运行的过程中，会产生大量的对象，其中有些对象是与业务信息相关，比如Http请求中的Session对象、线程、Socket连接，这类对象跟业务直接挂钩，因此生命周期比较长。但是还有一些对象，主要是程序运行过程中生成的临时变量，这些对象生命周期会比较短，比如：String对象，由于其不变类的特性，系统会产生大量的这些对象，有些对象甚至只用一次即可回收。

  目前几乎所有的GC都采用分代手机算法执行垃圾回收的。

  在HotSpot中，基于分代的概念，GC所使用的内存回收算法必须结合年轻代和老年代各自的特点。

  ##### 年轻代（Young Gen）

  年轻代特点：区域相对老年代较小，对象生命周期短、存活率低，回收频繁。

  

  这种情况复制算法的回收整理，速度是最快的。复制算法的效率只和当前存活对象大小有关，因此很适用于年轻代的回收。而复制算法内存利用率不高的问题，通过hotspot中的两个survivor的设计得到缓解。

  ##### 老年代（Tenured Gen）

  老年代特点：区域较大，对象生命周期长、存活率高，回收不及年轻代频繁。

  

  这种情况存在大量存活率高的对象，复制算法明显变得不合适。一般是由标记-清除或者是标记-清除与标记-整理的混合实现。

* #### 增量收集算法、分区算法

  ##### 增量收集算法

  如果一次性将所有的垃圾进行处理，需要造成系统长时间的停顿，那么就可以让垃圾收集线程和应用程序线程交替执行。**每次，垃圾收集线程只收集一小片区域的内存空间，接着切换到应用程序线程。依次反复，直到垃圾收集完成。**

  

  总的来说，增量收集算法的基础仍是传统的标记-清除和复制算法。增量收集算法通过对线程间冲突的妥善处理，允许垃圾收集线程以分阶段的方式完成标记、清理或复制工作

  ##### 缺点

  使用这种方式，由于在垃圾回收过程中，间断性地还执行了应用程序代码，所以能减少系统的停顿时间。但是，因为线程切换和上下文转换的消耗，会使得垃圾回收的总体成本上升，造成系统吞吐量的下降。

  

  ##### 分区算法

  一般来说，在相同条件下，堆空间越大，一次Gc时所需要的时间就越长，有关GC产生的停顿也越长。为了更好地控制GC产生的停顿时间，将一块大的内存区域分割成多个小块，根据目标的停顿时间，每次合理地回收若干个小区间，而不是整个堆空间，从而减少一次GC所产生的停顿。

  

  分代算法将按照对象的生命周期长短划分成两个部分，分区算法将整个堆空间划分成连续的不同小区间。

  

  每一个小区间都独立使用，独立回收。这种算法的好处是可以控制一次回收多少个小区间。

  ![](image\屏幕截图 2024-01-14 151112.png)

  



### 垃圾回收相关概念

Reference子类中只有终结器引用是包内可见的，其他3种引用类型均为public，可以在应用程序中直接使用



- **强引用**（StrongReference）：最传统的“引用”的定义，是指在程序代码之中普遍存在的引用赋值，即类似“`Object obj = new Object()`”这种引用关系。无论任何情况下，只要强引用关系还存在，垃圾收集器就永远不会回收掉被引用的对象。

- **软引用**（SoftReference）：**在系统将要发生内存溢出之前，将会把这些对象列入回收范围之中进行第二次回收。如果这次回收后还没有足够的内存，才会抛出内存流出异常**。

- **弱引用**（WeakReference）：被弱引用关联的对象只能生存到下一次垃圾收集之前。当垃圾收集器工作时，无论内存空间是否足够，都会回收掉被弱引用关联的对象。

  > **面试题：你开发中使用过WeakHashMap吗？**

  WeakHashMap用来存储图片信息，可以在内存不足的时候，及时回收，避免了OOM

  

- **虚引用**（PhantomReference）：一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来获得一个对象的实例。为一个对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知。



### 垃圾回收器

* #### 垃圾回收器发展史

  有了虚拟机，就一定需要收集垃圾的机制，这就是Garbage Collection，对应的产品我们称为Garbage Collector。

  

  - 1999年随JDK1.3.1一起来的是串行方式的serialGc，它是第一款GC。ParNew垃圾收集器是Serial收集器的多线程版本

  - 2002年2月26日，Parallel GC和Concurrent Mark Sweep GC跟随JDK1.4.2一起发布·

  - Parallel GC在JDK6之后成为HotSpot默认GC。

  - 2012年，在JDK1.7u4版本中，G1可用。

  - 2017年，JDK9中G1变成默认的垃圾收集器，以替代CMS。

  - 2018年3月，JDK10中G1垃圾回收器的并行完整垃圾回收，实现并行性来改善最坏情况下的延迟。

  - 2018年9月，JDK11发布。引入Epsilon 垃圾回收器，又被称为 "No-Op(无操作)“ 回收器。同时，引入ZGC：可伸缩的低延迟垃圾回收器（Experimental）

  - 2019年3月，JDK12发布。增强G1，自动返回未用堆内存给操作系统。同时，引入Shenandoah GC：低停顿时间的GC（Experimental）。·

  - 2019年9月，JDK13发布。增强ZGC，自动返回未用堆内存给操作系统。

  - 2020年3月，JDK14发布。删除CMS垃圾回收器。扩展ZGC在macos和Windows上的应用

* #### 7种经典的垃圾回收器

  * 串行回收器：Serial、Serial Old（Serial 串行）
  * 并行回收器：ParNew、Parallel Scavenge、Parallel old（parallel [ˈpærəˌlel]平行）
  * 并发回收器：CMS、G1

* #### 7中垃圾回收器与垃圾分代的关系

  ![](image\屏幕截图 2024-01-14 225951.png)

* #### Serial 回收器：串行回收

  Serial收集器作为HotSpot中client模式下的默认新生代垃圾收集器。

  

  **Serial收集器采用复制算法、串行回收和"stop-the-World"机制的方式执行内存回收**。

  

  除了年轻代之外，Serial收集器还提供用于执行老年代垃圾收集的Serial Old收集器。**Serial Old收集器同样也采用了串行回收和"Stop the World"机制，只不过内存回收算法使用的是标记-压缩算法**。

  

  - Serial old是运行在Client模式下默认的老年代的垃圾回收器

  - Serial 0ld在Server模式下主要有两个用途：① 与新生代的Parallel scavenge配合使用 ② 作为老年代CMS收集器的后备垃圾收集方案

* #### ParNew 回收器：并行回收

  如果说Serial GC是年轻代中的单线程垃圾收集器，那么ParNew收集器则是Serial收集器的多线程版本。Par是Parallel的缩写，New：只能处理的是新生代

  

  **ParNew 收集器除了采用并行回收的方式执行内存回收外，两款垃圾收集器之间几乎没有任何区别。ParNew收集器在年轻代中同样也是采用复制算法、"Stop-the-World"机制。**

  

  ParNew 是很多JVM运行在Server模式下新生代的默认垃圾收集器。

  - 对于新生代，回收次数频繁，使用并行方式高效。

  - 对于老年代，回收次数少，使用串行方式节省资源。（CPU并行需要切换线程，串行可以省去切换线程的资源）

  **在单个CPU的环境下，ParNew收集器不比Serial 收集器更高效**

  

* #### Parallel回收器：吞吐量优先

  HotSpot的年轻代中除了拥有ParNew收集器是基于并行回收的以外，**Parallel Scavenge收集器同样也采用了复制算法、并行回收和"Stop the World"机制。**

  

  那么Parallel 收集器的出现是否多此一举？

  

  - **和ParNew收集器不同，ParallelScavenge收集器的目标则是达到一个可控制的吞吐量（Throughput），它也被称为吞吐量优先的垃圾收集器**。

    可以设置年轻代并行收集的线程数，设置垃圾收集器最大停顿时间。

  - **自适应调节策略**也是Parallel Scavenge与ParNew一个重要区别。

    > - - 在这种模式下，年轻代的大小、Eden和Survivor的比例、晋升老年代的对象年龄等参数会被自动调整，已达到在堆大小、吞吐量和停顿时间之间的平衡点。
    >
    > - - 在手动调优比较困难的场合，可以直接使用这种自适应的方式，仅指定虚拟机的最大堆、目标的吞吐量（`GCTimeRatio`）和停顿时间（`MaxGCPauseMills`），让虚拟机自己完成调优工作。

  

  高吞吐量则可以高效率地利用CPU时间，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的任务。因此，常见在服务器环境中使用。**例如，那些执行批量处理、订单处理、工资支付、科学计算的应用程序。**

  

  Parallel 收集器在JDK1.6时提供了用于执行老年代垃圾收集的Parallel Old收集器，用来代替老年代的Serial Old收集器。

  

  **Parallel Old收集器采用了标记-压缩算法，但同样也是基于并行回收和"Stop-the-World"机制。**



* #### CMS回收器：低延迟

  > 在JDK1.5时期，Hotspot推出了一款在强交互应用中几乎可认为有划时代意义的垃圾收集器：CMS（Concurrent-Mark-Sweep）收集器，**这款收集器是HotSpot虚拟机中第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程同时工作。**
  >
  > CMS收集器的关注点是尽可能缩短垃圾收集时用户线程的停顿时间。停顿时间越短（低延迟）就越适合与用户交互的程序，良好的响应速度能提升用户体验。

  **CMS的垃圾收集算法采用标记-清除算法，并且也会"Stop-the-World"**

  

  不幸的是，CMS作为老年代的收集器，却无法与JDK1.4.0中已经存在的新生代收集器Parallel Scavenge配合工作，所以在JDK1.5中使用CMS来收集老年代的时候，新生代只能选择ParNew或者Serial收集器中的一个。

  在G1出现之前，CMS使用还是非常广泛的。一直到今天，仍然有很多系统使用CMS GC。

  ##### CMS执行过程：

  CMS整个过程比之前的收集器要复杂，整个过程分为4个主要阶段，即初始标记阶段、并发标记阶段、重新标记阶段和并发清除阶段

  - **初始标记**（Initial-Mark）阶段：在这个阶段中，程序中所有的工作线程都将会因为“Stop-the-World”机制而出现短暂的暂停，**这个阶段的主要任务仅仅只是标记出GCRoots能直接关联到的对象**。一旦标记完成之后就会恢复之前被暂停的所有应用线程。由于直接关联对象比较小，所以这里的速度非常快。

  - **并发标记**（Concurrent-Mark）阶段：从GC Roots的直接关联对象开始遍历整个对象图的过程，这个过程耗时较长但是不需要停顿用户线程，可以与垃圾收集线程一起并发运行。

  - **重新标记**（Remark）阶段：由于在并发标记阶段中，程序的工作线程会和垃圾收集线程同时运行或者交叉运行，因此为了**修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录**，这个阶段的停顿时间通常会比初始标记阶段稍长一些，但也远比并发标记阶段的时间短。

  - **并发清除**（Concurrent-Sweep）阶段：此阶段清理删除掉标记阶段判断的已经死亡的对象，释放内存空间。由于不需要移动存活对象，所以这个阶段也是可以与用户线程同时并发的

  

  尽管CMS收集器采用的是并发回收（非独占式），但是在其初始化标记和再次标记这两个阶段中仍然需要执行“Stop-the-World”机制暂停程序中的工作线程，不过暂停时间并不会太长，因此可以说明目前所有的垃圾收集器都做不到完全不需要“stop-the-World”，只是尽可能地缩短暂停时间。

  由于最耗费时间的并发标记与并发清除阶段都不需要暂停工作，所以整体的回收是低停顿的。

  ![](image\屏幕截图 2024-01-14 231517.png)

  **注意：由于在垃圾收集阶段用户线程没有中断，所以在CMS回收过程中，还应该确保程序用户线程有足够的内存可用。** CMS收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集，而是**当堆内存使用率达到某一阈值时，便开始进行回收**

  ##### CMS的弊端：

  - **会产生内存碎片**，导致并发清除后，用户线程可用的空间不足。在无法分配大对象的情况下，不得不提前触发FullGC。

  - **CMS收集器对CPU资源非常敏感**。在并发阶段，它虽然不会导致用户停顿，但是会因为占用了一部分线程而导致应用程序变慢，总吞吐量会降低。

  - **CMS收集器无法处理浮动垃圾**。可能出现“`Concurrent Mode Failure`"失败而导致另一次Full GC的产生。在并发标记阶段由于程序的工作线程和垃圾收集线程是同时运行或者交叉运行的，那么在并发标记阶段如果产生新的垃圾对象，CMS将无法对这些垃圾对象进行标记，最终会导致这些新产生的垃圾对象没有被及时回收，从而只能在下一次执行GC时释放这些之前未被回收的内存空间。

  

* #### G1回收器：区域化分代式

  > **既然我们已经有了前面几个强大的GC，为什么还要发布Garbage First（G1）？**

  原因就在于应用程序所应对的业务越来越庞大、复杂，用户越来越多，没有GC就不能保证应用程序正常进行，而经常造成STW的GC又跟不上实际的需求，所以才会不断地尝试对GC进行优化。**G1（Garbage-First）**垃圾回收器是在Java7 update4之后引入的一个新的垃圾回收器，是当今收集器技术发展的最前沿成果之一。

  与此同时，为了适应现在不断扩大的内存和不断增加的处理器数量，进一步降低暂停时间（pause time），同时兼顾良好的吞吐量。

  官方给G1设定的目标是在延迟可控的情况下获得尽可能高的吞吐量，所以才担当起“全功能收集器”的重任与期望。

  > 为什么名字叫 Garbage First（G1）呢？

  因为G1是一个并行回收器，它把堆内存分割为很多不相关的区域（Region）（物理上不连续的）。使用不同的Region来表示Eden、幸存者0区，幸存者1区，老年代等。

  **G1 GC有计划地避免在整个Java堆中进行全区域的垃圾收集。G1跟踪各个Region里面的垃圾堆积的价值大小（回收所获得的空间大小以及回收所需时间的经验值），在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的Region**。

  由于这种方式的侧重点在于回收垃圾最大量的区间（Region），所以我们给G1一个名字：垃圾优先（Garbage First）。

  G1（Garbage-First）是一款面向服务端应用的垃圾收集器，主要针对配备多核CPU及大容量内存的机器，以极高概率满足GC停顿时间的同时，还兼具高吞吐量的性能特征。

  在JDK1.7版本正式启用，移除了Experimenta1的标识，是JDK9以后的默认垃圾回收器，取代了CMS回收器以及Parallel+Parallel Old组合。被Oracle官方称为“全功能的垃圾收集器”。

  与此同时，CMS已经在JDK9中被标记为废弃（deprecated）。在jdk8中还不是默认的垃圾回收器，需要使用`-XX:+UseG1GC`来启用。

  ##### G1 回收器的优势

  与其他GC收集器相比，G1使用了全新的分区算法，其特点如下所示：

  ##### 并行与并发

  - 并行性：G1在回收期间，可以有多个GC线程同时工作，有效利用多核计算能力。此时用户线程STW

  - 并发性：G1拥有与应用程序交替执行的能力，部分工作可以和应用程序同时执行，因此，一般来说，不会在整个回收阶段发生完全阻塞应用程序的情况

  ##### 分代收集

  - 从分代上看，G1依然属于分代型垃圾回收器，它会区分年轻代和老年代，年轻代依然有Eden区和Survivor区。但从堆的结构上看，它不要求整个Eden区、年轻代或者老年代都是连续的，也不再坚持固定大小和固定数量。

  - 将**堆空间分为若干个区域（Region），这些区域中包含了逻辑上的年轻代和老年代。**

  - 和之前的各类回收器不同，它同时兼顾年轻代和老年代。对比其他回收器，或者工作在年轻代，或者工作在老年代；

  ![](image\屏幕截图 2024-01-14 233658.png)

* #### 空间整合

  - CMS：“标记-清除”算法、内存碎片、若干次Gc后进行一次碎片整理

  - G1将内存划分为一个个的region。内存的回收是以region作为基本单位的。**Region之间是复制算法，但整体上实际可看作是标记-压缩（Mark-Compact）算法，两种算法都可以避免内存碎片**。这种特性有利于程序长时间运行，分配大对象时不会因为无法找到连续内存空间而提前触发下一次GC。**尤其是当Java堆非常大的时候，G1的优势更加明显。**

* #### 分区Region：化整为零

  所有的Region大小相同，且在JVM生命周期内不会被改变。

  G1垃圾收集器还增加了一种新的内存区域，叫做Humongous内存区域，如图中的H块。主要用于存储大对象，如果超过1.5个region，就放到H。

  

  设置H的原因：对于堆中的对象，默认直接会被分配到老年代，但是如果它是一个短期存在的大对象就会对垃圾收集器造成负面影响。为了解决这个问题，G1划分了一个Humongous区，它用来专门存放大对象。如果一个H区装不下一个大对象，那么G1会寻找连续的H区来存储。为了能找到连续的H区，有时候不得不启动Full GC。G1的大多数行为都把H区作为老年代的一部分来看待。

  每个Region都是通过指针碰撞来分配空间

* #### G1 垃圾回收器的回收过程

  G1GC的垃圾回收过程主要包括如下三个环节：

  -  年轻代GC（Young GC） 

  -  老年代并发标记过程（Concurrent Marking） 

  -  混合回收（Mixed GC）
    （如果需要，单线程、独占式、高强度的Full GC还是继续存在的。它针对GC的评估失败提供了一种失败保护机制，即强力回收。） 

  ![](image\屏幕截图 2024-01-14 235504.png)

  

  应用程序分配内存，**当年轻代的Eden区用尽时开始年轻代回收过程**；G1的年轻代收集阶段是一个并行的独占式收集器。在年轻代回收期，**G1GC暂停所有应用程序线程**，启动多线程执行年轻代回收。**然后从年轻代区间移动存活对象到Survivor区间或者老年区间，也有可能是两个区间都会涉及。**

  

  当堆内存使用达到一定值（默认45%）时，开始**老年代并发标记过程。**

  

  **标记完成马上开始混合回收过程**。对于一个混合回收期，G1 GC从老年区间移动存活对象到空闲区间，这些空闲区间也就成为了老年代的一部分。和年轻代不同，老年代的G1回收器和其他GC不同，**G1的老年代回收器不需要整个老年代被回收，一次只需要扫描/回收一小部分老年代的Region就可以了**。同时，这个老年代Region是和年轻代一起被回收的。

  

  举个例子：一个Web服务器，Java进程最大堆内存为4G，每分钟响应1500个请求，每45秒钟会新分配大约2G的内存。G1会每45秒钟进行一次年轻代回收，每31个小时整个堆的使用率会达到45%，会开始老年代并发标记过程，标记完成后开始四到五次的混合回收。

* #### Remembered Set

  -  一个对象被不同区域引用的问题 

  -  一个Region不可能是孤立的，一个Region中的对象可能被其他任意Region中对象引用，判断对象存活时，是否需要扫描整个Java堆才能保证准确？ 

  -  在其他的分代收集器，也存在这样的问题（而G1更突出）回收新生代也不得不同时扫描老年代？ 

  -  这样的话会降低MinorGC的效率； 

  ##### 解决方法：

  无论G1还是其他分代收集器，JVM都是使用Remembered Set来避免全局扫描：

  **每个Region都有一个对应的Remembered Set；**每次Reference类型数据写操作时，都会产生一个Write Barrier暂时中断操作；

  然后检查将要**写入的引用指向的对象**是否和该**Reference类型数据**在不同的Region（其他收集器：检查老年代对象是否引用了新生代对象）；

  如果不同，通过CardTable把相关引用信息记录到引用指向对象的所在Region对应的Remembered Set中；

  当进行垃圾收集时，在GC根节点的枚举范围加入Remembered Set；就可以保证不进行全局扫描，也不会有遗漏。

* #### G1 回收过程一：年轻代GC

  > JVM启动时，G1先准备好Eden区，程序在运行过程中不断创建对象到Eden区，当Eden空间耗尽时，G1会启动一次年轻代垃圾回收过程。
  >
  > 年轻代垃圾回收只会回收Eden区和Survivor区。
  >
  > 首先G1停止应用程序的执行（Stop-The-World），G1创建回收集（Collection Set），回收集是指需要被回收的内存分段的集合，年轻代回收过程的回收集包含年轻代Eden区和Survivor区所有的内存分段。

  1. **第一阶段，扫描根**。根是指static变量指向的对象，正在执行的方法调用链条上的局部变量等。根引用连同RSet记录的外部引用作为扫描存活对象的入口。
  2. **第二阶段，更新RSet**。处理dirty card queue（见备注）中的card，更新RSet。此阶段完成后，RSet可以准确的反映老年代对所在的内存分段中对象的引用。
  3. **第三阶段，处理RSet**。识别被老年代对象指向的Eden中的对象，这些被指向的Eden中的对象被认为是存活的对象。
  4. **第四阶段，复制对象**。此阶段，对象树被遍历，Eden区内存段中存活的对象会被复制到Survivor区中空的内存分段，Survivor区内存段中存活的对象如果年龄未达阈值，年龄会加1，达到阀值会被会被复制到Old区中空的内存分段。如果Survivor空间不够，Eden空间的部分数据会直接晋升到老年代空间。
  5. **第五阶段，处理引用**。处理Soft，Weak，Phantom，Final，JNI Weak 等引用。最终Eden空间的数据为空，GC停止工作，而目标内存中的对象都是连续存储的，没有碎片，所以复制过程可以达到内存整理的效果，减少碎片。

* #### G1 回收过程二：并发标记过程

  1. **初始标记阶段**：标记从根节点直接可达的对象。这个阶段是STW的，并且会触发一次年轻代GC。

  2. **根区域扫描（Root Region Scanning）**：G1 GC扫描Survivor区直接可达的老年代区域对象，并标记被引用的对象。这一过程必须在YoungGC之前完成。

  3. **并发标记（Concurrent Marking）**：在整个堆中进行并发标记（和应用程序并发执行），此过程可能被YoungGC中断。在并发标记阶段，若发现区域对象中的所有对象都是垃圾，那这个区域会被立即回收。同时，并发标记过程中，会计算每个区域的对象活性（区域中存活对象的比例）。

  4. **再次标记（Remark**）：由于应用程序持续进行，需要修正上一次的标记结果。是STW的。G1中采用了比CMS更快的初始快照算法：snapshot-at-the-beginning（SATB）。

  5. **独占清理（cleanup，STW）**：计算各个区域的存活对象和GC回收比例，并进行排序，识别可以混合回收的区域。为下阶段做铺垫。是STW的。这个阶段并不会实际上去做垃圾的收集

  6. **并发清理阶段**：识别并清理完全空闲的区域。

     

* #### G1 回收过程三：混合回收

  当越来越多的对象晋升到老年代o1d region时，为了避免堆内存被耗尽，虚拟机会触发一个混合的垃圾收集器，即Mixed GC，该算法并不是一个Old GC，除了回收整个Young Region，还会回收一部分的Old Region。**这里需要注意：是一部分老年代，而不是全部老年代。**可以选择哪些Old Region进行收集，从而可以对垃圾回收的耗时时间进行控制。也要注意的是Mixed GC并不是Full GC。



## 第三章 字节码与类的加载篇



### 类的加载过程（类的生命周期）

在Java中数据类型分为基本数据类型和引用数据类型。基本数据类型由虚拟机预先定义，引用数据类型则需要进行类的加载。



按照Java虚拟机规范，从class文件到加载到内存中的类，到类卸载出内存为止，它的整个生命周期包括如下7个阶段：

![](image\屏幕截图 2024-01-15 171136.png)



#### 过程一：Loading（加载阶段）

> 所谓加载，简而言之就是将Java类的字节码文件加载到机器内存中，并在内存中构建出Java类的原型——类模板对象

在加载类时，Java虚拟机必须完成以下3件事情：

加载阶段，简言之，查找并加载类的二进制数据，生成Class文件的实例

-  通过类的全名，获取类的二进制数据流。 

-  解析类的二进制数据流为方法区内的数据结构（Java类模型） 

-  创建java.lang.Class类的实例，表示该类型。作为方法区这个类的各种数据的访问入口 





* #### 3.2.2. 二进制流的获取方式

  对于类的二进制数据流，虚拟机可以通过多种途径产生或获得。（只要所读取的字节码符合JVM规范即可）

  - 虚拟机可能通过文件系统读入一个class后缀的文件**（最常见的）**

  - 读入jar、zip等归档数据包，提取类文件。

  - 事先存放在数据库中的类的二进制数据

  - 使用类似于HTTP之类的协议通过网络进行加载

  - 在运行时生成一段class的二进制信息等

  - 在获取到类的二进制信息后，Java虚拟机就会处理这些数据，并最终转为一个java.lang.Class的实例。

  如果输入数据不是ClassFile的结构，则会抛出ClassFormatError。

  

* #### 类模型与Class实例的位置

  **类模型的位置**

  

  加载的类在JVM中创建相应的类结构，类结构会存储在方法区（JDKl.8之前：永久代；J0Kl.8及之后：元空间）。

  

  **Class实例的位置**

  

  类将.class文件加载至元空间后，会在堆中创建一个Java.lang.Class对象，用来封装类位于方法区内的数据结构，该Class对象是在加载类的过程中创建的，每个类都对应有一个Class类型的对象。

  

  

  

#### 过程二：Linking（链接）阶段



* #### 环节一：Verification（验证）

  ![](image\屏幕截图 2024-01-15 171941.png)

* #### 环节2：Preparation（准备）

  > 准备阶段（Preparation），简言之，为类的静态变分配内存，并将其初始化为默认值。

  这里不包括基本数据类型的字段用`static final`修饰的情况，因为`final`在编译的时候就分配了，准备阶段显式赋值。

* #### 环节3：Resolution（解析）

  在准备阶段完成后，就进入了解析阶段。解析阶段（Resolution），简言之，将类、接口、字段和方法的符号引用转为直接引用。

  **具体描述**：

  符号引用就是一些字面量的引用，和虚拟机的内部数据结构和和内存布局无关。比较容易理解的就是在Class类文件中，通过常量池进行了大量的符号引用。但是在程序实际运行时，只有符号引用是不够的，比如当如下println()方法被调用时，系统需要明确知道该方法的位置

  **总结：通过解析操作，符合应用就可以转变为目标方法在类中方法表中的位置，从而使得方法被成功调用**



#### 过程三：Initialization（初始化）阶段

**说明**：使用static+ final修饰的字段的显式赋值的操作，到底是在哪个阶段进行的赋值？

-  情况1：在链接阶段的准备环节赋值 

-  情况2：在初始化阶段<clinit>()中赋值 

**结论**： 在链接阶段的准备环节赋值的情况：

-  对于基本数据类型的字段来说，如果使用static final修饰，则显式赋值(直接赋值常量，而非调用方法通常是在链接阶段的准备环节进行 

-  对于String来说，如果使用字面量的方式赋值，使用static final修饰的话，则显式赋值通常是在链接阶段的准备环节进行 

-  在初始化阶段<clinit>()中赋值的情况： 排除上述的在准备环节赋值的情况之外的情况。 

**最终结论**：使用static+final修饰，且显式赋值中不涉及到方法或构造器调用的基本数据类到或String类型的显式赋值，是在链接阶段的准备环节进行。



* #### < clinit >()的线程安全性

  对于`< clinit >()`方法的调用，也就是类的初始化，虚拟机会在内部确保其多线程环境中的安全性。

  虚拟机会保证一个类的()方法在多线程环境中被正确地加锁、同步，如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的`<clinit>()`方法，其他线程都需要阻塞等待，直到活动线程执行`<clinit>()`方法完毕。

  正是因为它是带线程安全的，因此，如果在一个类的`<clinit>()`方法中有耗时很长的操作，就可能造成多个线程阻塞，引发死锁。并且这种死锁是很难发现的，因为看起来它们并没有可用的锁信息。

  如果之前的线程成功加载了类，则等在队列中的线程就没有机会再执行`<clinit>()`方法了。那么，当需要使用这个类时，虚拟机会直接返回给它已经准备好的信息。

* #### 类的初始化情况：主动使用vs被动使用

  **主动使用：**

  1.  实例化：当创建一个类的实例时，比如使用new关键字，或者通过反射、克隆、反序列化。 
  2. 静态方法：当调用类的静态方法时，即当使用了字节码invokestatic指令。 
  3. 静态字段：当使用类、接口的静态字段时（final修饰特殊考虑），比如，使用getstatic或者putstatic指令。（对应访问变量、赋值变量操作） 
  4.  反射：当使用java.lang.reflect包中的方法反射类的方法时。比如：Class.forName("com.atguigu.java.Test") 
  5.  继承：当初始化子类时，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。 
  6. default方法：如果一个接口定义了default方法，那么直接实现或者间接实现该接口的类的初始化，该接口要在其之前被初始化。 
  7.  main方法：当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类。 
  8.  MethodHandle：当初次调用MethodHandle实例时，初始化该MethodHandle指向的方法所在的类。（涉及解析REF getStatic、REF_putStatic、REF invokeStatic方法句柄对应的类） 

#### 

#### 过程四：类的Using（使用）

任何一个类型在使用之前都必须经历过完整的加载、链接和初始化3个类加载步骤。一旦一个类型成功经历过这3个步骤之后，便“厉事俱备只欠东风”，就等着开发者使用了。



开发人员可以在程序中访问和调用它的静态类成员信息（比如：静态字段、静态方法），或者使用new关键字为其创建对象实例。



#### 过程五：类的Unloading（卸载）

* ##### 类、类的加载器、类的实例之间的引用关系

  在类加载器的内部实现中，用一个Java集合来存放所加载类的引用。另一方面，一个Class对象总是会引用它的类加载器，调用Class对象的getClassLoader()方法，就能获得它的类加载器。由此可见，代表某个类的Class实例与其类的加载器之间为双向关联关系。

  一个类的实例总是引用代表这个类的Class对象。在Object类中定义了getClass()方法，这个方法返回代表对象所属类的Class对象的引用。此外，所有的java类都有一个静态属性class，它引用代表这个类的Class对象。

* ##### 类的生命周期

  当Sample类被加载、链接和初始化后，它的生命周期就开始了。当代表Sample类的Class对象不再被引用，即不可触及时，Class对象就会结束生命周期，Sample类在方法区内的数据也会被卸载，从而结束Sample类的生命周期。























