# 面试题

## 1.  make 和 new 的区别

> * ## 总结
>
> 所以从这里可以看的很明白了，二者都是内存的分配（堆上），但是`make`只用于slice、map以及channel的初始化（非零值）；而`new`用于类型的内存分配，并且内存置为零。所以在我们编写程序的时候，就可以根据自己的需要很好的选择了。
>
> `make`返回的还是这三个引用类型本身；而`new`返回的是指向类型的指针。
>
> make与new对堆栈分配处理是相同的，编译器优先进行逃逸分析，逃逸的才分配到堆上逃逸分析就是看函数内部的变量会不会返回出去，如果返回出去到外面的作用域去了，就属于逃逸，就会分配在堆上

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

* ### new

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

* ## make

`make`也是用于内存分配的，但是和`new`不同，它只用于`chan`、`map`以及切片的内存创建，而且它返回的类型就是这三个类型本身，而不是他们的指针类型，因为这三种类型就是引用类型，所以就没有必要返回他们的指针了。

注意，因为这三种类型是引用类型，所以必须得初始化，但是不是置为零值，这个和`new`是不一样的。

```go
func make(t Type, size ...IntegerType) Type
```

从函数声明中可以看到，返回的还是该类型。



## 2. 数组和切片的区别

相同点：都是存储相同类型的数据结构

不同点：

1）**容量机制不同**。数组的长度是固定的，当访问超过长度的下标就会发生panic；而切片的长度和容量可以自动扩容。

2）**类型不同**。数组是值类型，切片是引用类型，每个切片都引用了一个底层数组。

### 切片的扩容机制

* 会先检查容量是否足够，不够再进行扩容。
* 当旧的长度小于 1024 时，会两倍两倍的扩大。
* 当旧的长度大于1024 时，不同的版本扩容算法不同，一般的扩容原长度的四分之一



## 3. select

**select 的特性**

1）select 操作至少要有一个 case 语句，出现读写 nil 的 channel 该分支会忽略，在 nil 的 channel 上操作则会报错。

2）select 仅支持管道，而且是单协程操作。

3）每个 case 语句仅能处理一个管道，要么读要么写。

4）多个 case 语句的执行顺序是随机的。

5）存在 default 语句，select 将不会阻塞，但是存在 default 会影响性能。



## 4. map相关

* ### map的特性

  1. map是无序的。map循环也是无序的（map底层结构是hash表，扩容发生变化）
  2. map是线程不安全的，map类型是容易发生并发访问问题的。
  3. map是不支持并发读写的，当出现这种情况就会报错。

* ### map的内存释放

  * 如果删除的元素是值类型，如int，float。。。以及数组和struct，map的内存不会自动释放
  * 如果删除的元素是引用类型，如指针，slice，map等，map的内存会自动释放，但释放的内存是子元素应用类型的内存占用。
  * 将map设置为nil后，内存被回收

  **note:如果一个键或值超过 128 字节，Go 不会将其直接存储在 map bucket 中。相反，Go 存储一个指针来引用键或值。**

  

* ### 如何解决并发问题

  1. 使用` sync.Mutex`加锁来保证并发安全，但是导致读写效率大大降低。
  2. 在 `sync` 包中提供了 `sync.RWMutex`，这个锁可以允许进行并发读，但是写的时候还是需要等待锁。 也就是说，**一个协程在持有写锁的时候，其他协程是既不能读也不能写的，只能等待写锁释放才能进行读写**。

  2. 使用`sync.Map`，例子：

  ```go
  func main() {
  	var m sync.Map
  	// 1. 写入
  	m.Store("data01", 1)
  	m.Store(02, "data02")
  	// 2. 读取
  	data, _ := m.Load(2)
  	fmt.Println("02:", data)
  	// 3. 遍历
  	m.Range(func(key, value any) bool {
  		fmt.Printf("key:%v,value:%v\n", key, value)
  		return true
  	})
  	// 4. 删除
  	m.Delete(02)
  	data, _ = m.Load("data01")
  	fmt.Println(data)
  }
  ```



### sync.Map

* `sync.Map`原理

  而使用 `sync.map` 之后，对 map 的读写，不需要加锁。并且它通过空间换时间的方式，使用 read 和 dirty 两个 map 来进行读写分离，降低锁时间来提高效率。对于写多的场景，提升的效果比较明显。

  ### 数据结构

  ![](image\屏幕截图 2024-03-02 160756.png)

  ```go
  // go 1.21.6
  type Map struct {
      mu Mutex
      read atomic.Pointer[readOnly] // readOnly
      dirty map[any]*entry
      misses int
  }
  type readOnly struct {
  	m       map[any]*entry
  	amended bool // true if the dirty map contains some key not in m.
  }
  type entry struct {
      p atomic.Pointer[any]
  }
  ```

  * `mu` 是互斥锁，用来保护`read`和`dirty`

  * `read` 是一个原子类指针，对其的访问是原子操作，底层关联着一个原始的map和一个标识符` amended`。相当于一个读缓存。这个map和dirty数据结构是一样的，相同`key`指向的entry也是同一个。

  * `dirty` 是一个原始的map，用来存储实际数据key和存储有value指针`p`的`entry`

  * `amended` 标识符用来表示 `read`关联的map和`dirty`是否有数据没有存储在 `read` 中，而在`dirty`中

  * `misses` 用来记录读取数据未命中 `read` 的次数，达到一定阈值就会将`dirty`拷贝到`read`中。因为数据的插入操作都是在  `dirty` 中进行的。

  * `p`指针，用来指向value值，有三种状态`nil`、`expunged`(已删除)、`[value]`。

    如果是`expunged `状态，表示在`dirty`中没有该映射，只有在首次设置 `m.dirty[key] = e `后才能更新条目的关联值，以便使用脏映射的查找找到该条目。当成功设置映射时，就会将状态改变为`nil`
    
    

  关于 `sync.Map` 的插入与删除, 底层都用到了大量的`CAS`操作。

  * **查询操作**
  
    1. 先到`read`中去查找key是否存在，如果不存在并且`amended`字段为true就会对`sync.Map`加锁；对`read`进行二次查找，如果还是没找到，就会去`dirty`中查找。
    2. 在`dirty`查找，并且让`misses`加一，**如果`misses`的值大于等于`dirty`的长度，就会让`read` 中的map更新为`dirty`，并设置`dirty` 为 nil。**

  * **插入操作**
  
    1. 先在 `read`中查找key是否存在，如果存在就会，判断`entry`中的指针 p 状态是否为`expunged`（删除），如果不是就执行`CAS`操作更新值；
    2. 如果`read`中 key 不存在，或者`CAS`执行失败，就会对`sync.Map`加锁，再次查找`read`中key是否存在，如果存在使用函数`e.unexpungeLock()`判断`entry`中`p`的状态，如果是`expunged`，改成`nil`，成功后增加该`entry`到`dirty`中；最后执行`swapLocked()`更新值操作。**这些底层都是原子操作**。
    3. 如果二次查找 `read` 还不存在，检查 `dirty` 中是否存在，如果存在就直接执行`swapLocked()`更新值操作。
    4. 如果都不存在并且`amended`为 false ，就会触发`dirtyLocked()`函数，如果`dirty`为nil，对`dirty`初始化，并遍历`read`中的map，如果`p`为nil，转变成`expunged`，否则，将`entry`添加到`dirty`；设置`amended`为true。（这一步是保证当`dirty`由于`Range()`函数或者`misses`达到阈值时使得其未nil后的第一次添加元素时，为`dirty`初始化分配内存空间）之后就会在`dirty`中创建插入新的`entry`

  * **删除操作**
  
    1. 先在`read`中查找，如果不存在就加锁再次在`read`中查找，如果查找到就让key对应`entry`中的指针为nil
    2. 如果还不在就在`dirty`中寻找，并且让`misses`加一
    3. 如果在 `dirty`中找到，就删除`dirty`中对应的`entry`，即变为nil

  * **Range 函数**

    **如果`amended`为true，`read` 中的map更新为`dirty`，并将`dirty`设置为nil，amended设置为false**；之久后开始遍历`read`中的map。

  * ### **总结**
  
    

### map的扩容机制

* ### 数据结构

  ```go
  // Map contains Type fields specific to maps.
  type Map struct {
      Key  *Type // Key type
      Elem *Type // Val (elem) type
  
      Bucket *Type // internal struct type representing a hash bucket
      Hmap   *Type // internal struct type representing the Hmap (map header object)
      Hiter  *Type // internal struct type representing hash iterator state
  }
  
  type hmap struct {
  	count     int
  	flags     uint8
  	B         uint8  
  	noverflow uint16
  	hash0     uint32
  	buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
  	oldbuckets unsafe.Pointer
  	nevacuate  uintptr
  	extra *mapextra // optional fields
  }
  type mapextra struct {
  	overflow    *[]*bmap
  	oldoverflow *[]*bmap
  	nextOverflow *bmap
  }
  type bmap struct {
  	tophash [bucketCnt]uint8
  }
  // bucketCnt = 8
  //桶的结构体 runtime.bmap 在 Go 语言源代码中的定义只包含一个简单的 tophash 字段，tophash 存储了键的哈希的高 8 位，通过比较不同键的哈希的高 8 位可以减少访问键值对次数以提高性能.
  //key 的定位高八位用于定位 bucket，也就是nextOverflow字段；低八位用于定位 key，也就是overflow字段.
  ```

  ![](image\屏幕截图 2024-03-03 180541.png)

* ### 简单总结

**Go 语言使用拉链法来解决哈希碰撞的问题实现了哈希表**，它的访问、写入和删除等操作都在编译期间转换成了运行时的函数或者方法。

哈希在每一个桶中存储键对应哈希的前 8 位，当对哈希进行操作时，这些 `tophash` 就成为可以帮助哈希快速遍历桶中元素的缓存。

哈希表的每个桶都只能存储 8 个键值对，一旦当前哈希的某个桶超出 8 个，新的键值对就会存储到哈希的溢出桶中。

随着键值对数量的增加，溢出桶的数量和哈希的装载因子也会逐渐升高，超过一定范围就会触发扩容，扩容会将桶的数量翻倍，元素再分配的过程也是在调用写操作时增量进行的，不会造成性能的瞬时巨大抖动。

装载因子的计算公式是：**装载因子=填入表中的元素个数/散列表的长度**，装载因子越大，说明空闲位置越少，冲突越多，散列表的性能会下降。装载因子超过阈值，源码里定义的阈值是 6.5。



### slices 能作为 map 类型的key吗？

答案是：**在golang规范中，可比较的类型都可以作为map key；**这个问题又延伸到在：golang规范中，哪些数据类型可以比较？

**不能作为map key 的类型包括：**

- slices
- maps
- functions



## 5. context 相关

> 答：Go 的 Context 的数据结构包含 Deadline，Done，Err，Value，Deadline 方法返回一个 time.Time，表示当前 Context 应该结束的时间，ok 则表示有结束时间，Done 方法当 Context 被取消或者超时时候返回的一个 close 的 channel，告诉给 context 相关的函数要停止当前工作然后返回了，Err 表示 context 被取消的原因，Value 方法表示 context 实现共享数据存储的地方，是协程安全的。context 在业务中是经常被使用的，
>
> **其主要的应用 ：**
>
> 1：上下文控制，2：多个 goroutine 之间的数据交互等，3：超时控制：到某个时间点超时，过多久超时。

小结：

1. context包通过构建树形关系的context，来达到上一层goroutine对下一层goroutine的控制。对于处理一个request请求操作，需要通过goroutine来层层控制goroutine，以及传递一些变量来共享。

2. context变量的请求周期一般为一个请求的处理周期。即针对一个请求创建context对象；在请求处理结束后，撤销此ctx变量，释放资源。

3. 每创建一个goroutine，要不将原有context传递给子goroutine，要么创建一个子context传递给goroutine.

4. Context能灵活地存储不同类型、不同数目的值，并且使多个`Goroutine`安全地读写其中的值。

5. 当通过父 Context对象创建子Context时，可以同时获得子Context的撤销函数，这样父goroutine就获得了子goroutine的撤销权。

 

原则：

1. 不要把context放到一个结构体中，应该作为第一个参数显式地传入函数

2. 即使方法允许，也不要传入一个nil的context，如果不确定需要什么context的时候，传入一个`context.TODO`

3. 使用context的Value相关方法应该传递和请求相关的元数据，不要用它来传递一些可选参数

4. 同样的context可以传递到多个goroutine中，Context在多个goroutine中是安全的

5. 在子context传入`goroutine`中后，应该在子`goroutine`中对该子context的`Done channel`进行监控，一旦该channel被关闭，应立即终止对当前请求的处理，并释放资源。



## 6. channel 相关



## 7. defer 

defer执行顺序和调用顺序相反，类似于栈**后进先出**(LIFO)。

defer在return之后执行，但在函数退出之前，defer可以修改返回值。下面是一个例子：

**对于无名返回值会创建一个临时变量保存返回值**

上面这个例子中，test返回值并没有修改，这是由于Go的返回机制决定的，执行Return语句后，Go会创建一个临时变量保存返回值

```go
func test() int {
	i := 0
	defer func() {
		fmt.Println("defer1")
	}()
	defer func() {
		i += 1
		fmt.Println("defer2")
	}()
	return i
}

func main() {
	fmt.Println("return", test())
}
// defer2
// defer1
// return 0
```

**如果是有名返回值函数，执行 return 语句时，并不会再创建临时变量保存，因此，defer 语句修改了 i，即对返回值产生了影响**

```go
func test() (i int) {
	i = 0
	defer func() {
		i += 1
		fmt.Println("defer2")
	}()
	return i
}

func main() {
	fmt.Println("return", test())
}
// defer2
// return 1
```



## 8. 函数

### Go 支持默认参数或可选参数吗

不支持。但是可以利用结构体参数，或者...传入参数切片数组

```go
func sum(nums ...int) {
    total := 0
    for _, num := range nums {
        total += num
    }
    fmt.Println(total)
}
```

## 9. 切片

### 如何判断 2 个字符串切片是相等的？

`reflect.DeepEqual()`, 但是反射非常影响性能



## 其它

### Go 语言 tag 的用处？

tag可以为结构体成员提供属性。常见的：

1. json序列化或反序列化时字段的名称
2. db: sqlx模块中对应的数据库字段名
3. form: gin框架中对应的前端的数据字段名
4. binding: 搭配 form 使用, 默认如果没查找到结构体中的某个字段则不报错值为空, binding为 required 代表没找到返回错误给前端



### 如何获取一个结构体中的所有tag？

```go
import reflect
type Author struct {
    Name int `jons:Name`
    Publications []string `json:Publication, omitempty`
}
func main() {
    t := reflect.TypeOf(Author{})
    for i := 0; i < t.NumField(); i++ {
        name := t.Field(i).Name
        s, _ := t.FieldByName(name)
        fmt.Println(name, s.Tag)
    }
}
```

### 

### 结构体打印时，`%v` 和 `%+v` 的区别

`%v`输出结构体各成员的值；

`%+v`输出结构体各成员的**名称**和**值**；

`%#v`输出结构体名称和结构体各成员的名称和值



### Go 语言中如何表示枚举值（enums）

在常量中用 `iota`可以表示枚举。iota从0开始

```go
/* 
1KB = 1,000Byte
1MB = 1,000KB
1GB = 1,000,000 KB

1KiB = 1024Byte
1Mib = 1024KiB
...
*/
const (
	B = 1 << (10 * iota)
    //代表一个二进制数：10000000000    该数转换为十进制为2的10次方即1024
	KiB 
	MiB
	GiB
	TiB
	PiB
	EiB
)
```



### Go 里面的int和int32是同一个概念吗

不是同一个概念，千万不要混淆。go语言中的int的大小是和操作系统位数相关的，如果是32位操作系统，int类型的大小就是4字节。如果是64位，就是8字节。



## 实现原理

### 01. init() 函数是什么时候执行的？

在main函数之前执行。

**详细**：init()函数是go初始化的一部分，由runtime初始化每个导入的包，初始化不是按照从上到下的导入顺序，**而是按照解析的依赖关系，没有依赖的包最先初始化。**

每个包首先初始化包作用域的常量和变量（常量优先于变量），然后执行包的`init()`函数。同一个包，甚至是同一个源文件可以有多个`init()`函数。**`init()`函数没有入参和返回值，不能被其他函数调用，同一个包内多个`init()`函数的执行顺序不作保证。**

执行顺序：`import –> const –> var –>init()–>main()`



### 02. 如何知道一个对象是分配在栈上还是堆上？

Go和C++不同，Go局部变量会进行**逃逸分析**。如果**变量离开作用域后没有被引用**，则**优先**分配到栈上，否则分配到堆上。那么如何判断是否发生了逃逸呢？

`go build -gcflags '-m -m -l' xxx.go`.

关于逃逸的可能情况：变量大小不确定，变量类型不确定，变量分配的内存超过用户栈最大值，暴露给了外部指针。



### 03. 2个 interface 可以比较吗？

Go语言中，interface 的内部实现包括了2个字段，一个是类型T，和值V。两个interface可以通过 `!=` 和 `==` 来判断是否相等，但是会出现两种情况：

1. 两个都是nil，T和V都是unset状态，这时是相等的
2. 两个类型T相等，V相等，==才是true



### 04. 2 个nil可能不相等吗？

可能不等。interface在运行时绑定值，只有值为nil接口值才为nil，但是与指针的nil不相等。举个例子：

```text
var p *int = nil
var i interface{} = nil
if(p == i){
	fmt.Println("Equal")
}
```

两者并不相同。总结：**两个nil只有在类型相同时才相等**。



### 05.  简述 Go 语言GC的工作原理



### 06. 函数返回局部变量的指针是否安全？

这一点和C++不同，在Go里面返回局部变量的指针是安全的。因为Go会进行**逃逸分析**，如果发现局部变量的作用域超过该函数则会**把指针分配到堆区**，避免内存泄漏。







### 08. go slice 是如何扩容的？

**Go <= 1.17**

如果当前容量小于1024，则判断所需容量是否大于原来容量2倍，如果大于，当前容量加上所需容量；否则当前容量乘2。

如果当前容量大于1024，则每次按照1.25倍速度递增容量，也就是每次加上cap/4。

**Go 1.18 之后，引入了新的扩容规则：**

首先会判断两倍容量是否满足所需最小容量，如果不满足，预估容量就是所需最小容量。否则进入下面的判断。

如果当前容量小于256，就按照2倍是速率扩容。

对于超过等于256的容量的切片就是按照下面这个公式来扩容。

```go
const threshold = 256
newcap += (newcap + 3*threshold) / 4
/* 
这个公式，对于容量小的切片，按照2倍的速率扩容和对于容量大的切片，按照1.25倍的速度扩容，为两者提供了平滑的过渡。
*/
```



获取到预估容量后，还要进行内存对齐：以整型为例，预估容量 *元素类型的大小，也即是 5* 8 = 40 bytes （64位环境下）。

那么经过roundupsize函数调整，得到结果为 48 bytes，而48 bytes可以装下6个元素，对应调整代码为:

```go
newcap = int(capmem / et.size)
```

所以，最终容量的大小被调整为6。

其中`roundupsize`函数位于在`./src/runtime/msize.go`文件中。

它的作用是：返回mallocgc将分配的内存块的大小。

**也就是，由Go语言的内存管理模块返回给你需要的内存块，通常这些内存块都是预先申请好，并且被分为常用的规格，比如8，16， 32， 48， 64等。**

**这里我们需要的内存是40 bytes，所以会分配一个足够用，且最接近的内存块。所以给48bytes，这时，重新调整后的容量 newcap就为6。











1 Go的垃圾回收机制？GMP模型？

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

## .  Kafka

- kafka同步租户时如何防止信息丢失（事务：commit、autocommit）（展盟）

## .  Kubernetes相关知识

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

## . 服务治理与微服务架构

- kitex框架：服务治理（负载均衡、服务降级...）、可观测（链路追踪...）（展盟）
- istio是做数据面还是控制面?（优咔科技）
- Istio负载均衡的策略（优咔科技）
- Service mesh中南北向和东西向流量处理？（优咔科技）

1. Linux命令和系统知识

- Linux的awk、strace命令（展盟）
- linux根据服务名称查看端口号（netstat -tunlp | grep 服务名称）（柯莱特-外派小红书）
- linux根据服务名称查看pid（ps -aux | 服务名称）（柯莱特-外派小红书）
- linux进程间通信的方式（socket、管道.....）（柯莱特-外派小红书）

