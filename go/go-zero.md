# 一、安装配置

## goctl 安装

goctl 是 go-zero 的内置脚手架，是提升开发效率的一大利器，可以一键生成代码、文档、部署 k8s yaml、dockerfile 等。

```shell
$ GO111MODULE=on go install github.com/zeromicro/go-zero/tools/goctl@latest
# 验证是否安装成功
$ goctl --version
```



## protoc 安装

方式一：

```shell
# 通过 goctl 一键安装
goctl env check --install --verbose --force
```

方式二：

```shell
# 下载压缩包并解压压缩包到 $GOBIN 目录，GOBIN 为 $GOPATH/bin
# 获取 $GOPATH
go env GOPATH

# 这个protoc可以将proto文件编译为任何语言的文件，想要编译为go语言的，还需要下载另外一个可执行文件
# 安装 protoc-gen-go 和 protoc-gen-go-grpc
goctl env check --install --verbose --force
```



**注意：我们需要将下载得到的可执行文件protoc所在的 bin 目录加到我们电脑的环境变量中**

验证：

```shell
$ goctl env check --verbose
[goctl-env]: preparing to check env

[goctl-env]: looking up "protoc"
[goctl-env]: "protoc" is installed

[goctl-env]: looking up "protoc-gen-go"
[goctl-env]: "protoc-gen-go" is installed

[goctl-env]: looking up "protoc-gen-go-grpc"
[goctl-env]: "protoc-gen-go-grpc" is installed

[goctl-env]: congratulations! your goctl environment is ready!
```



## go-zero 安装

```shell
$ mkdir <project name> && cd <project name> # project name 为具体值
$ go mod init <module name> # module name 为具体值
$ go get -u github.com/zeromicro/go-zero@latest
```



## goctl-intellij 安装

goctl-intellij 是 go-zero api 描述语言的 intellij 编辑器插件，支持 api 描述语言高亮、语法检测、快速提示、创建模板特性。



# 二、入门

## DSL 介绍

> 领域特定语言domain-specific language （DSL）是一种旨在特定领域下的上下文的语言

### api 语法

> api 是 go-zero 自研的领域特性语言（下文称 api 语言 或 api 描述语言），旨在实现人性化的基础描述语言，作为生成 HTTP 服务最基本的描述语言。
>
> api 领域特性语言包含语法版本，info 块，结构体声明，服务描述等几大块语法组成，其中结构体和 Golang 结构体 语法几乎一样，只是移出了 `struct` 关键字。





















