# ETCD

## 安装配置

etcd是使用Go语言开发的一个开源的、高可用的分布式key-value存储系统，可以用于配置共享和服务的注册和发现。

类似项目有zookeeper和consul。

etcd具有以下特点：

- 完全复制：集群中的每个节点都可以使用完整的存档
- 高可用性：Etcd可用于避免硬件的单点故障或网络问题
- 一致性：每次读取都会返回跨多主机的最新写入
- 简单：包括一个定义良好、面向用户的API（gRPC）
- 安全：实现了带有可选的客户端证书身份验证的自动化TLS
- 快速：每秒10000次写入的基准速度
- 可靠：使用Raft算法实现了强一致、高可用的服务存储目录





### 单机配置

* #### 创建本机数据存储目录

```shell
# 创建 etcd 数据目录,创建3个
# -p 是新建嵌套文件夹
rm -rf /tmp/etcd/single/data 
mkdir -p /tmp/etcd/single/data
mkdir -p /tmp/etcd/single/conf
# mkdir -p etcd-node{1,2,3}
```

* #### 编写 etcd.yml

```shell
vim /tmp/etcd/single/conf/etcd.yml

name: etcd-node0
data-dir: /etcd-data
listen-client-urls: http://0.0.0.0:2379
advertise-client-urls: http://0.0.0.0:2379
listen-peer-urls: http://0.0.0.0:2380
initial-advertise-peer-urls: http://0.0.0.0:2380
initial-cluster: etcd-node0=http://0.0.0.0:2380
initial-cluster-token: etcd-cluster-token
initial-cluster-state: new
log-level: info
logger: zap
log-outputs: stderr
```

1. `name` 节点的名字,在集群中必须唯一
2. `data-dir` 存储数据的路径,**注意这是容器内部路径**
3. `listen-client-urls`：这个参数用于指定 etcd 服务监听客户端请求的地址和端口。客户端（如 etcdctl 命令行工具、其他使用 etcd 客户端库的应用程序）将通过这些地址与 etcd 服务进行通信，以执行各种操作，如读取、写入和修改键值对.`http://127.0.0.1:2379`就只能内网访问,`http://0.0.0.0:2379` 则表示 etcd 服务将在所有可用的 IPv4 网络接口上监听客户端请求
4. `advertise-client-urls`：这个参数用于告知客户端和其他 etcd 成员如何访问 etcd 服务器。这些 URL 通常是可以从集群外部访问的地址。客户端和其他 etcd 服务器将使用这些地址与 etcd 服务进行通信。
5. `listen-peer-urls`：这个参数用于指定 etcd 服务监听其他 etcd 集群成员节点间通信请求的地址和端口。**这个参数对于 etcd 集群的节点之间的数据同步、心跳检测和集群管理等操作至关重要。**
6. `initial-advertise-peer-urls`：这个参数用于指定 etcd 服务在初始化时向其他 etcd 集群成员宣告其可用于集群间通信的地址和端口。其他 etcd 成员节点将使用这些地址与当前 etcd 服务进行通信，进行数据同步、心跳检测和集群管理等操作。
7. `initial-cluster`：这个参数用于在 etcd 集群初始化时定义集群内所有成员的列表。它是一个逗号分隔的键值对列表，键是成员的名称，值是对应的 initial-advertise-peer-urls。当一个新的 etcd 节点加入集群时，它需要知道集群内其他成员的信息，这个参数就是提供这些信息的途径。

* #### 下载 etcd image

```shell
docker pull quay.io/coreos/etcd:v3.5.5
```

* #### 使用 docker run 启动 etcd

```shell
docker run \
  -d \
  -p 12379:2379 \
  -p 12380:2380 \
  -e ETCDCTL_API=3 \
  --mount type=bind,source=/tmp/etcd/single,destination=/tmp/etcd \
  --name etcd-node0 \
  quay.io/coreos/etcd:v3.5.5 \
  /usr/local/bin/etcd \
  --config-file /tmp/etcd/conf/etcd.yml
```

`--mount` 在启动容器时挂载volume或绑定主机目录，这里使用bind绑定刚才创建的主机目录,

* #### 使用 `docker-compose` 启动集群

```yaml
# --detach 在后台运行容器
# up 启动或重启配置的所有容器
docker-compose up  --detach 
```

```shell
# 安装操作 etcd 相关库
go get go.etcd.io/etcd/clientv3@latest
```



## put和get操作

```go
func main() {
    cli, err := clientv3.New(clientv3.Config{
       Endpoints:   []string{"127.0.0.1:2379"},
       DialTimeout: 5 * time.Second,
    })
    if err != nil {
       fmt.Println("connect to etcd failed, err:", err)
       return
    }
    fmt.Println("connect to etcd success")
    defer cli.Close()

    // put
    ctx, cancel := context.WithTimeout(context.Background(), time.Second)
    _, err = cli.Put(ctx, "yrings-key", "yrings-value")
    cancel()
    if err != nil {
       fmt.Println("put to etcd failed, err:", err)
       return
    }
    // get
    ctx, cancel = context.WithTimeout(context.Background(), time.Second)
    resp, err := cli.Get(ctx, "yrings-key")
    cancel()
    if err != nil {
       fmt.Println("get by etcd failed, err:", err)
       return
    }
    for _, ev := range resp.Kvs {
       fmt.Printf("data----%s:%s", ev.Key, ev.Value)
    }
    ctx, cancel = context.WithTimeout(context.Background(), time.Second)
	Dresp, err := cli.Delete(ctx, "yrings-key")
	cancel()
	if err != nil {
		fmt.Println("delete by etcd failed, err:", err)
		return
	}
	fmt.Println("Dresp:", Dresp)
}

-------------
删除操作：
Dresp: &{cluster_id:14841639068965178418 member_id:10276657743932975437 revision:10 raft_term:3  1}
```

## watch 操作

使用`watch`来获取未来更改的通知。**可以实现配置热加载**

```go
func main() {
	cli, err := clientv3.New(clientv3.Config{
		Endpoints:   []string{"127.0.0.1:2379"},
		DialTimeout: 5 * time.Second,
	})
	if err != nil {
		fmt.Println("connect to etcd failed, err :", err)
		return
	}
	fmt.Println("connect to etcd success")
	defer cli.Close()
	rch := cli.Watch(context.Background(), "yrings-key")
	for wresp := range rch {
		for _, ev := range wresp.Events {
			fmt.Printf("Type: %s Key:%s Value:%s\n", ev.Type, ev.Kv.Key, ev.Kv.Value)
		}
	}
}
```



## 租约操作



```go
func main() {
	cli, err := clientv3.New(clientv3.Config{
		Endpoints:   []string{"127.0.0.1:2379"},
		DialTimeout: time.Second * 5,
	})
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("connect to etcd success.")
	defer cli.Close()

	// 创建一个10秒的租约
	leaseCreateResp, err := cli.Grant(context.TODO(), 10)
	if err != nil {
		fmt.Println("get a lease failed,", err)
		return
	}
	leaseId := leaseCreateResp.ID
	// 租约绑定key
	if _, err = cli.Put(context.TODO(), "yrings-key2", "hhh", clientv3.WithLease(clientv3.LeaseID(leaseId))); err != nil {
		fmt.Println("put failed,", err)
		return
	}
	for {
		getResp, err := cli.Get(context.TODO(), "yrings-key2")
		if err != nil {
			fmt.Println(err)
			return
		}
		if len(getResp.Kvs) == 0 {
			fmt.Println("kv过期了")
			break
		}
		fmt.Println("还没过期:", getResp.Kvs)
		time.Sleep(2 * time.Second)
	}

}
```

### 版本问题

这是本地的版本，直接使用了 `go get github.com/coreos/etcd/clientv3`
或者
他这里默认拉取的是2.3.8，但是看一下远程的地址
[https://pkg.go.dev/go.etcd.io/etcd@v2.3.8+incompatible/clientv3?tab=versions](https://links.jianshu.com/go?to=https%3A%2F%2Fpkg.go.dev%2Fgo.etcd.io%2Fetcd@v2.3.8%2Bincompatible%2Fclientv3%3Ftab%3Dversions)
版本已经到了3以上，这个差距太大了吧
试着换成最新的版本
`go get github.com/coreos/etcd/clientv3@v3.3.25`

然后换一下grpc版本

`go get google.golang.org/grpc@v1.26.0`



## keepAlive

keepAlive 能让租约一直续约下去，直到程序关闭后，就不会再续约。

```go
func main() {
	cli, err := clientv3.New(clientv3.Config{
		Endpoints:   []string{"127.0.0.1:2379"},
		DialTimeout: time.Second * 5,
	})
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("connect to etcd success.")
	defer cli.Close()

	resp, err := cli.Grant(context.TODO(), 5)
	if err != nil {
		log.Fatal(err)
	}

	_, err = cli.Put(context.TODO(), "/test-key/", "test", clientv3.WithLease(resp.ID))
	if err != nil {
		log.Fatal(err)
	}

	// the key '/test-key/' will be kept forever
	ch, kaerr := cli.KeepAlive(context.TODO(), resp.ID)
	if kaerr != nil {
		log.Fatal(kaerr)
	}
	for {
		ka := <-ch
		fmt.Println("ttl:", ka.TTL)
	}
}
```

## 分布式锁

```go
func main() {
	cli, err := clientv3.New(clientv3.Config{Endpoints: []string{"127.0.0.1:2379"}})
	if err != nil {
		log.Fatal(err)
	}
	defer cli.Close()

	// 创建两个单独的会话用来演示锁竞争
	s1, err := concurrency.NewSession(cli)
	if err != nil {
		log.Fatal(err)
	}
	defer s1.Close()
	m1 := concurrency.NewMutex(s1, "/my-lock/")

	s2, err := concurrency.NewSession(cli)
	if err != nil {
		log.Fatal(err)
	}
	defer s2.Close()
	m2 := concurrency.NewMutex(s2, "/my-lock/")

	// 会话s1获取锁
	if err := m1.Lock(context.TODO()); err != nil {
		log.Fatal(err)
	}
	fmt.Println("acquired lock for s1")

	m2Locked := make(chan struct{})
	go func() {
		defer close(m2Locked)
		// 等待直到会话s1释放了/my-lock/的锁
		fmt.Println("会话s2正在获取锁。。。。")
		if err := m2.Lock(context.TODO()); err != nil {
			log.Fatal(err)
		}
		fmt.Println("会话s2获取到锁")
	}()

	if err := m1.Unlock(context.TODO()); err != nil {
		log.Fatal(err)
	}
	fmt.Println("released lock for s1")

	<-m2Locked
	fmt.Println("acquired lock for s2")
}
```









