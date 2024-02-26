# 面试题

* #### golang 中 make 和 new 的区别？（基本必问）

  > 所以从这里可以看的很明白了，二者都是内存的分配（堆上），但是`make`只用于slice、map以及channel的初始化（非零值）；而`new`用于类型的内存分配，并且内存置为零。所以在我们编写程序的时候，就可以根据自己的需要很好的选择了。
  >
  > `make`返回的还是这三个引用类型本身；而`new`返回的是指向类型的指针。

  对于变量的声明，我们可以通过关键字`var`。当我们使用`var`关键字对变量进行声明后，变量会有默认的值，例如整型是0，对于引用类型是nil.

  那我们来看看下面这个例子：

  ```go
  func main() {
  	var i *int
  	*i = 10
  	fmt.Println(i)
  }
  -------------------
  panic: runtime error: invalid memory address or nil pointer dereference
  ```

  从结果来看，对于引用类型，我们不仅要声明，还要申请内存空间。

  **第一个知识点是：`new`和`make`都是分配内存的关键字 **

  ### new

  > ```go
  > // The new built-in function allocates memory. The first argument is a type,
  > // not a value, and the value returned is a pointer to a newly
  > // allocated zero value of that type.
  > func new(Type) *Type
  > ```

  解决问题：

  ```go
  func main() {
  	var i *int
      i = new(int)
  	*i = 10
  	fmt.Println(i)
  }
  ```

  再看下一个例子

  ```go
  func main() {
      u:=new(user)
      u.lock.Lock()
      u.name = "张三"
      u.lock.Unlock()
  
      fmt.Println(u)
  }
  
  type user struct {
      lock sync.Mutex
      name string
      age int
  }
  ```

  这里的关键点在于`user`结构体中的引用属性不用初始化，就直接可以拿来使用。

  **是因为`new`它的返回值类型永远是指针，指向分配类型的内存空间**

  ### make

  `make`也是用于内存分配的，但是和`new`不同，它只用于`chan`、`map`以及切片的内存创建，而且它返回的类型就是这三个类型本身，而不是他们的指针类型，因为这三种类型就是引用类型，所以就没有必要返回他们的指针了。

  注意，因为这三种类型是引用类型，所以必须得初始化，但是不是置为零值，这个和`new`是不一样的。

  ```go
  func make(t Type, size ...IntegerType) Type
  ```

  从函数声明中可以看到，返回的还是该类型。

  

## 1 Go的垃圾回收机制？GMP模型？

Go的垃圾回收机制？GMP模型？（展盟，百度，滴滴，小米）

Golang如何优雅关闭一个channel？（展盟）

Go里面的map是怎么决绝hash冲突的？（展盟）

slice是引用传递还是值传递？slice 参数传递过去，修改之后，外部变量是否也会被修改？（展盟）

Go读写锁的概念？读的时候会影响别人的读么？读优先还是写优先？（展盟）

context的应用场景？（展盟）

select的作用？项目中怎么使用的？（展盟）

数组和切片的区别（柯莱特-外派小红书，优咔科技）

map是否是线程安全的，如何在Go中使用线程安全的map（柯莱特-外派小红书，优咔科技）

sync.map的原理（柯莱特-外派小红书，优咔科技）

Go数据类型有哪些？（优咔科技）

如何判断两个interface{}相等？（优咔科技）

Go map中删除一个key的内存是否会立即释放？（优咔科技）

init()方法的特性（优咔科技）

switch-case语句，强制执行下一个case（优咔科技）

encoding/json 包解码通过 HTTP 请求接收的 JSON 数据时，它会默认将所有数字解析为 float64 类型（优咔科技）

Go里面的类型断言？（优咔科技）

Go静态类型声明？（优咔科技）

sync包使用？（优咔科技，360）

gin的并发请求、错误处理、路由处理（优咔科技）

CSP并发模型（百度）

逃逸分析的介绍？逃逸分析怎么看（-gcflags = "-m -l"）?工作中是否用过逃逸分析解决问题？（百度）

对关闭的channel进行读写channel会发生什么？对关闭的channel写为什么会panic？（百度）

字符串转byte数组会发生内存拷贝么？为什么？（百度）

如何实现字符串转切片无内存拷贝（unsafe）？（百度）

Go语言channel的特性？channel阻塞信息是怎么处理的？channel底层实现？（360）


## .  数据库知识

- MySQL的事务隔离级别，可重复读是什么样的概念？（展盟，360）
- MySQL联合索引最左匹配原则（展盟）
- MySQL的慢查询是怎么解决的？（展盟，360，小米）
- Redis遍历key的命令，可否用keys命令?（展盟）
- MySQL的优化？（优咔科技）
- MonGoDB和MySQL的区别，MonGoDB的索引了解过么？（优咔科技）
- Redis的数据类型？（优咔科技，360）
- Redis持久化的方式（优咔科技）
- MySQL的索引，聚簇索引和非聚簇索引的区别？索引是用什么实现的（b+ tree）?（360）
- MySQL事务隔离级别（RU/RC/RR/S），可重复读是怎么实现的？幻读是怎么解决的？（360）
- Redis ZSet底层是怎么实现的（压缩链表、跳表）（360）
- Redis的持久化机制？RDB的原理（save、bgsave）？（360）

## 3.  Kafka

- kafka同步租户时如何防止信息丢失（事务：commit、autocommit）（展盟）

## 4.  Kubernetes相关知识

- 介绍istio相关概念（展盟，优咔科技，360）
- k8s的service，集群用的什么网络插件（calico、flannel）？（展盟）
- 聊聊云原生是什么？（优咔科技）
- 常见的容器运行时（优咔科技）
- 数据库是如何部署的(k8s的statefulset）（优咔科技）
- 介绍k8s master的组件（优咔科技）
- 容器和虚机的不同点？（优咔科技）
- k8s的源码看过么？（360）
- k8s创建一个Pod的全过程（小米）
- client-Go Informer机制全过程？（小米）
- Delta-FIFO和普通队列的区别？Store是什么？Reflector是什么？（小米）
- k8s官方sample-controller和kubebuilder生成controller有什么区别？（小米）

## 5. 服务治理与微服务架构

- kitex框架：服务治理（负载均衡、服务降级...）、可观测（链路追踪...）（展盟）
- istio是做数据面还是控制面?（优咔科技）
- Istio负载均衡的策略（优咔科技）
- Service mesh中南北向和东西向流量处理？（优咔科技）

1. Linux命令和系统知识

- Linux的awk、strace命令（展盟）
- linux根据服务名称查看端口号（netstat -tunlp | grep 服务名称）（柯莱特-外派小红书）
- linux根据服务名称查看pid（ps -aux | 服务名称）（柯莱特-外派小红书）
- linux进程间通信的方式（socket、管道.....）（柯莱特-外派小红书）

