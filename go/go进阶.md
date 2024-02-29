# Go rune 类型

刚接触 Go 语言时，就听说有一个叫 `rune` 的数据类型，即使查阅过一些资料，对它的理解依旧比较模糊，加之对陌生事物的天然排斥，在之后很长一段时间的编程工作中，我都没有让它出现在我的代码里。

逃避虽然有用，但是似乎有些可耻，想要成为一名成熟、优秀的 Go 语言开发工程师，必须要有直面陌生事物并且成功运用的勇气和能力，带着这样的觉悟，让我们一起走近 `rune`，直视它！

* ## 了解一下，`rune`类型究竟是什么？

`rune` 类型是 Go 语言的一种特殊数字类型。在 `builtin/builtin.go` 文件中，它的定义：`type rune = int32`；官方对它的解释是：`rune` 是类型 `int32` 的别名，在所有方面都等价于它，用来区分字符值跟整数值。使用单引号定义 ，返回采用 UTF-8 编码的 Unicode 码点。Go 语言通过 `rune` 处理中文，支持国际化多语言。

众所周知，Go 语言有两种类型声明方式：一种叫类型定义声明，另一种叫类型别名声明。其中，别名的使用在大型项目重构中作用最为明显，它能解决代码升级或迁移过程中可能存在的类型兼容性问题。而`rune` 跟 `byte` 是 Go 语言中仅有的两个类型别名，专门用来处理字符。当然，我们也可以通过 `type` 关键字加等号的方式声明更多的类型别名。

* ## 学习一下，`rune`类型怎么用？

我们知道，字符串由字符组成，字符的底层由字节组成，而一个字符串在底层的表示是一个字节序列。在 Go 语言中，字符可以被分成两种类型处理：对占 1 个字节的英文类字符，可以使用 `byte`（或者 `unit8` ）；对占 1 ~ 4 个字节的其他字符，可以使用 `rune`（或者 `int32` ），如中文、特殊符号等。

下面，我们通过示例应用来具体感受一下。

- 统计带中文字符串长度

```go
// 使用内置函数 len() 统计字符串长度
fmt.Println(len("Go语言编程"))  // 输出：14  
```

前面说到，字符串在底层的表示是一个字节序列。其中，英文字符占用 1 字节，中文字符占用 3 字节，所以得到的长度 14 显然是底层占用字节长度，而不是字符串长度，这时，便需要用到 `rune` 类型。

```go
// 转换成 rune 数组后统计字符串长度
fmt.Println(len([]rune("Go语言编程")))  // 输出：6
```

这回对了。很容易，我们解锁了 `rune` 类型的第一个功能，即统计字符串长度。

- 截取带中文字符串

如果想要截取字符串中 ”Go语言“ 这一段，考虑到底层是一个字节序列，或者说是一个数组，通常情况下，我们会这样：

```go
s := "Go语言编程"
// 8=2*1+2*3
fmt.Println(s[0:8])  // 输出：Go语言
```

结果符合预期。但是，按照字节的方式进行截取，必须预先计算出需要截取字符串的字节数，如果字节数计算错误，就会显示乱码，比如这样：

```go
s := "Go语言编程"
fmt.Println(s[0:7]) // 输出：Go语�
```

此外，如果截取的字符串较长，那通过字节的方式进行截取显然不是一个高效准确的办法。那有没有不用计算字节数，简单又不会出现乱码的方法呢？不妨试试这样：

```go
s := "Go语言编程"
// 转成 rune 数组，需要几个字符，取几个字符
fmt.Println(string([]rune(s)[:4])) // 输出：Go语言    
```

到这里，我们解锁了 `rune` 类型的第二个功能，即截取字符串。

* ## 思考一下，为什么 `rune` 类型可以做到？

通过上面的示例，我们发现似乎在处理带中文的字符串时，都需要用到 `rune` 类型，这究竟是为什么呢？除了使用 `rune` 类型，还有其他方法吗？

在深入思考之前，我们需要首先弄清楚 `string` 、`byte`、`rune` 三者间的关系。

字符串在底层的表示是由单个字节组成的一个不可修改的字节序列，字节使用 UTF-8[1] 编码标识 Unicode[2] 文本。Unicode 文本意味着 `.go` 文件内可以包含世界上的任意语言或字符，该文件在任意系统上打开都不会乱码。UTF-8 是 Unicode 的一种实现方式，是一种针对 Unicode 可变长度的字符编码，它定义了字符串具体以何种方式存储在内存中。UFT-8 使用 1 ~ 4 为每个字符编码。

Go 语言把字符分 `byte` 和 `rune` 两种类型处理。`byte` 是类型 `unit8` 的别名，用于存放占 1 字节的 ASCII 字符，如英文字符，返回的是字符原始字节。`rune` 是类型 `int32` 的别名，用于存放多字节字符，如占 3 字节的中文字符，返回的是字符 Unicode 码点值。如下图所示：

```go
s := "Go语言编程"
// byte
fmt.Println([]byte(s)) // 输出：[71 111 232 175 173 232 168 128 231 188 150 231 168 139]
// rune
fmt.Println([]rune(s)) // 输出：[71 111 35821 35328 32534 31243]
```

![](image\640.png)

# Go Tag标签

> 在 Go 语言中，标签（Tag）是附加到结构体字段的元信息，它是以字符串的形式存储的。这些标签可以通过反射（reflection）机制来获取，并可以被用于各种目的。例如，一些库会使用标签来控制序列化和反序列化，如 JSON 或 XML 库；还有一些库可能会使用标签来进行数据验证或进行数据库到结构体的映射。
>

多个标签示例：

```go
type Person struct {
 Name    string `json:"name" xml:"name"`
 Email   string `json:"email" xml:"email"`
 Age     int    `json:"age,omitempty" xml:"age,omitempty"`
}
```

通过反射获取标签内容：

注意，如果你想在反射时获取标签，你需要导入` reflect `包，并使用` Type.Field(i).Tag.Get(key) `方法，其中，`Type `是一个结构体的类型，`i `是字段的索引，`key` 是你想获取的标签的键。

# Go 泛型

> 争议非常大但同时也备受期待的泛型终于伴随着`Go1.18`发布了。

Go1.18也是通过这种方式实现的泛型，但是单纯的形参实参是远远不能实现泛型编程的，所以Go还引入了非常多全新的概念：

- 类型形参 (Type parameter)
- 类型实参(Type argument)
- 类型形参列表( Type parameter list)
- 类型约束(Type constraint)
- 实例化(Instantiations)
- 泛型类型(Generic type)
- 泛型接收器(Generic receiver)
- 泛型函数(Generic function)
  ........

## 泛型类型

```go
type Slice[T int | float32 | float64] []T
```

`T` 就是上面介绍过的**类型形参(Type parameter)**，在定义Slice类型的时候 T 代表的具体类型并不确定，类似一个占位符

`int|float32|float64` 这部分被称为**类型约束(Type constraint)**，中间的 `|` 的意思是告诉编译器，类型形参 T 只可以接收 `int `或 `float32 `或` float64` 这三种类型的实参

中括号里的 `T int|float32|float64` 这一整串因为定义了所有的类型形参(在这个例子里只有一个类型形参T），所以我们称其为 **类型形参列表(type parameter list)**

这里新定义的类型名称叫 `Slice[T]`

想要使用泛型类型的话，就要先进行声明类型，即**实例化**

**类型定义中带 类型形参的类型，称之为 泛型类型(Generic type)**

使用：

```go
type MyMap[KEY int | string, VALUE float32 | float64] map[KEY]VALUE  
type Slice[T int | float32 | float64] []T

func main() {
	var a Slice[int] = []int{1, 2, 3}
	var b Slice[float64] = []float64{1.0, 2.0, 3.0}

	fmt.Println(a)
	fmt.Println(b)
    // 用类型实参 string 和 flaot64 替换了类型形参 KEY 、 VALUE，泛型类型被实例化为具体的类型：MyMap[string, float64]
	var a MyMap[string, float64] = map[string]float64{
    	"jack_score": 9.6,
    	"bob_score":  8.4,
	}
}

```

## 其它类型泛型

```go
// 一个泛型类型的结构体。
type MyStruct[T int | string] struct {
    Name string
    Data T
}
// 一个泛型类型接口
type IprintlnData[T int | float32 | string] interface {
    Print(data T)
}

// 一个泛型通道
type MyChan[T int | string] chan T
```

## 类型套用

```go
type WowStruct[T int | float32, S []T] struct {
    Data     S
    MaxValue T
    MinValue T
}

/*  
套娃
*/
// 先定义个泛型类型 Slice[T]
type Slice[T int|string|float32|float64] []T

// ✗ 错误。泛型类型Slice[T]的类型约束中不包含uint, uint8
type UintSlice[T uint|uint8] Slice[T]  

// ✓ 正确。基于泛型类型Slice[T]定义了新的泛型类型 FloatSlice[T] 。FloatSlice[T]只接受float32和float64两种类型
type FloatSlice[T float32|float64] Slice[T] 

// ✓ 正确。基于泛型类型Slice[T]定义的新泛型类型 IntAndStringSlice[T]
type IntAndStringSlice[T int|string] Slice[T]  
// ✓ 正确 基于IntAndStringSlice[T]套娃定义出的新泛型类型
type IntSlice[T int] IntAndStringSlice[T] 

// 在map中套一个泛型类型Slice[T]
type WowMap[T int|string] map[string]Slice[T]
// 在map中套Slice[T]的另一种写法
type WowMap2[T Slice[int] | Slice[string]] map[string]T
```

## 语法错误

```go
//✗ 错误。T *int会被编译器误认为是表达式 T乘以int，而不是int指针
type NewType[T *int] []T
// 上面代码再编译器眼中：它认为你要定义一个存放切片的数组，数组长度由 T 乘以 int 计算得到
type NewType [T * int][]T 

//✗ 错误。和上面一样，这里不光*被会认为是乘号，| 还会被认为是按位或操作
type NewType2[T *int|*float64] []T 

//✗ 错误
type NewType2 [T (int)] []T 
```

解决办法是，给类型约束加上`interface{}`，或加上逗号（只适用一个参数的情况）

```go
type NewType[T interface{*int}] []T
type NewType2[T interface{*int|*float64}] []T 

// 如果类型约束中只有一个类型，可以添加个逗号消除歧义
type NewType3[T *int,] []T
```



## 匿名结构体

> 匿名结构体是不支持泛型的

```go
testCase := struct[T int|string] {
        caseName string
        got      T
        want     T
    }[int]{
        caseName: "test OK",
        got:      100,
        want:     100,
    }
```

## 泛型receiver

给泛型结构添加方法

```go
type MySlice[T int | float32] []T

func (s MySlice[T]) Sum() T {
    var sum T
    for _, value := range s {
        sum += value
    }
    return
}
func main() {
    var s MySlice[int] = []int{1, 2, 3, 4}
fmt.Println(s.Sum()) // 输出：10

var s2 MySlice[float32] = []float32{1.0, 2.0, 3.0, 4.0}
fmt.Println(s2.Sum()) // 输出：10.0
}
```

## 泛型判断变量

使用接口的时候经常会用到类型断言或 `type swith` 来确定接口具体类型。

```go
var i interface{} = 123
i.(int) // 类型断言

// type switch
switch i.(type) {
    case int:
        // do something
    case string:
        // do something
    default:
        // do something
    }
}
```

但是对于泛型，不能够使用，需要使用反射机制来完成。但仔细想想，我们使用泛型的初衷就是避免使用接口加反射，如果又加入了反射，这有点“绕远路了”

## 泛型函数

```go
func Add[T int | float32 | float64](a T, b T) T {
    return a + b
}
// 使用
func main() {
    Add[int](1, 2)
    // 自动判断类型
    Add(1.0, 2.0)
}
```

* #### 匿名函数不支持泛型

* #### 非泛型类型不支持泛型方法

## 泛型简化

```go

// 一个可以容纳所有int,uint以及浮点类型的泛型切片，但不易维护
type Slice[T int | int8 | int16 | int32 | int64 | uint | uint8 | uint16 | uint32 | uint64 | float32 | float64] []T

// 简化后
type IntUintFloat interface {
    int | int8 | int16 | int32 | int64 | uint | uint8 | uint16 | uint32 | uint64 | float32 | float64
}

type Slice[T IntUintFloat] []T

// 而接口和接口、接口和普通类型之间也是可以通过 `|` 进行组合：
type Int interface {
    int | int8 | int16 | int32 | int64
}
type Uint interface {
    uint | uint8 | uint16 | uint32
}
type Float interface {
    float32 | float64
}
type Slice[T Int | Uint | Float] []T  // 使用 '|' 将多个接口类型组合

// ~ 指定底层类型
/*
1. 后面类型不能为接口
2. 后面类型必须为基本类型
*/
```

## 泛型与接口

> 在泛型出现之前，官方对于接口的定义是`An interface type specifies a **method set** called its interface`, 是一个方法集。但是随着泛型的出现，接口就发生了非常大的变化。`An interface type defines a type set (一个接口类型定义了一个类型集)`

### 接口实现

既然接口定义发生了变化，那么从Go1.18开始 `接口实现(implement)` 的定义自然也发生了变化：

当满足以下条件时，我们可以说 **类型 T 实现了接口 I ( type T implements interface I)**：

- T 不是接口时：类型 T 是接口 I 代表的类型集中的一个成员 (T is an element of the type set of I)
- T 是接口时： T 接口代表的类型集是 I 代表的类型集的子集(Type set of T is a subset of the type set of I)

**接口类型集合：**

```go
// 类型的并集
type Uint interface {  // 类型集 Uint 是 ~uint 和 ~uint8 等类型的并集
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64
}

// 类型的交集
type AllInt interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 | ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint32
}
type Uint interface {
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64
}
type A interface { // 接口A代表的类型集是 AllInt 和 Uint 的交集
    AllInt
    Uint
}
type B interface { // 接口B代表的类型集是 AllInt 和 ~int 的交集
    AllInt
    ~int
}

// 空集
type Bad interface {
    int
    float32 
} // 类型 int 和 float32 没有相交的类型，所以接口 Bad 代表的类型集为空

```

### 空接口和 any

> 空接口代表了所有类型的集合

1. 虽然空接口内没有写入任何的类型，但它代表的是所有类型的集合，而非一个 **空集**
2. 类型约束中指定 **空接口** 的意思是指定了一个包含所有类型的类型集，并不是类型约束限定了只能使用 **空接口** 来做类型形参

```go
// 空接口代表所有类型的集合。写入类型约束意味着所有类型都可拿来做类型实参
type Slice[T interface{}] []T

var s1 Slice[int]    // 正确
var s2 Slice[map[string]string]  // 正确
var s3 Slice[chan int]  // 正确
var s4 Slice[interface{}]  // 正确
```

因为空接口是一个包含了所有类型的类型集，所以我们经常会用到它。于是，`Go1.18`开始提供了一个和空接口 `interface{}` 等价的新关键词 `any` ，用来使代码更简单

### comparable

对于一些数据类型，我们需要在类型约束中限制只接收能`!=` 和 `==` 对比的类型，如map：

```go
// 错误，因为 map 中的类型必须是可以进行 != 和 == 比较的类型
type MyMap[KEY any, VALUE any] map[KEY]VALUE
```

Go直接内置了一个叫 `comparable` 的接口，它代表了所有可用 `!=` 以及 `==` 对比的类型

```go
type MyMap[KEY comparable, VALUE any] map[KEY]VALUE // 正确
```

### ordered

而可进行大小比较的类型被称为 `Orderd` 。目前Go语言并没有像 `comparable` 这样直接内置对应的关键词，所以想要的话需要自己来定义相关接口，比如我们可以参考Go官方包`golang.org/x/exp/constraints` 如何定义：

```go
// Ordered 代表所有可比大小排序的类型
type Ordered interface {
    Integer | Float | ~string
}
type Integer interface {
    Signed | Unsigned
}
type Signed interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64
}
type Unsigned interface {
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 | ~uintptr
}
type Float interface {
    ~float32 | ~float64
}
```

## 接口类型

### 基本接口

接口定义中如果只有方法的话，那么这种接口被称为**基本接口(Basic interface)**。

```go
type MyError interface { // 接口中只有方法，所以是基本接口
    Error() string
}

// 用法和 Go1.18之前保持一致
var err MyError = fmt.Errorf("hello world")
```

### 一般接口

如果接口内不光只有方法，还有类型的话，这种接口被称为 **一般接口(General interface)** ，如下例子都是一般接口：

```go
type Uint interface { // 接口 Uint 中有类型，所以是一般接口
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64
}

type ReadWriter interface {  // ReadWriter 接口既有方法也有类型，所以是一般接口
    ~string | ~[]rune

    Read(p []byte) (n int, err error)
    Write(p []byte) (n int, err error)
}
```

## 泛型接口

```go
type DataProcessor[T any] interface {
    Process(oriData T) (newData T)
    Save(data T) error
}

type DataProcessor2[T any] interface {
    int | ~struct{ Data interface{} }

    Process(data T) (newData T)
    Save(data T) error
}
```



## 接口定义的限制

* 用 `|` 连接多个类型的时候，类型之间不能有相交的部分(即必须是不交集):但是相交的类型中是接口的话，则不受这一限制

* 类型的并集中不能有类型形参

* 接口不能直接或间接地并入自己

* 接口的并集成员个数大于一的时候不能直接或间接并入 `comparable` 接口

  ```go
  type OK interface {
      comparable // 正确。只有一个类型的时候可以使用 comparable
  }
  
  type Bad1 interface {
      []int | comparable // 错误，类型并集不能直接并入 comparable 接口
  }
  
  type CmpInf interface {
      comparable
  }
  type Bad2 interface {
      chan int | CmpInf  // 错误，类型并集通过 CmpInf 间接并入了comparable
  }
  type Bad3 interface {
      chan int | interface{comparable}  // 理所当然，这样也是不行的
  }
  ```

* 带方法的接口，都不能写入接口的并集中

  ```go
  type _ interface {
      ~int | ~string | error // 错误，error是带方法的接口(一般接口) 不能写入并集中
  }
  
  type DataProcessor[T any] interface {
      ~string | ~[]byte
  
      Process(data T) (newData T)
      Save(data T) error
  }
  
  // 错误，实例化之后的 DataProcessor[string] 是带方法的一般接口，不能写入类型并集
  type _ interface {
      ~int | ~string | DataProcessor[string] 
  }
  
  type Bad[T any] interface {
      ~int | ~string | DataProcessor[T]  // 也不行
  }
  ```

  











CA0FGNME2N
