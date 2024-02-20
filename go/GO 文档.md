# GO 文档

## LOOP:

LOOP 是 Go 语言中的一个标签（label），用于标记[循环语句](https://so.csdn.net/so/search?q=循环语句&spm=1001.2101.3001.7020)。它由用户定义的标识符后跟一个冒号（:）构成。在这段代码中，使用标签的目的是为了同时跳出内部的 select 语句和外部的 for 循环。



## time

> time.Now() 获取时间戳

```go
time.Now().Unix()				//时间戳（秒） 
time.Now().UnixNano()			//时间戳（纳秒）
time.Now().UnixNano() / 1e6 	//时间戳（毫秒）
time.Now().UnixNano() / 1e9 	//时间戳（纳秒转换为秒）
```

> 时间格式化

```go
nowtime:=time.Now().Format("2006-01-02 15:04:05")  //要显示的时间格式，2006-01-02 15:04:05，这几个值是固定写法,golang的诞生时间.
 
fmt.Println(nowtime)    //打印结果：2021-02-02 13:22:04
```

> 时间戳与字符串相互转换

```go
// 时间戳转字符串
timeUnix:=time.Now().Unix() //时间戳
 
formatTimeStr:=time.Unix(timeUnix,0).Format("2006-01-02 15:04:05")
 
fmt.Println(formatTimeStr) //打印结果：2021-02-02 13:22:04

// 字符串转时间
formatTimeStr := "2021-02-02 13:22:04" 
 
formatTime, err := time.Parse("2006-01-02 15:04:05", formatTimeStr)
 
fmt.Println(formatTime) //打印结果：2021-02-02 13:22:04 +0000 UTC
```



## math

> "math/rand"获取随机数，随机种子

```go
// 获取随机种子
rand.New(rand.NewSource(time.Now().UnixNano()))

// 获取随机数
rand.Int()
```



## io/ioutil

```go
// Discard 是一个 io.Writer 接口，调用它的 Write 方法将不做任何事情
// 并且始终成功返回。
var Discard io.Writer = devNull(0)
​
// ReadAll 读取 r 中的所有数据，返回读取的数据和遇到的错误。
// 如果读取成功，则 err 返回 nil，而不是 EOF，因为 ReadAll 定义为读取
// 所有数据，所以不会把 EOF 当做错误处理。
func ReadAll(r io.Reader) ([]byte, error)
​
// ReadFile 读取文件中的所有数据，返回读取的数据和遇到的错误。
// 如果读取成功，则 err 返回 nil，而不是 EOF
func ReadFile(filename string) ([]byte, error)
​
// WriteFile 向文件中写入数据，写入前会清空文件。
// 如果文件不存在，则会以指定的权限创建该文件。
// 返回遇到的错误。
func WriteFile(filename string, data []byte, perm os.FileMode) error
​
// ReadDir 读取指定目录中的所有目录和文件（不包括子目录）。
// 返回读取到的文件信息列表和遇到的错误，列表是经过排序的。
func ReadDir(dirname string) ([]os.FileInfo, error)
​
// NopCloser 将 r 包装为一个 ReadCloser 类型，但 Close 方法不做任何事情。
func NopCloser(r io.Reader) io.ReadCloser
​
// TempFile 在 dir 目录中创建一个以 prefix 为前缀的临时文件，并将其以读
// 写模式打开。返回创建的文件对象和遇到的错误。
// 如果 dir 为空，则在默认的临时目录中创建文件（参见 os.TempDir），多次
// 调用会创建不同的临时文件，调用者可以通过 f.Name() 获取文件的完整路径。
// 调用本函数所创建的临时文件，应该由调用者自己删除。
func TempFile(dir, prefix string) (f *os.File, err error)
​
// TempDir 功能同 TempFile，只不过创建的是目录，返回目录的完整路径。
func TempDir(dir, prefix string) (name string, err error)
```

## regexp

> 正则表达式相关包
>
> 提供很多方法：
>
> ```go
> Find(All)?(String)?(Submatch)?(Index)?
> ```

```go
// 获取正则表达式实例
re := regexp.MustCompile(正则表达式) //`(\d+)@qq.com`
// -1代表取全部
results := re.FindAllStringSubmatch(pageStr, -1)
```

## strconv

> 在Go语言中，我们往往需要对一些常用的数据类型进行转换，如string，int，int64 ,float等数据类型之间的转换，基于此Go语言为我们提供了一个名为[strconv](https://github.com/golang/go/tree/master/src/strconv)的包来为我们提供这些类型的转换方法。

```go
// int64转string
s := strconv.Itoa(i)  
//等价于s := strconv.FormatInt(int64(i), 10)

// string转int64
i, err := strconv.ParseInt(s, 10, 64)

// string转int
i, err := strconv.Atoi(s)
```

[sqlx](https://zhuanlan.zhihu.com/github.com/jmoiron/sqlx) 是 Go 语言中一个流行的第三方包，它提供了对 Go 标准库 `database/sql` 的扩展，旨在简化和改进 Go 语言中使用 SQL 的体验，并提供了更加强大的数据库交互功能。`sqlx` 保留了 `database/sql` 接口不变，是 `database/sql` 的超集，这使得将现有项目中使用的 `database/sql` 替换为 `sqlx` 变得相当轻松。

本文重点讲解 `sqlx` 在 `database/sql` 基础上扩展的功能，对于 `database/sql` 已经支持的功能则不会详细讲解。如果你对 `database/sql` 不熟悉，可以查看我的另一篇文章[《在 Go 中如何使用 database/sql 来操作数据库》](https://link.zhihu.com/?target=https%3A//jianghushinian.cn/2023/06/05/how-to-use-database-sql-to-operate-database-in-go/)。

## sqlx

‘**安装**

`sqlx` 安装方式同 Go 语言中其他第三方包一样：

```bash
$ go get github.com/jmoiron/sqlx
```

**sqlx** 类型设计

`sqlx` 的设计与 `database/sql` 差别不大，编码风格较为统一，参考 `database/sql` 标准库，`sqlx` 提供了如下几种与之对应的数据类型：

- `sqlx.DB`：类似于 `sql.DB`，表示数据库对象，可以用来操作数据库。
- `sqlx.Tx`：类似于 `sql.Tx`，事务对象。
- `sqlx.Stmt`：类似于 `sql.Stmt`，预处理 SQL 语句。
- `sqlx.NamedStmt`：对 `sqlx.Stmt` 的封装，支持具名参数。
- `sqlx.Rows`：类似于 `sql.Rows`，`sqlx.Queryx` 的返回结果。
- `sqlx.Row`：类似于 `sql.Row`，`sqlx.QueryRowx` 的返回结果。

以上类型与 `database/sql` 提供的对应类型在功能上区别不大，但 `sqlx` 为这些类型提供了更友好的方法。



## context

> 在 Go http包的Server中，每一个请求在都有一个对应的 goroutine 去处理。请求处理函数通常会启动额外的 goroutine 用来访问后端服务，比如数据库和RPC服务。用来处理一个请求的 goroutine 通常需要访问一些与请求特定的数据，比如终端用户的身份认证信息、验证相关的token、请求的截止时间。 当一个请求被取消或超时时，所有用来处理该请求的 goroutine 都应该迅速退出，然后系统才能释放这些 goroutine 占用的资源。
>
> `Go` 在 **1.7** 引入了 `context` 包，目的是为了在不同的 `goroutine` 之间或跨 `API` 边界传递超时、取消信号和其他请求范围内的值（与该请求相关的值。这些值可能包括用户身份信息、请求处理日志、跟踪信息等等）。
>
> 在 `Go` 的日常开发中，`Context` 上下文对象无处不在，无论是处理网络请求、数据库操作还是调用 `RPC` 等场景下，都会使用到 `Context`。那么，你真的了解它吗？熟悉它的正确用法吗？了解它的使用注意事项吗？喝一杯你最喜欢的饮料，随着本文一探究竟吧。

`Context` 接口的定义：即四个核心方法

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)

    Done() <-chan struct{}

    Err() error

    Value(key any) any
}
```

> ### Deadline()

`Deadline() (deadline time.Time, ok bool)` 方法返回 `Context` 的截止时间，表示在这个时间点之后，`Context` 会被自动取消。如果 `Context` 没有设置截止时间，该方法返回一个零值 `time.Time` 和一个布尔值 `false`。

```go
deadline, ok := ctx.Deadline()
if ok {
    // Context 有截止时间
} else {
    // Context 没有截止时间
}
```

> ### Done()

`Done()`方法返回一个只读通道，当 `Context` 被取消时，这个通道就会关闭。可以通过监听这个通道。

```go
select {
case <-ctx.Done():
    // Context 已取消
default:
    // Context 尚未取消
}
```

> ### Err()

返回一个 error 值，表示 Context 被取消时产生的错误。如果尚未取消，这个方法返回nil

```go
if err := ctx.Err(); err != nil {
    // Context 已取消，处理错误
}
```

> ### Value()

`Value(key any) any` 方法返回与 `Context` 关联的键值对，一般用于在 `Goroutine` 之间传递请求范围内的信息。如果没有关联的值，则返回 `nil`。

```go
value := ctx.Value(key)
if value != nil {
    // 存在关联的值
}
```

> ## 创建context

> ### context.Backgrund()

`context.Background()` 函数返回一个非 `nil` 的空 `Context`，它没有携带任何的值，也没有取消和超时信号。通常作为根 `Context` 使用。

```go
ctx := context.Background()
```

> ### context.TODO()

`context.TODO()` 函数返回一个非 `nil` 的空 `Context`，它没有携带任何的值，也没有取消和超时信号。虽然它的返回结果和 `context.Background()` 函数一样，但是它们的使用场景是不一样的，如果不确定使用哪个上下文时，可以使用 `context.TODO()`。

```go
ctx := context.TODO()
```

> ### context.WithValue()

`context.WithValue(parent Context, key, val any)` 函数接收一个父 `Context` 和一个键值对 `key`、`val`，返回一个新的子 `Context`，并在其中添加一个 `key-value` 数据对。

```go
ctx := context.WithValue(parentCtx, "username", "陈明勇")
```

> ### context.WithCancel()

`context.WithCancel(parent Context) (ctx Context, cancel CancelFunc)` 函数接收一个父 `Context`，返回一个新的子 `Context` 和一个取消函数，当取消函数被调用时，子 `Context` 会被取消，同时会向子 `Context` 关联的 `Done()` 通道发送取消信号，届时其衍生的子孙 `Context` 都会被取消。这个函数适用于手动取消操作的场景。

```go
ctx, cancelFunc := context.WithCancel(parentCtx)  
defer cancelFunc()
```

> ### context.WithCancelCause() 与 context.Cause()

`context.WithCancelCause(parent Context) (ctx Context, cancel CancelCauseFunc)` 函数是 `Go 1.20` 版本才新增的，其功能类似于 `context.WithCancel()`，但是它可以设置额外的取消原因，也就是 `error` 信息，返回的 `cancel` 函数被调用时，需传入一个 `error` 参数。

```go
ctx, cancelFunc := context.WithCancelCause(parentCtx)
defer cancelFunc(errors.New("原因"))
```

`context.Cause(c Context) error` 函数用于返回取消 `Context` 的原因，即错误值 `error`。如果是通过 `context.WithCancelCause()` 函数返回的取消函数 `cancelFunc(myErr)` 进行的取消操作，我们可以获取到 `myErr` 的值。否则，我们将得到与 `c.Err()` 相同的返回值。如果 `Context` 尚未被取消，将返回 `nil`。

```go
err := context.Cause(ctx)
```

> ### context.WithDeadline()

`context.WithDeadline(parent Context, d time.Time) (Context, CancelFunc)` 函数接收一个父 `Context` 和一个截止时间作为参数，返回一个新的子 `Context`。当截止时间到达时，子 `Context` 其衍生的子孙 `Context` 会被自动取消。这个函数适用于需要在特定时间点取消操作的场景。

```go
deadline := time.Now().Add(time.Second * 2)
ctx, cancelFunc := context.WithTimeout(parentCtx, deadline)
defer cancelFunc()
```

> ### context.WithTimeout()

`context.WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)` 函数和 `context.WithDeadline()` 函数的功能是一样的，其底层会调用 `WithDeadline()` 函数，只不过其第二个参数接收的是一个超时时间，而不是截止时间。这个函数适用于需要在一段时间后取消操作的场景。

```go
ctx, cancelFunc := context.WithTimeout(parentCtx, time.Second * 2)
defer cancelFunc()
```



**案例：传递共享数据**

编写中间件函数，用于向 `HTTP` 处理链中添加处理请求 `ID` 功能

```go
type key int

const (
   requestIDKey key = iota
)

func WithRequestId(next http.Handler) http.Handler {
   return http.HandlerFunc(func(rw http.ResponseWriter, req *http.Request) {
      // 从请求中提取请求ID和用户信息
      requestID := req.Header.Get("X-Request-ID")

      // 创建子 context，并添加一个请求 Id 的信息
      ctx := context.WithValue(req.Context(), requestIDKey, requestID)

      // 创建一个新的请求，设置新 ctx
      req = req.WithContext(ctx)

      // 将带有请求 ID 的上下文传递给下一个处理器
      next.ServeHTTP(rw, req)
   })
}
```

首先，我们从请求的头部中提取请求 `ID`。然后使用 `context.WithValue` 创建一个子上下文，并将请求 `ID` 作为键值对存储在子上下文中。接着，我们创建一个新的请求对象，并将子上下文设置为新请求的上下文。最后，我们将带有请求 `ID` 的上下文传递给下一个处理器。 这样，通过使用 `WithRequestId` 中间件函数，我们可以在处理请求的过程中方便地获取和使用请求 `ID`，例如在 **日志记录、跟踪和调试等方面**。



## encoding

### /gob

> gob和json的pack之类的方法一样，由发送端使用Encoder对数据结构进行编码。在接收端收到消息之后，接收端使用Decoder将序列化的数据变化成本地变量。与json编码格式相比，**gob编码可以实现json所不支持的struct的方法序列化，利用gob包序列化struct保存到本地也十分方便。**
>

```go
1.记录数据的类型和名称
Register
// Register记录value下层具体值的类型和其名称。
// 该名称将用来识别发送或接受接口类型值时下层的具体类型。
// 本函数只应在初始化时调用，如果类型和名字的映射不是一一对应的，会panic。
func Register(value interface{})
RegisterName

// RegisterName类似Register，区别是这里使用提供的name代替类型的默认名称。
func RegisterName(name string, value interface{})


2. 编码
   数据在传输时会先经过编码后再进行传输，与编码相关的有三种方法：

NewDecoder
// NewEncoder返回一个将编码后数据写入w的*Encoder。
func NewEncoder(w io.Writer) *Encoder


Encode
// Encode方法将e编码后发送，并且会保证所有的类型信息都先发送。
func (enc *Encoder) Encode(e interface{}) error

EncodeValue
// EncodeValue方法将value代表的数据编码后发送，并且会保证所有的类型信息都先发送。
func (enc *Encoder) EncodeValue(value reflect.Value) error


3. 解码
   接收到数据后需要对数据进行解码，与解码相关的有三种方法：

NewDecoder
// 函数返回一个从r读取数据的*Decoder，如果r不满足io.ByteReader接口，则会包装r为bufio.Reader。
func NewDecoder(r io.Reader) *Decoder

Decode
// Decode从输入流读取下一个之并将该值存入e。
// 如果e是nil，将丢弃该值,否则e必须是可接收该值的类型的指针。
// 如果输入结束，方法会返回io.EOF并且不修改e（指向的值）。
func (dec *Decoder) Decode(e interface{}) error


DecodeValue
// DecodeValue从输入流读取下一个值，如果v是reflect.Value类型的零值（v.Kind() == Invalid），方法丢弃该值；否则它会把该值存入v。
// 此时，v必须代表一个非nil的指向实际存在值的指针或者可写入的reflect.Value（v.CanSet()为真）。
// 如果输入结束，方法会返回io.EOF并且不修改e（指向的值）。
func (dec *Decoder) DecodeValue(v reflect.Value) error
```



### /binary

### Go 处理固定长度字节序

Go中处理大小端序的代码位于 encoding/binary ,包中的全局变量BigEndian用于操作大端序数据，LittleEndian用于操作小端序数据，这两个变量所对应的数据类型都实行了ByteOrder接口：

```go
type ByteOrder interface {  
    Uint16([]byte) uint16  
    Uint32([]byte) uint32  
    Uint64([]byte) uint64  
    PutUint16([]byte, uint16)
    PutUint32([]byte, uint32)  
    PutUint64([]byte, uint64)  
    String() string
}
```

其中，前三个方法用于读取数据，后三个方法用于写入数据。

## log

log包定义了Logger类型，该类型提供了一些格式化输出的方法。

```go
type Logger struct {
    mu     sync.Mutex // ensures atomic writes; protects the following fields
    prefix string     // prefix on each line to identify the logger (but see                            Lmsgprefix)
    flag   int        // properties
    out    io.Writer  // destination for output
    buf    []byte     // for accumulating text to write
}
```

`mu`属性主要是为了确保原子操作，`prefix`设置每一行的前缀，`flag`设置输出的各种属性，比如时间、行号、文件路径等。`out`输出的方向，用于把日志存储文件。

本包也提供了一个预定义的“标准”logger，可以通过调用函数Print系列`(Print|Printf|Println）`、Fatal系列`（Fatal|Fatalf|Fatalln）`、和Panic系列`（Panic|Panicf|Panicln）`来使用，比自行创建一个logger对象更容易使用。

例如，我们可以像下面的代码一样直接通过log包来调用上面提到的方法，默认它们会将日志信息打印到终端界面：

```go
package main

import (
    "log"
)

func main() {
    log.Println("这是一条优雅的日志。")
    v := "优雅的"
    log.Printf("这是一个%s日志\n", v)
    //fatal系列函数会在写入日志信息后调用os.Exit(1)。Panic系列函数会在写入日志信息后panic。
    log.Fatalln("这是一天会触发fatal的日志") 
    log.Panicln("这是一个会触发panic的日志。") //执行后会自动触发一个异常
}
```

