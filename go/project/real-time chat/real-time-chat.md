# 项目概述

> 一个实时聊天项目，使用了webSocket
>
> ### webSocket是什么
>
> - WebSocket是一种在单个TCP连接上进行全双工通信的协议
> - WebSocket使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据
> - 在WebSocket API中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输
> - 需要安装第三方包：
>   - cmd中：go get -u -v github.com/gorilla/websocket

**项目需要**

* Golang：1.20

## 服务端

Prompt (Windows) or Terminal (Mac and Linux),execute the following:

```shell
go get github.com/gorilla/websocket
go get github.com/satori/go.uuid
```

每个用户都会通过多路复用库来创建一个`websocket`来进行数据交换，所以我们需要一个`UUID Library`来为每个用户分配id来标识这个`websocket`。

会涉及三种结构体：

```go
type ClientManager struct {
    clients    map[*Client]bool
    broadcast  chan []byte
    register   chan *Client
    unregister chan *Client
}

type Client struct {
    id     string
    socket *websocket.Conn
    send   chan []byte
}

type Message struct {
    Sender    string `json:"sender,omitempty"`
    Recipient string `json:"recipient,omitempty"`
    Content   string `json:"content,omitempty"`
}
```

对于 `ClientManager`用于跟踪各个客户端对应的`websocker`，当有数据读写，就会调用相应的方法。

服务端会启用一个主`goroutine` 来处理请求连接，然后每个连接启动两个`goroutine`进行读写数据。

### 第三方依赖

使用到的开源库：

gin —— 路由、路由组、中间件
zap —— 高性能日志方案
gorm —— ORM 数据操作
cobra —— 命令行结构
viper —— 配置信息
cast —— 类型转换
redis —— Redis 操作
jwt —— JWT 操作
base64Captcha —— 图片验证码
govalidator —— 请求验证器
limiter —— 限流器
email —— SMTP 邮件发送
aliyun-communicate —— 发送阿里云短信
ansi —— 终端高亮输出
strcase —— 字符串大小写操作
pluralize —— 英文字符单数复数处理
faker —— 假数据填充
imaging —— 图片裁切

### 自定义的包

现在来看下我们自建的库：

app —— 应用对象
auth —— 用户授权
cache —— 缓存
captcha —— 图片验证码
config —— 配置信息
console —— 终端
database —— 数据库操作
file —— 文件处理
hash —— 哈希
helpers —— 辅助方法
jwt —— JWT 认证
limiter —— API 限流
logger —— 日志记录
mail —— 邮件发送
migrate —— 数据库迁移
paginator —— 分页器
redis —— Redis 数据库操作
response —— 响应处理
seed —— 数据填充
sms —— 发送短信
str —— 字符串处理
verifycode —— 数字验证码



## 登陆服务

> 对于登陆功能，是我们最常见的服务，这个服务技术最重要的就是**登陆鉴权**
>
> 登陆方案：
>
> * Cookie + Session 登陆
> * Token 登陆
> * SSO 单点登陆
> * OAuth 第三方登陆
> * 扫码 登陆
> * 一键登陆

![](images\屏幕截图 2024-02-23 223008.png)





创建用户表：

```sql
CREATE TABLE userInfo (
    id bigint AUTO_INCREMENT,
    name varchar(255) NULL COMMENT '用户名',
    password varchar(255) NOT NULL DEFAULT '' COMMENT '密码',
    mobile varchar(255) NOT NULL DEFAULT '' COMMENT '电话号码',
    email varchar(255) NOT NULL DEFAULT '' COMMENT '邮箱',
    gender char(10) NOT NULL DEFAULT '未知' COMMENT '性别,男|女|未知',
    nickname varchar(255) NULL DEFAULT '' COMMENT '昵称',
    type tinyint(1) NULL DEFAULT 0 COMMENT '用户类型, 0:normal,1:vip',
    exp mediumint NULL COMMENT '用户经验值',
    birthday NULL COMMENT '生日',
    login_at timestamp NULL COMMENT '上次登陆时间',
    create_at timestamp NULL,
    update_at timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE mobile_index (mobile),
    UNIQUE name_index (name),
    UNIQUE email_index (email)
    PRIMARY KEY (id)
) ENGINE = InnoDB COLLATE utf8mb4_general_ci COMMENT 'userInfo table';
```



## 工程维度

```text
.
├── consumer
├── go.mod
├── internal
│   └── model
├── job
├── pkg
├── restful
├── script
└── service
```

- consumer： 队列消费服务
- internal： 工程内部可访问的公共模块
- job： cron job 服务
- pkg： 工程外部可访问的公共模块
- restful：HTTP 服务目录，下存放以服务为维度的微服务
- script：脚本服务目录，下存放以脚本为维度的服务
- service：gRPC 服务目录，下存放以服务为维度的微服务

## 服务维度

```text
example
├── etc
│   └── example.yaml
├── main.go
└── internal
    ├── config
    │   └── config.go
    ├── handler
    │   ├── xxxhandler.go
    │   └── xxxhandler.go
    ├── logic
    │   └── xxxlogic.go
    ├── svc
    │   └── servicecontext.go
    └── types
        └── types.go
```

Copy

- example：单个服务目录，一般是某微服务名称
- etc：静态配置文件目录
- main.go：程序启动入口文件
- internal：单个服务内部文件，其可见范围仅限当前服务
- config：静态配置文件对应的结构体声明目录
- handler：handler 目录，可选，一般 http 服务会有这一层做路由管理，`handler` 为固定后缀
- logic：业务目录，所有业务编码文件都存放在这个目录下面，`logic` 为固定后缀
- svc：依赖注入目录，所有 logic 层需要用到的依赖都要在这里进行显式注入
- types：结构体存放目录



