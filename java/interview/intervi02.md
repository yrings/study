# intervi02

## 35-AQS详解

> AQS,**AbstractQueuedSynchronizer**,AQS内部是使用了**双向链表**将等待线程链接起来，当发生并发竞争的时候，就会初始化该队列并让线程进入睡眠等待唤醒，同时每个节点会根据是否为共享锁标记状态为共享模式或独占模式。
>
> `AbstractQueuedSynchronizer(AQS)`提供了一套可用于实现锁同步机制的框架，不夸张地说，**`AQS`是`JUC`同步框架的基石。`AQS`通过一个`FIFO`队列维护线程同步状态，实现类只需要继承该类，并重写指定方法即可实现一套线程同步机制。**
>
> `AQS`根据资源互斥级别提供了**独占和共享**两种资源访问模式；同时其定义`Condition`结构提供了`wait/signal`等待唤醒机制。在`JUC`中，诸如`ReentrantLock`、`CountDownLatch`等都基于`AQS`实现。

* ### AQS框架

  #### AQS原理

  `AQS`的原理并不复杂，`AQS`维护了一个`volatile int state`变量和一个`CLH(三个人名缩写)双向队列`，队列中的节点持有线程引用，每个节点均可通过`getState()`、`setState()`和`compareAndSetState()`对`state`进行修改和访问。

  ![](image\屏幕截图 2024-01-17 151826.png)

  当线程获取锁时，即试图对`state`变量做修改，如修改成功则获取锁；如修改失败则包装为节点挂载到队列中，等待持有锁的线程释放锁并唤醒队列中的节点

  ### 核心结构

  #### Node

  `Node`主要包含5个核心字段：

  - waitStatus：当前节点状态，该字段共有5种取值：
    - `CANCELLED = 1`。节点引用线程由于等待超时或被打断时的状态。
    - `SIGNAL = -1`。后继节点线程需要被唤醒时的当前节点状态。当队列中加入后继节点被挂起`(block)`时，其前驱节点会被设置为`SIGNAL`状态，表示该节点需要被唤醒。
    - `CONDITION = -2`。当节点线程进入`condition`队列时的状态。(见`ConditionObject`)
    - `PROPAGATE = -3`。仅在释放共享锁`releaseShared`时对头节点使用。(见共享锁分析)
    - `0`。节点初始化时的状态。
  - `prev`：前驱节点。
  - `next`：后继节点。
  - `thread`：引用线程，头节点不包含线程。
  - `nextWaiter`：`condition`条件队列。(见`ConditionObject`)

  #### 独占锁分析

  `acquire`核心为`tryAcquire`、`addWaiter`和`acquireQueued`三个函数，其中`tryAcquire`需具体类实现。 每当线程调用`acquire`时都首先会调用`tryAcquire`，失败后才会挂载到队列，因此`acquire`实现**默认为非公平锁**。

  `addWaiter`将线程包装为独占节点，尾插式加入到队列中，如队列为空，则会添加一个空的头节点。**值得注意的是`addWaiter`中的`enq`方法，通过`CAS+自旋`的方式处理尾节点添加冲突。**

  ![](image\屏幕截图 2024-01-17 153200.png)

  

  `acquireQueue`在线程节点加入队列后判断是否可再次尝试获取资源，如不能获取则将其前驱节点标志为`SIGNAL`状态(表示其需要被`unpark`唤醒)后，则通过`park`进入阻塞状态。

  ![](image\屏幕截图 2024-01-17 153352.png)

  参照流程图，`acquireQueued`方法核心逻辑为`for(;;)`和`shouldParkAfterFailedAcquire`。`tail`节点默认初始状态为0，**当新节点被挂载到队列后，将其前驱即原`tail`节点状态设为`SIGNAL`，表示该节点需要被唤醒，返回`true`后即被`park`陷入阻塞。`for`循环直到节点前驱为`head`后才尝试进行资源获取。**

  

  #### 共享锁分析

  `acquireShared`和`releaseShared`整体流程与独占锁类似，`tryAcquireShared`获取失败后以`Node.SHARED`挂载到队尾阻塞，直到队头节点将其唤醒。在`doAcquireShared`与独占锁不同的是，**由于共享锁是可以被多个线程获取的，因此在首个阻塞节点被唤醒后，会通过`setHeadAndPropagate`传递唤醒后续的阻塞节点。**

  ![](image\屏幕截图 2024-01-17 154110.png)

  

  #### 总结

  1、独占锁共享锁默认都是非公平获取策略，可能被插队。

  2、独占锁只有一个线程可获取，其他线程均被阻塞在队列中；共享锁可以有多个线程获取。

  3、独占锁释放仅唤醒一个阻塞节点，共享锁可以根据可用数量，一次唤醒多个阻塞节点

  ### AQS模板方法（使用）

  `AQS`内部封装了队列维护逻辑，采用模版方法的模式提供实现类以下方法：

  ```Java
  tryAcquire(int);        // 尝试获取独占锁，可获取返回true，否则false
  tryRelease(int);        // 尝试释放独占锁，可释放返回true，否则false
  tryAcquireShared(int);  // 尝试以共享方式获取锁，失败返回负数，只能获取一次返回0，否则返回个数
  tryReleaseShared(int);  // 尝试释放共享锁，可获取返回true，否则false
  isHeldExclusively();    // 判断线程是否独占资源
  ```

  如实现类只需实现独占锁/共享锁功能，可只实现`tryAcquire/tryRelease`或`tryAcquireShared/tryReleaseShared`。虽然实现`tryAcquire/tryRelease`可自行设定逻辑，但建议使用`state`方法对`state`变量进行操作以实现同步类。

  如下是一个简单的同步锁实现示例：

  ```Java
  Java复制代码public class Mutex extends AbstractQueuedSynchronizer {
      
      @Override
      public boolean tryAcquire(int arg) {
          return compareAndSetState(0, 1);
      }
      
      @Override
      public boolean tryRelease(int arg) {
          return compareAndSetState(1, 0);
      }
      
      public static void main(String[] args) {
          final Mutex mutex = new Mutex();
          
          new Thread(() -> {
              System.out.println("thread1 acquire mutex");
              mutex.acquire(1);
              // 获取资源后sleep保持
              try {
                  TimeUnit.SECONDS.sleep(5);
              } catch(InterruptedException ignore) {
                  
              }
              mutex.release(1);
              System.out.println("thread1 release mutex");
          }).start();
          
          new Thread(() -> {
              // 保证线程2在线程1启动后执行
              try {
                  TimeUnit.SECONDS.sleep(1);
              } catch(InterruptedException ignore) {
                  
              }
              // 等待线程1 sleep结束释放资源
              mutex.acquire(1);
              System.out.println("thread2 acquire mutex");
              mutex.release(1);
          }).start()
      }
  }
  ```

  示例代码简单通过`AQS`实现一个互斥操作，线程1获取`mutex`后，线程2的`acquire`陷入阻塞，直到线程1释放。其中`tryAcquire/acquire/tryRelease/release`的`arg`参数可按实现逻辑自定义传入值，无具体要求。





## 36-@Autowired和@Resource的区别是什么

* **@Autowired**

  是先根据属性的类型去Spring容器中找Bean对象，如果找到多个，就会根据属性名去确定其中一个，如果根据属性名字没有找到就会报错。

  @Autowired是Spring层面提供的。

* **@Resource**

  会先根据属性名字去Spring容器中找Bean对象，如果没有找到，则会根据属性类型找，如果找到多个就会报错。另外可以利用@Resource指定name，如果配置了这个name属性，如果没有在Spring容器中找到Bean对象，则会直接报错。

  @Resource是JDK层面提供的。
  
  

## 37-volatile是如何保证可见性的

### volatile 详解

* **volatile的作用**

  1. **保证变量的可变性**

     对于多核CPU，它们都有自己的三级缓存空间。当CPU1、CPU2都去读取一个变量a，由于在缓存中没有目标，它们都会到主存中获取值，然后刷新到缓存中。但是如果此时CPU1修改了这个值，只会刷新到自己的缓存和主存中，当另一个CPU2读取时，会读取到缓存中的旧数据。

  2. **保证赋值操作的原子性**

     原子性就是不能被线程调度打断的操作，是线程安全的操作，对于原子性操作，即使在多线程环境下，也不用担心线程安全问题或者数据不一致的问题。有些变量的赋值本身就是原子性的，比如对boolean，对int的赋值，但是像对于long或者double则不一定，如果是32位的处理器，对于64位的变量的操作可能会被分解成为二个步骤：高32位和低32位，由此可能会发生线程切换，从而导致线程不安全。如果变量声明为volatile，那么虚拟机会保证赋值是原子的，是不可被打断的。

  3. **禁止指令重排**

     正常情况下，虚拟机会在不影响程序结果的正确性的前提下，对指令进行重排。上面两个作用中也能体现这个作用。

### volatile保证多线程的可见性

我们先通过一个例子来看看，可见性导致的线程安全问题：

```java
public class Main {

    static int a = 0;

    public static void main(String[] args)  throws Exception {
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                while (a == 0) {

                }
                System.out.println("T1得知a = 1");
            }
        });

        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                    a = 1;
                    System.out.println("T2修改a = 1");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        t1.start();
        t2.start();
    }
}
```

线程T2再休眠1秒之后，修改了a的值为1，此时T1应该退出while循环并打印，但是结果并非如此：



![img](https://pic3.zhimg.com/80/v2-6ab816f00e3f14e19bdcc0e28f828d16_1440w.webp)



T1没有退出循环，程序也就不会结束。但是如果对a使用volatile关键字修饰就会解决该问题。

这个问题的源头就在于**可见性**问题。为什么会出现这种问题呢？这里我们需要从CPU多核缓存架构讲起。

* CPU多核缓存架构

一个双核CPU架构可以如下图所示：



![img](https://pic1.zhimg.com/80/v2-f1d1253d4babc807bc2e44a83e2e2894_1440w.webp)



首先需要明确的一点是，计算机实际上是分为多级缓存的，因为读取缓存的数据性能十分快

1. 当CPU1需要读取共享变量的值a时，首先会找缓存（即L1、L2、L3三级高速缓存），看看这个值是不是在L1。
2. 很明显，缓存没办法给CPU1它想要的数据，于是只能去主内存读取共享变量的值
3. 缓存得到共享变量的值之后，把数据交给寄存器，但是缓存留了个心眼，它把a的值存了起来，这样下次别的线程再需要a的值时，就不用再去主内存问了

至此，一次完整的数据访问流程走完了。L1和L2、L3都是高速缓存，从高速缓存和主内存读取数据的速度完全是两个概念。所以才会有主内存和缓存的设计。

* **写数据时刷新内存**

针对上述模型，当CPU1读取完数据后，假如对数据进行了修改，那么它会将缓存 —> 主内存的顺序将修改后的数据刷新一遍，完成对数据的更新。

从读到写这一整个流程看起来似乎都是完美的，而且每次修改都把数据重新写回到主内存，讲道理不会有问题啊？

实际上问题正是出在这个看似完美的读写操作中：对于CPU1来说的确是完美的，但如果这时候CPU2来插一脚呢？我们思考下面这个流程：

1. CPU1读取数据a=1，CPU1的缓存中都有数据a的副本
2. CPU2也执行读取操作，同样CPU2也有数据a=1的副本
3. CPU1修改数据a=2，同时CPU1的缓存以及主内存a=2
4. CPU2再次读取a，但是CPU2在缓存中命中数据，此时a=1

问题到这里已经很明显了，CPU2并不知道CPU1改变了共享变量的值，因此造成了不可见问题。

* **缓存一致性协议**

为了解决这个问题，在早期的CPU当中，是通过在总线上直接加锁的形式来解决缓存不一致的问题。

但是正如Java中Synchronized一样，直接加锁太粗暴了，由于在锁住总线期间，其他CPU无法访问内存，导致效率低下。很明显这样做是不可取的。

所以就出现了缓存一致性协议。缓存一致性协议有MSI，MESI，MOSI，Synapse，Firefly及DragonProtocol等等。

* **MESI协议**

最出名的就是Intel 的MESI协议，MESI协议保证了每个缓存中使用的共享变量的副本是一致的。

1. **M**odify（修改）：当缓存行中的数据被修改时，该缓存行置为M状态
2. **E**xclusive（独占）：当只有一个缓存行使用某个数据时，置为E状态
3. **S**hared（共享）：当其他CPU中也读取某数据到缓存行时，所有持有该数据的缓存行置为S状态
4. **I**nvalid（无效）：当某个缓存行数据修改时，其他持有该数据的缓存行置为I状态

它核心的思想是：当CPU写数据时，如果发现操作的变量是共享变量，即在其他CPU中也存在该变量的副本，会发出信号通知其他CPU将该变量的缓存行置为无效状态，因此当其他CPU需要读取这个变量时，发现自己缓存中缓存该变量的缓存行是无效的，那么它就会从内存重新读取。

而这其中，监听和通知又基于总线嗅探机制来完成。

* **总线嗅探机制**

嗅探机制其实就是一个监听器，回到我们刚才的流程，如果是加入MESI缓存一致性协议和总线嗅探机制之后：

1. CPU1读取数据a=1，CPU1的缓存中都有数据a的副本，该缓存行置为（E）状态
2. CPU2也执行读取操作，同样CPU2也有数据a=1的副本，此时总线嗅探到CPU1也有该数据，则CPU1、CPU2两个缓存行都置为（S）状态
3. CPU1修改数据a=2，CPU1的缓存以及主内存a=2，同时CPU1的缓存行置为（S）状态，总线发出通知，CPU2的缓存行置为（I）状态
4. CPU2再次读取a，虽然CPU2在缓存中命中数据a=1，但是发现状态为（I），因此直接丢弃该数据，去主内存获取最新数据

当我们使用volatile关键字修饰某个变量之后，就相当于告诉CPU：我这个变量需要使用MESI和总线嗅探机制处理。从而也就保证了可见性。

* **指令重排序**

在加入MESI和总线嗅探机制后，当CPU2发现当前缓存行数据无效时，会丢弃该数据，并前往主内存获取最新数据。

但是这里又会产生一个问题：CPU1把数据刷回主内存是需要时间的，假如CPU2在主内存拿数据时，CPU1还没有把数据刷回来呢？

很明显，CPU2不会把资源浪费在这里傻等。它会先跳过和该数据有关的语句，继续处理后面的逻辑。

比如说如下代码：

```text
a = 1;
b = 2;
b++;
```

假如第一条语句需要等待CPU1数据刷新，那么CPU2可能就会先回来执行后面两条语句。因为对于CPU2来说，先执行后面两条语句不会对最终结果造成任何影响。



## 38-进程和线程

### 进程的三种状态

![](image\屏幕截图 2024-01-09 152707.png)

1. **就绪状态**在等待获取CPU
2. **执行状态**某进程占用CPU并在CPU上执行程序
3. **阻塞状态**也称为等待状态或封锁状态。如：请求I/O。（多个等待队列）    

### 进程和线程的区别

* #### 进程

  每个进程都有自己独立的一块内存空间，一个进程可以有多个线程。进程实体（进程映像）：程序段、数据段、PCB三部分组成了进程实体（进程映像）

* #### 线程

  与进程不同的是同类的多个线程共享进程的**堆**和**方法区**资源，但每个线程有自己的**程序计数器**、**虚拟机栈**和**本地方法栈**，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多，也正因为如此，线程也被称为轻量级进程。

**根本区别：**进程是操作系统资源分配的基本单位。线程是任务调度和执行的基本单位。

**资源开销：**每个进程都有自己的独立代码和数据空间，在切换进程时会有很大的开销。线程可以看做轻量级的进程，同一类线程共享代码和数据空间，每个线程都有自己独立的运行栈和程序计数器，线程之间的切换开销较小。

**包含关系**：如果一个进程内有多个线程，则执行过程不是一条线的，而是多条线（线程）共同完成的；线程是进程的一部分，所以线程也被称为轻权进程或者轻量级进程。

**内存分配**：同一进程的线程共享本进程的地址空间和资源，而进程之间的地址空间和资源是相互独立的

**影响关系**：一个进程崩溃后，在保护模式下不会对其他进程产生影响，但是一个线程崩溃整个进程都死掉。所以多进程要比多线程健壮。

执行过程：每个独立的进程有程序运行的入口、顺序执行序列和程序出口。但是线程不能独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制，两者均可并发执行


### 从JVM角度谈

![](image\屏幕截图 2024-01-09 153957.png)

一个进程中可以有多个线程，多个线程共享进程的**堆**和**方法区 (JDK1.8 之后的元空间)\**资源，但是每个线程有自己的\**程序计数器**、**虚拟机栈** 和 **本地方法栈**。

>  程序计数器为什么是私有的?

> 程序计数器主要有下面两个作用：

1. 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理。
2. 在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。
   需要注意的是，如果执行的是 native 方法，那么程序计数器记录的是 undefined 地址，只有执行的是 Java 代码时程序计数器记录的才是下一条指令的地址。

所以，程序计数器私有主要是为了线程切换后能恢复到正确的执行位置。

>  虚拟机栈和本地方法栈为什么是私有的?

* **虚拟机栈**：每个 Java 方法在执行的同时会创建一个栈帧用于存储局部变量表、操作数栈、常量池引用等信息。从方法调用直至执行完成的过程，就对应着一个栈帧在 Java 虚拟机栈中入栈和出栈的过程。
* **本地方法栈**：和虚拟机栈所发挥的作用非常相似，区别是： 虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。 在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一

所以，为了**保证线程中的局部变量不被别的线程访问到**，虚拟机栈和本地方法栈是线程私有的。





## 39-OSI的七层结构

> 开放系统互联（Open System Interconnection，OSI），是一种概念模型。
>
> HTTP协议在应用层，IP协议在网络层

![](image\屏幕截图 2024-01-13 115336.png)



![](image\屏幕截图 2024-01-13 115517.png)

#### 网络层和传输层负责的内容有什么区别？

* #### 网络层

  计算机网络中如果有多台计算机，怎么找到要发的那台？如果中间有多个节点，怎么选择路径？这就是路由要做的事。

  该层的主要任务就是：通过路由选择算法，为报文（该层的数据单位，由上一层数据打包而来）通过通信子网选择最适当的路径。这一层定义的是IP地址，通过IP地址寻址，所以产生了IP协议。

* #### 传输层

  当发送大量数据时，很可能会出现丢包的情况，另一台电脑要告诉是否完整接收到全部的包。如果缺了，就告诉丢了哪些包，然后再发一次，直至全部接收为止。

  简单来说，传输层的主要功能就是：监控数据传输服务的质量，保证报文的正确传输。



## 40-TCP拥塞控制

> 在某段时间，若对网络中某一资源的需求超过了该资源所能提供的可用部分，网络性能就要变差，这种情况就叫做网络拥塞。
>
> 若**出现拥塞而不进行控制**，整个网络的**吞吐量将随输入负荷的增大而下降**。路由器缓存溢出，拥塞会导致丢包。

![](image\屏幕截图 2024-01-13 120149.png)



* ### TCP四种拥塞控制算法

  1. 慢开始
  2. 拥塞避免
  3. 快重传
  4. 快恢复

* ### 拥塞控制过程

  > 假定：
  >
  > 1. 数据是单方向传送的，而另一个方向只传送确认
  > 2. 接收方总是有足够大的缓存空间，因而发送发发送窗口的大小由网络的拥塞程度来决定
  > 3. 以TCP报文段的个数为讨论问题的单位，而不是以字节为单位

  * #### 慢启动

  ![](image\屏幕截图 2024-01-13 121346.png)

  慢启动会**指数**地增加拥塞窗口的大小，直到超过或等于`ssthreash`，就用拥塞避免算法。

  * #### 拥塞避免

    ![](image\屏幕截图 2024-01-13 122538.png)

    拥塞避免算法会**线性**增加拥塞窗口大小，知道发生重传（收到三次重复确认），即快重传，之后就会发生快重传。将`ssthreash`设置为发生拥塞时的`cwnd`的一半

  * 快恢复

    * 当发送端收到第三个重复确认的报文时，会更新ssthresh的值，然后立即重传丢失的报文段，并且设置：cwnd = ssthresh+3*SMSS，进入拥塞避免阶段。
    * 当收到一个重复确认的报文时，设置cwnd = cwnd +SMSS。此时发送端可以发送新的TCP报文（如果新的cwnd允许）
    * 当收到新数据的确认时，设置cwnd=ssthresh。进入拥塞避免阶段。
      

> 新的TCP Reno 版本在快重传之后采用快回复算法而不是采用慢开始

#### 拥塞控制窗口和流量控制窗口的区别

> 两者作用上的区别:
>
> 流量控制是为了解决发送方和接收方速度不同而导致的数据丢失问题,当发送方发送的太快,接收方来不及接受就会导致数据丢失,流量控制用滑动窗口的形式解决问题
>
> 拥塞控制是为了解决过多的数据注入到网络,导致网络奔溃,超过负荷.当发送方发送数据大量的数据会注入到网络,如果没有限制,网络就会超负荷变卡,拥塞控制的用的是拥塞窗口解决的问题的
> 



## 41-HashMap有什么线程问题

* ### 循环链表

  > 当多个线程进行put操作，触发了扩容事件时，可能会产生循环链表。造成线程安全问题。

  



## 42-JVM中为什么要有两个Survivor区

**Survivor的存在意义，就是减少被送到老年代的对象，进而减少Full GC的发生，Survivor的预筛选保证，只有经历16次Minor GC还能在新生代中存活的对象，才会被送到老年代。**

**设置两个Survivor区最大的好处就是解决了碎片化**

### 对象的头信息

将对象的所属类（即类的元数据信息）、对象的HashCode和对象的GC信息、锁信息等数据存储在对象的对象头中。这个过程的具体设置方式取决于JVM实现。



## 43-Token的无感刷新

> 在Token认证的基础上，用户完成登录返回两个token，一个短期的access_token，一个长期的refresh_token。短期access_token**过期**，使用长期refresh_token换取**新token**，也就是会**发三次请求**，第一次401，第二次换token，第三次重新请求。如果网速正常、接口正常基本就能无感知实现刷新token了。

**access_token**：用作接口请求令牌,一般有效期十几分钟、几小时（具体看公司需求） **refresh_token**：在短期token过期后，换取新Token，一般是几天、一周



## 44-synchronized详解

* ### synchronized关键字三大特性

  > 面试时经常拿`synchronized`关键字和`volatile`关键字的特性进行对比，`synchronized`关键字可以保证并发编程的三大特性：原子性、可见性、有序性，而`volatile`关键字只能保证可见性和有序性，不能保证原子性，也称为是轻量级的`synchronized`。

  - 原子性：一个或多个操作要么全部执行成功，要么全部执行失败。`synchronized`关键字可以保证只有一个线程拿到锁，访问共享资源。
  - 可见性：当一个线程对共享变量进行修改后，其他线程可以立刻看到。执行`synchronized`时，会对应执行 `lock`、`unlock`原子操作，保证可见性。
  - 有序性：程序的执行顺序会按照代码的先后顺序执行。

  > synchronized 如何保证可见性的呢？

  其实进入`synchronized`块就是把在`synchronized`块内使用到的变量从线程的本地缓存中擦除，这样在`synchronized`块中再次使用到该变量就不能从本地内存中获取了，需要从主内存中获取，解决了内存不可见问题。

* ### synchronized关键字可以实现是类型的锁

  - **悲观锁**：`synchronized`关键字实现的是悲观锁，每次访问共享资源时都会上锁。
  - **非公平锁**：`synchronized`关键字实现的是非公平锁，即线程获取锁的顺序并不一定是按照线程阻塞的顺序。
  - **可重入锁**：`synchronized`关键字实现的是可重入锁，即已经获取锁的线程可以再次获取锁。
  - **独占锁或者排他锁**：`synchronized`关键字实现的是独占锁，即该锁只能被一个线程所持有，其他线程均被阻塞。



* ### 底层原理

  > 这个问题也是面试比较高频的一个问题，也是比较难理解的，理解`synchronized`需要一定的Java虚拟机的知识。

  在jdk1.6之前，`synchronized`被称为重量级锁，在jdk1.6中，为了减少获得锁和释放锁带来的性能开销，引入了偏向锁和轻量级锁。下面先介绍jdk1.6之前的`synchronized`原理。

  #### 对象头

  在HotSpot虚拟机中，Java对象在内存中的布局大致可以分为三部分：**对象头**、**实例数据**和**填充对齐**。因为`synchronized`用的锁是存在对象头里的，这里我们需要重点了解对象头。如果对象头是数组类型，则对象头由**Mark Word**、**Class MetadataAddress**和**Array length**组成，如果对象头非数组类型，对象头则由**Mark Word**和**Class MetadataAddress**组成。在32位虚拟机中，数组类型的Java对象头的组成如下表：

  ![](image\屏幕截图 2024-01-15 161417.png)

  #### 重量级锁的底部实现原理：Monitor

  在jdk1.6之前，`synchronized`只能实现重量级锁，Java虚拟机是基于Monitor对象来实现重量级锁的，所以首先来了解下Monitor，在Hotspot虚拟机中，Monitor是由ObjectMonitor实现的，其源码是用C++语言编写的，首先我们先下载Hotspot的源码，源码下载链接：[http://hg.openjdk.java.net/jdk8/jdk8/hotspot](https://link.zhihu.com/?target=http%3A//hg.openjdk.java.net/jdk8/jdk8/hotspot)，找到ObjectMonitor.hpp文件，路径是`src/share/vm/runtime/objectMonitor.hpp`，这里只是简单介绍下其数据结构

  ```c++
  ObjectMonitor() {
      _header       = NULL;
      _count        = 0; //锁的计数器，获取锁时count数值加1，释放锁时count值减1，直到
      _waiters      = 0, //等待线程数
      _recursions   = 0; //锁的重入次数
      _object       = NULL; 
      _owner        = NULL; //指向持有ObjectMonitor对象的线程地址
      _WaitSet      = NULL; //处于wait状态的线程，会被加入到_WaitSet
      _WaitSetLock  = 0 ;
      _Responsible  = NULL ;
      _succ         = NULL ;
      _cxq          = NULL ; //阻塞在EntryList上的单向线程列表
      FreeNext      = NULL ;
      _EntryList    = NULL ; //处于等待锁block状态的线程，会被加入到该列表
      _SpinFreq     = 0 ;
      _SpinClock    = 0 ;
      OwnerIsThread = 0 ;
    }
  ```

  ![](image\屏幕截图 2024-01-15 161619.png)

  从上图可以总结获取Monitor和释放Monitor的流程如下：

  1. 当多个线程同时访问同步代码块时，首先会进入到EntryList中，然后通过CAS的方式尝试将Monitor中的owner字段设置为当前线程，同时count加1，若发现之前的owner的值就是指向当前线程的，recursions也需要加1。如果CAS尝试获取锁失败，则进入到EntryList中。
  2. 当获取锁的线程调用`wait()`方法，则会将owner设置为null，同时count减1，recursions减1，当前线程加入到WaitSet中，等待被唤醒。
  3. 当前线程执行完同步代码块时，则会释放锁，count减1，recursions减1。当recursions的值为0时，说明线程已经释放了锁。

  > 之前提到过一个常见面试题，为什么`wait()`、`notify()`等方法要在同步方法或同步代码块中来执行呢，这里就能找到原因，是因为`wait()`、`notify()`方法需要借助ObjectMonitor对象内部方法来完成。

  同步代码块的实现是由`monitorenter` 和`monitorexit`指令完成的，其中`monitorenter`指令所在的位置是同步代码块开始的位置，第一个`monitorexit`指令是用于正常结束同步代码块的指令，第二个`monitorexit`指令是用于异常结束时所执行的释放Monitor指令。

  发现这个没有`monitorenter` 和 `monitorexit` 这两个指令了，而在查看该方法的class文件的结构信息时发现了`Access flags`后边的synchronized标识，该标识表明了该方法是一个同步方法。Java虚拟机通过该标识可以来辨别一个方法是否为同步方法，如果有该标识，线程将持有Monitor，在执行方法，最后释放Monitor。

  > 原理大概就是这样，最后总结一下，面试中应该简洁地如何回答`synchroized`的底层原理这个问题。

  答：Java虚拟机是通过进入和退出Monitor对象来实现代码块同步和方法同步的，代码块同步使用的是`monitorenter`和 `monitorexit` 指令实现的，而方法同步是通过`Access flags`后面的标识来确定该方法是否为同步方法。

  #### 为什么要对synchronized进行优化

  因为Java虚拟机是通过进入和退出Monitor对象来实现代码块同步和方法同步的，而Monitor是依靠底层操作系统的`Mutex Lock`（互斥锁）来实现的，操作系统实现线程之间的切换需要从用户态转换到内核态，这个切换成本比较高，对性能影响较大。

  

  #### 偏向锁

  > 常见面试题：偏向锁的原理（或偏向锁的获取流程）、偏向锁的好处是什么（获取偏向锁的目的是什么）

  引入偏向锁的目的：减少只有一个线程执行同步代码块时的性能消耗，即在没有其他线程竞争的情况下，一个线程获得了锁。

  

  ##### 偏向锁的获取流程：

  1. 检测对象头中Mark Word 是否为可偏向状态，如果不是直接升级为轻量级锁
  2. 如果是，判断Mark Work中的线程ID是否指向当前线程，如果是，就执行同步代码块
  3. 如果不是，则进行CAS操作竞争锁，如果竞争到锁，则将Mark Work中的线程ID设为当前线程ID，执行同步代码块
  4. 如果竞争失败，升级为轻量级锁

  ##### 偏向锁的撤销：

  只有等到竞争，持有偏向锁的线程才会撤销偏向锁。偏向锁撤销后会恢复到无锁或者轻量级锁的状态。

  1. 偏向锁的撤销需要到达全局安全点，全局安全点表示一种状态，该状态下所有线程都处于暂停状态。
  2. 判断锁对象是否处于无锁状态，即获得偏向锁的线程如果已经退出了临界区，表示同步代码已经执行完了。重新竞争锁的线程会进行CAS操作替代原来线程的ThreadID。
  3. 如果获得偏向锁的线程还处于临界区之内，表示同步代码还未执行完，将获得偏向锁的线程升级为轻量级锁。

  **一句话简单总结偏向锁原理：使用CAS操作将当前线程的ID记录到对象的Mark Word中。**

  

  #### 轻量级锁

  引入轻量级锁的目的：在多线程交替执行同步代码块时（未发生竞争），避免使用互斥量（重量锁）带来的性能消耗。但多个线程同时进入临界区（发生竞争）则会使得轻量级锁膨胀为重量级锁。

  ##### 轻量级锁获取流程：

  * 首先判断当前对象是否处于一个无锁的状态，如果是，Java虚拟机将在当前线程的栈帧建立一个锁记录（Lock Record），用于存储对象目前的Mark Word的拷贝，如图所示。
  * 将对象的Mark Word复制到栈帧中的Lock Record中，并将Lock Record中的owner指向当前对象，**并使用CAS操作将对象的Mark Word更新为指向Lock Record的指针**，如图所示。

  - 如果第二步执行成功，表示该线程获得了这个对象的锁，将对象Mark Word中锁的标志位设置为“00”，执行同步代码块。
  - 如果第二步未执行成功，需要先判断当前对象的Mark Word是否指向当前线程的栈帧，如果是，表示当前线程已经持有了当前对象的锁，这是一次重入，直接执行同步代码块。**如果不是表示多个线程存在竞争，该线程通过自旋尝试获得锁，即重复步骤2，自旋超过一定次数，轻量级锁升级为重量级锁。**

  ##### 轻量级锁的解锁：

  轻量级锁的解锁：

  轻量级的解锁同样是通过CAS操作进行的，**线程会通过CAS操作将Lock Record中的Mark Word（官方称为Displaced Mark Word）替换回来**。如果成功表示没有竞争发生，成功释放锁，恢复到无锁的状态；如果失败，表示当前锁存在竞争，升级为重量级锁。

  一句话总结轻量级锁的原理：将对象的Mark Word复制到当前线程的Lock Record中，并将对象的Mark Word更新为指向Lock Record的指针。

  

  **一句话总结轻量级锁的原理：将对象的Mark Word复制到当前线程的Lock Record中，并将对象的Mark Word更新为指向Lock Record的指针。**

  

  #### 自旋锁

  引入自旋锁的原因：因为阻塞和唤起线程都会引起操作系统用户态和核心态的转变，对系统性能影响较大，而自旋等待可以避免线程切换的开销。

  **自适应自旋锁**：JDK1.6引入了自适应自旋锁，自适应自旋锁的自旋次数不在固定，而是由上一次在同一个锁上的自旋时间及锁的拥有者的状态来决定的。如果对于某个锁对象，刚刚有线程自旋等待成功获取到锁，那么虚拟机将认为这次自旋等待的成功率也很高，会允许线程自旋等待的时间更长一些。如果对于某个锁对象，线程自旋等待很少成功获取到锁，那么虚拟机将会减少线程自旋等待的时间。

  

  #### **当线程1进入到一个对象的synchronized方法A后，线程2是否可以进入到此对象的synchronized方法B?**

  不能，线程2只能访问该对象的非同步方法。因为执行同步方法时需要获得对象的锁，而线程1在进入`sychronized`修饰的方A时已经获取到了锁，线程2只能等待，无法进入到`synchronized`修饰的方法B，但可以进入到其他非`synchronized`修饰的方法。

  #### **synchronized和volatile的区别？**

  - `volatile`主要是保证内存的可见性，即变量在寄存器中的内存是不确定的，需要从主存中读取。`synchronized`主要是解决多个线程访问资源的同步性。
  - `volatile`作用于变量，`synchronized`作用于代码块或者方法。
  - `volatile`仅可以保证数据的可见性，不能保证数据的原子性。`synchronized`可以保证数据的可见性和原子性。
  - `volatile`不会造成线程的阻塞，`synchronized`会造成线程的阻塞。

  #### **synchronized和Lock的区别？**

  - Lock是显示锁，需要手动开启和关闭。synchronized是隐士锁，可以自动释放锁。
  - Lock是一个接口，是JDK实现的。synchronized是一个关键字，是依赖JVM实现的。
  - Lock是可中断锁，synchronized是不可中断锁，需要线程执行完才能释放锁。
  - 发生异常时，Lock不会主动释放占有的锁，必须通过unlock进行手动释放，因此可能引发死锁。synchronized在发生异常时会自动释放占有的锁，不会出现死锁的情况。
  - Lock可以判断锁的状态，synchronized不可以判断锁的状态。
  - Lock实现锁的类型是可重入锁、公平锁。synchronized实现锁的类型是可重入锁，非公平锁。
  - Lock适用于大量同步代码块的场景，synchronized适用于少量同步代码块的场景。



## 45-IOC容器初始化过程

* ### IOC容器初始化过程？

1. 从XML中读取配置文件。
2. 将bean标签解析成 BeanDefinition，如解析 property 元素， 并注入到 BeanDefinition 实例中。
3. 将 BeanDefinition 注册到容器 BeanDefinitionMap 中。
4. BeanFactory 根据 BeanDefinition 的定义信息创建实例化和初始化 bean。

单例bean的初始化以及依赖注入一般都在容器初始化阶段进行，只有懒加载（lazy-init为true）的单例bean是在应用第一次调用getBean()时进行初始化和依赖注入。

```java
// AbstractApplicationContext
// Instantiate all remaining (non-lazy-init) singletons.
finishBeanFactoryInitialization(beanFactory);
```

多例bean 在容器启动时不实例化，即使设置 lazy-init 为 false 也没用，只有调用了getBean()才进行实例化。

`loadBeanDefinitions`采用了模板模式，具体加载 `BeanDefinition` 的逻辑由各个子类完成。



## 46-Bean的生命周期

![](image\屏幕截图 2024-01-15 202648.png)

1.调用bean的构造方法创建Bean

2.通过反射调用setter方法进行属性的依赖注入

3.如果Bean实现了`BeanNameAware`接口，Spring将调用`setBeanName`()，设置 `Bean`的name（xml文件中bean标签的id）

4.如果Bean实现了`BeanFactoryAware`接口，Spring将调用`setBeanFactory()`把bean factory设置给Bean

5.如果存在`BeanPostProcessor`，Spring将调用它们的`postProcessBeforeInitialization`（预初始化）方法，在Bean初始化前对其进行处理

6.如果Bean实现了`InitializingBean`接口，Spring将调用它的`afterPropertiesSet`方法，然后调用xml定义的 init-method 方法，两个方法作用类似，都是在初始化 bean 的时候执行

7.如果存在`BeanPostProcessor`，Spring将调用它们的`postProcessAfterInitialization`（后初始化）方法，在Bean初始化后对其进行处理

8.Bean初始化完成，供应用使用，这里分两种情况：

8.1 如果Bean为单例的话，那么容器会返回Bean给用户，并存入缓存池。如果Bean实现了`DisposableBean`接口，Spring将调用它的`destory`方法，然后调用在xml中定义的 `destory-method`方法，这两个方法作用类似，都是在Bean实例销毁前执行。

8.2 如果Bean是多例的话，容器将Bean返回给用户，剩下的生命周期由用户控制。

```java
public interface BeanPostProcessor {
    @Nullable
    default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
    @Nullable
    default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}
public interface InitializingBean {
    void afterPropertiesSet() throws Exception;
}
```





## 47-Redis数据结构底层实现原理

* ### hash

  哈希对象的编码有两种，分别是：**ziplist、hashtable**。

  当哈希对象保存的键值对数量小于 512，并且所有键值对的长度都小于 64 字节时，使用ziplist(压缩列表)存储；否则使用 hashtable 存储。

  Redis中的hashtable跟Java中的HashMap类似，都是通过"数组+链表"的实现方式解决部分的哈希冲突。直接看源码定义。

  #### 扩容机制

  1、如果执行扩展操作，会基于原哈希表创建一个大小等于 ht[0].used*2n 的哈希表（也就是每次扩展都是根据原哈希表已使用的空间扩大一倍创建另一个哈希表）。相反如果执行的是收缩操作，每次收缩是根据已使用空间缩小一倍创建一个新的哈希表。

  2、重新利用哈希算法，计算索引值，然后将键值对放到新的哈希表位置上。

  3、所有键值对都迁徙完毕后，释放原哈希表的内存空间。

* ### list（链表）

  列表对象的编码有两种，分别是：ziplist、linkedlist。当列表的长度小于 512，并且所有元素的长度都小于 64 字节时，使用ziplist存储；否则使用 linkedlist 存储。

  Redis中的linkedlist类似于Java中的LinkedList，是一个链表，底层的实现原理也和LinkedList类似。这意味着list的插入和删除操作效率会比较快，时间复杂度是O(1)。我们看源码：

  ![](image\屏幕截图 2024-01-16 155916.png)

* ### set（集合）

  set类型的特点很简单，无序，不重复，跟Java的HashSet类似。它的编码有两种，分别是intset和hashtable。如果value可以转成整数值，并且长度不超过512的话就使用intset存储，否则采用hashtable。

  hashtable在前面讲hash类型时已经讲过，这里的set集合采用的hashtable几乎一样，只是哈希表的value都是NULL。这个不难理解，比如用Java中的HashMap实现一个HashSet，我们只用HashMap的key就是了。

  我们讲一讲intset，先看源码。

  ```text
  typedef struct intset{
      uint32_t encoding;//编码方式
  
      uint32_t length;//集合包含的元素数量
  
      int8_t contents[];//保存元素的数组
  }intset;
  ```

* ### zset(有序集合)

  zset是Redis中比较有特色的数据类型，它和set一样是不可重复的，区别在于多了score值，用来代表排序的权重。也就是当你需要一个有序的，不可重复的集合列表时，就可以考虑使用这种数据类型。

  zset的编码有两种，分别是：ziplist、skiplist。当zset的长度小于 128，并且所有元素的长度都小于 64 字节时，使用ziplist存储；否则使用 skiplist 存储。

  这里要讲一下skiplist，也就是跳跃表。它的底层实现比较复杂，这里简单地提一下。

  




## 48-Arrays.sort()底层采用什么算法

> ，Arrays.sort 底层实际上采用了三种排序方式。当数据量在 [0 - 47) 时，采用插入排序，[47, 286) 时采用双轴快速排序，大于等于 286 时采用归并排序。不过这还有一个我在看源码注释时发现的点，当数据量大于 286 但数组不是高度结构化的时候采用双轴快速排序替换归并排序。



## 49-String底层原理

String在JDK1.8中的底层实现，如下：（char数组）

```java
private final char value[];
```

String在JDK1.9中的底层实现，如下：（byte数组）

```java
@Stable //表示下方属性 最多被赋值1次！
private final byte[] value;
```



* ### String 的创建流程

  #### 第一种情况

  ```java
  public static void main (String[] args){
      String s1 = "hello";
      String s2 = new String("hello");
      System.out.println(s1 == s2);//false
  }
  ```

  对于第一种字符串s1的赋值，它会先在堆内存中的字符串常量池中寻找地址，如果找到就会根据地址获取到值，然后将地址赋值给s1；如果没有，就会在堆内存中创建这个字符串，并将地址注册到常量池中。

  对于`new String()`方法，不管常量池也没有，都会在堆内存中创建一个新的对象。所以s1和s2很容易知道不是同一个对象。

  #### 第二种情况

  ```java
  String s1 = new String("hello ") + new String("world");
  s1.intern();
  String s2 = "hello world";
  System.out.println(s1 == s2);//true
  ```

  对于s1的执行情况，是

  1. 会首先在堆内存中创建“hello”和“world”两个字符串对象
  2. 然后将它们拼接起来（底层使用的是StringBuilder）
  3. 在拼接完成后会产生新的“hello world”对象，然后s1指向这个变量

  ##### intern()函数

  当调用 intern()方法时, 首先会去常量池中查找是否有该字符串对应的引用, 如果有就直接返回该字符串; 如果没有, 就会在常量池中注册该字符串的引用, 然后返回该字符串。

  #### 第三种情况

  ```java
  String s1 = "hello ";
  String s2 = "world";
  String s3 = s1 + s2;
  String s4 = "hello world";
  System.out.println(s3 == s4);
  ```

  根据上面的例子，这种情况我们目前还无法判断。因为不知道两个字符串拼接会不会创建对象。

  看源码可以知道，s3会调用常量池中的两个字符串，然后创建StringBuilder来新建一个新的字符串对象，并不会注册到常量池中。

  所以结果为false；



## 50-Spring用到哪些设计模式

### Spring的优点

- 通过控制反转和依赖注入实现**松耦合**。
- 支持**面向切面**的编程，并且把应用业务逻辑和系统服务分开。
- 通过切面和模板减少样板式代码。
- 声明式事务的支持。可以从单调繁冗的事务管理代码中解脱出来，通过声明式方式灵活地进行事务的管理，提高开发效率和质量。
- 方便集成各种优秀框架。内部提供了对各种优秀框架的直接支持（如：Hessian、Quartz、MyBatis等）。
- 方便程序的测试。Spring支持Junit4，添加注解便可以测试Spring程序。

### Spring 用到了哪些设计模式？

1、**简单工厂模式**：`BeanFactory`就是简单工厂模式的体现，根据传入一个唯一标识来获得 Bean 对象。

```java
@Override
public Object getBean(String name) throws BeansException {
    assertBeanFactoryActive();
    return getBeanFactory().getBean(name);
}
```

2、**工厂方法模式**：`FactoryBean`就是典型的工厂方法模式。spring在使用`getBean()`调用获得该bean时，会自动调用该bean的`getObject()`方法。每个 Bean 都会对应一个 `FactoryBean`，如 `SqlSessionFactory` 对应 `SqlSessionFactoryBean`。

3、**单例模式**：一个类仅有一个实例，提供一个访问它的全局访问点。Spring 创建 Bean 实例默认是单例的。

4、**适配器模式**：SpringMVC中的适配器`HandlerAdatper`。由于应用会有多个Controller实现，如果需要直接调用Controller方法，那么需要先判断是由哪一个Controller处理请求，然后调用相应的方法。当增加新的 Controller，需要修改原来的逻辑，违反了开闭原则（对修改关闭，对扩展开放）。为此，Spring提供了一个适配器接口，每一种 Controller 对应一种 `HandlerAdapter` 实现类，当请求过来，SpringMVC会调用`getHandler()`获取相应的Controller，然后获取该Controller对应的 `HandlerAdapter`，最后调用`HandlerAdapter`的`handle()`方法处理请求，实际上调用的是Controller的`handleRequest()`。每次添加新的 Controller 时，只需要增加一个适配器类就可以，无需修改原有的逻辑

5、**代理模式**：spring 的 aop 使用了动态代理，有两种方式`JdkDynamicAopProxy`和`Cglib2AopProxy`。

6、**观察者模式**：spring 中 observer 模式常用的地方是 listener 的实现，如`ApplicationListener`。

7、**模板模式**： Spring 中 `jdbcTemplate`、`hibernateTemplate` 等，就使用到了模板模式。



## 51-MySQL有哪些锁

* ### 锁的分类

  行级锁、表级锁、页级锁

  共享锁、排它锁

  乐观锁、悲观锁

  > MyISAM和MEMORY采用表级锁(table-level locking)
  >
  > BDB采用页面锁(page-level locking)或表级锁，默认为页面锁
  >
  > InnoDB支持行级锁(row-level locking)和表级锁,默认为行级锁

* ### 行级锁

  > 行级锁是mysql中锁定粒度最细的一种锁。表示只针对当前操作的行进行加锁。行级锁能大大减少数据库操作的冲突，其加锁粒度最小，但加锁的开销也最大。**行级锁分为共享锁和排他锁**
  >
  > 开销大，加锁慢，会出现死锁。发生锁冲突的概率最低，并发度也最高。

  #### InnoDB的三种行锁算法：

  1，Record Lock（记录锁）：单个行记录上的锁。这个也是我们日常认为的行锁。

  2，Gap Lock（间隙锁）：间隙锁，锁定一个范围，但不包括记录本身（只不过它的锁粒度比记录锁的锁整行更大一些，他是锁住了某个范围内的多个行，包括根本不存在的数据）。GAP锁的目的，是为了防止同一事务的两次当前读，出现幻读的情况。该锁只会在隔离级别是RR或者以上的级别内存在。间隙锁的目的是为了让其他事务无法在间隙中新增数据。

  3，Next-Key Lock（临键锁）：它是记录锁和间隙锁的结合，锁定一个范围，并且锁定记录本身。对于行的查询，都是采用该方法，主要目的是解决幻读的问题。next-key锁是InnoDB默认的锁

  上面这三种锁都是排它锁（X锁）

  

## 52-sleep方法会一直占用CPU吗

Object.wait()方法，会释放锁资源以及CPU资源。

Thread.sleep()方法，不会释放锁资源，但是会释放CPU资源。

因为wait/notify是基于共享内存来实现线程通信的工具，这个通信涉及到条件的竞争，所以在调用这两个方法之前必须要竞争锁资源。

当线程调用wait方法的时候，表示当前线程的工作处理完了，意味着让其他竞争同一个共享资源的线程有机会去执行。

但前提是其他线程需要竞争到锁资源，所以wait方法必须要释放锁，否则就会导致死锁的问题。

然后，Thread.sleep()方法，只是让一个线程单纯进入睡眠状态，这个方法并没有强制要求加synchronized同步锁。


## 53-分布式id的生成策略

* #### UUID

* #### 数据库自增id

* ####  基于数据库的好段模式

  号段模式是当下分布式ID生成器的主流实现方式之一，号段模式可以理解为从数据库批量的获取自增ID，每次从数据库取出一个号段范围，例如 (1,1000] 代表1000个ID，具体的业务服务将本号段，生成1~1000的自增ID并加载到内存。表结构如下：

  ```text
  CREATE TABLE id_generator (
    id int(10) NOT NULL,
    max_id bigint(20) NOT NULL COMMENT '当前最大id',
    step int(20) NOT NULL COMMENT '号段的布长',
    biz_type    int(20) NOT NULL COMMENT '业务类型',
    version int(20) NOT NULL COMMENT '版本号',
    PRIMARY KEY (`id`)
  ) 
  ```

  biz_type ：代表不同业务类型

  max_id ：当前最大的可用id

  step ：代表号段的长度

  version ：是一个乐观锁，每次都更新version，保证并发时数据的正确性

  等这批号段ID用完，再次向数据库申请新号段，对`max_id`字段做一次`update`操作，`update max_id= max_id + step`，update成功则说明新号段获取成功，新的号段范围是`(max_id ,max_id +step]`。

* ### Redis

* #### 雪花算法

  > 雪花算法（Snowflake）是twitter公司内部分布式项目采用的ID生成算法，开源后广受国内大厂的好评，在该算法影响下各大公司相继开发出各具特色的分布式生成器。

  ![](image\屏幕截图 2024-01-17 155125.png)

  

  `Snowflake`生成的是Long类型的ID，一个Long类型占8个字节，每个字节占8比特，也就是说一个Long类型占64个比特。
  Snowflake ID组成结构：`正数位`（占1比特）+ `时间戳`（占41比特）+ `机器ID`（占5比特）+ `数据中心`（占5比特）+ `自增值`（占12比特），总共64比特组成的一个Long类型。

  - 第一个bit位（1bit）：Java中long的最高位是符号位代表正负，正数是0，负数是1，一般生成ID都为正数，所以默认为0。
  - 时间戳部分（41bit）：毫秒级的时间，不建议存当前时间戳，而是用（当前时间戳 - 固定开始时间戳）的差值，可以使产生的ID从更小的值开始；41位的时间戳可以使用69年，(1L << 41) / (1000L * 60 * 60 * 24 * 365) = 69年
  - 工作机器id（10bit）：也被叫做`workId`，这个可以灵活配置，机房或者机器号组合都可以。
  - 序列号部分（12bit），自增值支持同一毫秒内同一个节点可以生成4096个ID

  根据这个算法的逻辑，只需要将这个算法用Java语言实现出来，封装为一个工具方法，那么各个业务应用可以直接使用该工具方法来获取分布式ID，只需保证每个业务应用有自己的工作机器id即可，而不需要单独去搭建一个获取分布式ID的应用。

* #### 百度（uid-generator）

  `uid-generator`是由百度技术部开发，项目GitHub地址 [https://github.com/baidu/uid-generator](https://link.zhihu.com/?target=https%3A//links.jianshu.com/go%3Fto%3Dhttps%3A%2F%2Fgithub.com%2Fbaidu%2Fuid-generator)
  `uid-generator`是基于`Snowflake`算法实现的，与原始的`snowflake`算法不同在于，`uid-generator`支持自`定义时间戳`、`工作机器ID`和 `序列号` 等各部分的位数，而且`uid-generator`中采用用户自定义`workId`的生成策略。
  `uid-generator`需要与数据库配合使用，需要新增一个`WORKER_NODE`表。当应用启动时会向数据库表中去插入一条数据，插入成功后返回的自增ID就是该机器的`workId`数据由host，port组成。


  **对于\**\**`uid-generator` ID组成结构**：


  `workId`，占用了22个bit位，时间占用了28个bit位，序列化占用了13个bit位，需要注意的是，和原始的`snowflake`不太一样，时间的单位是秒，而不是毫秒，`workId`也不一样，而且同一应用每次重启就会消费一个`workId`

* #### 美团(Leaf)

  `Leaf`同时支持号段模式和`snowflake`算法模式，可以切换使用

* #### **滴滴（Tinyid）**

  

## 54-什么是回表查询？如何避免？

> 回表查询就是执行了两次B+树查找。
>
> 举个例子：当对一个表创建了一个普通索引，那么这个索引就只存储了**主键和索引的值**，如果你要查询的字段不包括在这个索引里面，它就会回到主键索引中寻找数据，就发生了两次B+树查找。

* ### 覆盖索引

  覆盖索引就是建立联合索引，让查询的字段都命中在索引里。



## 55-static关键字

>  一句话描述就是：方便在没有创建对象的情况下进行调用(方法/变量)。
>
> static方法是属于类的，非实例对象，在JVM加载类时，就已经存在内存中，不会被虚拟机GC回收掉，这样内存负荷会很大，但是非static方法会在运行完毕后被虚拟机GC掉，减轻内存压力



## 56-ThreadLoacl详解

> **ThreadLocal 是 JDK 中常用的工具类，它提供了在与当前线程绑定的局部变量，不同线程都会取到不同的值，这在一些并发、变量传递等场景下非常好用。**然而 ThreadLocal 其实有内存泄漏的隐患，如果平时使用过程不注意，很有可能会暴露问题。作为一名合格的程序猿，我们需要对日常用的的工具内部原理都了如指掌，因此在这里探究下关于 ThreadLocal 的那些事。
>
> 从名字我们就可以看到`ThreadLocal` 叫做本地线程变量，意思是说，`ThreadLocal` 中填充的的是当前线程的变量，该变量对其他线程而言是封闭且隔离的，`ThreadLocal` 为变量在每个线程中创建了一个副本，这样每个线程都可以访问自己内部的副本变量。
>
> 从字面意思很容易理解，但是实际角度就没那么容易了，作为一个面试常问的点，使用场景也是很丰富。
>
> - 1、在进行对象跨层传递的时候，使用ThreadLocal可以避免多次传递，打破层次间的约束。
> - 2、**线程间数据隔离**
> - 3、进行事务操作，用于存储线程事务信息。
> - 4、数据库连接，`Session`会话管理。

* ### ThreadLocal 基本结构

  我们知道每个线程都有一个 ThreadLocalMap 结构，其中就保存着当前线程所持有的所有 ThreadLocal。ThreadLocal 本身只是一个引用，没有直接保存值，值是保存在 ThreadLocalMap 中，ThreadLocal 作为 key，值作为 value。可以用下面的图来概括：

  ![](image\屏幕截图 2024-01-17 164842.png)

* ### ThreadLocalMap 详解

  - ThreadLocalMap 类似 HashMap，key 为 ThreadLocal，value 为 ThreadLocal 的值。ThreadLocal 只是一个引用，它本身并不保存值；
  - 当发生哈希冲突时，ThreadLocalMap 采用的是**开放寻址法**，这与 HashMap 的拉链式寻址法不同；
  - ThreadLocalMap 本身的操作时，因为是线程私有变量，所以不存在并发操作；但是需要考虑 GC 因素，因为其 key 是弱引用，随时可能被 GC，导致开放寻址法中，出现一个个“坑”，因此在查找、新增、删除等操作时，需要进行 rehash 操作，也就是把元素重新进行哈希映射，放置到正确的位置，填平这些“坑”；
  - ThreadLocaMap 也会进行扩容，扩容之前会进行全量的 rehash 操作。

### 开放性寻址法

> **开放寻址法**：又称开放定址法，当哈希碰撞发生时，从发生碰撞的那个单元起，按照一定的次序，从哈希表中寻找一个空闲的单元，然后把发生冲突的元素存入到该单元。这个空闲单元又称为开放单元或者空白单元。
>
>   查找时，如果探查到空白单元，即表中无待查的关键字，则查找失败。开放寻址法需要的表长度要大于等于所需要存放的元素数量，非常适用于装载因子较小（小于0.5）的散列表。
>
> **开放定址法的缺点在于删除元素的时候不能真的删除，否则会引起查找错误**，只能做一个特殊标记，直到有下个元素插入才能真正删除该元素。

**由于在开放寻址法删除元素会引起查找错误，所以在`ThreadLocalMap`中，删除元素以后要对后面的元素进行`rehash`操作。**

### ThreadLoacl内存泄露

过上面的原理分析，结合前面的 ThreadLocal 结构图，为什么 ThreadLocal 会发生内存泄露就很明显了：

存在着一条从 currentThread 到 ThreadLocal value 的强引用链，即使 ThreadLocal 本身已经被回收了，value 因为强引用链的缘故，是依旧存活在内存中的。所以如果后续一直没有经过主动的 remove 或者被动的 rehash 操作的话，value 就会持续占用内存空间，造成泄露。

前面 ThreadLocalMap 中，因为 ThreadLocal 为弱引用，加上开放寻址法，所以查询、插入、删除操作都非常复杂，那么现在知道为什么 ThreadLocal 是弱引用了，原因就是为了尽量去避免内存泄漏问题。只要 ThreadLocal 本身不再用到了，即没有强引用了，ThreadLocalMap 中的弱引用就会被自动 GC 回收。**后面 ThreadLocalMap 会不断的进行懒式扫描，剔除掉 key 为 null 的元素，释放在这部分内存。**

某种意义上，ThreadLocal 本身已经最大化的去解决内存泄露问题了。不过凡事总有万一，万一这个元素在后续的 rehash 操作中一直没有被扫描到，那就会一直存在于内存。所以最好的方式就是，对于不再使用的 ThreadLocal，主动进行 remove，就像用完锁要解锁一样。



## 57-tomcat线程池深入理解

* ### 工作机制：

  Tomcat启动时如果没有请求过来，那么线程数（都是指线程池的）为0；

  一旦有请求，Tomcat会初始化minSpareThreads设置的线程数；

* ### 线程池作用：

  Tomcat的线程池的线程数跟你的瞬间并发有关系，比如maxThreads设置为1000，当瞬间并发达到1000那么Tomcat就会起1000个线程来处理，这时候跟你应用的快慢关系不大。

* ###  参数分析：

  //编辑tomcat安装目录下的conf目录下的server.xml文件

  ```xml
  <Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
  
  maxThreads="3000" minSpareThreads="800"/>
  
  <Connector executor="tomcatThreadPool" port="8084" protocol="org.apache.coyote.http11.Http11AprProtocol"
  
  connectionTimeout="60000"
  
  keepAliveTimeout="30000"
  
  maxKeepAliveRequests="8000"
  
  maxHttpHeaderSize="8192"
  
  URIEncoding="UTF-8"
  
  enableLookups="false"
  
  acceptCount="1000"
  
  disableUploadTimeout="true"
  
  redirectPort="8443" />
  ```

  

  **maxThreads**：Tomcat线程池最多能起的线程数；

  **maxConnections**：Tomcat瞬间最多能并发处理的请求（连接）；

  **acceptCount**：Tomcat维护最大的队列数（当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理）；

  **minSpareThreads**：Tomcat初始化的线程池大小或者说Tomcat线程池最少会有这么多线程。

  

  比较容易弄混的是`maxThreads`和`maxConnections`这两个参数：

  比如maxThreads=1000，maxConnections=800，假设某一瞬间的并发时1000，那么最终Tomcat的线程数将会是800，即同时处理800个请求，剩余200进入队列"排队"，如果acceptCount=100，那么有100个请求会被拒掉。

  注意：根据前面所说，只是并发那一瞬间Tomcat会起800个线程处理请求，但是稳定后，某一瞬间可能只有很少的线程处于RUNNABLE状态，大部分线程是TIMED_WAITING，如果你的应用处理时间够快的话。所以真正决定Tomcat最大可能达到的线程数是maxConnections这个参数和并发数，当并发数超过这个参数则请求会排队，这时响应的快慢就看你的程序性能了。

  并发的概念：不管什么并发肯定是有一个时间单位的（一般是1s），准确的来讲应该是当时Tomcat处理一个请求的时间内并发数，比如当时Tomcat处理某一个请求花费了1s，那么如果这1s过来的请求数达到了3000，那么Tomcat的线程数就会为3000，maxConnections只是Tomcat做的一个限制。

 

* ### Tomcat会停止长时间闲置的线程

  这个时间就是maxIdleTime。但我之前的测试中确实没有发现线程释放的现象，这是为什么呢？我发现除了这个参数线程池线程是否释放？释放多少？还跟当前Tomcat每秒处理的请求数（从Jmeter或LoadRunner来看可以理解为TPS）有关系。通过下表可以清晰的看出来线程数，TPS和maxIdleTime之间的关系：

  ![img](https://img2018.cnblogs.com/blog/743229/201901/743229-20190123155030035-1701526713.png)

  当然这个Thread Count不会小于minSpareThreads。





## 58-java反射调用方法与直接调用方法有什么区别

> 用反射调用方法会有性能的损失。

* ### 原因

  反射是基于程序和元数据的，由于虚拟机不知道你要使用的是什么类，所以在使用反射的时候，会搜索元数据，进行参数校验。

  因为方法传递的参数通常是object类型的，调用过程中的会有很多装箱与拆箱操作产生了额外的开销。(装箱相当于新创建了对象。分配了内存空间、初始化对象、返回对象地址)

  > **装箱**:就是将基础数据类型转换为对应的包装器类型；
  >
  > **拆箱**:就是将包装器类型转换为对应的基础数据类型；



## 59-Http为什么不安全，怎么解决？

HTTP协议没有任何的加密以及身份验证的机制，非常容易遭到窃听、劫持、篡改等。不安全的原因主要包含以下三个方面：

1、通信使用明文，内容可能被窃听。

2、不验证通信方的身份，因此有可能遭到伪装。

3、无法验证报文的完整性，所以有可能被篡改。

4、传统的HTTP请求过程都是明文传输的，所谓的明文指的是没有经过加密的信息，如果HTTP请求和响应被黑客拦截，并且里面含有密码等敏感数据的话，会非常危险。



**HTTPS**为了解决这个问题，Netscape 公司制定了HTTPS协议，HTTPS可以将数据加密传输，也就是传输的是密文，即便黑客在传输过程中拦截到数据也无法破译，这就保证了网络通信的安全。HTTPS就是 披着 TLS/SSL 的HTTP。

1、首先需要了解下加密算法这块的基础：

2、下面是一些加密算法的知识。

3、明文： 明文指的是未被加密过的原始数据。

4、密文：明文被某种加密算法加密之后，会变成密文，从而确保原始数据的安全。密文也可以被解密，得到原始的明文。

5、密钥：密钥是一种参数，它是在明文转换为密文或将密文转换为明文的算法中输入的参数。密钥分为对称密钥与非对称密钥，分别应用在对称加密和非对称加密上。





## 60-TCP为什么是可靠性的？

可靠传输就是通过TCP连接传送的数据是没有差错、不会丢失、不重复并且按序到达的。TCP是通过序列号、检验和、确认应答信号、重发机制、连接管理、窗口控制、流量控制、拥塞控制一起保证TCP传输的可靠性的。

* ### 总结：

TCP保证可靠性一般有以下几种方法：

1. **检验和**：在数据传输过程中，吧传输的数据当作一个16位整数。吧所有的数据加起来，最前面的进位补到最后一位，然后取反得到校验和。发送方和接收方验证校验和是否相同。不相同则数据传输有误，相同也可能有问题。
2. **确认应答**：ACK和序列号（一应一答）机制保证数据的完整性（三次握手于四次挥手过程中通过比对Seq和ACK来实现）
3. **超时重传**：发送数据包在一定的时间周期内没有收到相应的ACK，等待一定的时间，超时之后就认为这个数据包丢失，就会重新发送（也就是发送数据后，长时间没收到回应，会把数据再发一次。）
4. **连接管理**：三次握手，四次挥手
5. **最大消息长度**：理想的情况下是该长度的数据刚好不被网络层分块。
6. **拥塞控制**：控制传输上流量（发送数据时开始是慢启动，先发送一点点数据去探测网络拥塞不拥塞，如果不拥塞了，则大量的发送数据。如果突然拥塞了，则又很慢的发送数据。这样是为了尽可能快的发送数据，避免网络拥塞造成一系列问题）
7. **流量控制**：TCP利用滑动窗口实现流量控制，流量控制是为了控制发送方的发送速率，保证接收方来的及时接收





## 61-Cookie和Session的关系和区别

1. cookie 数据存放在客户端浏览器上，而session数据放在服务器上，当时服务器的session的实现对客户端的cookie有依赖关系。

   因为当触发session机制时，服务端会向客户端发送sessionId，而这个sessionid就是保存在cookie中的。

2. cookie容易被人分析修改，相比session更不安全。



## 62-虚拟内存与物理内存的区别

1. 每个进程都是由自己的内存空间的，这个存储空间会将代码从磁盘中拷贝到自己的进程空间中去，由进程控制表来进行数据的读写。

> 如果每个程序都讲数据拷贝到内存空间中，需要的内存空间就会非常大

2. 所以其实每个进程都是将数据拷贝到虚拟内存空间，每次访问时，都会将虚拟内存空间的某个地址转化为实际物理的内存空间。
3. 故所有的进程都是共享同一个物理内存，**每个进程只把目前所需要的虚拟内存空间映射存储到物理内存空间上**
4. 进程要知道哪些内存地址上的数据在物理内存上，这时就出现了**页表**。
5. 页表用来记录内存地址在物理内存空间上，还有物理内存地址
6. 若访问虚拟地址，查询页表，发现不在，**发生缺页异常**
7. 缺页异常处理：把需要数据从磁盘拷贝到物理内存，若满了就会找一个覆盖，若被覆盖的页被修改就会写回磁盘。

### 分页和分段机制

页表实现了虚拟内存到物理内存的映射，当访问一个虚拟内存页面时，页面的虚拟地址将作为一个索引指向页表，如果页表中存在对应物理内存的映射，则直接返回物理内存的地址，否则将引发一个缺页异常，从而陷入到内核中分配物理内存，返回对应的物理地址，然后更新页表。

分页是实现虚拟内存的技术，虚拟内存按照固定的大小分为页面，物理内存也会按照固定的大小分成页框，页面和页框大小通常是一样的，一般是 4KB，页面和页框可以实现一对一的映射。**分页是一维的，主要是为了获得更大的线性地址空间。**但是一个地址空间可能存在很多个表，表的数据大小是动态增长的，由于多个表都在一维空间中，有可能导致一个表的数据覆盖了另一个表。

分段是把虚拟内存划分为多个独立的地址空间，每个地址空间可以动态增长，互不影响。每个段可以单独进行控制，有助于保护和共享。

> 为什么要引进二级页表

为了加快虚拟地址到物理地址的转换，多数系统会引入一个转换检测缓冲区（TLB）的设备，通常又称为**快表**，当请求访问一个虚拟地址时，处理器检查是否缓存了该虚拟

地址的映射，如果命中则直接返回物理地址，否则就通过页表搜索对应的物理地址。



## 63-线程通讯方式

> 线程通信主要可以分为三种方式，分别为**共享内存、消息传递和管道流**

- 共享内存：线程之间共享程序的公共状态，线程之间通过读-写内存中的公共状态来隐式通信。

> volatile共享内存

- 消息传递：线程之间没有公共的状态，线程之间必须通过明确的发送信息来显示的进行通信。

> wait/notify等待通知方式
> join方式

- 管道流

> 管道输入/输出流的形式



## 64-SpringMVC的执行流程

![](image\屏幕截图 2024-01-23 214625.png)



## 65-MySQL什么时候索引会失效


















