# GO 语言入门

## 目录：常用函数

* ### strings.Trim函数

  所属包：strings

  函数声明：`func Trim(s string, cutset string) string`

  说明：返回将 s 前后端所有 cutset 包含的 utf-8 码值都去掉的字符串。

  示例代码：

  ```go
  package main
  
  import (
      "fmt"
      "strings"
   _  "test/subpac"
  )
  
  func main(){
      fmt.Println("[ !!! Achtung! Achtung! !!! ]:[]:[", strings.Trim(" !!! Achtung! Achtung! !!! ", "") ,"\b]")
      fmt.Println("[ !!! Achtung! Achtung! !!! ]:[ ]:[", strings.Trim(" !!! Achtung! Achtung! !!! ", " ") ,"\b]")
      fmt.Println("[ !!! Achtung! Achtung! !!! ]:[!]:[", strings.Trim(" !!! Achtung! Achtung! !!! ", "!") , "\b]")
      fmt.Println("[ !!! Achtung! Achtung! !!! ]:[! ]:[", strings.Trim(" !!! Achtung! Achtung! !!! ", "! "), "\b]" )
  }
  ------------------输出结果-----------------------
  [ !!! Achtung! Achtung! !!! ]:[]:[  !!! Achtung! Achtung! !!! ]
  [ !!! Achtung! Achtung! !!! ]:[ ]:[ !!! Achtung! Achtung! !!!]
  [ !!! Achtung! Achtung! !!! ]:[!]:[  !!! Achtung! Achtung! !!! ]
  [ !!! Achtung! Achtung! !!! ]:[! ]:[ Achtung! Achtung]
  
  第一行 cutset 为空（不是空格）：因此输出原字符串。
  第二行 cutset 为 ” “（空格）：因此串首尾的两个空格字符被删除了。
  第三行 cutset 为 “!” ：收尾未匹配到该 cutset，因此输出原字符串。
  第四行 cutset 为 “! “：首先匹配到空格，串首尾空格字符被删除，然后匹配到 “!”，继续删除首尾的各三个 “!”，于是得到该结果串。
  ```

  

## 一、数据类型

### 1.1 切片（slice）和数组

数组的长度不能改变，如果想拼接2个数组，或是获取子数组，需要使用切片。切片是数组的抽象。 切片使用数组作为底层结构。**切片包含三个组件：容量，长度和指向底层数组的指针,切片可以随时进行扩展**

```go
package main

import "fmt"

func main() {
    
    // 数组的初始化
    a := [4]float64{67.7, 89.8, 21, 78}
	b := [...]int{2, 3, 5}
    
	//slice1 := make([]float32, 0)
	slice2 := make([]float32, 3, 5)
	fmt.Println(len(slice2), cap(slice2))
	// 添加元素，切片容量可以根据需要自动扩展
	slice2 = append(slice2, 1, 2, 3, 4)   // [0, 0, 0, 1, 2, 3, 4]
	fmt.Println(len(slice2), cap(slice2)) // 7 12
	// 子切片 [start, end)
	sub1 := slice2[3:]  // [1 2 3 4]
	sub2 := slice2[:3]  // [0 0 0]
	sub3 := slice2[1:4] // [0 0 1]
	// 合并切片
	combined := append(sub1, sub2...) // [1, 2, 3, 4, 0, 0, 0]
	fmt.Println(sub1, sub2, sub3, combined)
}
```

**`cap()`函数是获取切片的当前容量，就是初始设置的空闲容量**

### 1.2 字典（map）

```go
// 仅声明
m1 := make(map[string]int)
// 声明时初始化
m2 := map[string]string{
	"Sam": "Male",
	"Alice": "Female",
}
// 赋值/修改
m1["Tom"] = 18
```

### 1.3 指针（pointer）

指针即某个值的地址，类型定义时使用符号`*`，对一个已经存在的变量，使用 `&` 获取该变量的地址。

```go
str := "Golang"
var p *string = &str // p 是指向 str 的指针
*p = "Hello"
fmt.Println(str) // Hello 修改了 p，str 的值也发生了改变
```

一般来说，指针通常在函数传递参数，或者给某个类型定义新的方法时使用。Go 语言中，参数是按值传递的，如果不使用指针，函数内部将会拷贝一份参数的副本，对参数的修改并不会影响到外部变量的值。如果参数使用指针，对参数的传递将会影响到外部变量。

```java
func add(num int) {
	num += 1
}
func realAdd(num *int) {
	*num += 1
}
func main() {
	num := 100
	add(num)
	fmt.Println(num)  // 100，num 没有变化

	realAdd(&num)
	fmt.Println(num)  // 101，指针传递，num 被修改
}
```

* ### make和new的区别

```
二者都是用来做内存分配的。
    make只用于slice、map以及channel的初始化，返回的还是这三个引用类型本身；
    而new用于类型的内存分配，并且内存对应的值为类型零值，返回的是指向类型的指针。
```

## 二、流程控制

### 2.1 else if

```go
age := 18
if age < 18 {
	fmt.Printf("Kid")
} else {
	fmt.Printf("Adult")
}
// 可以简写为：
if age := 18; age < 18 {
	fmt.Printf("Kid")
} else {
	fmt.Printf("Adult")
}
```



### 2.2 switch

```go
type Gender int8
const (
	MALE   Gender = 1
	FEMALE Gender = 2
)

gender := MALE

switch gender {
case FEMALE:
	fmt.Println("female")
case MALE:
	fmt.Println("male")
default:
	fmt.Println("unknown")
}
// male
```

- 在这里，使用了`type` 关键字定义了一个新的类型 Gender。
- 使用 const 定义了 MALE 和 FEMALE 2 个常量，Go 语言中没有枚举(enum)的概念，一般可以用常量的方式来模拟枚举。
- **和其他语言不同的地方在于，Go 语言的 switch 不需要 break**，匹配到某个 case，执行完该 case 定义的行为后，默认不会继续往下执行。如果需要继续往下执行，需要使用 **fallthrough**，例如：

### 2.3 遍历

> 对数组(arr)、切片(slice)、字典(map) 使用 for range 遍历：

```go
nums := []int{10, 20, 30, 40}
for i, num := range nums {
	fmt.Println(i, num)
}
// 0 10
// 1 20
// 2 30
// 3 40
m2 := map[string]string{
	"Sam":   "Male",
	"Alice": "Female",
}

for key, value := range m2 {
	fmt.Println(key, value)
}
// Sam Male
// Alice Female
```



## 三、函数（function）

* ### 特点

  ```txt
      • 无需声明原型。
      • 支持不定 变参。
      • 支持多返回值。
      • 支持命名返回参数。 
      • 支持匿名函数和闭包。
      • 函数也是一种类型，一个函数可以赋值给变量。
  
      • 不支持 嵌套 (nested) 一个包不能有两个名字一样的函数。
      • 不支持 重载 (overload) 
      • 不支持 默认参数 (default parameter)。
      
      命名如果以小写字母开头，则对包外是不可见的，但是他们在整个包的内部是可见并且可用的
  ```



### 3.1 内置函数

Go 语言拥有一些不用导入操作就可以使用的函数。

```txt
    append          -- 用来追加元素到数组、slice中,返回修改后的数组、slice
    close           -- 主要用来关闭channel
    delete            -- 从map中删除key对应的value
    panic            -- 停止常规的goroutine  （panic和recover：用来做错误处理）
    recover         -- 允许程序定义goroutine的panic动作
    imag            -- 返回complex的实部   （complex、real imag：用于创建和操作复数）
    real            -- 返回complex的虚部
    make            -- 用来分配内存，返回Type本身(只能应用于slice, map, channel)
    new                -- 用来分配内存，主要用来分配值类型，比如int、struct。返回指向Type的指针
    cap                -- capacity是容量的意思，用于返回某个类型的最大容量（只能用于切片和 map）
    copy            -- 用于复制和连接slice，返回复制的数目
    len                -- 来求长度，比如string、array、slice、map、channel ，返回长度
    print、println     -- 底层打印函数，在部署环境中建议使用 fmt 包
```

### 3.2 参数与返回值

一个典型的函数定义如下，使用关键字 `func`，参数可以有多个，返回值也支持有多个。特别地，`package main` 中的`func main()` 约定为可执行程序的入口。

```go
func funcName(param1 Type1, param2 Type2, ...) (return1 Type3, ...) {
    // body
}
```

可以给返回值命名，简化 return

```go
func add(num1 int, num2 int) (ans int) {
	ans = num1 + num2
	return
}
```

### 3. 3 错误处理（error handling）

如果函数实现过程中，如果出现不能处理的错误，可以返回给调用者处理。比如我们调用标准库函数`os.Open`读取文件，`os.Open` 有2个返回值，第一个是 `*File`，第二个是 `error`， 如果调用成功，error 的值是 nil，如果调用失败，例如文件不存在，我们可以通过 error 知道具体的错误信息。

```go
import (
	"fmt"
	"os"
)

func main() {
	_, err := os.Open("filename.txt")
	if err != nil {
		fmt.Println(err)
	}
}

// open filename.txt: no such file or directory
```

可以通过 `errorw.New` 返回自定义的错误

```go
import (
	"errors"
	"fmt"
)
func hello(name string) error {
	if len(name) == 0 {
		return errors.New("error: name is null")
	}
	fmt.Println("Hello,", name)
	return nil
}

func main() {
	if err := hello(""); err != nil {
		fmt.Println(err)
	}
}
// error: name is null
```

error 往往是能预知的错误，但是也可能出现一些不可预知的错误，例如数组越界，这种错误可能会导致程序非正常退出，在 Go 语言中称之为 **panic**(类似java中的运行时异常？)。

```go
func get(index int) int {
	arr := [3]int{2, 3, 4}
	return arr[index]
}

func main() {
	fmt.Println(get(5))
	fmt.Println("finished")
}

-------------
 go run .
panic: runtime error: index out of range [5] with length 3
goroutine 1 [running]:
exit status 2
```

在 Python、Java 等语言中有 `try...catch` 机制，在 `try` 中捕获各种类型的异常，在 `catch` 中定义异常处理的行为。Go 语言也提供了类似的机制 `defer` 和 `recover`。

```go
func get(index int) (ret int) {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("Some error happened!", r)
			ret = -1
		}
	}()
	arr := [3]int{2, 3, 4}
	return arr[index]
}

func main() {
	fmt.Println(get(5))
	fmt.Println("finished")
}
-------------------------------------
$ go run .
Some error happened! runtime error: index out of range [5] with length 3
-1
finished
```

- 在 get 函数中，使用 defer 定义了异常处理的函数，在协程退出前，会执行完 defer 挂载的任务。因此如果触发了 panic，控制权就交给了 defer。
- 在 defer 的处理逻辑中，使用 recover，使程序恢复正常，并且将返回值设置为 -1，在这里也可以不处理返回值，如果不处理返回值，返回值将被置为默认值 0。

### 3.4 特殊函数

#### 3.4.1 init()

Go语言中`init()` 函数用于包`(package)`的初始化，该函数是go语言的一个重要的特性。

有以下特性：

```txt
1.该函数是用于程序执行前做包的初始化的函数，比如初始化包里的变量，是不能有返回值的。
2.每个包都可以有多个init函数，包的每个源文件也可以拥有多个init函数
3.同一个包中的执行顺序没有明确定义。
4.不同包的执行顺序会根据包的导入依赖关系来执行
5.不能被其它函数调用，而是在main函数执行之前，自己被调用
```

#### 3.4.2 main()

是默认的入口函数

* #### inti函数和main函数的异同

  ```txt
      相同点：
          两个函数在定义时不能有任何的参数和返回值，且Go程序自动调用。
      不同点：
          init可以应用于任意包中，且可以重复定义多个。
          main函数只能用于main包中，且只能定义一个。
  ```

### 3.5 匿名函数

匿名函数由一个不带函数名的函数声明和函数体组成。匿名函数的优越性在于可以直接使用函数内的变量，不必申明。

```go
package main

import "fmt"

func main() {
    getSqrt := func(a float64) float64{
        return math.Sqrt(a)
    }
    fmt.Println(getSqrt(4));
}
```

匿名函数可赋值给变量，作为结构字段，或者在channel里传送

```go
package main

func main() {
    // --- function variable ---
    fn := func() { println("Hello, World!") }
    fn()

    // --- function collection ---
    fns := [](func(x int) int){
        func(x int) int { return x + 1 },
        func(x int) int { return x + 2 },
    }
    println(fns[0](100))

    // --- function as field ---
    d := struct {
        fn func() string
    }{
        fn: func() string { return "Hello, World!" },
    }
    println(d.fn())

    // --- channel of function ---
    fc := make(chan func() string, 2)
    fc <- func() string { return "Hello, World!" }
    println((<-fc)())
}
```

### 3.6 闭包

> 什么是闭包？

闭包就是一个函数+引用环境。指的是一个拥有许多变量和**绑定**了这些变量的环境表达式，故这些变量也是该表达式的一部分。

例如在javascript中的例子：

```javascript
function a(){
    var i=0;
    function b(){
        console.log(++i);
        document.write("<h1>"+i+"</h1>");
    }
    return b;
}
$(function(){
    var c=a();
    c();
    c();
    c();
    //a(); //不会有信息输出
    document.write("<h1>=============</h1>");
    var c2=a();
    c2();
    c2();
});
```

这里是将`b()`函数返回给了c，然后调用c这时就形成了一个闭包，它绑定了a函数中的i变量，不会被GC掉，并且对于c2这个变量的闭包的不同的闭包，他们的环境是不同的。

**go中的闭包**

```go
package main

import (
    "fmt"
)

func a() func() int {
    i := 0
    b := func() int {
        i++
        fmt.Println(i)
        return i
    }
    return b
}
func main() {
    c := a()
    c()
    c()
    c()

    a() //不会输出i
}
```



### 3.7 延迟调用（defer）

**defer关键字特点：**

```txt
1.用于注册延迟调用
2.知道return前才被执行。因此，可以用来做资源清理。
3.多个defer语句，按先进后出方式执行
4.defer语句中的变量，在defer声明时就决定了
```

**defer的用途**

```txt
1.关闭文件句柄
2.锁资源释放
3.数据库连接释放
```





## 四、结构体，方法和接口

结构体类似于其他语言中的 class，可以在结构体中定义多个字段，为结构体实现方法，实例化等。接下来我们定义一个结构体 Student，并为 Student 添加 name，age 字段，并实现 `hello()` 方法。

```go
type Student struct {
	name string
	age  int
}

func (stu *Student) hello(person string) string {
	return fmt.Sprintf("hello %s, I am %s", person, stu.name)
}

func main() {
	stu := &Student{
		name: "Tom",
	}
	msg := stu.hello("Jack")
	fmt.Println(msg) // hello Jack, I am Tom
}
```

- 使用 `Student{field: value, ...}`的形式创建 Student 的实例，字段不需要每个都赋值，没有显性赋值的变量将被赋予默认值，例如 age 将被赋予默认值 0。
- **实现方法与实现函数的区别在于，`func` 和函数名`hello` 之间，加上该方法对应的实例名 `stu` 及其类型 `*Student`，可以通过实例名访问该实例的字段`name`和其他方法了。**
- 调用方法通过 `实例名.方法名(参数)` 的方式。
- `Springtf`格式化返回一个字符串

还可以用`new`实例化

```go
func main() {
	stu2 := new(Student)
	fmt.Println(stu2.hello("Alice")) // hello Alice, I am  , name 被赋予默认值""
}
```

### 4.1 接口（interfaces）

```go
type Person interface {
	getName() string
}

type Student struct {
	name string
	age  int
}

func (stu *Student) getName() string {
	return stu.name
}

type Worker struct {
	name   string
	gender string
}

func (w *Worker) getName() string {
	return w.name
}

func main() {
	var p Person = &Student{
		name: "Tom",
		age:  18,
	}

	fmt.Println(p.getName()) // Tom
}
```

- Go 语言中，并不需要显式地声明实现了哪一个接口，只需要直接实现该接口对应的方法即可。
- 实例化 `Student`后，强制类型转换为接口类型 Person。

在上面的例子中，我们在 main 函数中尝试将 Student 实例类型转换为 Person，如果 Student 没有完全实现 Person 的方法，比如我们将 `(*Student).getName()` 删掉，编译时会出现如下报错信息。

```
*Student does not implement Person (missing getName method)
```

### 4.2 类型转换

* #### 类型断言

  ```go
  var s = x.(T)
  ```

  如果x不是nil，且x可以转换成T类型，就会断言成功，返回T类型的变量s。**如果T不是接口类型，则要求x的类型就是T，如果T是一个接口，要求x实现了T接口。**

  如果断言类型成立，则表达式返回值就是T类型的x，如果断言失败就会触发panic。

  上述表所示再断言失败就会panic，go提供了另外一种带返回是否成立的断言语法：

  ```go
  s, ok := x.(T)
  ```

  该方法和第一种差不多一样，但是ok会返回是否断言成功不会出现panic，ok就表示是否是成功了。

  

```go
var _ Person = (*Student)(nil)
var _ Person = (*Worker)(nil)
```

- 将空值 nil 转换为 *Student 类型，再转换为 Person 接口，如果转换失败，说明 Student 并没有实现 Person 接口的所有方法。
- Worker 同上。

实例可以强制类型转换为接口，接口也可以强制类型转换为实例。

```go
func main() {
	var p Person = &Student{
		name: "Tom",
		age:  18,
	}

	stu := p.(*Student) // 接口转为实例
	fmt.Println(stu.getAge())
}
```



### 4.3 空接口

如果定义了一个没有任何方法的空接口，那么这个接口可以表示任意类型。例如

```go
func main() {
	m := make(map[string]interface{})
	m["name"] = "Tom"
	m["age"] = 18
	m["scores"] = [3]int{98, 99, 85}
	fmt.Println(m) // map[age:18 name:Tom scores:[98 99 85]]
}
```



### 4.4 匿名字段

go支持只提供类型而不写字段名的方式，也就是匿名字段，也称嵌入式字段。

```go
type Person struct{
    name string
    sex string
    age int
}
type Student struct{
    Person
    id int
    addr string
}
```

**所有的内置类型和自定义类型都是可以作为匿名字段去使用**

```go
type Person struct{
    name string
    sex string
    age int
}
// 自定义类型
type mystr string
type Student struct{
    Person
    int
    mystr
}
```



## 五、网络编程

### 5.1 TCP

> 一个TCP服务端可以同时连接很多个客户端。因为GO语言中创建多个goroutine实现并发非常方便和高效，所以我们可以**每建立一次链接就创建一个goroutine去处理**

#### 5.1.1 TCP服务端

TCP服务端程序的处理流程：

```txt
1. 监听端口
2. 接收客户端请求，建立连接
3. 创建goroutine处理连接
```

我们使用GO语言的`net`包实现的TCP服务端代码如下：

```go
// tcp/server/main.go
package main

import (
	"bufio"
	"fmt"
	"net"
)
func process(conn net.Conn) {
    defer conn.Close() // 关闭连接
    for {
        reader := bufio.NewReader(conn)
        var buf [128]byte
        n,  err := reader.Read(buf[:])//读取数据
        if err != nil {
            fmt.Println("read from client failed, err:" , err)
            break
        }
        recvStr := string(buf[:n])
        fmt.Println("client:", recvStr)
        //修改数据
        sendStr := string("server:" + recvStr)
        
        conn.Write([]byte(sendStr))//发送数据
    }
}
func main() {
    listen, err := net.Listen("tcp", "127.0.0.1:9999")
    if err != nil {
        fmt.Println("listen failed, err:", err)
        return
    }
    for {
        conn, err := listen.Accept() //建立连接
        if err != nil {
            fmt.Println("accept failed, err:", err)
            continue
        }
        go process(conn) //启动一个goroutine处理连接 
    }
}
```

#### 5.1.2 TCP客户端

TCP客户端流程

```tx
1. 建立与服务端的连接
2. 进行数据收发
3. 关闭连接
```

使用Go语言的net包是实现客户端

```go
// tcp/client/main.go
package main

import (
	"bufio"
	"fmt"
	"net"
	"os"
	"strings"
)

func main() {
	conn, err := net.Dial("tcp", "127.0.0.1:9999")
	if err != nil {
		fmt.Println("err :", err)
		return
	}
	defer conn.Close()
	inputReader := bufio.NewReader(os.Stdin)
	for {
		input, _ := inputReader.ReadString('\n') //读入控制台内容
		inputInfo := strings.Trim(input, "\r\n")
		if strings.ToUpper(inputInfo) == "Q" { //如果输入q就退出
			return
		}
		_, err = conn.Write([]byte(inputInfo)) //发送数据
		if err != nil {
			return
		}
		buf := [512]byte{}
		n, err := conn.Read(buf[:])
		if err != nil {
			fmt.Println("recv failed, err:", err)
			return
		}
		fmt.Println(string(buf[:n]))
	}
}
```

### 5.2 UDP

#### 5.2.1 UDP 服务端

```go
package main

import (
	"fmt"
	"net"
)

func main() {
	listen, err := net.ListenUDP("udp", &net.UDPAddr{
		IP:   net.IPv4(0, 0, 0, 0),
		Port: 30000,
	})
	if err != nil {
		fmt.Println("listen failed, err:", err)
		return
	}
	defer listen.Close()
	for {
		var data [1024]byte
		n, addr, err := listen.ReadFromUDP(data[:]) //接收数据
		if err != nil {
			fmt.Println("read udp failed, err:", err)
			continue
		}
		fmt.Printf("data:%v addr:%v count:%v \n", string(data[:n]), addr, n)
		sendData := string(data[:n])
		sendData = "server:" + sendData
		_, err = listen.WriteToUDP([]byte(sendData), addr) // 发数据
		if err != nil {
			fmt.Println("write to UDP failed, err:", err)
			continue
		}

	}
}

```

#### 5.2.2 UDP 客户端

```go
package main

import (
	"fmt"
	"net"
)

func main() {
	socket, err := net.DialUDP("udp", nil, &net.UDPAddr{
		IP:   net.IPv4(0, 0, 0, 0),
		Port: 30000,
	})
	if err != nil {
		fmt.Println("connect to server failed, err:", err)
		return
	}
	defer socket.Close()
	sendData := []byte("hello server")
	_, err = socket.Write(sendData)
	if err != nil {
		fmt.Println("send data failed, err:", err)
		return
	}
	data := make([]byte, 4096)

	n, remoteAddr, err := socket.ReadFromUDP(data) //接收数据
	if err != nil {
		fmt.Println("recevice data failed, err:", err)
		return
	}
	fmt.Printf("recv:%v addr:%v count:%v\n", string(data[:n]), remoteAddr, n)

}
```





### 5.3 TCP 黏包

> 主要原因就是tcp数据传递模式是流模式，在保持长连接的时候可以进行多次的收和发。
>
> “粘包”可发生在发送端也可发生在接收端：
>
> ```
>     1.由Nagle算法造成的发送端的粘包：Nagle算法是一种改善网络传输效率的算法。简单来说就是当我们提交一段数据给TCP发送时，TCP并不立刻发送此段数据，而是等待一小段时间看看在等待期间是否还有要发送的数据，若有则会一次把这两段数据发送出去。
>     2.接收端接收不及时造成的接收端粘包：TCP会把接收到的数据存在自己的缓冲区中，然后通知应用层取数据。当应用层由于某些原因不能及时的把TCP的数据取出来，就会造成TCP缓冲区中存放了几段数据。
> ```



* #### 解决办法

  > 出现"黏包"的关键在于双发不知道将要传输的数据包的大小，因此我们可以对数据包进行封装。自定义协议：我们可以自己定义一个协议，比如数据包的前4个字节为包头，里面存储的是发送的数据的长度。

  创建 proto 包，用于对数据的解码和编码

  ```go
  package proto
  
  import (
  	"bufio"
  	"bytes"
  	"encoding/binary"
  )
  
  // Encode 将消息编码
  func Encode(message string) ([]byte, error) {
  	// 读取消息长度，转化成int32类型（占4个字节）
  	var length = int32(len(message))
  	var pkg = new(bytes.Buffer)
  
  	// 写入消息头
  	err := binary.Write(pkg, binary.LittleEndian, length)
  	if err != nil {
  		return nil, err
  	}
  	// 写入消息实体
  	err = binary.Write(pkg, binary.LittleEndian, []byte(message))
  	if err != nil {
  		return nil, err
  	}
  	return pkg.Bytes(), nil
  
  }
  
  // Decode 解码消息
  func Decode(reader *bufio.Reader) (string, error) {
  	lengthByte, _ := reader.Peek(4) //读取前四个字节
  	lengthBuff := bytes.NewBuffer(lengthByte)
  	var length int32
  	err := binary.Read(lengthBuff, binary.LittleEndian, &length)
  	if err != nil {
  		return "", err
  	}
  	// Buffered返回缓存中现有的可读取的字节数
  	if int32(reader.Buffered()) < length+4 {
  		return "", err
  	}
  	// 读取真正的消息数据
  	pack := make([]byte, int(4+length))
  	_, err = reader.Read(pack)
  	if err != nil {
  		return "", err
  	}
  	return string(pack[4:]), nil
  }
  ```
  

### 5.4 http协议

**服务端server：**

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	// http://127.0.0.1:8000/go
	// 单独写回调函数
	http.HandleFunc("/go", myHandler)
	/*
		addr: 监听的地址和端口
		handler: 回调函数
	*/
	http.ListenAndServe("127.0.0.1:8000", nil)
}

func myHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Println(r.RemoteAddr, "-连接成功")
	fmt.Println("method:", r.Method)
	// /go
	fmt.Println("url:", r.URL.Path)
	fmt.Println("header:", r.Header)
	fmt.Println("body:", r.Body)

	w.Write([]byte("server: receive success!!!"))

}

```

**客户端client：**

```go
package main

import (
	"fmt"
	"io"
	"net/http"
)

func main() {
	resp, _ := http.Get("http://127.0.0.1:8000/go")

	defer resp.Body.Close()

	fmt.Println("status:", resp.Status)
	fmt.Println("header:", resp.Header)

	buf := make([]byte, 1024)

	for {
		// 接收服务端信息
		n, err := resp.Body.Read(buf)
		if err != nil && err != io.EOF {
			fmt.Println(err)
			return
		} else {
			fmt.Println("读取完毕")
			res := string(buf[:n])
			fmt.Println(res)
			break
		}
	}
}
```





### 5.5 WebSocket

> 什么是websocket？

- WebSocket是一种在单个TCP连接上进行全双工通信的协议

- WebSocket使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据

- 在WebSocket API中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输

- 需要安装第三方包：

  - cmd中：

    ```shell
    go get -u -v github.com/gorilla/websocket`
    go get -u github.com/gorilla/mux  
    ```

Go 版本>=1.11 设置GOPROXY
在 Linux 或 macOS 上面，需要运行下面命令：
Bash / 复制

```shell
# 启用 Go Modules 功能
export GO111MODULE=on
# 配置 GOPROXY 环境变量
export GOPROXY=https://goproxy.io
# 使用阿里云仓库
export GOPROXY=https://mirrors.aliyun.com/goproxy/
```

或者，可以把上面的命令写到 .bashrc 或 .bash_profile 文件当中。

在 Windows 上，需要运行下面命令：
PowerShell  复制

```shell
go env -w GOPROXY=https://goproxy.cn
```

现在，当你构建或运行你的应用时，Go 将会通过 goproxy.io 获取依赖。更多信息请查看 goproxy 仓库。

#### 5.5.2 聊天室案例

在同一级目录下新建四个go文件`connection.go|data.go|hub.go|server.go`

**运行**

```shell
go run server.go hub.go data.go connection.go
```













## 六、并发编程（goroutine）

> goroutine 知识由官方实现的“超级线程池”。
>
> 每个实例 `4~5KB`的栈内存占用和由于实现机制而大幅减少的创建和销毁开销是go高并发的根本原因。



### 6.1 Goroutine

> 在java/c++中我们要实现并发编程的时候，我们通常需要自己来维护一个线程池，将线程池资源分配给任务使用。这就要我们自己来管理、调度线程执行任务并维护上下文切换。
>
> Go语言中的 goroutine 就减少了大量的维护操作，goroutine的概念类似于线程，它的调度都是由运行时（runtime）进行的。Go程序会智能地将gorontine中的任务合理地分配给CPU处理。并且go语言内置有一套上下文切换算法。
>
> #### 使用goroutine
>
> 在Go语言中使用goroutine是十分地简单粗暴，只需要在调用函数时加上一个go关键字，就能为一个函数创建一个goroutine

#### 6.1.1 Goroutine 特性

**案例：**

```go
func hello(){
    fmt.Println("Hello Goroutine!")
}
func main(){
    go hello()
    fmt.Println("main goroutine done!")
}
```

在执行之后，会发现只打印了`"main goroutine done!"`

```
原因是当main函数启动时，Go会为main创建一个goroutine，在主协程结束时，会GC所有在main函数中创建的goroutine。
我们可以让主程序暴力地等待一会
time.Sleep(time.Second)
```

**启动多个goroutine：**

```go
var wg sync.WaitGroup

func hellos(i int) {
	// goroutine结束就登记-1
	defer wg.Done()
	fmt.Println("Hello Goroutine!", i)
}
func main() {

	for i := 0; i < 10; i++ {
		// 启动一个goroutine 就登记+1
		wg.Add(1)
		go hellos(i)
	}
	// 等待所有登记的goroutine都结束
	wg.Wait()
}
```



#### 6.1.2 goroutine与线程

##### 可增长的栈

OS线程（操作系统线程）一般都有固定的占内存（通常为2MB），一个goroutine的栈在其生命周期开始时只有很小的栈（2KB），goroutine的栈不是固定的，他可以按需增大和缩小。

##### goroutine调度

**GMP**是Go语言运行时（runtime）层面的实现，是go语言自己实现的一套调度系统。区别于操作系统调度OS线程。

* **G（goroutine）：**里面存放本 goroutine 信息外，还有与所在P的绑定信息
* **P（processor）：**即处理器，每个P会管理一组goroutine队列，P会存储当前goroutine运行的上下文环境（函数指针，堆栈地址及地址边界）。P会对自己管理的goroutine队列做一些调度（比如把占用CPU时间较长的goroutine暂停、运行后续的goroutine等等）当自己的队列消费完了就去全局队列里取，如果全局队列里也消费完了会去其他P的队列里抢任务。
* **M（machine）：**代表系统线程，负责执行goroutine。是Go运行时（runtime）对操作系统内核线程的虚拟， M与内核线程一般是一一映射的关系， 一个groutine最终是要放到M上执行的；

P与M一般也是一一对应的。他们关系是： P管理着一组G挂载在M上运行。当一个G长久阻塞在一个M上时，runtime会新建一个M，阻塞G所在的P会把其他的G 挂载在新建的M上。当旧的G阻塞完成或者认为其已经死掉时 回收旧的M。

P的个数是通过runtime.GOMAXPROCS设定（最大256），Go1.5版本之后默认为物理线程数。 在并发量大的时候会增加一些P和M，但不会太多，切换太频繁的话得不偿失。

单从线程调度讲，Go语言相比起其他语言的优势在于OS线程是由OS内核来调度的，goroutine则是由Go运行时（runtime）自己的调度器调度的，这个调度器使用一个称为m:n调度的技术（复用/调度m个goroutine到n个OS线程）。 其一大特点是goroutine的调度是在用户态下完成的， 不涉及内核态与用户态之间的频繁切换，包括内存的分配与释放，都是在用户态维护着一块大的内存池， 不直接调用系统的malloc函数（除非内存池需要改变），成本比调度OS线程低很多。 另一方面充分利用了多核的硬件资源，近似的把若干goroutine均分在物理线程上， 再加上本身goroutine的超轻量，以上种种保证了go调度方面的性能。



### 6.2 runtime 包

#### runtime.Gosched()

> 让出CPU时间片，重新等待安排任务

#### runtime.Goexit()

> 退出当前协程

```go
func main() {
	go func() {
		defer fmt.Println("A.defer")
		func() {
			defer fmt.Println("B.defer")
			// 结束协程
			runtime.Goexit()
			defer fmt.Println("C.defer")
			fmt.Println("B")
		}()
		fmt.Println("A")
	}()
	for {
	}
}
------------------------------
B.defer
A.defer
```

#### runtime.GOMAXPROCS

> Go运行时的调度器使用GOMAXPROCS参数来确定需要使用多少个OS线程来同时执行Go代码。默认值是机器上的CPU核心数。例如在一个8核心的机器上，调度器会把Go代码同时调度到8个OS线程上（GOMAXPROCS是m:n调度中的n）。
>
> Go语言中可以通过runtime.GOMAXPROCS()函数设置当前程序并发时占用的CPU逻辑核心数。
>
> Go1.5版本之前，默认使用的是单核心执行。Go1.5版本之后，默认使用全部的CPU逻辑核心数。

我们可以通过将任务分配到不同的CPU逻辑核心上实现并行的效果，这里举个例子：

```go
func a() {
    for i := 1; i < 10; i++ {
        fmt.Println("A:", i)
    }
}

func b() {
    for i := 1; i < 10; i++ {
        fmt.Println("B:", i)
    }
}

func main() {
    runtime.GOMAXPROCS(1)
    go a()
    go b()
    time.Sleep(time.Second)
}
-------------------
B: 1
B: 2
B: 3
B: 4
B: 5
B: 6
B: 7
B: 8
B: 9
A: 1
A: 2
A: 3
A: 4
A: 5
A: 6
A: 7
A: 8
A: 9
```



### 6.3 Channel

> 单纯地将函数并发执行是没有意义的。函数与函数间需要交换数据才能体现并发执行函数的意义。
>
> 虽然可以使用共享内存进行数据交换，但是共享内存在不同的goroutine中容易发生竞态问题。为了保证数据交换的正确性，必须使用互斥量对内存进行加锁，这种做法势必造成性能问题。
>
> Go语言的并发模型是CSP（Communicating Sequential Processes），**提倡通过通信共享内存而不是通过共享内存而实现通信。**
>
> 如果说goroutine是Go程序并发的执行体，channel就是它们之间的连接。channel是可以让一个goroutine发送特定值到另一个goroutine的通信机制。
>
> Go 语言中的通道（channel）是一种特殊的类型。通道像一个传送带或者队列，总是遵循先入先出（First In First Out）的规则，保证收发数据的顺序。每一个通道都是一个具体类型的导管，也就是声明channel的时候需要为其指定元素类型。

#### 6.3.1**创建channel：**

```go
var 变量 chan 元素类型
------------------
var ch1 chan int   // 声明一个传递整型的通道
var ch2 chan bool  // 声明一个传递布尔型的通道
var ch3 chan []int // 声明一个传递int切片的通道


chan是引用类型，需要make函数初始化
make(chan 元素类型, [缓冲大小])
-------------------------
ch4 := make(chan int)
ch5 := make(chan bool)
ch6 := make(chan []int)
```



#### 6.3.2 channel操作

通道有发送（send）、接收(receive）和关闭（close）三种操作。

发送和接收都使用<-符号。

现在我们先使用以下语句定义一个通道：

```go
ch := make(chan int)
```

**发送**

将一个值发送到通道中。

```go
ch <- 10 // 把10发送到ch中
```

**接收**

从一个通道中接收值。

```go
x := <- ch // 从ch中接收值并赋值给变量x
<-ch       // 从ch中接收值，忽略结果
```

通道有发送（send）、接收(receive）和关闭（close）三种操作。

发送和接收都使用<-符号。

现在我们先使用以下语句定义一个通道：

```go
ch := make(chan int)
```

**关闭**

我们通过调用内置的close函数来关闭通道。

```go
    close(ch)
```

关于关闭通道需要注意的事情是，只有在通知接收方goroutine所有的数据都发送完毕的时候才需要关闭通道。通道是可以被垃圾回收机制回收的，它和关闭文件是不一样的，在结束操作之后关闭文件是必须要做的，但关闭通道不是必须的。

关闭后的通道有以下特点：

```
    1.对一个关闭的通道再发送值就会导致panic。
    2.对一个关闭的通道进行接收会一直获取值直到通道为空。
    3.对一个关闭的并且没有值的通道执行接收操作会得到对应类型的零值。
    4.关闭一个已经关闭的通道会导致panic
```



#### 6.3.3 无缓冲通道

无缓冲的通道又称为阻塞的通道。我们来看一下下面的代码：

```go
func main() {
    ch := make(chan int)
    ch <- 10
    fmt.Println("发送成功")
}
```

上面这段代码能够通过编译，但是执行的时候会出现以下错误：

```
    fatal error: all goroutines are asleep - deadlock!

    goroutine 1 [chan send]:
    main.main()
            .../src/github.com/pprof/studygo/day06/channel02/main.go:8 +0x54
```

为什么会出现deadlock错误呢？

因为我们使用ch := make(chan int)创建的是无缓冲的通道，无缓冲的通道只有在有人接收值的时候才能发送值。就像你住的小区没有快递柜和代收点，快递员给你打电话必须要把这个物品送到你的手中，简单来说就是无缓冲的通道必须有接收才能发送。

上面的代码会阻塞在ch <- 10这一行代码形成死锁，那如何解决这个问题呢？

一种方法是启用一个goroutine去接收值，例如：

```go
func recv(c chan int) {
    ret := <-c
    fmt.Println("接收成功", ret)
}
func main() {
    ch := make(chan int)
    go recv(ch) // 启用goroutine从通道接收值
    ch <- 10
    fmt.Println("发送成功")
}
```

无缓冲通道上的发送操作会阻塞，直到另一个goroutine在该通道上执行接收操作，这时值才能发送成功，两个goroutine将继续执行。相反，如果接收操作先执行，接收方的goroutine将阻塞，直到另一个goroutine在该通道上发送一个值。

使用无缓冲通道进行通信将导致发送和接收的goroutine同步化。因此，无缓冲通道也被称为同步通道。



#### 6.3.4 缓冲通道

我们可以在使用make函数初始化通道的时候为其指定通道的容量，例如：

```go
func main() {
    ch := make(chan int, 1) // 创建一个容量为1的有缓冲区通道
    ch <- 10
    fmt.Println("发送成功")
}
```

只要通道的容量大于零，那么该通道就是有缓冲的通道，通道的容量表示通道中能存放元素的数量。就像你小区的快递柜只有那么个多格子，格子满了就装不下了，就阻塞了，等到别人取走一个快递员就能往里面放一个。

我们可以使用内置的len函数获取通道内元素的数量，使用cap函数获取通道的容量，虽然我们很少会这么做。



#### 6.3.5 从通道循环取值

> 当通过通道发送数据时，通道关闭了，我们要如何知道通道是否关闭呢？

```go
package main

import "fmt"

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)

	// 开启goroutine将0~100的数发送到ch1
	go func() {
		for i := 0; i < 100; i++ {
			ch1 <- i
		}
		close(ch1)
	}()

	// 开启goroutine从ch1中取值，并平方后发送给ch2
	go func() {
		for {
			i, ok := <-ch1
			if !ok {
				break
			}
			ch2 <- i * i
		}
		close(ch2)
	}()

	for i := range ch2 {
		fmt.Println(i)
	}
}
```



#### 6.3.6 单向通道

的时候我们会将通道作为参数在多个任务函数间传递，很多时候我们在不同的任务函数中使用通道都会对其进行限制，比如限制通道在函数中只能发送或只能接收。

```go
package main

import "fmt"

func counter(out chan<- int) {
	for i := 0; i < 100; i++ {
		out <- i
	}
	close(out)
}

func squarer(out chan<- int, in <-chan int) {
	for i := range in {
		out <- i * i
	}
	close(out)
}

func printer(in <-chan int) {
	for i := range in {
		fmt.Println(i)
	}
}

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	go counter(ch1)
	go squarer(ch2, ch1)
	printer(ch2)
}

-------------------
   1.chan<- int是一个只能发送的通道，可以发送但是不能接收；
    2.<-chan int是一个只能接收的通道，可以接收但是不能发送。
```





### 6.4 Goroutine 池

#### worker pool

> - 本质上是生产者消费者模型
> - 可以有效控制goroutine数量，防止暴涨
> - 需求：
>   - 计算一个数字的各个位数之和，例如数字123，结果为1+2+3=6
>   - 随机生成数字进行计算

```go
package main

import (
	"fmt"
	"math/rand"
)
// 任务结构
type Job struct {
	// id
	Id int
	// 需要计算的数
	RandNum int
}
// 结果实例
type Result struct {
	// 任务实例
	job *Job
	// 求和结果
	sum int
}
func main() {
	// 1. job管道
	jobChan := make(chan *Job, 128)
	// 2. 结果管道
	resultChan := make(chan *Result, 128)
	// 3. 创建工作池
	createPool(64, jobChan, resultChan)
	// 4. 启动一个打印协程
	go func(resultChan chan *Result) {
		for result := range resultChan {
			fmt.Printf("job id:%v randnum: %v result:%d\n", result.job.Id, result.job.RandNum, result.sum)

		}
	}(resultChan)
	var id int
	// 循环创建job，输入到管道
	for {
		id++
		// 生成随机数
		r_num := rand.Int()
		job := &Job{
			Id:      id,
			RandNum: r_num,
		}
		jobChan <- job
	}
}
func createPool(num int, jobChan chan *Job, resultChan chan *Result) {
	// 根据开协程个数，启动goroutine
	for i := 0; i < num; i++ {
		go func(jobChan chan *Job, resultChan chan *Result) {
			// 执行运算
			// 遍历job管道所有数据，进行相加
			for job := range jobChan {
				r_num := job.RandNum
				// 随机数没一位相加
				var sum int
				for r_num != 0 {
					tmp := r_num % 10
					sum += tmp
					r_num /= 10
				}
				r := &Result{
					job: job,
					sum: sum,
				}
				resultChan <- r
			}

		}(jobChan, resultChan)
	}
}
```



### 6.5 定时器 Timer

当定时器的时间未到，会进行阻塞。

```go
package main

import (
	"fmt"
	"time"
)

// 定时器的基本使用
func main() {
	fmt.Println("----定时器的基本使用-----")
	timer1 := time.NewTimer(2 * time.Second)
	t1 := time.Now()
	fmt.Printf("t1:%v\n", t1)
	t2 := <-timer1.C
	fmt.Printf("t2:%v\n", t2)

	fmt.Println("---验证timer只能使用一次---")
	timer2 := time.NewTimer(time.Second)
	for {
		<-timer2.C
		fmt.Println("时间到")
		break
	}

	fmt.Println("---timer实现延时功能---")
	time.Sleep(time.Second)
	timer3 := time.NewTimer(2 * time.Second)
	<-timer3.C
	fmt.Println("两秒到")
	<-time.After(2 * time.Second)
	fmt.Println("两秒到")

	fmt.Println("---停止定时器---")
	timer4 := time.NewTimer(2 * time.Second)
	go func() {
		<-timer4.C
		fmt.Println("定时器执行了")
	}()
	b := timer4.Stop()
	if b {
		fmt.Println("time4已经关闭")
	}

	fmt.Println("---重置定时器---")
	timer5 := time.NewTimer(3 * time.Second)
	timer5.Reset(1 * time.Second)
	fmt.Println(time.Now())
	fmt.Println(<-timer5.C)
}

---------------------------------------
----定时器的基本使用-----
t1:2024-02-11 22:16:28.9814112 +0800 CST m=+0.001593101
t2:2024-02-11 22:16:30.990381 +0800 CST m=+2.010562901
---验证timer只能使用一次---
时间到
---timer实现延时功能---
两秒到
两秒到
---停止定时器---
time4已经关闭
---重置定时器---
2024-02-11 22:16:37.0337932 +0800 CST m=+8.053975101
2024-02-11 22:16:38.0470336 +0800 CST m=+9.067215501
```

#### 定时器多次执行

> `time.NewTicker(d Duration)`

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	// 获取ticker对象
	ticker := time.NewTicker(time.Second)
	i := 0
	// 子协程
	go func() {
		for {
			i++
			fmt.Println(<-ticker.C)
			if i == 5 {
				// 停止
				ticker.Stop()
			}
		}
	}()
	for {
	}
}
----------------------------
2024-02-11 22:21:52.6693366 +0800 CST m=+1.004385301
2024-02-11 22:21:53.6688886 +0800 CST m=+2.003937301
2024-02-11 22:21:54.6690899 +0800 CST m=+3.004138601
2024-02-11 22:21:55.6691911 +0800 CST m=+4.004239801
2024-02-11 22:21:56.6810198 +0800 CST m=+5.016068501
```



### 6.6 select 多路复用

select的使用类似于switch语句，它有一系列case分支和一个默认的分支。每个case会对应一个通道的通信（接收或发送）过程。select会一直等待，直到某个case的通信操作完成时，就会执行case分支对应的语句。具体格式如下：

```go
 select {
    case <-chan1:
       // 如果chan1成功读到数据，则进行该case处理语句
    case chan2 <- 1:
       // 如果成功向chan2写入数据，则进行该case处理语句
    default:
       // 如果上面都没有成功，则进入default处理流程
    }
```

* select可以同时监听一个或多个channel，直到其中一个channel ready,如果多个channel同时ready，则随机选择一个执行

  ```go
  package main
  
  import "fmt"
  
  func main() {
  	// 创建2个管道
  	int_chan := make(chan int, 1)
  	string_chan := make(chan string, 1)
  	go func() {
  		string_chan <- "hello"
  	}()
  	go func() {
  		int_chan <- 1
  	}()
  	select {
  	case value := <-int_chan:
  		fmt.Println("int:", value)
  	case value := <-string_chan:
  		fmt.Println("string:", value)
  	}
  	fmt.Println("main结束")
  }
  ------------------
  int: 1
  main结束
  后创建的goroutine要先执行？
  ```

* 可以用于判断管道是否存满

  ```go
  package main
  
  import (
  	"fmt"
  	"time"
  )
  
  // 判断管道有没有存满
  func main() {
  	// 创建管道
  	output1 := make(chan string, 10)
  	// 子协程写数据
  	go write(output1)
  	// 取数据
  	for s := range output1 {
  		fmt.Println("res:", s)
  		time.Sleep(time.Second)
  	}
  }
  
  func write(ch chan string) {
  	for {
  		select {
  		// 写数据
  		case ch <- "hello":
  			fmt.Println("write hello")
  		default:
  			fmt.Println("channel full")
  		}
  		time.Sleep(time.Millisecond * 500)
  	}
  }
  ```



### 6.7 并发安全和锁

互斥锁

```go
var x int64
var wg sync.WaitGroup
var lock sync.Mutex

func add() {
    for i := 0; i < 5000; i++ {
        lock.Lock() // 加锁
        x = x + 1
        lock.Unlock() // 解锁
    }
    wg.Done()
}
func main() {
    wg.Add(2)
    go add()
    go add()
    wg.Wait()
    fmt.Println(x)
}
```

读写锁

```go
var (
    x      int64
    wg     sync.WaitGroup
    lock   sync.Mutex
    rwlock sync.RWMutex
)
func write() {
    // lock.Lock()   // 加互斥锁
    rwlock.Lock() // 加写锁
    x = x + 1
    time.Sleep(10 * time.Millisecond) // 假设读操作耗时10毫秒
    rwlock.Unlock()                   // 解写锁
    // lock.Unlock()                     // 解互斥锁
    wg.Done()
}
func read() {
    // lock.Lock()                  // 加互斥锁
    rwlock.RLock()               // 加读锁
    time.Sleep(time.Millisecond) // 假设读操作耗时1毫秒
    rwlock.RUnlock()             // 解读锁
    // lock.Unlock()                // 解互斥锁
    wg.Done()
}
func main() {
    start := time.Now()
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go write()
    }
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go read()
    }
    wg.Wait()
    end := time.Now()
    fmt.Println(end.Sub(start))
}
```



### 6.8 sync

Go 语言提供了 sync 和 channel 两种方式支持协程(goroutine)的并发。

例如我们希望并发下载 N 个资源，**多个并发协程之间不需要通信，那么就可以使用 `sync.WaitGroup`**，等待所有并发协程执行结束

```go
import (
	"fmt"
	"sync"
	"time"
)

var wg sync.WaitGroup

func download(url string) {
	fmt.Println("start to download", url)
	time.Sleep(time.Second) // 模拟耗时操作
	wg.Done()
}

func main() {
	for i := 0; i < 3; i++ {
		wg.Add(1)
		go download("a.com/" + string(i+'0'))
	}
	wg.Wait()
	fmt.Println("Done!")
}
```

- `wg.Add(1)`：为 wg 添加一个计数，wg.Done()，减去一个计数。
- `go download()`：启动新的协程并发执行 download 函数。
- `wg.Wait()`：等待所有的协程执行结束。



#### 6.8.1 sync.Once

说在前面的话：这是一个进阶知识点。

在编程的很多场景下我们需要确保某些操作在高并发的场景下只执行一次，例如只加载一次配置文件、只关闭一次通道等。

Go语言中的sync包中提供了一个针对只执行一次场景的解决方案–sync.Once。

sync.Once只有一个Do方法，其签名如下：

```go
var icons map[string]image.Image

var loadIconsOnce sync.Once

func loadIcons() {
    icons = map[string]image.Image{
        "left":  loadIcon("left.png"),
        "up":    loadIcon("up.png"),
        "right": loadIcon("right.png"),
        "down":  loadIcon("down.png"),
    }
}

// Icon 是并发安全的
func Icon(name string) image.Image {
    loadIconsOnce.Do(loadIcons)
    return icons[name]
}
```

sync.Once其实内部包含一个互斥锁和一个布尔值，互斥锁保证布尔值和数据的安全，而布尔值用来记录初始化是否完成。这样设计就能保证初始化操作的时候是并发安全的并且初始化操作也不会被执行多次。

#### 6.8.2 sync.Map

> Go语言的sync包中提供了一个开箱即用的并发安全版map–sync.Map。开箱即用表示不用像内置的map一样使用make函数初始化就能直接使用。同时sync.Map内置了诸如Store、Load、LoadOrStore、Delete、Range等操作方法。

```go
var m = sync.Map{}

func main() {
    wg := sync.WaitGroup{}
    for i := 0; i < 20; i++ {
        wg.Add(1)
        go func(n int) {
            key := strconv.Itoa(n)
            m.Store(key, n)
            value, _ := m.Load(key)
            fmt.Printf("k=:%v,v:=%v\n", key, value)
            wg.Done()
        }(i)
    }
    wg.Wait()
}
```



### 6.9 原子操作（atomic包）

 ```go
 var x int64
 var l sync.Mutex
 var wg sync.WaitGroup
 
 // 普通版加函数
 func add() {
     // x = x + 1
     x++ // 等价于上面的操作
     wg.Done()
 }
 
 // 互斥锁版加函数
 func mutexAdd() {
     l.Lock()
     x++
     l.Unlock()
     wg.Done()
 }
 
 // 原子操作版加函数
 func atomicAdd() {
     atomic.AddInt64(&x, 1)
     wg.Done()
 }
 
 func main() {
     start := time.Now()
     for i := 0; i < 10000; i++ {
         wg.Add(1)
         // go add()       // 普通版add函数 不是并发安全的
         // go mutexAdd()  // 加锁版add函数 是并发安全的，但是加锁性能开销大
         go atomicAdd() // 原子操作版add函数 是并发安全，性能优于加锁版
     }
     wg.Wait()
     end := time.Now()
     fmt.Println(x)
     fmt.Println(end.Sub(start))
 }
 ```

atomic包提供了底层的原子级内存操作，对于同步算法的实现很有用。这些函数必须谨慎地保证正确使用。除了某些特殊的底层应用，使用通道或者sync包的函数/类型实现同步更好。



### 6.10 GMP 原理与调度





## 七、数据操作

### 7.1 操作MySQL

使用第三方开源的mysql库: github.com/go-sql-driver/mysql （mysql驱动） github.com/jmoiron/sqlx （基于mysql驱动的封装）

命令行输入 ：

```shell
    go get github.com/go-sql-driver/mysql 
    go get github.com/jmoiron/sqlx
```

链接mysql

```go
    database, err := sqlx.Open("mysql", "root:XXXX@tcp(127.0.0.1:3306)/test")
    //database, err := sqlx.Open("数据库类型", "用户名:密码@tcp(地址:端口)/数据库名")
```

操作

```go
 
1） import (“github.com/jmoiron/sqlx")
    2)  Db.Begin()        开始事务
    3)  Db.Commit()        提交事务
    4)  Db.Rollback()     回滚事务
```



### 7.2 操作Redis

```shell
  go get github.com/garyburd/redigo/redis  #已过时
  go get github.com/gomodule/redigo/redis
```

连接

```go
package main

import (
    "fmt"
    "github.com/garyburd/redigo/redis"
)

func main() {
    c, err := redis.Dial("tcp", "localhost:6379")
    if err != nil {
        fmt.Println("conn redis failed,", err)
        return
    } 

    fmt.Println("redis conn success")

    defer c.Close()
}
```

#### redis连接池

```go
package main
import(
    "fmt"
    "github.com/garyburd/redigo/redis"
)

var pool *redis.Pool  //创建redis连接池

func init(){
    pool = &redis.Pool{     //实例化一个连接池
        MaxIdle:16,    //最初的连接数量
        // MaxActive:1000000,    //最大连接数量
        MaxActive:0,    //连接池最大连接数量,不确定可以用0（0表示自动定义），按需分配
        IdleTimeout:300,    //连接关闭时间 300秒 （300秒不使用自动关闭）    
        Dial: func() (redis.Conn ,error){     //要连接的redis数据库
            return redis.Dial("tcp","localhost:6379")
        },
    }
}

func main(){
    c := pool.Get() //从连接池，取一个链接
    defer c.Close() //函数运行结束 ，把连接放回连接池

        _,err := c.Do("Set","abc",200)
        if err != nil {
            fmt.Println(err)
            return
        }

        r,err := redis.Int(c.Do("Get","abc"))
        if err != nil {
            fmt.Println("get abc faild :",err)
            return
        }
        fmt.Println(r)
        pool.Close() //关闭连接池
}
```





-+
