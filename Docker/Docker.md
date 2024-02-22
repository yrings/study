# Docker

## 一、Docker简介

Docker 是一个开源的应用容器引擎，基于 [Go 语言](https://www.runoob.com/go/go-tutorial.html) 并遵从 Apache2.0 协议开源。

Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。

容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。

### 1.1 DevOps

是一种管理模式，主要是用于促进开发与运维部门间的沟通、协作的一种思想。



### 1.2 三大要素

镜像（image）、容器（container）和仓库（repository）



### 1.3 Docker底层原理

采用的是C/S模式

![](image\屏幕截图 2024-01-28 153051.png)

1. 用户是使用 Docker Client 与 Docker Daemon 建立通信，并发送请求给后者
2. Docker Daemon 作为 Docker 架构的主体部分，首先提供 Docker Server 的功能使其可以接收 Docker Client 的请求
3. Docker Engine 执行 Docker 内部的一系列工作，每一项工作都是以一个 Job 的形式存在的
4. Job 的运行过程中，当需要容器镜像时，则会从 Docker Registy 中下载镜像，并通过镜像管理驱动 Graph driver 将下载的镜像以 Graph 的形式存储。
5. 当需要为 Docker 创建网络环境时，通过网络管理驱动 Network driver 创建并配置 Docker 容器网络环境。
6. 当需要限制 Docker 容器运行资源或执行用户指令等操作时，则通过Exec driver来完成
7. Libcontainer 是一项独立的容器管理包，Network driver 以及Exec driver 都是通过 Libcontainer来实现具体对容器进行操作。

### 1.4 在Centos中安装

```txt
# 安装gcc
yum -y install gcc
# 安装gcc-c++
yum -y install gcc-c++
# 安装软件包
yum install -y yum-utils
# 添加镜像仓库
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 重建yum索引，加快加载
yum makecache fast
# 安装docker ce
yum -y install docker-ce docker-cecli containerd.io
```

**初次启动docker**

```txt
# 启动docker
systemctl start docker
# 查看线程
ps -ef|grep docker
# 启动初始镜像
docker run hello-world
```

**卸载docker**

```txt
# 停止docker进程
systemctl stop docker
# 卸载软件
yum remove docker-ce docker-ce-cli containerd.io
# 删除相关文件
rm -rf /var/lib/docker
rm -rf /var/lib/containerd
```

 **配置阿里云镜像加速器**

登陆阿里云，找到工作台，找到容器镜像服务

https://uk0rxcwz.mirror.aliyuncs.com

针对Docker客户端版本大于 1.10.0 的用户

您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

```txt
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://uk0rxcwz.mirror.aliyuncs.com"]
}
EOF
systemctl daemon-reload
systemctl restart docker
```

### 1.5 run命令

![](.\image\屏幕截图 2024-01-28 223907.png)

### 1.6 为什么Docker要比VM快

* #### docker有比虚拟机更少的抽象层

  由于docker不需要`Hypervisor`（虚拟机）实现硬件资源虚拟化，运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。

  因此在cpu、内存利用率上docker将会在效率上有明显的优势

* #### docker利用的是宿主主机的内核，而不需要加载操作系统

  当新建一个容器时，docker不需要和虚拟机一样重新加载一个操作系统内核。进而避免引寻、加载操作系统内核返回等比较费时资源的过程，当新建一个虚拟机时，虚拟机软件需要加载os，返回新建过程是分钟级别的。而docker由于直接利用宿主机的操作系统，则省略了返回过程，因此新建了一个docker容器只需要几秒钟。



## 二、Docker命令

### 2.1 帮助启动类命令

```shell
# 启动docker
systemctl start docker

# 停止docker
systemctl stop docker

# 重启docker
systemctl restart docker

# 查看docker状态
systemctl status docker

# 开机启动
systemctl enable docker

# 查看docker概要信息
docker info

# 查看docker总体帮助文档
docker --help

# 查看docker命令帮助文档
docker [具体命令] --help
```

### 2.2 镜像命令

* **docker images**

```txt
docker images
-----------
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d2c94e258dcb   9 months ago   13.3kB
各个选项说明：
REPOSITORY：镜像的仓库源
TAG：镜像的标签版本号
IMAGE ID：镜像ID
CREATED：创建时间
SIZE：大小
----------
-a:列出本地所有的镜像，含历史映像层
-q:只显示镜像ID
```

* **docker search**

搜索某个镜像

```txt
docker search redis
--------------------
NAME           DESCRIPTION                                     STARS     OFFICIAL
redis          Redis is an open source key-value store that…   12631     [OK]
-----------------
docker search --limit 5 redis
只显示五条记录
```

* **docker pull**

```txt
docker pull redis [:TAG]
```



* **docker system df**

查看docker镜像容器本地所占硬盘空间

> 如何查看linux的硬盘空间？
>
> linux:`df -h`



* **docker rmi**

```txt
docker rmi hello-world
---------------------
删除某个指定镜像，可以使用镜像名或者镜像id

docker rmi -f hello-world
----------------------
强制删除

docker rmi -f $(docker images -qa)
----------------------
删除全部
```

#### 2.2.1 什么是docker的虚悬镜像？

> 仓库名、标签都是<none> 的镜像，俗称就是虚悬镜像dangling image



### 2.3 容器命令

> docker是必须在linux内核环境下才能运行的，下面用ubuntu镜像来实验容器命令。

* **docker run**

```txt
docker run [OPTION] IMAGE [COMMAND] [ARG...]
------------------
新建+启动容器实例
-----------------
OPTION：
--name：容器新名字
-d：后台运行容器并返回容器ID，也即启动守护式容器（后台运行）

-i：以交互模式运行容器，通常与-t同时使用
-t：为容器重新分配一个伪输入终端，通常与-i同时使用；
也就是启动交互式容器
--------------------
docker run -it --name=myu1 ubuntu /bin/bash
docker run -it --name=myu1 ubuntu bash

-P：随机端口映射，大写
-p：自定义端口映射，小写
--------------------
docker run -p 6379:6379 redis
表示 docker宿主机暴露的端口:容器的实际端口
```

* **docker ps**

```
docker ps
---------------
罗列所有正在运行的容器实例
------------------
OPTION:
-a 罗列所有历史上运行过的容器（包括删除的）
-i 显示最近创建的容器
-n 显示最近n个创建的容器
-q 静默模式，只显示容器编号
---------------------
docker ps -n 10,显示最近创建的容器，也会显示已经关闭的容器
```

* **退出容器**

```txt
exit
# 退出容器，并关闭容器实例
ctrl + p + q
# 转为后台运行
```

* **简单命令**

```
# 启动已停止运行的容器
docker start [容器id或者容器名]

# 重启容器
docker restart [容器id或者容器名]

# 停止容器
docker stop [容器id或者容器名]

# 强制停止容器
docker kill [容器id或者容器名]

# 删除已停止的容器
docker rm [容器id或者容器名]
# 强制删除
docker rm -f [容器id或者容器名]
# 删除多个
docker rm -f $(docker ps -a -q)
docker ps -a -q | xargs docker rm

# 设置容器是否自启动
docker update --restart=always 容器名或容器ID
docker update --restart=always <CONTAINER ID>
# 例如将tomcat设为自启动
docker update --restart=always tomcat
# docker update --restart=no 容器名或容器ID
docker update --restart=no <CONTAINER ID>
# 例如取消tomcat的自启动
docker update --restart=no tomcat
```

* **重要命令**

> 在大多数情况下，我们希望docker容器服务是在后台运行的，我们可以通过 -d 选项指定容器的后台运行模式。
>
> 当我们使用 `docker run -d ubuntn`后台启动，然后用`docker ps`发现，容器已经退出了，这时docker的机制问题，对于某些容器，**容器运行的命令如果不是一直挂起的命令（top，tail），就是会自动退出的**。
>
> 下面运用redis镜像作为例子。

**可以把每一个容器实例看做是一个简易版的linux环境**

```
# 启动redis容器实例
docker run -d reids

# 查看日志
docker logs [容器id]

# 查看docker运行的进程
docker top
# 查看容器内的运行情况
docker inspect [容器id]

# 进入正在运行的容器并以命令行交互
dcoker exec -it [容器id] /bin/bash
docker attch [容器id]

# 从容器内拷贝文件到主机上
docker cp [容器id:容器内路径 主机目录]

# 导入和导出容器

# 导出容器的内容留作为一个tar归档文件
docker export [容器id] > 文件名.tar

# 从tar包中的内容创建一个新的文件系统再导入为镜像
cat 文件名.tar | docker import -[镜像用户/镜像名:镜像版本号]
```

#### 2.3.1 exec 和 attch的区别

attach 直接进入容器启动命令的终端，不会启动新的进程用exit退出，会导致容器的停止。

exec 是在容器中打开新的终端，并且可以启动新的进程，用exit退出不会导致容器的停止。

* ##### docker stats

  可以查看docker cpu、内存的使用情况

  

##  三、Docker  镜像



### 3.1 镜像分层

#### 3.1.1 UnionFS 联合文件系统

> Union 文件系统 是一种分层、轻量级并且高性能的文件系统，**它支持对文件系统的修改为一次提交来层层的叠加**，同时可以将不同目录挂载到同一个虚拟文件系统下。Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来继承，基于基础镜像，可以制作各种具体应用的镜像。

#### 3.1.2 镜像加载原理

docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。

**bootfs（boot file system）**主要包含 **bootloader、kernel（内核：操作系统的核心组件）**，bootloader主要是引导加载kernel，Linux刚启动时，会加载bootfs 文件系统，在 Docker 镜像的最底层是引导文件系统 `bootfs`。**这一层bootfs与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核**。当boot加载完成之后，整个内核就都在内存中了，此时内存的使用权已由bootfd转交给内核，此时系统也会卸载bootfs。

**rootfs（root file system）**, 在bootfs之上。包含的就是典型 Linux 系统中的 `/dev、/proc、/bin、/etc`等标准目录和文件。

rootfd是由镜像提供的；

对于一个精简的OS，rootfs可以很小，只需要包含最基本的命令、工具和程序库就可以了，因为底层直接用Host的kernel，自己只需要提供 rootfs就行了。由此可见对于不同的linux发行版，bootfs基本是一直的，rootfs会有差别，因此不同的发行版本可以公用 bootfs。

> 优点

镜像分层最大的一个好处就是共享资源，方便复制迁移，就是为了复用。比如说有多个镜像都从相同的 base 镜像构建而来，那么 docker host 只需在磁盘上保存一份 base 镜像；同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。而镜像的每一层都可以共享。

> 重点理解

**Docker 镜像层都是只读的，容器层是可写的**

当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称作“容器层”，“容器层”之下的都叫做“镜像层”；

**所以就如我们之前所说的，可以吧镜像看成一个小的linux系统**

### 3.2 commit 命令

> docker commit命令作用是提交容器副本使之成为一个新的镜像。
>
> ```
> docker commit -m="[提交的描述信息]" -a="[作者]" [容器id] [要创建的目标镜像名]:[标签名（版本号）]
> ```

#### 3.2.1 实例

> 要在官服的ubuntu镜像中安装vim工具，并创建新的镜像

```
// 启动ubuntu容器
docker run -it --name=demo_ubuntu ubuntu bash
// 更新安装包
apt-get update
// 安装vim工具
apt-get -y install vim
// commit新的镜像
docker commit -m="add vim cmd" -a="yrings" a8f7b94e44b2 yrings-vim/myubuntu:1.1
------------------返回一个id------------
在容器实例关闭后也可以创建

// 查看镜像是否有
docker images
// 利用镜像id或者 镜像名 + TAG，启动容器
docker run -it yrings-vim/myubuntu:1.1
```

#### 3.2.2 发布本地镜像到阿里云

阿里云镜像流程

![](image\屏幕截图 2024-02-01 210752.png)



1. 进入阿里云工作台中的 **容器镜像服务**
2. 创建个人实例，设置仓库密码
3. 创建命名空间`yrings-test`
4. 选择刚刚创建的命名空间，创建镜像仓库`my-ubuntu`
5. 根据给出的步骤和命令登陆、注册、推送镜像。

* ##### 1. 登录阿里云Docker Registry

  ```
  docker login --username=yrings registry.cn-hangzhou.aliyuncs.com
  ```

  用于登录的用户名为阿里云账号全名，密码为开通服务时设置的密码。

  您可以在访问凭证页面修改凭证密码。

  ##### 2. 从Registry中拉取镜像

  ```
  docker pull registry.cn-hangzhou.aliyuncs.com/yrings-test/my-ubuntu:[镜像版本号]
  ```

  ##### 3. 将镜像推送到Registry

  ```
  docker login --username=yrings registry.cn-hangzhou.aliyuncs.com
  
  docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/yrings-test/my-ubuntu:[镜像版本号]
  
  docker push registry.cn-hangzhou.aliyuncs.com/yrings-test/my-ubuntu:[镜像版本号]
  ```

  请根据实际镜像信息替换示例中的[ImageId]和[镜像版本号]参数。

  ##### 4. 选择合适的镜像仓库地址

  从ECS推送镜像时，可以选择使用镜像仓库内网地址。推送速度将得到提升并且将不会损耗您的公网流量。

  如果您使用的机器位于VPC网络，请使用 registry-vpc.cn-hangzhou.aliyuncs.com 作为Registry的域名登录。

  ##### 5. 示例

  使用"docker tag"命令重命名镜像，并将它通过专有网络地址推送至Registry。

  ```
  docker imagesREPOSITORY                                                         TAG                 IMAGE ID            CREATED             VIRTUAL SIZEregistry.aliyuncs.com/acs/agent                                    0.7-dfb6816         37bb9c63c8b2        7 days ago          37.89 MB$ docker tag 37bb9c63c8b2 registry-vpc.cn-hangzhou.aliyuncs.com/acs/agent:0.7-dfb6816
  ```

  使用 "docker push" 命令将该镜像推送至远程。

  ```
  docker push registry-vpc.cn-hangzhou.aliyuncs.com/acs/agent:0.7-dfb6816
  ```



**测试：**

```
// 删除yrinigs-vim/myubuntu 镜像
docker rmi -f 255ae43a2a39
// 拉取
docker pull registry.cn-hangzhou.aliyuncs.com/yrings-test/my-ubuntu:1.1
// 运行
docker run -it 255ae43a2a39 bash
//测试
vim test.txt
```



#### 3.2.3 docker私人仓库

> 官方的Docker Hub地址，类似github，https://hub.docker.com/，但是在中国大陆访问太慢了。
>
> Docker Registry是官方提供的工具，用来构建私人仓库

```
// 下载镜像Docker Registry
docker pull registry
// 运行镜像，创建本地私有库
docker run -d -p 5000:5000 -v /yrings/myregistry/:/tmp/registry --privileged=true registry
--------------------
-d: 后台运行
-p 5000:5000 : 指定端口映射 
默认情况下，仓库被创建在容器的/var/lib/registry目录下，建议自行用容器卷映射，方便宿机联调
```



案例：创建一个新的镜像：ubuntu安装ifconfig的镜像

```
# 启动官方镜像
docker run -it ubuntu bash
# 安装ifconfig工具
apt-get update
apt-get install net-tools

# 退出容器，查看容器id
docker ps -n 3 ----3b34cf4726cb
# commit新的镜像
docker commit -m="ifconfig cmd add" -a="yrings" 3b34cf4726cb yrings-test/myubuntu:1.2
# 查看镜像id，并测试
docker images -------9692f7e0eb
docker run -it 9692f7e0eb bash
```

提交到私人仓库

```
# 查看私人仓库有什么镜像
curl -XGET http://192.168.30.100:5000/v2/_catalog
-------------------
{"repositories":[]}

# 将新镜像修改成符合规范要求的
docker tag yrings-test/myubuntu:1.2 192.168.30.100:5000/yrings-test/myubuntu:1.2
docker images
----------------
会复制一个镜像

# 配置5000端口为安全端口
vim /etc/docker/daemon.json
加上以下这段
"insecure-registries": ["192.168.30.100:5000"]

# 推送镜像
docker push 192.168.30.100:5000/yrings-test/myubuntu:1.2
# 验证
curl -XGET http://192.168.30.100:5000/v2/_catalog

# 先删除
docker rmi -f 192.168.30.100:5000/yrings-test/myubuntu:1.2
# 拉取到本地
docker pull 192.168.30.100:5000/yrings-test/myubuntu:1.2
```

> 配置安全端口的原因：
>
> docker默认不允许http方式推送镜像，通过配置选项来取消这个限制。
>
> 如果配置完没生效，重启docker





## 四、Docker 容器卷

> 容器数据卷的作用主要是将容器内的数据持久化到主机目录里，防止意外导致的数据丢失。类似于redis的rdb和aof。存在于一个或多个容器中，由docker挂载到容器，DNA不属于联合文件系统，因此能够要过 Union File System 提供一些用于持续存储或共享数据的特性。**docker 内产生的数据会映射、备份到主机配置的路径**
>
> 卷的设计目的就是数据的持久化，完全独立与容器的生命周期，因此Docker不会再容器删除时删除其挂载的数据卷

> ```shell
> docker run -d -p 5000:5000 -v /yrings/myregistry/:/tmp/registry --privileged=true registry
> ```
>
> Docker 挂载主机目录访问如果出现 `cannot open directory:Permission denied`
>
> 解决办法：在挂载目录后多加一个`--privileged=true`参数即可



### 4.1 数据卷的使用

**格式**：

```shell
docker run -it --privileged=true -v /[宿主机绝对路径]:/[容器内目录] [镜像名]
```

**特点：**

1. 数据卷可在容器之间共享或重用数据
2. 卷中的更改可以直接实时生效
3. 数据卷中的更改不会包含在镜像的更新中
4. 数据卷的生命周期一直持续到没有容器使用它为止



**案例：**

```shell
# 运行有数据卷的容器
docker run -it --privileged=true -v /tmp/host_data:/tmp/docker_data --name test_ubuntu ubuntu

# 进入到ubuntu容器的/tmp/docker_data目录创建文件
# 回到主机中创建文件
# 两个都能看到实时的更新

# 获取容器的具体信息，以json格式来展示，用来查看数据卷是否挂载成功
docker inspect [容器id]
-----------------------------
查看Mounts属性

#运行容器数据卷只有读功能,加上:ro
docker run -it --privileged=true -v /tmp/host_data:/tmp/docker_data:ro --name test_ubuntu ubuntu
```



#### 4.1.1 容器卷之间的继承

```shell
# 创建u1容器
docker run -it --privileged=true -v /tmp/host_data:/tmp/docker_data --name u1 ubuntu

# 创建u2容器并继承u1容器
docker run -it --privieged=true --volumes-from u1 --name u2 ubuntu
```

> 容器卷之间的继承是继承映射规则，与u1并没有实际的连续，所以当u1挂掉时，对u2不会有任何影响





## 五、Docker的实际使用



