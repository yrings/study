# Gin

> * Gin 是一个golang的微框架，封装比较优雅，API友好。
> * 对于golang而言，web框架的依赖要远比Python，Java之类的要小。自身的 `net/http`足够简单，性能也非常不错
> * 借助框架开发，不仅可以省去很多常用的封装带来的时间，也有助于团队的编码风格和形成规范



## 安装配置

要安装Gin软件包，您需要安装Go并首先设置Go工作区。

1.首先需要安装Go（需要1.10+版本），然后可以使用下面的Go命令安装Gin。

> go get -u github.com/gin-gonic/gin

2.将其导入您的代码中：

> import "github.com/gin-gonic/gin"

3.（可选）导入net/http。例如，如果使用常量，则需要这样做http.StatusOK。

> import "net/http"



**案例：**

```go
func main() {
	// 1. 创建路由
	r := gin.Default()
	// 2. 绑定路由规则，执行的函数
	// gin.Context, 封装了 request 和 response
	r.GET("/", func(c *gin.Context) {
		c.String(http.StatusOK, "hello world!")
		fmt.Println("request:", c.Request)
	})
	// 3. 监听端口，默认在8080
	// Run()
	r.Run(":8000")
}
```



## 一、router路由

### 1. 基本路由

 ```go
 func main() {
 	// 1. 创建路由
 	r := gin.Default()
 	// 2. 绑定路由规则，执行的函数
 	// gin.Context, 封装了 request 和 response
 	r.GET("/", func(c *gin.Context) {
 		c.String(http.StatusOK, "hello world!")
 		fmt.Println("request:", c.Request)
 	})
 	r.POST("/testPost", func(c *gin.Context) {
 		c.String(http.StatusOK, "receive success!")
 		fmt.Println("receive post request:", c.Request)
 	})
 
 	// 3. 监听端口，默认在8080
 	// Run()
 	r.Run(":8000")
 }
 
 ```

### 2. 获取参数

> #### API 参数

访问`localhost:8080/user/jerry/look/fight`

```go
func main() {
	r := gin.Default()
	r.GET("/user/:name/*action", func(c *gin.Context) {
		name := c.Param("name")
		action := c.Param("action")
		fmt.Println("action:", action)
        // action: /look/fight
		action = strings.Trim(action, "/")

		c.String(http.StatusOK, "name is "+name+"action is "+action)
		// name is jerryaction is look/fight
	})
	r.Run()
}
```



> #### URL 参数

- URL参数可以通过DefaultQuery()或Query()方法获取
- DefaultQuery()若参数不村则，返回默认值，Query()若不存在，返回空串

```go
func main() {
    r := gin.Default()
    r.GET("/user", func(c *gin.Context) {
        //指定默认值
        //http://localhost:8080/user 才会打印出来默认的值
        name := c.DefaultQuery("name", "枯藤")
        c.String(http.StatusOK, fmt.Sprintf("hello %s", name))
    })
    r.Run()
}
```



> 表单参数

- 表单传输为post请求，http常见的传输格式为四种：
  - application/json
  - application/x-www-form-urlencoded
  - application/xml
  - multipart/form-data
- 表单参数可以通过PostForm()方法获取，该方法默认解析的是x-www-form-urlencoded或from-data格式的参数

```go
func main() {
    r := gin.Default()
    r.POST("/form", func(c *gin.Context) {
        types := c.DefaultPostForm("type", "post")
        username := c.PostForm("username")
        password := c.PostForm("userpassword")
        // c.String(http.StatusOK, fmt.Sprintf("username:%s,password:%s,type:%s", username, password, types))
        c.String(http.StatusOK, fmt.Sprintf("username:%s,password:%s,type:%s", username, password, types))
    })
    r.Run()
}
```

### 3. 文件上传

- multipart/form-data格式用于文件上传
- gin文件上传与原生的net/http方法类似，不同在于gin把原生的request封装到c.Request中

```go
func main() {
    r := gin.Default()
    r.POST("/upload", func(c *gin.Context) {
        _, headers, err := c.Request.FormFile("file")
        if err != nil {
            log.Printf("Error when try to get file: %v", err)
        }
        //headers.Size 获取文件大小
        if headers.Size > 1024*1024*2 {
            fmt.Println("文件太大了")
            return
        }
        //headers.Header.Get("Content-Type")获取上传文件的类型
        if headers.Header.Get("Content-Type") != "image/png" {
            fmt.Println("只允许上传png图片")
            return
        }
        c.SaveUploadedFile(headers, "./video/"+headers.Filename)
        c.String(http.StatusOK, headers.Filename)
    })
    r.Run()
}
```



### 4. 热加载调试 Hot Reload

Python 的 `Flask`框架，有 *debug* 模式，启动时传入 *debug=True* 就可以热加载(Hot Reload, Live Reload)了。即更改源码，保存后，自动触发更新，浏览器上刷新即可。免去了杀进程、重新启动之苦。

Gin 原生不支持，但有很多额外的库可以支持。例如

- github.com/codegangsta/gin
- github.com/pilu/fresh

这次，我们采用 *github.com/pilu/fresh* 。

```
go get -v -u github.com/pilu/fresh
```

安装好后，只需要将`go run main.go`命令换成`fresh`即可。每次更改源文件，代码将自动重新编译(Auto Compile)。



## 二、数据解析与绑定

### 1. Json 数据解析和绑定

 ```go
 package main
 
 import (
    "github.com/gin-gonic/gin"
    "net/http"
 )
 
 // 定义接收数据的结构体
 type Login struct {
    // binding:"required"修饰的字段，若接收为空值，则报错，是必须字段
    User    string `form:"username" json:"user" uri:"user" xml:"user" binding:"required"`
    Pssword string `form:"password" json:"password" uri:"password" xml:"password" binding:"required"`
 }
 
 func main() {
    // 1.创建路由
    // 默认使用了2个中间件Logger(), Recovery()
    r := gin.Default()
    // JSON绑定
    r.POST("loginJSON", func(c *gin.Context) {
       // 声明接收的变量
       var json Login
       // 将request的body中的数据，自动按照json格式解析到结构体
       if err := c.ShouldBindJSON(&json); err != nil {
          // 返回错误信息
          // gin.H封装了生成json数据的工具
          c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
          return
       }
       // 判断用户名密码是否正确
       if json.User != "root" || json.Pssword != "admin" {
          c.JSON(http.StatusBadRequest, gin.H{"status": "304"})
          return
       }
       c.JSON(http.StatusOK, gin.H{"status": "200"})
    })
    r.Run(":8000")
 }
 ```



### 2. 表单数据解析和绑定

```go
if err := c.Bind(&form); err != nil {
            c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
            return
        }
```



### 3. URL数据解析与绑定

```go 
if err := c.ShouldBindUri(&login); err != nil {
            c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
            return
        }
```



## 三、 渲染

### 1. 响应数据

```go
func main() {
	// 1. 创建路由
	// 默认使用了两个中间件Logger()、Recovery()
	r := gin.Default()
	// 1. json
	r.GET("/resJson", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"message": "resJson", "status": 200})
	})
	//{"message":"resJson","status":200}

	// 2. 结构体
	r.GET("/resStruct", func(c *gin.Context) {
		var msg struct {
			Name    string
			Message string
			Number  int
		}
		msg.Name = "root"
		msg.Message = "message"
		msg.Number = 123
		c.JSON(http.StatusOK, msg)
	})
	/*
		{
		    "Name": "root",
		    "Message": "message",
		    "Number": 123
		}
	*/

	// 3. XML
	r.GET("/resXML", func(c *gin.Context) {
		c.XML(200, gin.H{"message": "abc"})
	})
	/*
		<map>
		    <message>abc</message>
		</map>
	*/

	// 4. YAML响应
	r.GET("/resYAML", func(c *gin.Context) {
		c.YAML(200, gin.H{"name": "zhangsan"})
	})
	// name: zhangsan

	// 5.protobuf格式,谷歌开发的高效存储读取的工具
	// 数组？切片？如果自己构建一个传输格式，应该是什么格式？
	// 二进制文件
	r.GET("/resProtoBuf", func(c *gin.Context) {
		reps := []int64{int64(1), int64(2)}
		// 定义数据
		label := "label"
		// 传protobuf格式数据
		data := &protoexample.Test{
			Label: &label,
			Reps:  reps,
		}
		c.ProtoBuf(200, data)
	})
	r.Run()
}
```

### 2. HTML 模板渲染

```go
package main

import (
	"net/http"

	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()
    // 默认是从工作路径开始
	r.LoadHTMLGlob("gin-study/rendering/static/*")
	r.GET("/index.html", func(c *gin.Context) {
		c.HTML(http.StatusOK, "index.html", gin.H{"title": "我是测试", "ce": "123456"})
	})
	r.Run()
}

```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>{{.title}}</title>
</head>
    <body>
        fgkjdskjdsh{{.ce}}
    </body>
</html>
```

目录结构:

**注意：路径是从工作路径开始的**

```
static
 -index.html
main.go
```



### 3. 重定向

```go
package main

import (
    "net/http"

    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default()
    r.GET("/index", func(c *gin.Context) {
        c.Redirect(http.StatusMovedPermanently, "http://www.5lmh.com")
    })
    r.Run()
}
```



### 4. 同步异步

- goroutine机制可以方便地实现异步处理
- 另外，在启动新的goroutine时，不应该使用原始上下文，必须使用它的只读副本

```go
func main() {
    // 1.创建路由
    // 默认使用了2个中间件Logger(), Recovery()
    r := gin.Default()
    // 1.异步
    r.GET("/long_async", func(c *gin.Context) {
        // 需要搞一个副本
        copyContext := c.Copy()
        // 异步处理
        go func() {
            time.Sleep(3 * time.Second)
            log.Println("异步执行：" + copyContext.Request.URL.Path)
        }()
    })
    // 2.同步
    r.GET("/long_sync", func(c *gin.Context) {
        time.Sleep(3 * time.Second)
        log.Println("同步执行：" + c.Request.URL.Path)
    })

    r.Run(":8000")
}
```



## 四、中间件

### 1. 全局中间件

所有的请求都会经过这个中间件

```go
func MiddleWare() gin.HandlerFunc {
	return func(c *gin.Context) {
		t := time.Now()
		fmt.Println("中间件开始执行了")
		// 设置变量到Context的key中，可以通过Get()取
		c.Set("request", "中间件")
		status := c.Writer.Status()
		fmt.Println("中间件执行完毕", status)
		t2 := time.Since(t)
		fmt.Println("time:", t2)
	}
}

func main() {
	// 1.创建路由
	// 默认使用了2个中间件Logger(), Recovery()
	r := gin.Default()
	// 注册中间件
	r.Use(MiddleWare())
	// {}为了代码规范
	{
		r.GET("/ce", func(c *gin.Context) {
			// 取值
			req, _ := c.Get("request")
			fmt.Println("request:", req)
			// 页面接收
			c.JSON(200, gin.H{"request": req})
		})

	}
	r.Run()
}

```



### 2. Next()和Abort()方法

> #### Next()

- 分割前置 / 后置拦截：`Next()`之前的代码会在到达`HandlerFunc`前执行，`Next()`方法之后的代码则在`HandlerFunc`处理后执行。
- 数据传递：当前中间件调用`next()`后 *请求* 将交由内一层中间件。 

**案例：**

```go
func MiddleWare1() gin.HandlerFunc {
	return func(c *gin.Context) {
		fmt.Println("中间件开始执行了1")
		// 设置变量到Context的key中，可以通过Get()取
		c.Set("request1", "中间件1")
		c.Next()
		status := c.Writer.Status()
		fmt.Println("中间件执行完毕1", status)
	}
}
func MiddleWare2() gin.HandlerFunc {
	return func(c *gin.Context) {
		fmt.Println("中间件开始执行了2")
		// 设置变量到Context的key中，可以通过Get()取
		c.Set("request2", "中间件2")
		status := c.Writer.Status()
		// 获取上一层中间件的数据
		res, _ := c.Get("request1")
		fmt.Println("request1-data:", res)
		c.Next()
		fmt.Println("中间件执行完毕2", status)
	}
}
func MiddleWare3() gin.HandlerFunc {
	return func(c *gin.Context) {
		fmt.Println("中间件开始执行了3")
		// 设置变量到Context的key中，可以通过Get()取
		c.Set("request3", "中间件3")
		status := c.Writer.Status()
		// 获取上两层层中间件的数据
		res1, _ := c.Get("request1")
		fmt.Println("request1-data:", res1)
		res2, _ := c.Get("request2")
		fmt.Println("request2-data:", res2)
		c.Next()
		fmt.Println("中间件执行完毕3", status)
	}
}
func main() {
	// 1.创建路由
	// 默认使用了2个中间件Logger(), Recovery()
	r := gin.Default()
	// 注册中间件
	r.Use(MiddleWare1(), MiddleWare2(), MiddleWare3())
	// {}为了代码规范
	{
		r.GET("/ce", func(c *gin.Context) {
			fmt.Println("执行请求处理")
			// 取值
			req1, _ := c.Get("request1")
			fmt.Println("request1-data:", req1)
			req2, _ := c.Get("request2")
			fmt.Println("request2-data:", req2)
			req3, _ := c.Get("request3")
			fmt.Println("request3-data:", req3)
			// 页面接收
			c.JSON(200, gin.H{"request1": req1, "requset2": req2, "request3": req3})
		})

	}
	r.Run()
}
-----------执行结果----------
中间件开始执行了1
中间件开始执行了2
request1-data: 中间件1
中间件开始执行了3
request1-data: 中间件1
request2-data: 中间件2
执行请求处理
request1-data: 中间件1
request2-data: 中间件2
request3-data: 中间件3
中间件执行完毕3 200
中间件执行完毕2 200
中间件执行完毕1 200
```

> #### Abort()

* Abort()方法是阻止请求向内层中间件乃至`HandlerFunc()`流动
* 在Abort()方法会直接执行之前未执行完的中间件（`Next()方法导致`），并直接跳过`HandleFunc`

### 3. 局部中间件

```go
//局部中间键使用
    r.GET("/ce", MiddleWare(), func(c *gin.Context) {
        // 取值
        req, _ := c.Get("request")
        fmt.Println("request:", req)
        // 页面接收
        c.JSON(200, gin.H{"request": req})
    })
```



## 五、会话技术

### 1. Cookie 的使用

```go
func main() {
   // 1.创建路由
   // 默认使用了2个中间件Logger(), Recovery()
   r := gin.Default()
   // 服务端要给客户端cookie
   r.GET("cookie", func(c *gin.Context) {
      // 获取客户端是否携带cookie
      cookie, err := c.Cookie("key_cookie")
      if err != nil {
         cookie = "NotSet"
         // 给客户端设置cookie
         //  maxAge int, 单位为秒
         // path,cookie所在目录
         // domain string,域名
         //   secure 是否智能通过https访问
         // httpOnly bool  是否允许别人通过js获取自己的cookie
         c.SetCookie("key_cookie", "value_cookie", 60, "/",
            "localhost", false, true)
      }
      fmt.Printf("cookie的值是： %s\n", cookie)
   })
   r.Run(":8000")
}
```



## 六、其它

### 1. 日志文件

```go
func main() {
    gin.DisableConsoleColor()

    // Logging to a file.
    f, _ := os.Create("gin.log")
    gin.DefaultWriter = io.MultiWriter(f)

    // 如果需要同时将日志写入文件和控制台，请使用以下代码。
    // gin.DefaultWriter = io.MultiWriter(f, os.Stdout)
    r := gin.Default()
    r.GET("/ping", func(c *gin.Context) {
        c.String(200, "pong")
    })
    r.Run()
}
```



### 2. Air 实时加载

> 我们要介绍一个神器——Air能够实时监听项目的代码文件，在代码发生变更之后自动重新编译并执行，大大提高gin框架项目的开发效率

**安装**

```shell
go get -v -u github.com/cosmtrek/air
```



**air_example.conf 示例文件**

```toml
# [Air](https://github.com/cosmtrek/air) TOML 格式的配置文件

# 工作目录
# 使用 . 或绝对路径，请注意 `tmp_dir` 目录必须在 `root` 目录下
root = "."
tmp_dir = "tmp"

[build]
# 只需要写你平常编译使用的shell命令。你也可以使用 `make`
cmd = "go build -o ./tmp/main ."
# 由`cmd`命令得到的二进制文件名
bin = "tmp/main"
# 自定义的二进制，可以添加额外的编译标识例如添加 GIN_MODE=release
full_bin = "APP_ENV=dev APP_USER=air ./tmp/main"
# 监听以下文件扩展名的文件.
include_ext = ["go", "tpl", "tmpl", "html"]
# 忽略这些文件扩展名或目录
exclude_dir = ["assets", "tmp", "vendor", "frontend/node_modules"]
# 监听以下指定目录的文件
include_dir = []
# 排除以下文件
exclude_file = []
# 如果文件更改过于频繁，则没有必要在每次更改时都触发构建。可以设置触发构建的延迟时间
delay = 1000 # ms
# 发生构建错误时，停止运行旧的二进制文件。
stop_on_error = true
# air的日志文件名，该日志文件放置在你的`tmp_dir`中
log = "air_errors.log"

[log]
# 显示日志时间
time = true

[color]
# 自定义每个部分显示的颜色。如果找不到颜色，使用原始的应用程序日志。
main = "magenta"
watcher = "cyan"
build = "yellow"
runner = "green"

[misc]
# 退出时删除tmp目录
clean_on_exit = true
```

### 3. 验证码

。。。

