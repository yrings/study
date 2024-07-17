# Go语言学习笔记

## Go语言的优势

![](.\image\Screenshot 2024-07-17 183949.png)

## 一、起源于发展

Go 通过以下的 Logo 来展示它的速度，并以囊地鼠 (Gopher) 作为它的吉祥物

这是一门完全开源的编程语言，因为它使用 BSD 授权许可，所以任何人都可以进行商业软件的开发而不需要支付任何费用。

> ### go语言的发展目标

在go语言之前，开发者们总是面临非常艰难的抉择，究竟是使用执行速度快但是编译速度并不理想的语言（如：C++），还是使用编译速度较快但执行效率不佳的语言（如：.NET、Java），或者说开发难度较低但执行速度一般的动态语言呢？显然，**Go 语言在这 3 个条件之间做到了最佳的平衡：快速编译，高效执行，易于开发。**

Go语言的主要目标是将**静态语言和安全性和高效性**与**动态语言的易开发性**进行有机结合，达到完美平衡，从而使编程变得更加有乐趣，而不是在艰难抉择中痛苦前行。

因此，Go 语言是一门**类型安全**和**内存安全**的编程语言。**虽然 Go 语言中仍有指针的存在，但并不允许进行指针运算。**

GO语言的另一个目标是对于**网络通信、并发和并行编程的极佳支持**，从而更好地利用大量的分布式和多核计算机。设计者通过 `goroutine` 这种轻量级线程的概念来实现这个目标，然后通过 `channel` 来实现各个 `goroutine` 之间的通信。他们实现了**分段栈增长**和`goroutine` 在线程基础上 **多路复用技术和自动化**

Go语言中另一个非常重要的特性就是它的**构建速度（编译和链接到机器代码的速度）**，这不仅极大地提升了开发者的生产力，同时也使得软件开发过程中的代码测试环节更加紧凑，不必浪费大量的时间在等待程序的构建上。

依赖管理是现今软件开发的一个重要组成部分，人们越来越需要一门具有严格的、简洁的依赖关系分析系统从而能够快速编译的编程语言。Go 语言采用**包模型**的根本原因，**这个模型通过严格的依赖关系检查机制来加快程序构建的速度**，提供了非常好的可量测性。

由于内存问题（通常称为内存泄漏）长期以来一直伴随着 C++ 的开发者们，Go 语言的设计者们认为内存管理不应该是开发人员所需要考虑的问题。因此尽管 Go 语言像其它静态语言一样执行本地代码，但它依旧运行在某种意义上的虚拟机，以此来实现高效快速的垃圾回收（使用了一个简单的标记-清除算法）。



> ### 指导设计原则

Go语言通过减少关键字的数量（25 个）来简化编码过程中的混乱和复杂度。干净、整齐和简洁的语法也能够提高程序的编译速度，因为这些关键字在编译过程中少到甚至不需要符号表来协助解析。

这些方面的工作都是为了减少编码的工作量，甚至可以与 Java 的简化程度相比较。

Go 语言有一种极简抽象艺术家的感觉，因为它只提供了一到两种方法来解决某个问题，这使得开发者们的代码都非常容易阅读和理解。众所周知，代码的可读性是软件工程里最重要的一部分（ **译者注：代码是写给人看的，不是写给机器看的** ）。

它不像 Ruby 那样通过实现过程来定义编码规范。作为一门具有明确编码规范的语言，它要求可以采用不同的编译器如 `gc `和` gccgo`进行编译工作，这对语言本身拥有更好的编码规范起到很大帮助。



> ### 语言的特性

在传统的面向对象语言中，使用面向对象编程技术显得非常臃肿，它们总是通过复杂的模式来构建庞大的类型层级，这违背了编程语言应该提升生产力的宗旨。

函数是 Go 语言中的基本构件，它们的使用方法非常灵活。在[第六章](https://github.com/unknwon/the-way-to-go_ZH_CN/blob/master/eBook/06.0.md)，我们会看到 Go 语言在函数式编程方面的基本概念。

o 语言使用静态类型，所以它是类型安全的一门语言，加上通过构建到本地代码，程序的执行速度也非常快。

**作为强类型语言，隐式的类型转换是不被允许的，记住一条原则：让所有的东西都是显式的。**

Go 语言其实也有一些动态语言的特性（通过关键字 `var`），所以它对那些逃离 Java 和 .Net 世界而使用 Python、Ruby、PHP 和 JavaScript 的开发者们也具有很大的吸引力。

**Go 语言支持交叉编译**，比如说你可以在运行 Linux 系统的计算机上开发运行 Windows 下运行的应用程序。这是第一门完全支持 UTF-8 的编程语言，这不仅体现在它可以处理使用 UTF-8 编码的字符串，就连它的源码文件格式都是使用的 UTF-8 编码。Go 语言做到了真正的国际化！



> ### 环境配置

Go 开发环境依赖于一些操作系统环境变量，你最好在安装 Go 之前就已经设置好他们。如果你使用的是 Windows 的话，你完全不用进行手动设置，Go 将被默认安装在目录 `c:/go` 下。这里列举几个最为重要的环境变量：

- **$GOROOT** 表示 Go 在你的电脑上的安装位置，它的值一般都是 `$HOME/go`，当然，你也可以安装在别的地方。
- **$GOARCH** 表示目标机器的处理器架构，它的值可以是 386、amd64 或 arm。
- **$GOOS** 表示目标机器的操作系统，它的值可以是 darwin、freebsd、linux 或 windows。
- **$GOBIN** 表示编译器和链接器的安装位置，默认是 `$GOROOT/bin`，如果你使用的是 Go 1.0.3 及以后的版本，一般情况下你可以将它的值设置为空，Go 将会使用前面提到的默认值。

目标机器是指你打算运行你的 Go 应用程序的机器。

Go 编译器支持交叉编译，也就是说你可以在一台机器上构建能够在不同操作系统和处理器架构上运行的应用程序，也就是说编写源代码的机器可以和目标机器有完全不同的特性（操作系统与处理器架构）。

为了区分本地机器和目标机器，你可以使用 `$GOHOSTOS` 和 `$GOHOSTARCH` 设置本地机器的操作系统名称和编译体系结构，这两个变量只有在进行交叉编译的时候才会用到，如果你不进行显示设置，他们的值会和本地机器（`$GOOS` 和 `$GOARCH`）一样。

- **$GOPATH** 默认采用和 `$GOROOT` 一样的值，但从 Go 1.1 版本开始，你必须修改为其它路径。它可以包含多个 Go 语言源码文件、包文件和可执行文件的路径，而这些路径下又必须分别包含三个规定的目录：`src`、`pkg` 和 `bin`，这三个目录分别用于存放源码文件、包文件和可执行文件。
- **$GOARM** 专门针对基于 arm 架构的处理器，它的值可以是 5 或 6，默认为 6。
- **$GOMAXPROCS** 用于设置应用程序可使用的处理器个数与核数，

> ### Go 运行时 (runtime)

尽管 Go 编译器产生的是本地可执行代码，这些代码仍旧运行在 Go 的 runtime（这部分的代码可以在 `runtime` 包中找到）当中。这个 runtime 类似 Java 和 .NET 语言所用到的虚拟机，它负责管理包括内存分配、垃圾回收（[第 10.8 节](https://github.com/unknwon/the-way-to-go_ZH_CN/blob/master/eBook/10.8.md)）、栈处理、goroutine、channel、切片 (slice)、map 和反射 (reflection) 等等。

runtime 主要由 C 语言编写（Go 1.5 开始自举），并且是每个 Go 包的最顶级包。你可以在目录 [`$GOROOT/src/runtime`](https://github.com/golang/go/tree/master/src/runtime) 中找到相关内容。

**垃圾回收器** Go 拥有简单却高效的标记-清除回收器。它的主要思想来源于 IBM 的可复用垃圾回收器，旨在打造一个高效、低延迟的并发回收器。目前 gccgo 还没有回收器，同时适用 gc 和 gccgo 的新回收器正在研发中。使用一门具有垃圾回收功能的编程语言不代表你可以避免内存分配所带来的问题，分配和回收内容都是消耗 CPU 资源的一种行为。

Go 的可执行文件都比相对应的源代码文件要大很多，这恰恰说明了 Go 的 runtime 嵌入到了每一个可执行文件当中。当然，在部署到数量巨大的集群时，较大的文件体积也是比较头疼的问题。但总的来说，Go 的部署工作还是要比 Java 和 Python 轻松得多。因为 Go 不需要依赖任何其它文件，它只需要一个单独的静态文件，这样你也不会像使用其它语言一样在各种不同版本的依赖文件之间混淆。



## 二、基础语法

* ### 1. 如何高效地拼接字符串

  Go语言中，字符串是只读的，也就意味着每次拼接字符串都会创建新的字符串。如果要多次拼接字符串，应该使用`stirng.Builder`，最小化内存拷贝次数

  ```go
  var str string.Builder
  for i := 0; i < 1000; i++ {
      str.WriteString("a")
  }
  fmt.Println(str.String())
  ```

* ### 2. 什么是 rune 类型

  ASCII 码只需要 7 bit 就可以完整地表示，但只能表示英文字母在内的128个字符，为了表示世界上大部分的文字系统，发明了 Unicode， 它是ASCII的超集，包含世界上书写系统中存在的所有字符，并为每个代码分配一个标准编号（称为Unicode CodePoint），在 Go 语言中称之为 rune，是 int32 类型的别名。

  Go 语言中，字符串的底层表示是 byte (8 bit) 序列，而非 rune (32 bit) 序列。例如下面的例子中 `语` 和 `言` 使用 UTF-8 编码后各占 3 个 byte，因此 `len("Go语言")` 等于 8，当然我们也可以将字符串转换为 rune 序列。

  ```go
  fmt.Println(len("Go语言")) // 8
  fmt.Println(len([]rune("Go语言"))) // 4
  ```

* ### 3. 如何判断 map 中是否包含某个key

  ```go
  func main() {
  	dict := make(map[string]string)
  	dict["foo"] = "testdata"
  
  	if val, ok := dict["foo"]; ok {
  		fmt.Println("value:", val)
  	}
  }
  ```

* ### 4. 如何交换 2 个变量的值

  ```
  a, b := "A", "B"
  a, b = b, a
  fmt.Println(a, b) // B A
  ```

* ### 5. Go 语言 tag 的用处

  tag 可以理解为 struct 字段的注解，可以用来定义字段的一个或多个属性。框架/工具可以通过反射获取到某个字段定义的属性，采取相应的处理方式。tag 丰富了代码的语义，增强了灵活性。

  ```go
  package main
  
  import "fmt"
  import "encoding/json"
  
  type Stu struct {
  	Name string `json:"stu_name"`
  	ID   string `json:"stu_id"`
  	Age  int    `json:"-"`
  }
  
  func main() {
  	buf, _ := json.Marshal(Stu{"Tom", "t001", 18})
  	fmt.Printf("%s\n", buf)
  }
  ```

  这个例子使用 tag 定义了结构体字段与 json 字段的转换关系，Name -> stu_name, ID -> stu_id，忽略 Age 字段。很方便地实现了 Go 结构体与不同规范的 json 文本之间的转换。

* ### 6. 字符串打印时，`%v` 和 `%+v` 的区别

  `%v` 和 `%+v` 都可以用来打印 struct 的值，区别在于前者仅打印各个字段的值，后者还会打印各个字段的名称。

  ```go
  type Stu struct {
  	Name string
  }
  func main() {
  	fmt.Printf("%v\n", Stu{"Tom"}) // {Tom}
  	fmt.Printf("%+v\n", Stu{"Tom"}) // {Name:Tom}
  }
  ```

* ### 7. Go语言枚举

  ```go
  type StuType int32
  
  const (
  	Type1 StuType = iota
  	Type2
  	Type3
  	Type4
  )
  
  func main() {
  	fmt.Println(Type1, Type2, Type3, Type4)
  }
  
  ```

* ### 8. 空 `struct{}` 的用途

  使用空结构体 struct{} 可以节省内存，一般作为占位符使用，表明这里并不需要一个值。

  ```
  fmt.Println(unsafe.Sizeof(struct{}{})) // 0
  ```

  比如使用 map 表示集合时，只关注 key，value 可以使用 struct{} 作为占位符。如果使用其他类型作为占位符，例如 int，bool，不仅浪费了内存，而且容易引起歧义。

  ```go
  type Set map[string]struct{}
  
  func main() {
  	set := make(Set)
  
  	for _, item := range []string{"A", "A", "B", "C"} {
  		set[item] = struct{}{}
  	}
  	fmt.Println(len(set)) // 3
  	if _, ok := set["A"]; ok {
  		fmt.Println("A exists") // A exists
  	}
  }
  ```

  再比如，使用信道(channel)控制并发时，我们只是需要一个信号，但并不需要传递值，这个时候，也可以使用 struct{} 代替。

  ```go
  func main() {
  	ch := make(chan struct{}, 1)
  	go func() {
  		<-ch
  		// do something
  	}()
  	ch <- struct{}{}
  	// ...
  }
  ```

  再比如，声明只包含方法的结构体。

  ```go
  type Lamp struct{}
  
  func (l Lamp) On() {
          println("On")
  
  }
  func (l Lamp) Off() {
          println("Off")
  }
  ```



## 三、实现原理

* ### 1. init() 函数是什么时候执行的

  `init()` 函数是 Go 程序初始化的一部分。初始化优先于 main 函数，由 runtime 初始化每个导入的包，初始化顺序不是按照从上到下的导入顺序，而是按照解析的依赖关系，没有依赖的包最先初始化

  每个包首先初始化包作用域的常量和变量（常量优先于变量），然后执行包的 `init()` 函数。同一个包，甚至是同一个源文件可以有多个 `init()` 函数。`init()` 函数没有入参和返回值，不能被其他函数调用，同一个包内多个 `init()` 函数的执行顺序不作保证。

  一句话总结：

  ```
   import –> const –> var –> `init()` –> `main()`
  ```



* ### 2. Go 语言的局部变量分配在栈上还是堆上

  由编译器决定。Go 语言编译器会自动决定把一个变量放在堆上还是放在栈上。编译器会做**逃逸分析（escape analysis）**, 当发现变量的作用域没有超出函数范围，就可以在栈上，反之则必须分配在堆上。

  ```go
  func foo() *int {
  	v := 11
  	return &v
  }
  
  func main() {
  	m := foo()
  	println(*m) // 11
  }
  ```

  `foo()`函数中，如果 v 分配在栈上，foo函数返回时，`&v` 就会被回收，但是这段函数是能够正常运行的。Go 编译器发现 v 的引用脱离了 foo 的作用域，会将其分配在堆上。因此，main 函数中仍能够正常访问该值。



* ### 3. 两个 interface 可以比较吗

  Go 语言中，interface 的内部实现包含了 2 个字段，类型 `T` 和 值 `V`，interface 可以使用 `==` 或 `!=` 比较。2 个 interface 相等有以下 2 种情况

  1. 两个 interface 均等于 nil（此时 V 和 T 都处于 unset 状态）
  2. 类型 T 相同，且对应的值 V 相等。

  ```go
  type Stu struct {
  	Name string
  }
  
  type StuInt interface{}
  
  func main() {
  	var stu1, stu2 StuInt = &Stu{"Tom"}, &Stu{"Tom"}
  	var stu3, stu4 StuInt = Stu{"Tom"}, Stu{"Tom"}
  	fmt.Println(stu1 == stu2) // false
  	fmt.Println(stu3 == stu4) // true
  }
  ```

  `stu1` 和 `stu2` 对应的类型是 `*Stu`，值是 Stu 结构体的地址，两个地址不同，因此结果为 false。
  `stu3` 和 `stu4` 对应的类型是 `Stu`，值是 Stu 结构体，且各字段相等，因此结果为 true。



* ### 4. 两个 nil 可能不相等吗

  可能。

  接口（interface）是对非接口值（例如指针、结构体等）的封装，内部实现包含两个字段，类型 `T` 和 值 `V`。一个接口等于 nil，当且仅当 T 和 V 处于 unset 状态（T=nil，V is unset）。

  - ##### 两个接口值比较时，会先比较 T，再比较 V。

  - ##### 接口值与非接口值比较时，会先将非接口值尝试转换为接口值，再比较。

  ```go
  func main() {
  	var p *int = nil
  	var i interface{} = p
  	fmt.Println(i == p) // true
  	fmt.Println(p == nil) // true
  	fmt.Println(i == nil) // false
  }
  ```

  上面这个例子中，将一个 nil 非接口值 p 赋值给接口 i，此时，i 的内部字段为`(T=*int, V=nil)`，i 与 p 作比较时，将 p 转换为接口后再比较，因此 `i == p`，p 与 nil 比较，直接比较值，所以 `p == nil`。

  但是当 i 与 nil 比较时，会将 nil 转换为接口 `(T=nil, V=nil)`，与i `(T=*int, V=nil)` 不相等，因此 `i != nil`。因此 V 为 nil ，但 T 不为 nil 的接口不等于 nil



* ### 5. 简述 Go 语言GC的工作原理

  最常见的垃圾回收算法有**标记清除(Mark-Sweep)** 和引用计数(Reference Count)，Go 语言采用的是标记清除算法。并在此基础上使用了**三色标记法**和**写屏障技术**，提高了效率。

  > #### 标记清除（Mark-Sweep）

  标记清除收集器是跟踪式垃圾收集器，其执行过程可以分成标记（Mark）和清除（Sweep）两个阶段：

  - 标记阶段 — 从根对象出发查找并标记堆中所有存活的对象；
  - 清除阶段 — 遍历堆中的全部对象，回收未被标记的垃圾对象并将回收的内存加入空闲链表。

  标记清除算法的一大问题是在标记期间，需要暂停程序（Stop the world，STW），标记结束之后，用户程序才可以继续执行。为了能够异步执行，减少 STW 的时间，Go 语言采用了三色标记法。

  > #### 三色标记法

  三色标记算法将程序中的对象分成白色、黑色和灰色三类。

  - 白色：不确定对象。
  - 灰色：存活对象，子对象待处理。
  - 黑色：存活对象。

  1. 标记开始时，所有对象加入白色集合（这一步需 STW ）。

  2. 首先将根对象标记为灰色，加入灰色集合，垃圾搜集器取出一个灰色对象，将其标记为黑色，并将其指向的对象标记为灰色，加入灰色集合。重复这个过程，直到灰色集合为空为止，标记阶段结束。
  3. 那么白色对象即可需要清理的对象，而黑色对象均为根可达的对象，不能被清理。

  三色标记法因为多了一个白色的状态来存放不确定对象，所以后续的标记阶段可以并发地执行。当然并发执行的代价是可能会造成一些遗漏，因为那些早先被标记为黑色的对象可能目前已经是不可达的了。所以三色标记法是一个 false negative（假阴性）的算法。

  **三色标记法并发执行仍存在一个问题，即在 GC 过程中，对象指针发生了改变。**比如下面的例子：

  ```
  A (黑) -> B (灰) -> C (白) -> D (白)
  ```

  正常情况下，D 对象最终会被标记为黑色，不应被回收。但在标记和用户程序并发执行过程中，用户程序删除了 C 对 D 的引用，而 A 获得了 D 的引用。标记继续进行，D 就没有机会被标记为黑色了（A 已经处理过，这一轮不会再被处理）。

  ```
  A (黑) -> B (灰) -> C (白) 
    ↓
   D (白)
  ```

  > #### 写屏障技术

  为了解决这个问题，Go 使用了内存屏障技术，它是在用户程序读取对象、创建新对象以及更新对象指针时执行的一段代码，类似于一个钩子。垃圾收集器使用了写屏障（Write Barrier）技术，当对象新增或更新时，会将其着色为灰色。这样即使与用户程序并发执行，对象的引用发生改变时，垃圾收集器也能正确处理了。

  一次完整的 GC 分为四个阶段：

  - 1）标记准备(Mark Setup，需 STW)，打开写屏障(Write Barrier)
- 2）使用三色标记法标记（Marking, 并发）
  - 3）标记结束(Mark Termination，需 STW)，关闭写屏障。
  - 4）清理(Sweeping, 并发)
  
  

* ### 6. 函数返回局部变量的指针是否安全？

  这在 Go 中是安全的，Go 编译器将会对每个局部变量进行逃逸分析。如果发现局部变量的作用域超出该函数，则不会将内存分配在栈上，而是分配在堆上

  

* ### 7. 非接口的任意类型T() 都能够调用 `*T` 的方法吗？反过来呢？

  - 一个T类型的值可以调用为`*T`类型声明的方法，**但是仅当此T的值是可寻址(addressable) 的情况下。**编译器在调用指针属主方法前，会自动取此T值的地址。因为不是任何T值都是可寻址的，所以并非任何T值都能够调用为类型`*T`声明的方法。
  - 反过来，一个`*T`类型的值可以调用为类型T声明的方法，这是因为解引用指针总是合法的。事实上，你可以认为对于每一个为类型 T 声明的方法，编译器都会为类型`*T`自动隐式声明一个同名和同签名的方法。

  哪些值是不可寻址的呢？

  - 字符串中的字节；
  - map 对象中的元素（slice 对象中的元素是可寻址的，slice的底层是数组）；
  - 常量；
  - 包级别的函数等。

  举一个例子，定义类型 T，并为类型 `*T` 声明一个方法 `hello()`，变量 t1 可以调用该方法，但是常量 t2 调用该方法时，会产生编译错误。

  ```go
  type T string
  
  func (t *T) hello() {
  	fmt.Println("hello")
  }
  
  func main() {
  	var t1 T = "ABC"
  	t1.hello() // hello
  	const t2 T = "ABC"
  	t2.hello() // error: cannot call pointer method on t
  }
  ```

## 四、并发编程

* ### 无缓冲的 channel 和 有缓冲的 channel 的区别

  对于无缓冲的 channel，发送方将阻塞该信道，直到接收方从该信道接收到数据为止，而接收方也将阻塞该信道，直到发送方将数据发送到该信道中为止。

  对于有缓存的 channel，发送方在没有空插槽（缓冲区使用完）的情况下阻塞，而接收方在信道为空的情况下阻塞。



* ### 什么是协程泄露（Goroutine Leak）

  协程泄露是指协程创建后，长时间得不到释放，并且还在不断地创建新的协程，最终导致内存耗尽，程序崩溃。常见的导致协程泄露的场景有以下几种：

  - 缺少接收器，导致发送阻塞

  这个例子中，每执行一次 query，则启动1000个协程向信道 ch 发送数字 0，但只接收了一次，导致 999 个协程被阻塞，不能退出。

  ```go
  func query() int {
  	ch := make(chan int)
  	for i := 0; i < 1000; i++ {
  		go func() { ch <- 0 }()
  	}
  	return <-ch
  }
  
  func main() {
  	for i := 0; i < 4; i++ {
  		query()
  		fmt.Printf("goroutines: %d\n", runtime.NumGoroutine())
  	}
  }
  // goroutines: 1001
  // goroutines: 2000
  // goroutines: 2999
  // goroutines: 3998
  ```

  - 缺少发送器，导致接收阻塞

  那同样的，如果启动 1000 个协程接收信道的信息，但信道并不会发送那么多次的信息，也会导致接收协程被阻塞，不能退出。

  - 死锁(dead lock)

  两个或两个以上的协程在执行过程中，由于竞争资源或者由于彼此通信而造成阻塞，这种情况下，也会导致协程被阻塞，不能退出。

  - 无限循环(infinite loops)

  这个例子中，为了避免网络等问题，采用了无限重试的方式，发送 HTTP 请求，直到获取到数据。那如果 HTTP 服务宕机，永远不可达，导致协程不能退出，发生泄漏。

  ```go
  func request(url string, wg *sync.WaitGroup) {
  	i := 0
  	for {
  		if _, err := http.Get(url); err == nil {
  			// write to db
  			break
  		}
  		i++
  		if i >= 3 {
  			break
  		}
  		time.Sleep(time.Second)
  	}
  	wg.Done()
  }
  
  func main() {
  	var wg sync.WaitGroup
  	for i := 0; i < 1000; i++ {
  		wg.Add(1)
  		go request(fmt.Sprintf("https://127.0.0.1:8080/%d", i), &wg)
  	}
  	wg.Wait()
  }
  ```

* ### Go 可以限制运行时操作系统线程的数量吗

  可以使用环境变量 `GOMAXPROCS` 或 `runtime.GOMAXPROCS(num int)` 设置，例如：

  ```go
  runtime.GOMAXPROCS(1) // 限制同时执行Go代码的操作系统线程数为 1
  ```

  从官方文档的解释可以看到，`GOMAXPROCS` 限制的是同时执行用户态 Go 代码的操作系统线程的数量，但是对于被系统调用阻塞的线程数量是没有限制的。`GOMAXPROCS` 的默认值等于 CPU 的逻辑核数，同一时间，一个核只能绑定一个线程，然后运行被调度的协程。因此对于 CPU 密集型的任务，若该值过大，例如设置为 CPU 逻辑核数的 2 倍，会增加线程切换的开销，降低性能。对于 I/O 密集型应用，适当地调大该值，可以提高 I/O 吞吐率。



## 五、代码输出

* ### 常量与变量

  #### 1.

  ```go
  func main() {
  	const (
  		a, b = "golang", 100
  		d, e
  		f bool = true
  		g
  	)
  	fmt.Println(d, e, g)
  }
  -----------------------
   golang 100 true
  ```

  在同一个 const group 中，如果常量定义与前一行的定义一致，则可以省略类型和值。编译时，会按照前一行的定义自动补全。

  #### 2.

  ```go
  func main() {
  	const N = 100
  	var x int = N
  
  	const M int32 = 100
  	var y int = M
  	fmt.Println(x, y)
  }
  --------------------
  编译失败：cannot use M (type int32) as type int in assignment
  ```

  

  Go 语言中，常量分为无类型常量和有类型常量两种，`const N = 100`，属于无类型常量，赋值给其他变量时，如果字面量能够转换为对应类型的变量，则赋值成功，例如，`var x int = N`。但是对于有类型的常量 `const M int32 = 100`，赋值给其他变量时，需要类型匹配才能成功，所以显示地类型转换：

  ```go
  var y int = int(M)
  ```

  #### 3. 

  ```go
  func main() {
  	var a int8 = -1
  	var b int8 = -128 / a
  	fmt.Println(b)
  }
  -----------------
  -128
  ```

  int8 能表示的数字的范围是 [-2^7, 2^7-1]，即 [-128, 127]。-128 是无类型常量，转换为 int8，再除以变量 -1，结果为 128，常量除以变量，结果是一个变量。变量转换时允许溢出，符号位变为1，转为补码后恰好等于 -128。

  对于有符号整型，最高位是是符号位，计算机用补码表示负数。补码 = 原码取反加

  #### 4. 

  ```go
  func main() {
  	const a int8 = -1
  	var b int8 = -128 / a
  	fmt.Println(b)
  }
  ----------------------
  编译失败：constant 128 overflows int8
  ```

  -128 和 a 都是常量，在编译时求值，-128 / a = 128，两个常量相除，结果也是一个常量，常量类型转换时不允许溢出，因而编译失败。

* ### 作用域

  #### 1. 

  ```go
  func main() {
  	var err error
  	if err == nil {
  		err := fmt.Errorf("err")
  		fmt.Println(1, err)
  	}
  	if err != nil {
  		fmt.Println(2, err)
  	}
  }
  -------------------------
  1 err
  ```

  `:=` 表示声明并赋值，`=` 表示仅赋值。

  变量的作用域是大括号，因此在第一个 if 语句 `if err == nil` 内部重新声明且赋值了与外部变量同名的局部变量 err。对该局部变量的赋值不会影响到外部的 err。因此第二个 if 语句 `if err != nil` 不成立。所以只打印了 `1 err`

* ### defer 延迟调用

  #### 1.

  ```go
  type T struct{}
  
  func (t T) f(n int) T {
  	fmt.Print(n)
  	return t
  }
  
  func main() {
  	var t T
  	defer t.f(1).f(2)
  	fmt.Print(3)
  }
  ---------------------
  132
  ```

  defer 延迟调用时，需要保存函数指针和参数，因此链式调用的情况下，除了最后一个函数方法外的函数方法都会在调用时直接执行。也就是说 `t.f(1)` 直接执行，然后执行 `fmt.Print(3)`，最后函数返回时再执行 `.f(2)`，因此输出是 132

  #### 2.

  ```go
  func f(n int) {
  	defer fmt.Println(n)
  	n += 100
  }
  
  func main() {
  	f(1)
  }
  -------------------
  1
  ```

  打印 1 而不是 101。defer 语句执行时，会将需要延迟调用的函数和参数保存起来，也就是说，执行到 defer 时，参数 n(此时等于1) 已经被保存了。因此后面对 n 的改动并不会影响延迟函数调用的结果

  #### 3.

  ```go
  func main() {
  	n := 1
  	defer func() {
  		fmt.Println(n)
  	}()
  	n += 100
  }
  -----------------
  101
  ```

  匿名函数没有通过传参的方式将 n 传入，因此匿名函数内的 n 和函数外部的 n 是同一个，延迟执行时，已经被改变为 101









## 六、高性能编程

### 一、性能分析

#### 1. benchmark 基准测试

> #### 1.1 benchma的使用

GO语言内置了支持 benchmark 的 `testing` 库，接下来看一个简单的例子：

创建一个新的mod，新增 `fib.go` 文件，实现计算第 N 个斐波那契数。

```go
// fib.go
package main

func fib(n int) int {
	if n == 0 || n == 1 {
		return n
	}
	return fib(n-2) + fib(n-1)
}
// fib_test.go
package main

import "testing"

func BenchmarkFib(b *testing.B) {
	for n := 0; n < b.N; n++ {
		fib(30) // run fib(30) b.N times
	}
}
```

- benchmark 和普通的单元测试用例一样，都位于 `_test.go` 文件中。
- 函数名以 `Benchmark` 开头，参数是 `b *testing.B`。和普通的单元测试用例很像，单元测试函数名以 `Test` 开头，参数是 `t *testing.T`。

> #### 1.2 运行用例

`go test <module name>/<package name>` 用来运行某个 package 内的所有测试用例。

- 运行当前 package 内的用例：`go test example` 或 `go test .`
- 运行子 package 内的用例： `go test example/<package name>` 或 `go test ./<package name>`
- 如果想递归测试当前目录下的所有的 package：`go test ./...` 或 `go test example/...`。

`go test` 命令默认不运行 benchmark 用例的，如果我们想运行 benchmark 用例，则需要加上 `-bench` 参数。例如：

```shell
$ go test -bench .
goos: darwin
goarch: amd64
pkg: example
BenchmarkFib-8               200           5865240 ns/op
PASS
ok      example 1.782s
```

`-bench` 参数支持传入一个正则表达式，匹配到的用例才会得到执行，例如，只运行以 `Fib` 结尾的 benchmark 用例：

```shell
$ go test -bench='Fib$' .
goos: darwin
goarch: amd64
pkg: example
BenchmarkFib-8               202           5980669 ns/op
PASS
ok      example 1.813s
```

> #### 1.3 benchmark 是如何工作的

**benchmark 用例的参数 `b *testing.B`，有个属性 `b.N` 表示这个用例需要运行的次数。`b.N` 对于每个用例都是不一样的。**

那这个值是如何决定的呢？`b.N` 从 1 开始，如果该用例能够在 1s 内完成，`b.N` 的值便会增加，再次执行。`b.N` 的值大概以 1, 2, 3, 5, 10, 20, 30, 50, 100 这样的序列递增，越到后面，增加得越快。我们仔细观察上述例子的输出：

```
BenchmarkFib-8               202           5980669 ns/op
```

`BenchmarkFib-8 `中的 `-8` 即 `GOMAXPROCS`，默认等于 CPU 核数。可以通过 `-cpu` 参数改变 `GOMAXPROCS`，`-cpu` 支持传入一个列表作为参数，例如：

```shell
$ go test -bench='Fib$' -cpu=2,4 .
goos: darwin
goarch: amd64
pkg: example
BenchmarkFib-2               206           5774888 ns/op
BenchmarkFib-4               205           5799426 ns/op
PASS
ok      example 3.563s
```

在这个例子中，改变 CPU 的核数对结果几乎没有影响，因为这个 Fib 的调用是串行的。

`202` 和 `5980669 ns/op` 表示用例执行了 202 次，每次花费约 0.006s。总耗时比 1s 略多



> #### 1.4 提高准确度

```shell
# 执行5s，但实际上执行时间要大于5秒，由于测试用例需要编译、执行、销毁也需要时间
$ go test -bench='Fib$' -benchtime=5s .

# 执行50次
$ go test -bench='Fib$' -benchtime=50x .

# -count 参数可以用来设置轮数
$ go test -bench='Fib$' -benchtime=5s -count=3 .
goos: darwin
goarch: amd64
pkg: example
BenchmarkFib-8               975           5946624 ns/op
BenchmarkFib-8              1023           5820582 ns/op
BenchmarkFib-8               961           6096816 ns/op
PASS
ok      example 19.463s
```



> #### 1.5 内存分配情况

`-benchmem` 参数可以度量内存分配的次数。内存分配次数也性能也是息息相关的，例如不合理的切片容量，将导致内存重新分配，带来不必要的开销。

在下面的例子中，`generateWithCap` 和 `generate` 的作用是一致的，生成一组长度为 n 的随机序列。唯一的不同在于，`generateWithCap` 创建切片时，将切片的容量(capacity)设置为 n，这样切片就会一次性申请 n 个整数所需的内存。

```go
package main

import (
    "math/rand"
    "testing"
    "time"
)

func generateWithCap(n int) []int {
    rand.New(rand.NewSource(time.Now().UnixNano()))
    nums := make([]int, 0, n)
    for i := 0; i < n; i++ {
       nums = append(nums, rand.Int())
    }
    return nums
}
func generate(n int) []int {
    rand.New(rand.NewSource(time.Now().UnixNano()))
    nums := make([]int, 0)
    for i := 0; i < n; i++ {
       nums = append(nums, rand.Int())
    }
    return nums
}
func BenchmarkGenerateWithCap(b *testing.B) {
    for n := 0; n < b.N; n++ {
       generateWithCap(1000000)
    }
}

func BenchmarkGenerate(b *testing.B) {
    for n := 0; n < b.N; n++ {
       generate(1000000)
    }
}
```

测试:

```shell
go test -bench='Generate' -benchmem .
BenchmarkGenerateWithCap-16          273           4451378 ns/op         8003587 B/op          1 allocs/op
BenchmarkGenerate-16                 141           8381807 ns/op        41678166 B/op         38 allocs/op
PASS
ok      benchmark       3.721s
```



> #### 1.6 测试不同的输入

不同的函数复杂度不同，O(1)，O(n)，O(n^2) 等，利用 benchmark 验证复杂度一个简单的方式，是构造不同的输入。对刚才的 benchmark 稍作改造，便能够达到目的。

```go
func BenchmarkGenerate1000(b *testing.B)    { benchmarkGenerate(1000, b) }
func BenchmarkGenerate10000(b *testing.B)   { benchmarkGenerate(10000, b) }
func BenchmarkGenerate100000(b *testing.B)  { benchmarkGenerate(100000, b) }
func BenchmarkGenerate1000000(b *testing.B) { benchmarkGenerate(1000000, b) }
```

查看运行时间是否是线性变化的



> #### 1.7 ResetTimer

如果在 benchmark 开始前，需要一些准备工作，如果准备工作比较耗时，则需要将这部分代码的耗时忽略掉。比如下面的例子：

```go
func BenchmarkFib(b *testing.B) {
	time.Sleep(time.Second * 3) // 模拟耗时准备任务
	for n := 0; n < b.N; n++ {
		fib(30) // run fib(30) b.N times
	}
}
```

我们就需要用 `ResetTimer`屏蔽到耗时的准备任务

```go
func BenchmarkFib(b *testing.B) {
	time.Sleep(time.Second * 3) // 模拟耗时准备任务
    b.ResetTimer() // 重置定时器
    for n := 0; n < b.N; n++ {
		fib(30) // run fib(30) b.N times
	}
}
```



> #### 1.8 StopTimer & StartTimer

使用类似于`ResetTimer()`





#### 2. pprof 性能分析

> benchmark(基准测试) 可以度量某个函数或方法的性能，也就是说，如果我们知道性能的瓶颈点在哪里，benchmark 是一个非常好的方式。但是面对一个未知的程序，如何去分析这个程序的性能，并找到瓶颈点呢？
>
> [pprof](https://github.com/google/pprof) 就是用来解决这个问题的。pprof 包含两部分：
>
> - 编译到程序中的 `runtime/pprof` 包
> - 性能剖析工具 `go tool pprof`



> #### 2.1 性能分析类型

* #### 1 CPU 性能分析

CPU 性能分析(CPU profiling) 是最常见的性能分析类型。

启动 CPU 分析时，运行时(runtime) 将每隔 10ms 中断一次，记录此时正在运行的协程(goroutines) 的堆栈信息。

程序运行结束后，可以分析记录的数据找到最热代码路径(hottest code paths)。

> Compiler hot paths are code execution paths in the compiler in which most of the execution time is spent, and which are potentially executed very often.
> – [What’s the meaning of “hot codepath”](https://english.stackexchange.com/questions/402436/whats-the-meaning-of-hot-codepath-or-hot-code-path)

一个函数在性能分析数据中出现的次数越多，说明执行该函数的代码路径(code path)花费的时间占总运行时间的比重越大。

* #### 2 内存性能分析

内存性能分析(Memory profiling) 记录堆内存分配时的堆栈信息，忽略栈内存分配信息。

内存性能分析启用时，默认每1000次采样1次，这个比例是可以调整的。因为内存性能分析是基于采样的，因此基于内存分析数据来判断程序所有的内存使用情况是很困难的。

* #### 3 阻塞性能分析

阻塞性能分析(block profiling) 是 Go 特有的。

阻塞性能分析用来记录一个协程等待一个共享资源花费的时间。在判断程序的并发瓶颈时会很有用。阻塞的场景包括：

- 在没有缓冲区的信道上发送或接收数据。
- 从空的信道上接收数据，或发送数据到满的信道上。
- 尝试获得一个已经被其他协程锁住的排它锁。

一般情况下，当所有的 CPU 和内存瓶颈解决后，才会考虑这一类分析。

* #### 4 锁性能分析

锁性能分析(mutex profiling) 与阻塞分析类似，但专注于因为锁竞争导致的等待或延时。



> #### 2.2 CPU 性能分析





### 二、常用数据结构

#### 1. 字符串拼接性能及原理

> 在 Go 语言中，字符串(string) 是不可变的，拼接字符串事实上是创建了一个新的字符串对象。如果代码中存在大量的字符串拼接，对性能会产生严重的影响。

> #### 1.1 常见的拼接方式

为了避免编译器优化，我们首先实现一个生成长度为 n 的随机字符串的函数。

```go
const letterBytes = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"

func randomString(n int) string {
	b := make([]byte, n)
	for i := range b {
		b[i] = letterBytes[rand.Intn(len(letterBytes))]
	}
	return string(b)
}
```

然后利用这个函数生成字符串 `str`，然后将 `str` 拼接 N 次。在 Go 语言中，常见的字符串拼接方式有如下 5 种：

- 使用 `+`

```go
func plusConcat(n int, str string) string {
	s := ""
	for i := 0; i < n; i++ {
		s += str
	}
	return s
}
```

- 使用 `fmt.Sprintf`

```go
func sprintfConcat(n int, str string) string {
	s := ""
	for i := 0; i < n; i++ {
		s = fmt.Sprintf("%s%s", s, str)
	}
	return s
}
```

- 使用 `strings.Builder`

```go
func builderConcat(n int, str string) string {
	var builder strings.Builder
	for i := 0; i < n; i++ {
		builder.WriteString(str)
	}
	return builder.String()
}
```

- 使用 `bytes.Buffer`

```go
func bufferConcat(n int, s string) string {
	buf := new(bytes.Buffer)
	for i := 0; i < n; i++ {
		buf.WriteString(s)
	}
	return buf.String()
}
```

- 使用 `[]byte`

```go
func byteConcat(n int, str string) string {
	buf := make([]byte, 0)
	for i := 0; i < n; i++ {
		buf = append(buf, str...)
	}
	return string(buf)
}
```

如果长度是可预知的，那么创建 `[]byte` 时，我们还可以预分配切片的容量(cap)。

```go
func preByteConcat(n int, str string) string {
	buf := make([]byte, 0, n*len(str))
	for i := 0; i < n; i++ {
		buf = append(buf, str...)
	}
	return string(buf)
}
```

> make([]byte, 0, n*len(str)) 第二个参数是长度，第三个参数是容量(cap)，切片创建时，将预分配 cap 大小的内存。

* strings.join

  `string.join` 也是基于 `strings.builder`来实现的，并且可以自定义分隔符，在join方法内调用了`b.Grow(n)`方法，这个是进行初步的容量分配，而前面计算的n的长度就是我们要拼接的slice的长度，因为我们传入切片长度固定，所以提前进行容量分配可以减少内存分配。

  ```go
  func main() {
      sa := []string{"a", "b", "c"}
      ret := string.Join(sa, "")
  }
  ```

  

> #### 1.2 benchmark 性能比拼

每个 benchmark 用例中，生成了一个长度为 10 的字符串，并拼接 1w 次。

```go
func benchmark(b *testing.B, f func(int, string) string) {
	var str = randomString(10)
	for i := 0; i < b.N; i++ {
		f(10000, str)
	}
}

func BenchmarkPlusConcat(b *testing.B)    { benchmark(b, plusConcat) }
func BenchmarkSprintfConcat(b *testing.B) { benchmark(b, sprintfConcat) }
func BenchmarkBuilderConcat(b *testing.B) { benchmark(b, builderConcat) }
func BenchmarkBufferConcat(b *testing.B)  { benchmark(b, bufferConcat) }
func BenchmarkByteConcat(b *testing.B)    { benchmark(b, byteConcat) }
func BenchmarkPreByteConcat(b *testing.B) { benchmark(b, preByteConcat) }
```

运行该用例：

```
$ go test -bench="Concat$" -benchmem .
goos: darwin
goarch: amd64
pkg: example
BenchmarkPlusConcat-8         19      56 ms/op   530 MB/op   10026 allocs/op
BenchmarkSprintfConcat-8      10     112 ms/op   835 MB/op   37435 allocs/op
BenchmarkBuilderConcat-8    8901    0.13 ms/op   0.5 MB/op      23 allocs/op
BenchmarkBufferConcat-8     8130    0.14 ms/op   0.4 MB/op      13 allocs/op
BenchmarkByteConcat-8       8984    0.12 ms/op   0.6 MB/op      24 allocs/op
BenchmarkPreByteConcat-8   17379    0.07 ms/op   0.2 MB/op       2 allocs/op
PASS
ok      example 8.627s
```

从基准测试的结果来看，使用 `+` 和 `fmt.Sprintf` 的效率是最低的，和其余的方式相比，性能相差约 1000 倍，而且消耗了超过 1000 倍的内存。当然 `fmt.Sprintf` 通常是用来格式化字符串的，一般不会用来拼接字符串。

`strings.Builder`、`bytes.Buffer` 和 `[]byte` 的性能差距不大，而且消耗的内存也十分接近，性能最好且消耗内存最小的是 `preByteConcat`，这种方式预分配了内存，在字符串拼接的过程中，不需要进行字符串的拷贝，也不需要分配新的内存，因此性能最好，且内存消耗最小。

> #### 1.3 建议

综合易用性和性能，一般推荐使用 `strings.Builder` 来拼接字符串。

这是 Go 官方对 `strings.Builder` 的解释：

> A Builder is used to efficiently build a string using Write methods. It minimizes memory copying.

`string.Builder` 也提供了预分配内存的方式 `Grow`：

```go
func builderConcat(n int, str string) string {
	var builder strings.Builder
	builder.Grow(n * len(str))
	for i := 0; i < n; i++ {
		builder.WriteString(str)
	}
	return builder.String()
}
```

使用了 Grow 优化后的版本的 benchmark 结果如下：

```
BenchmarkBuilderConcat-8   16855    0.07 ns/op   0.1 MB/op       1 allocs/op
BenchmarkPreByteConcat-8   17379    0.07 ms/op   0.2 MB/op       2 allocs/op
```

与预分配内存的 `[]byte` 相比，因为省去了 `[]byte` 和字符串(string) 之间的转换，内存分配次数还减少了 1 次，内存消耗减半。

> ### 2 性能背后的原理

> #### 2.1 比较 strings.Builder 和 `+`

`strings.Builder` 和 `+` 性能和内存消耗差距如此巨大，是因为两者的内存分配方式不一样。

字符串在 Go 语言中是不可变类型，占用内存大小是固定的，当使用 `+` 拼接 2 个字符串时，生成一个新的字符串，那么就需要开辟一段新的空间，新空间的大小是原来两个字符串的大小之和。拼接第三个字符串时，再开辟一段新空间，新空间大小是三个字符串大小之和，以此类推。假设一个字符串大小为 10 byte，拼接 1w 次，需要申请的内存大小为：

```
10 + 2 * 10 + 3 * 10 + ... + 10000 * 10 byte = 500 MB 
```

而 `strings.Builder`，`bytes.Buffer`，包括切片 `[]byte` 的内存是以倍数申请的。例如，初始大小为 0，当第一次写入大小为 10 byte 的字符串时，则会申请大小为 16 byte 的内存（恰好大于 10 byte 的 2 的指数），第二次写入 10 byte 时，内存不够，则申请 32 byte 的内存，第三次写入内存足够，则不申请新的，以此类推。在实际过程中，超过一定大小，比如 2048 byte 后，申请策略上会有些许调整。我们可以通过打印 `builder.Cap()` 查看字符串拼接过程中，`strings.Builder` 的内存申请过程。

```go
func TestBuilderConcat(t *testing.T) {
	var str = randomString(10)
	var builder strings.Builder
	cap := 0
	for i := 0; i < 10000; i++ {
		if builder.Cap() != cap {
			fmt.Print(builder.Cap(), " ")
			cap = builder.Cap()
		}
		builder.WriteString(str)
	}
}
```

运行结果如下：

```go
$ go test -run="TestBuilderConcat" . -v
=== RUN   TestBuilderConcat
16 32 64 128 256 512 1024 2048 2688 3456 4864 6144 8192 10240 13568 18432 24576 32768 40960 57344 73728 98304 122880 --- PASS: TestBuilderConcat (0.00s)
PASS
ok      example 0.007s
```

我们可以看到，2048 以前按倍数申请，2048 之后，以 640 递增，最后一次递增 24576 到 122880。总共申请的内存大小约 `0.52 MB`，约为上一种方式的千分之一。

```
16 + 32 + 64 + ... + 122880 = 0.52 MB
```

> #### 2.2 比较 strings.Builder 和 bytes.Buffer

`strings.Builder` 和 `bytes.Buffer` 底层都是 `[]byte` 数组，但 `strings.Builder` 性能比 `bytes.Buffer` 略快约 10% 。一个比较重要的区别在于，`bytes.Buffer` 转化为字符串时重新申请了一块空间，存放生成的字符串变量，而 `strings.Builder` 直接将底层的 `[]byte` 转换成了字符串类型返回了回来。

- bytes.Buffer

```go
// To build strings more efficiently, see the strings.Builder type.
func (b *Buffer) String() string {
	if b == nil {
		// Special case, useful in debugging.
		return "<nil>"
	}
	return string(b.buf[b.off:])
}
```

- strings.Builder

```go
// String returns the accumulated string.
func (b *Builder) String() string {
	return *(*string)(unsafe.Pointer(&b.buf))
}
```

`bytes.Buffer` 的注释中还特意提到了：

> To build strings more efficiently, see the strings.Builder type.

#### 2. 切片slice性能及陷阱

在 Go 语言中，切片(slice)可能是使用最为频繁的数据结构之一，切片类型为处理同类型数据序列提供一个方便而高效的方式。

容量是当前切片已经预分配的内存能够容纳的元素个数，如果往切片中不断地增加新的元素。**如果超过了当前切片的容量，就需要分配新的内存，并将当前切片所有的元素拷贝到新的内存块上。**因此为了减少内存的拷贝次数，容量在比较小的时候，一般是以 2 的倍数扩大的，例如 2 4 8 16 …，当达到 2048 时，会采取新的策略，避免申请内存过大，导致浪费。

**切片操作并不复制切片指向的元素，创建一个新的切片会复用原来切片的底层数组，因此切片操作是非常高效的。**

> #### 切片操作

删除操作由于需要移动元素，故每次删除的复杂度为O(n)，并不适合大量删除场景



> #### 性能陷阱

在已有切片的基础上进行切片，不会创建新的底层数组。因为原来的底层数组没有发生变化，内存会一直占用，直到没有变量引用该数组。因此很可能出现这么一种情况，原切片由大量的元素构成，但是我们在原切片的基础上切片，虽然只使用了很小一段，但底层数组在内存中仍然占据了大量空间，得不到释放。比较推荐的做法，使用 `copy` 替代 `re-slice`。



#### 3. for 和 range 的性能比较

在Go语言中，range 可以用来很方便地遍历数组（array）、切片（slice）、字典（map） 和信道（channel）

> #### array/slice

```go
words := []string{"Go", "语言", "高性能", "编程"}
for i, s := range words {
    words = append(words, "test")
    fmt.Println(i, s)
}
// range 另一种写法
for i := range words {
	fmt.Println(i, words[i])
}
--------
0 Go
1 语言
2 高性能
3 编程
```

这个例子说明：

- 变量 words 在循环开始前，仅会计算一次，如果在循环中修改切片的长度不会改变本次循环的次数。
- 迭代过程中，每次迭代的下标和值被赋值给变量 i 和 s，第二个参数 s 是可选的。
- 针对 nil 切片，迭代次数为 0。



> #### map

```go
m := map[string]int{
    "one":   1,
    "two":   2,
    "three": 3,
}
for k, v := range m {
    delete(m, "two")
    m["four"] = 4
    fmt.Printf("%v: %v\n", k, v)
}
------------------
one: 1
four: 4
three: 3
```

- 和切片不同的是，迭代过程中，删除还未迭代到的键值对，则该键值对不会被迭代。
- 在迭代过程中，如果创建新的键值对，那么新增键值对，可能被迭代，也可能不会被迭代。
- 针对 nil 字典，迭代次数为 0



> #### channel

```go
ch := make(chan string)
go func() {
    ch <- "Go"
    ch <- "语言"
    ch <- "高性能"
    ch <- "编程"
    close(ch)
}()
for n := range ch {
    fmt.Println(n)
}
```

- 发送给信道(channel) 的值可以使用 for 循环迭代，直到信道被关闭。
- 如果是 nil 信道，循环将永远阻塞。



> #### for 和 range 性能比较

```go
type Item struct {
	id  int
	val [4096]byte
}

func BenchmarkForStruct(b *testing.B) {
	var items [1024]Item
	for i := 0; i < b.N; i++ {
		length := len(items)
		var tmp int
		for k := 0; k < length; k++ {
			tmp = items[k].id
		}
		_ = tmp
	}
}

func BenchmarkRangeIndexStruct(b *testing.B) {
	var items [1024]Item
	for i := 0; i < b.N; i++ {
		var tmp int
		for k := range items {
			tmp = items[k].id
		}
		_ = tmp
	}
}

func BenchmarkRangeStruct(b *testing.B) {
	var items [1024]Item
	for i := 0; i < b.N; i++ {
		var tmp int
		for _, item := range items {
			tmp = item.id
		}
		_ = tmp
	}
}
----------------------
$ go test -bench=Struct$ .
goos: darwin
goarch: amd64
pkg: example/hpg-range
BenchmarkForStruct-8             3769580               324 ns/op
BenchmarkRangeIndexStruct-8      3597555               330 ns/op
BenchmarkRangeStruct-8              2194            467411 ns/op
```

在仅遍历下标的情况下，for 和 range 的性能是差不多的。但是如果遍历结构体的元素的话， range的性能就会大大下降,**这是因为`range`对每个迭代值都创建了一个拷贝**

因此如果每次迭代的值内存占用很小的情况下，for 和 range 的性能几乎没有差异，但是如果每个迭代值内存占用很大，例如上面的例子中，每个结构体需要占据 4KB 的内存，这种情况下差距就非常明显了。

**但是可以使用指针来优化性能**

```go
func generateItems(n int) []*Item {
	items := make([]*Item, 0, n)
	for i := 0; i < n; i++ {
		items = append(items, &Item{id: i})
	}
	return items
}

func BenchmarkForPointer(b *testing.B) {
	items := generateItems(1024)
	for i := 0; i < b.N; i++ {
		length := len(items)
		var tmp int
		for k := 0; k < length; k++ {
			tmp = items[k].id
		}
		_ = tmp
	}
}

func BenchmarkRangePointer(b *testing.B) {
	items := generateItems(1024)
	for i := 0; i < b.N; i++ {
		var tmp int
		for _, item := range items {
			tmp = item.id
		}
		_ = tmp
	}
}
```



#### 4. 反射性能

> #### 案例

假设有一个配置类 Config，每个字段是一个配置项。为了简化实现，假设字段均为 string 类型：

```go
type Config struct {
	Name    string `json:"server-name"`
	IP      string `json:"server-ip"`
	URL     string `json:"server-url"`
	Timeout string `json:"timeout"`
}
```

配置默认从 `json` 文件中读取，如果环境变量中设置了某个配置项，则以环境变量中的配置为准。

配置项和环境变量对应的规则非常简单：将 json 字段的字母转为大写，将 `-` 转为下划线，并添加 `CONFIG_` 前缀。

最终的对应结果如下：

```go
type Config struct {
	Name    string `json:"server-name"` // CONFIG_SERVER_NAME
	IP      string `json:"server-ip"`   // CONFIG_SERVER_IP
	URL     string `json:"server-url"`  // CONFIG_SERVER_URL
	Timeout string `json:"timeout"`     // CONFIG_TIMEOUT
}
```

运用反射实现

```go
unc readConfig() *Config {
	// read from xxx.json，省略
	config := Config{}
	typ := reflect.TypeOf(config)
	value := reflect.Indirect(reflect.ValueOf(&config))
	for i := 0; i < typ.NumField(); i++ {
		f := typ.Field(i)
		if v, ok := f.Tag.Lookup("json"); ok {
			key := fmt.Sprintf("CONFIG_%s", strings.ReplaceAll(strings.ToUpper(v), "-", "_"))
			if env, exist := os.LookupEnv(key); exist {
				value.FieldByName(f.Name).Set(reflect.ValueOf(env))
			}
		}
	}
	return &config
}

func main() {
	os.Setenv("CONFIG_SERVER_NAME", "global_server")
	os.Setenv("CONFIG_SERVER_IP", "10.0.0.1")
	os.Setenv("CONFIG_SERVER_URL", "geektutu.com")
	c := readConfig()
	fmt.Printf("%+v", c)
}
```

实现逻辑其实是非常简单的：

- 在运行时，利用反射获取到 `Config` 的每个字段的 `Tag` 属性，拼接出对应的环境变量的名称。
- 查看该环境变量是否存在，如果存在，则将环境变量的值赋值给该字段。

运行该程序，输出为：

```
&{Name:global_server IP:10.0.0.1 URL:geektutu.com Timeout:}
```

> #### 性能测试

**创建对象：**

```go
func BenchmarkNew(b *testing.B) {
	var config *Config
	for i := 0; i < b.N; i++ {
		config = new(Config)
	}
	_ = config
}

func BenchmarkReflectNew(b *testing.B) {
	var config *Config
	typ := reflect.TypeOf(Config{})
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		config, _ = reflect.New(typ).Interface().(*Config)
	}
	_ = config
}
------------------------
$ go test -bench .          
goos: darwin
goarch: amd64
pkg: example/hpg-reflect
BenchmarkNew-8                  26478909                40.9 ns/op
BenchmarkReflectNew-8           18983700                62.1 ns/op
PASS
ok      example/hpg-reflect     2.382s
```

通过反射创建对象的耗时约为 `new` 的 1.5 倍，相差不是特别大。

**修改字段值：**

通过反射获取结构体的字段有两种方式，一种是 `FieldByName`，另一种是 `Field`(按照下标)。前面的例子中，我们使用的是 `FieldByName`。

```go
func BenchmarkSet(b *testing.B) {
	config := new(Config)
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		config.Name = "name"
		config.IP = "ip"
		config.URL = "url"
		config.Timeout = "timeout"
	}
}

func BenchmarkReflect_FieldSet(b *testing.B) {
	typ := reflect.TypeOf(Config{})
	ins := reflect.New(typ).Elem()
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		ins.Field(0).SetString("name")
		ins.Field(1).SetString("ip")
		ins.Field(2).SetString("url")
		ins.Field(3).SetString("timeout")
	}
}

func BenchmarkReflect_FieldByNameSet(b *testing.B) {
	typ := reflect.TypeOf(Config{})
	ins := reflect.New(typ).Elem()
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		ins.FieldByName("Name").SetString("name")
		ins.FieldByName("IP").SetString("ip")
		ins.FieldByName("URL").SetString("url")
		ins.FieldByName("Timeout").SetString("timeout")
	}
}
---------------
$ go test -bench="Set$" .          
goos: darwin
goarch: amd64
pkg: example/hpg-reflect
BenchmarkSet-8                          1000000000               0.302 ns/op
BenchmarkReflect_FieldSet-8             33913672                34.5 ns/op
BenchmarkReflect_FieldByNameSet-8        3775234               316 ns/op
PASS
ok      example/hpg-reflect     3.066s
```

总结一下，对于一个普通的拥有 4 个字段的结构体 `Config` 来说，使用反射给每个字段赋值，相比直接赋值，性能劣化约 100 - 1000 倍。其中，`FieldByName` 的性能相比 `Field` 劣化 10 倍。

```
通过源码可以看出来：
而 (t *structType) FieldByName 中使用 for 循环，逐个字段查找，字段名匹配时返回。也就是说，在反射的内部，字段是按顺序存储的，因此按照下标访问查询效率为 O(1)，而按照 Name 访问，则需要遍历所有字段，查询效率为 O(N)。结构体所包含的字段(包括方法)越多，那么两者之间的效率差距则越大。
```

> #### 提高性能

* #### 避免使用反射

使用反射赋值，效率非常低下，如果有替代方案，尽可能避免使用反射，特别是会被反复调用的热点代码。例如 RPC 协议中，需要对结构体进行序列化和反序列化，这个时候避免使用 Go 语言自带的 `json` 的 `Marshal` 和 `Unmarshal` 方法，因为标准库中的 json 序列化和反序列化是利用反射实现的。可选的替代方案有 [easyjson](https://github.com/mailru/easyjson)，在大部分场景下，相比标准库，有 5 倍左右的性能提升。

* #### 缓存

在上面的例子中可以看到，`FieldByName` 相比于 `Field` 有一个数量级的性能劣化。那在实际的应用中，就要避免直接调用 `FieldByName`。我们可以利用字典将 `Name` 和 `Index` 的映射缓存起来。避免每次反复查找，耗费大量的时间。

我们利用缓存，优化下刚才的测试用例：

#### 5. 空结构体节省内存

在 Go 语言中，我们可以使用 `unsafe.Sizeof` 计算出一个数据类型实例需要占用的字节数。

用来测试空结构体可以发现所占的空间为0；可以说明空结构体能够很好的节省空间。

> ###  2 空结构体的作用

因为空结构体不占据内存空间，因此被广泛作为各种场景下的占位符使用。一是节省资源，二是空结构体本身就具备很强的语义，即这里不需要任何值，仅作为占位符。

> #### 2.1 实现集合(Set)

Go 语言标准库没有提供 Set 的实现，通常使用 map 来代替。事实上，对于集合来说，只需要 map 的键，而不需要值。即使是将值设置为 bool 类型，也会多占据 1 个字节，那假设 map 中有一百万条数据，就会浪费 1MB 的空间。

因此呢，将 map 作为集合(Set)使用时，可以将值类型定义为空结构体，仅作为占位符使用即可。

```
type Set map[string]struct{}

func (s Set) Has(key string) bool {
	_, ok := s[key]
	return ok
}

func (s Set) Add(key string) {
	s[key] = struct{}{}
}

func (s Set) Delete(key string) {
	delete(s, key)
}

func main() {
	s := make(Set)
	s.Add("Tom")
	s.Add("Sam")
	fmt.Println(s.Has("Tom"))
	fmt.Println(s.Has("Jack"))
}
```

> #### 2.2 不发送数据的信道(channel)

```
func worker(ch chan struct{}) {
	<-ch
	fmt.Println("do something")
	close(ch)
}

func main() {
	ch := make(chan struct{})
	go worker(ch)
	ch <- struct{}{}
}
```

有时候使用 channel 不需要发送任何的数据，只用来通知子协程(goroutine)执行任务，或只用来控制协程并发度。这种情况下，使用空结构体作为占位符就非常合适了。

> #### 2.3 仅包含方法的结构体

```
type Door struct{}

func (d Door) Open() {
	fmt.Println("Open the door")
}

func (d Door) Close() {
	fmt.Println("Close the door")
}
```

在部分场景下，结构体只包含方法，不包含任何的字段。例如上面例子中的 `Door`，在这种情况下，`Door` 事实上可以用任何的数据结构替代。例如：

```
type Door int
type Door bool
```

无论是 `int` 还是 `bool` 都会浪费额外的内存，因此呢，这种情况下，声明为空结构体是最合适的。

#### 6. 内存对齐对性能的影响

在go语言中，我们可以使用`unsafe.Sizeof()`来计算出一个结构体所占的字节数

```go
type testS struct {
    num1 int16
    num2 int32
}
func main() {
    fmt.Println(unsafe.Sizeof(testS))
}
---------------------------------
8
```

这里的`testS`结构体的属性是2加上4字节，应该是6字节字节吗？为什么这里输出了8。多出来的2字节是内存对齐的结果。

> #### 为什么要内存对齐？

CPU访问内存时，并不是逐个字节访问，而是根据字节长来访问的。比如32位的CPU，访问字节长为4的字节，那么CPU访问内存的单位也就是4字节

这么设计的目的，是减少CPU访问内存的次数，加大CPU访问内存的吞吐量。

**例如：对于一个有两个3字节长属性的结构体，如果没有进行内存对齐。那么访问两个变量就需要CPU访问3次内存**

> unsafe.Alignof

在上面的例子中，`Flag{}` 两个字段占据了 6 个字节，但是最终对齐后的结果是 8 字节。Go 语言中内存对齐需要遵循什么规律呢？

`unsafe` 标准库提供了 `Alignof` 方法，可以返回一个类型的对齐值，也可以叫做对齐系数或者对齐倍数。例如：

```go
unsafe.Alignof(Args{}) // 8
unsafe.Alignof(Flag{}) // 4
```

- `Args{}` 的对齐倍数是 8，`Args{}` 两个字段占据 16 字节，是 8 的倍数，无需占据额外的空间对齐。
- `Flag{}` 的对齐倍数是 4，因此 `Flag{}` 占据的空间必须是 4 的倍数，因此，6 内存对齐后是 8 字节。

> #### 合理布局结构体属性减少内存占用

设一个 struct 包含三个字段，`a int8`、`b int16`、`c int64`，顺序会对 struct 的大小产生影响吗？我们来做一个实验：

```go
type demo1 struct {
	a int8
	b int16
	c int32
}

type demo2 struct {
	a int8
	c int32
	b int16
}

func main() {
	fmt.Println(unsafe.Sizeof(demo1{})) // 8
	fmt.Println(unsafe.Sizeof(demo2{})) // 12
}
```

答案是会产生影响。每个字段按照自身的对齐倍数来确定在内存中的偏移量，字段排列顺序不同，上一个字段因偏移而浪费的大小也不同。

接下来逐个分析，首先是 demo1：

- a 是第一个字段，默认是已经对齐的，从第 0 个位置开始占据 1 字节。
- b 是第二个字段，对齐倍数为 2，因此，必须空出 1 个字节，偏移量才是 2 的倍数，从第 2 个位置开始占据 2 字节。
- c 是第三个字段，对齐倍数为 4，此时，内存已经是对齐的，从第 4 个位置开始占据 4 字节即可。

因此 demo1 的内存占用为 8 字节。

其实是 demo2：

- a 是第一个字段，默认是已经对齐的，从第 0 个位置开始占据 1 字节。
- c 是第二个字段，对齐倍数为 4，因此，必须空出 3 个字节，偏移量才是 4 的倍数，从第 4 个位置开始占据 4 字节。
- b 是第三个字段，对齐倍数为 2，从第 8 个位置开始占据 2 字节。



## 七、GMP 原理与调度

> 调度器的由来：在单进程时代，程序都是线性运行的，CPU执行不需要进行调度，但是可能由于进程阻塞而导致CPU资源浪费。
>
> 对于多进程场景，就需要CPU调度器来合理使用CPU资源，当进程出现阻塞等情况，调度器就会运行其它的进程，不会浪费CPU资源。

### 协程

尽管多线程/多进程提高了系统的并发能力，但是为每个任务都创建一个线程是不现实的，因为线程会消耗大量的内存。

这时，协程就出现了。对于一个线程，其实是分为两部分的，一部分的“用户态”线程，一部分是“内核态”线程。一个“用户态线程”必须要绑定一个“内核态线程”，对于CPU来说，它关联的只是“内核线程”，而不关注“用户态线程”。

因此，我们可以再细化一下，将“用户态”线程叫做“协程（co-routine）”，“内核态”线程依旧叫做“线程（thread）”。

我们可以由这种关系引生出其它模型，例如

* `1:1`关系，一个协程绑定一个线程。协程的创建、删除和切换都由CPU完成，开销大
* `N:1`关系，多个协程经过协程调度器绑定一个线程。一个协程阻塞，其它协程无法执行
* `N:M`关系，多个协程绑定多个线程。实现复杂

### Go中的协程

Go 中，协程被称为 goroutine，它非常轻量，一个 goroutine 只占几 KB，并且这几 KB 就足够 goroutine 运行完，这就能在有限的内存空间内支持大量 goroutine，支持了更多的并发。虽然一个 goroutine 的栈只占几 KB，但实际是可伸缩的，如果需要更多内容，runtime 会自动为 goroutine 分配。

Goroutine 特点：

- 占用内存更小（几 kb）
- 调度更灵活 (runtime 调度)

### GMP

在新调度器中，出列 M (thread) 和 G (goroutine)，又引进了 P (Processor)

![](image\12.jpg)

- **全局队列（Global Queue）**：存放等待运行的 G。当要获取队列中的G，需要申请锁。
- **P 的本地队列**：同全局队列类似，存放的也是等待运行的 G，存的数量有限，不超过 256 个。新建 G’时，G’优先加入到 P 的本地队列，如果队列满了，则会把本地队列中一半的 G 移动到全局队列。
- **P 列表**：所有的 P 都在程序启动时创建，并保存在数组中，最多有 GOMAXPROCS(可配置) 个。
- **M**：线程想运行任务就得获取 P，从 P 的本地队列获取 G，P 队列为空时，M 也会尝试从全局队列拿一批 G 放到 P 的本地队列，或从其他 P 的本地队列偷一半放到自己 P 的本地队列。M 运行 G，G 执行之后，M 会从 P 获取下一个 G，不断重复下去。

**Goroutine 调度器和 OS 调度器是通过 M 结合起来的，每个 M 都代表了 1 个内核线程，OS 调度器负责把内核线程分配到 CPU 的核上执行。**

> ### 调度器设计策略

复用线程：避免频繁的创建、销毁线程，而是对线程的复用。

**1）work stealing 机制**

 当本线程无可运行的 G 时，尝试从其他线程绑定的 P 偷取 G，而不是销毁线程。

**2）hand off 机制**

 当本线程因为 G 进行系统调用阻塞时，线程释放绑定的 P，把 P 转移给其他空闲的线程执行。

**利用并行**：GOMAXPROCS 设置 P 的数量，最多有 GOMAXPROCS 个线程分布在多个 CPU 上同时运行。GOMAXPROCS 也限制了并发的程度，比如 GOMAXPROCS = 核数/2，则最多利用了一半的 CPU 核进行并行。

**抢占**：在 coroutine 中要等待一个协程主动让出 CPU 才执行下一个协程，在 Go 中，一个 goroutine **最多占用 CPU 10ms**，防止其他 goroutine 被饿死，这就是 goroutine 不同于 coroutine 的一个地方。

**全局 G 队列**：在新的调度器中依然有全局 G 队列，但功能已经被弱化了，当 M 执行 work stealing 从其他 P 偷不到 G 时，它可以从全局 G 队列获取 G。

**M和P的数量：**M 与 P 的数量没有绝对关系，一个 M 阻塞，P 就会去创建或者切换另一个 M，所以，即使 P 的默认数量是 1，也有可能会创建很多个 M 出来。

> ### go func() 调度流程

![](F:\study\go\image\13.jpg)

















## 八、Go中的内存逃逸分析

* ### 函数栈帧

![](F:\study\go\image\2546015635-90e85d897f74f991_fix732.webp)

当一个函数在运行时，需要为它在堆栈中创建一个栈帧（`stack frame`）用来记录运行时产生的相关信息，因此每个函数在执行前都会创建一个栈帧，在它返回时会销毁该栈帧。

通常用一个叫做栈基址（`bp`）的寄存器来保存正在运行函数栈帧的开始地址，由于栈指针（`sp`）始终保存的是栈顶的地址，所以栈指针保存的也就是正在运行函数栈帧的结束地址。

销毁时先把栈指针（`sp`）移动到此时栈基址（`bp`）的位置，此时栈指针和栈基址都指向同样的位置。

Go编译器会尽可能将变量分配到到栈上。但是，当编译器无法证明函数返回后，该变量没有被引用，那么编译器就必须在堆上分配该变量，以此避免悬挂指针（dangling pointer）。另外，如果局部变量非常大，也会将其分配在堆上。

那么，Go是如何确定的呢？答案就是：逃逸分析。编译器通过逃逸分析技术去选择堆或者栈，逃逸分析的基本思想如下：**检查变量的生命周期是否是完全可知的，如果通过检查，则可以在栈上分配。否则，就是所谓的逃逸，必须在堆上进行分配。**

Go语言虽然没有明确说明逃逸分析规则，但是有以下几点准则，是可以参考的。

- 逃逸分析是在编译器完成的，这是不同于jvm的运行时逃逸分析;
- 如果变量在函数外部没有引用，则优先放到栈中；
- 如果变量在函数外部存在引用，则必定放在堆中；

我们可通过`go build -gcflags '-m -l'`命令来查看逃逸分析结果，其中-m 打印逃逸分析信息，-l禁止内联优化。下面，我们通过一些案例，来熟悉一些常见的逃逸情况。

> ###  情况一：变量类型不确定

```go
package main

import "fmt"

func main() {
    a := 666
    fmt.Println(a)
}
```

逃逸分析结果如下

```go
$ go build -gcflags '-m -l' main.go
# command-line-arguments
./main.go:7:13: ... argument does not escape
./main.go:7:13: a escapes to heap
```

可以看到，分析结果告诉我们变量`a`逃逸到了堆上。但是，我们并没有外部引用啊，为啥也会有逃逸呢？为了看到更多细节，可以在语句中再添加一个`-m`参数。得到信息如下

```go
$ go build -gcflags '-m -m -l' main.go
# command-line-arguments
./main.go:7:13: a escapes to heap:
./main.go:7:13:   flow: {storage for ... argument} = &{storage for a}:
./main.go:7:13:     from a (spill) at ./main.go:7:13
./main.go:7:13:     from ... argument (slice-literal-element) at ./main.go:7:13
./main.go:7:13:   flow: {heap} = {storage for ... argument}:
./main.go:7:13:     from ... argument (spill) at ./main.go:7:13
./main.go:7:13:     from fmt.Println(... argument...) (call parameter) at ./main.go:7:13
./main.go:7:13: ... argument does not escape
./main.go:7:13: a escapes to heap
```

a逃逸是因为它被传入了`fmt.Println`的参数中，这个方法参数自己发生了逃逸。

```text
func Println(a ...interface{}) (n int, err error)
```

因为`fmt.Println`的函数参数为`interface`类型，编译期不能确定其参数的具体类型，所以将其分配于堆上。

> ### 情况二：暴露给外部指针

```go
package main

func foo() *int {
    a := 666
    return &a
}

func main() {
    _ = foo()
}
```

逃逸分析如下，变量a发生了逃逸。

```go
$ go build -gcflags '-m -m -l' main.go
# command-line-arguments
./main.go:4:2: a escapes to heap:
./main.go:4:2:   flow: ~r0 = &a:
./main.go:4:2:     from &a (address-of) at ./main.go:5:9
./main.go:4:2:     from return &a (return) at ./main.go:5:2
./main.go:4:2: moved to heap: a
```

这种情况直接满足我们上述中的原则：变量在函数外部存在引用。这个很好理解，因为当函数执行完毕，对应的栈帧就被销毁，但是引用已经被返回到函数之外。如果这时外部从引用地址取值，虽然地址还在，但是这块内存已经被释放回收了，这就是非法内存，问题可就大了。所以，很明显，这种情况必须分配到堆上。

> ### 情况三：变量所占内存较大

```go
func foo() {
    s := make([]int, 10000, 10000)
    for i := 0; i < len(s); i++ {
        s[i] = i
    }
}

func main() {
    foo()
}
```

逃逸分析结果

```go
$ go build -gcflags '-m -m -l' main.go
# command-line-arguments
./main.go:4:11: make([]int, 10000, 10000) escapes to heap:
./main.go:4:11:   flow: {heap} = &{storage for make([]int, 10000, 10000)}:
./main.go:4:11:     from make([]int, 10000, 10000) (too large for stack) at ./main.go:4:11
./main.go:4:11: make([]int, 10000, 10000) escapes to heap
```

可以看到，当我们创建了一个容量为10000的`int`类型的底层数组对象时，由于对象过大，它也会被分配到堆上。这里我们不禁要想一个问题，为啥大对象需要分配到堆上。

这里需要注意，在上文中没有说明的是：在Go中，执行用户代码的goroutine是一种用户态线程，其调用栈内存被称为用户栈，它其实也是从堆区分配的，但是我们仍然可以将其看作和系统栈一样的内存空间，它的分配和释放是通过编译器完成的。与其相对应的是系统栈，它的分配和释放是操作系统完成的。在GMP模型中，一个M对应一个系统栈（也称为M的`g0`栈），M上的多个goroutine会共享该系统栈。



## 九、Go GC

> ### go gc 的版本变化

- Go V1.3的标记-清除法(mark and sweep)
- Go V1.5的三色并发标记法
- Go V1.5的三色标记为什么需要STW
- Go V1.5的三色标记为什么需要屏障机制(“强-弱” 三色不变式、插入屏障、删除屏障 )
- Go V1.8混合写屏障机制
- Go V1.8混合写屏障机制的全场景分析







## 其它

### toml 配置文件

TOML文档：https://toml.io/cn/v1.0.0

### 雪花算法

**1.第一位** 占用1bit，其值始终是0，没有实际作用。 **2.时间戳** 占用41bit，精确到毫秒，总共可以容纳约69年的时间。 **3.工作机器id** 占用10bit，其中高位5bit是数据中心ID，低位5bit是工作节点ID，做多可以容纳1024个节点。 **4.序列号** 占用12bit，每个节点每毫秒0开始不断累加，最多可以累加到4095，一共可以产生4096个ID。

### RBAC 权限控制



