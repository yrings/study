# Kubernetes

## 一、K8S 入门

### 1.1 应用部署方式的演变

在部署应用程序的方式上，主要经历了三个时代：

> **传统部署**：互联网早期，会直接将应用程序部署在物理机上
>
> 优点：简单，不需要其它技术的参与
>
> 缺点：不能为应用程序定义资源使用边界，很难合理地分配计算资源，而且程序之间容易产生影响
>
> 

> **虚拟部署**：可以在一台物理机上运行多个虚拟机，每个虚拟机都是独立的一个环境
>
> 优点：程序环境不会相互产生影响，提供了一定程度的安全性
>
> 缺点：增加了操作系统，浪费了部分资源2
>
> 

> **容器化部署**：与虚拟化类似，但是共享了操作系统
>
> 优点：
>
> 可以保证每个容器拥有自己的文件系统、CPU、内存、进程空间等
>
> 运行应用程序所需要的资源都被容器包装，并和底层基础架构解耦
>
> 容器化的应用程序可以跨云服务商、跨Linux操作系统发行版进行部署

容器化部署方式给带来很多的便利，但是也会出现一些问题，比如说：

* 一个容器故障停机了，怎么样让另外一个容器立刻启动去替补停机的容器
* 当并发访问量变大的时候，怎么样做到横向扩展容器数量

这些容器管理的问题统称为容器编排问题，为了解决这些容器编排问题，就产生了一些容器编排的软件：

**Swarm**：Docker自己的容器编排工具
**Mesos**：Apache的一个资源统一管控的工具，需要和Marathon结合使用
**Kubernetes**：Google开源的的容器编排工具



### 1.2 Kubernetes 简介

```
kubernetes的本质是一组服务器集群，它可以在集群的每个节点上运行特定的程序，来对节点中的容器进行管理。目的是实现资源管理的自动化，主要提供了如下的主要功能：
```

* **自我修复**：一旦某一个容器崩溃，能够在1秒中左右迅速启动新的容器
* **弹性伸缩**：可以根据需要，自动对集群中正在运行的容器数量进行调整
* **服务发现**：服务可以通过自动发现的形式找到它所依赖的服务
* **负载均衡**：如果一个服务起动了多个容器，能够自动实现请求的负载均衡
* **版本回退**：如果发现新发布的程序版本有问题，可以立即回退到原来的版本
* **存储编排**：可以根据容器自身的需求自动创建存储卷



### 1.3 Kubernetes 组件

> 一个Kubernetes集群主要是由控制节点（master）、工作节点（node）构成；每个节点上都会配置有相应的组件来监控或控制节点。

![](image\屏幕截图 2024-01-30 173711.png)

#### 1.3.1 Master节点

![](image\屏幕截图 2024-01-30 173016.png)

构成组件：

* ##### etcd

  可以理解为 K8S 的数据库，键值类型存储的分布式数据库，提供了基于 Raft 算法实现自主集群高可用

  老版本：基于内存的

  新版本：持久化存储

* ##### kube-apiserver

  接口服务，基于REST风格开发k8s接口的服务

* ##### kube-controller-manager

  控制管理器，管理各个类型的控制器。负责运行控制器进程。从逻辑上说，**每个控制器都是一个单独的进程**，但是为了降低复杂性，它们都被编译到同一个可执行文件，并在同一个进程中运行。

  针对 k8s 中的各种资源进行管理

  控制器包括：

  ```txt
  节点控制器（Node Controller）：负责在节点出现故障时进行通知和响应
  任务控制器（Job Controller）：监测代表一次性任务的Job对象，然后创建Pods来运行这些任务直至完成
  端点分片控制器（EndpointSlice controller）：填充端点分片对象（以提供service 和 Pod 之间的链接）
  服务账号控制器（ServiceAccount Controller）：为新的命名空间创建默认的服务账号
  ```

* ##### cloud-controler-manager

  云控制管理器：第三方云平台提供的控制器 API 对接管理功能

* ##### kube-scheduler

  负责将 Pod 基于一定算法，将其调用到更合适的节点服务器上。                                                                                                

#### 1.3.2 Node 节点

构成组件：

* ##### kubelet:

  负责维护容器的生命周期，即通过控制docker，来创建、更新、销毁容器

* ##### kube-proxy:

  网络代理，负责提供集群内部的服务发现和负载均衡（4层负载）

* ##### container runtime:

  容器运行时环境（接口）：docker、containerd、CRI-C，负责节点上容器的各种操作



#### 1.3.3 其它组件

* ##### kube-dns

* ##### Ingress Controller

* ##### Heapster

* ##### Dashboard

* ##### Federation

* ##### Fluentd-elasticsearch

### 1.4 Kubernetes 概念

```
Master：集群控制节点，每个集群需要至少一个master节点负责集群的管控

Node：工作负载节点，由master分配容器到这些node工作节点上，然后node节点上的docker负责容器的运行

Pod：kubernetes的最小控制单元，容器都是运行在pod中的，一个pod中可以有1个或者多个容器

Controller：控制器，通过它来实现对pod的管理，比如启动pod、停止pod、伸缩pod的数量等等

Service：pod对外服务的统一入口，下面可以维护者同一类的多个pod

Label：标签，用于对pod进行分类，同一类pod会拥有相同的标签

NameSpace：命名空间，用来隔离pod的运行环境
```



### 1.5 K8S集群的搭建

####  1.5.1 搭建方案

* ##### minikube

* ##### kubeadm

* ##### 二进制安装

* ##### 命令行安装

#### 1.5.2 环境准备

* ##### 三台服务器（虚拟机） 系统环境

  master：192.168.30.100

  node1：192.168.30.101

  node2：192.168.30.102

  最低配置：2核心、2G内存、20G硬盘

  ```
  # 检查Linux系统版本，Kubernetes集群要求Centos版本在7.5或之上
  cat /etc/redhat-release
  -----------
  CentOS Linux release 7.9.2009 (Core)
  
  # 修改静态ip地址
  vi /etc/sysconfig/network-scripts/ifcfg-ens33
  systemctl restart network
  
  # 设置主机名字
  hostnamectl set-hostname <hostname>
  
  # 编辑master服务器/etc/hosts文件，添加以下内容
  cat >> /etc/hosts << EOF
  192.168.30.100 k8s-master
  192.168.30.101 k8s-node1
  192.168.30.102 k8s-node2
  EOF
  
  # 关闭防火墙
  systemctl stop firewalld
  systemctl disable firewalld
  
  # 关闭selinux,是linux系统下的一个安全服务
  sed -i 's/enforcing/disabled/' /etc/selinux/config && setenforce 0 	
  
  
  # 关闭swap，需要重启
  swapoff -a #临时
  sed -ri 's/.*swap.*/#&/' /etc/fstab # 永久
  
  # 将桥接的IPv4流量传到iptables链
  cat > /etc/sysctl.d/k8s.conf << EOF
  net.bridge.bridge-nf-call-ip6tables = 1
  net.bridge.bridge-nf-call-iptables = 1
  net.ipv4.ip_forward = 1
  EOF
  # 生效
  sysctl --system
  
  # 时间同步
  yum install ntpdate -y
  ntpdate time.windows.com
  
  # 重启
  shutdown -r now
  ```

* ##### 软件环境

  Docker：20+

  k8s：1.23.6

  > 在K8S 的1.24版本后，开始不支持docker，为了推出CRI-C规范，但是CRI现在也开始推出了一个CRI-dockerd，来支持CRI接口规范。
  >
  > 所以下面要基于CRI-dockerd来实现部署kubernetes集群

  * ##### 安装cri-dockerdd

    - 开源地址 https://github.com/Mirantis/cri-dockerd
    - 下载地址 https://github.com/Mirantis/cri-dockerd/releases

    ```
    # 上传解压并安装
    tar xf cri-dockerd-0.2.6.amd64.tgz
    cp cri-dockerd/cri-dockerd /usr/bin/
    chmod +x /usr/bin/cri-dockerd
    ```

    配置启动文件

    ```
    cat <<"EOF" > /usr/lib/systemd/system/cri-docker.service
    [Unit]
    Description=CRI Interface for Docker Application Container Engine
    Documentation=https://docs.mirantis.com
    After=network-online.target firewalld.service docker.service
    Wants=network-online.target
    Requires=cri-docker.socket
    [Service]
    Type=notify
    ExecStart=/usr/bin/cri-dockerd --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.7
    ExecReload=/bin/kill -s HUP $MAINPID
    TimeoutSec=0
    RestartSec=2
    Restart=always
    StartLimitBurst=3
    StartLimitInterval=60s
    LimitNOFILE=infinity
    LimitNPROC=infinity
    LimitCORE=infinity
    TasksMax=infinity
    Delegate=yes
    KillMode=process
    [Install]
    WantedBy=multi-user.target
    EOF
    ```

    生成socker文件

    ```
    cat <<"EOF" > /usr/lib/systemd/system/cri-docker.socket
    [Unit]
    Description=CRI Docker Socket for the API
    PartOf=cri-docker.service
    [Socket]
    ListenStream=%t/cri-dockerd.sock
    SocketMode=0660
    SocketUser=root
    SocketGroup=docker
    [Install]
    WantedBy=sockets.target
    EOF
    ```

    设置开机启动（**需要启动docker**）

    ```
    systemctl daemon-reload
    systemctl enable cri-docker --now
    systemctl is-active cri-docker
    ```

    删除

    ```
    rm -rf /usr/lib/systemd/system/cri-docker.socket
    rm -rf /usr/lib/systemd/system/cri-docker.serivce 
    rm -rf /usr/bin/cri-dockerd 
    rm -rf /root/cri-dockerd
    ```

    

  * ##### 安装kubernetes

    配置阿里云yum库

    ```
    cat > /etc/yum.repos.d/kubernetes.repo << EOF
    [kubernetes]
    name=Kubernetes
    baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
    enabled=1
    gpgcheck=0
    repo_gpgcheck=0
    gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
    EOF
    ```

    ```java
    yum install -y kubelet-1.23.6 kubeadm-1.23.6 kubectl-1.23.6
    # 开机启动
    systemctl enable kubelet
    ```

    **修改初始化系统管理器**

    > ubuntu 系统，debian 系统，centos7 系统，都是使用 systemd 初始化系统。systemd 这边已经有一套 cgroup 管理器了，如果容器运行时和 kubelet 使用 cgroupfs，此时就会存在 cgroups 和 systemd 两种 cgroup 管理器。也就意味着操作系统里面存在两种资源分配的视图，当操作系统上存在 CPU，内存等等资源不足的时候，操作系统上的进程会变得不稳定。

    ```
    cat <<EOF > /etc/sysconfig/kubelet
    KUBELET_EXTRA_ARGS="--cgroup-driver=systemd"
    EOF
    ```

    **初始化master节点**

    - **此命令只在master节点执行**，192.168.30.100 替换为你的master节点IP
    - 【若要重新初始化集群状态：kubeadm reset，然后再进行以下初始化操作】

    ```
    kubeadm init \
    --apiserver-advertise-address=192.168.30.100 \
    --image-repository registry.aliyuncs.com/google_containers \
    --kubernetes-version v1.23.6 \
    --service-cidr=10.96.0.0/12 \
    --pod-network-cidr=10.244.0.0/16
    ```

    **卸载kubernetes**

    ```
    yum remove -y kubelet kubeadm kubectl
    kubeadm reset -f
    modprobe -r ipip
    lsmod
    rm -rf ~/.kube/
    rm -rf /etc/kubernetes/
    rm -rf /etc/systemd/system/kubelet.service.d
    rm -rf /etc/systemd/system/kubelet.service
    rm -rf /usr/bin/kube*
    rm -rf /etc/cni
    rm -rf /opt/cni
    rm -rf /var/lib/etcd
    rm -rf /var/etcd
    ```

    ##### kubernetes初始化报错Get "http://localhost:10248/healthz": dial tcp 127.0.0.1:10248: connect: connection refused.的解决办法

    k8s使用kubeadm初始化报如下错误：

    [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp 127.0.0.1:10248: connect: connection refused.

    原因：

    docker的cgroup驱动程序默认设置为system。默认情况下Kubernetes cgroup为systemd，我们需要更改Docker cgroup驱动，

    解决办法：

    ```json
    kubeadm reset
    
    vim /etc/docker/daemon.json
    {
      "registry-mirrors": ["https://82m9ar63.mirror.aliyuncs.com"],
      "exec-opts": ["native.cgroupdriver=systemd"]
    }
    
    systemctl daemon-reload
    systemctl restart docker
    ```

    **注意：node和master节点中也要修改**

    重新初始化即可成功。

    ```
    # 查看当前状态
    systemctl status kubelet 
    ```

    配置kubelet

    ```
    mkdir -p $HOME/.kube
    cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    chown $(id -u):$(id -g) $HOME/.kube/config
    ```

  

#### 1.5.3 加入节点

k8s-node1 和 k8s-node2分别执行

```
# join命令
kubeadm join 192.168.30.100:6443 --token <master 控制台的token> --discovery-token-ca-cert-hash <master 控制台的hash>


----------------------
kubeadm join 192.168.30.100:6443 --token o0jn53.18p7mhiua7lavh0n --discovery-token-ca-cert-hash sha256:4c84826417ee058da8790231dafa50dd9f7b5a6d6166b9f2b83cc9f1bf727498
```

token和--discovery-token-ca-cert-hash在初始化时会打印在控制台：

```
# 在master中判断是否过期
kubeadm token list
# 创建token
kubeadm token create
# 获取--discovery-token-ca-cert-hash的值，然后在前面加上sha256:
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'

# 查看node节点
kubectl get nodes
# 查看组件状态
kubectl get componentstatus(cs)
# 获取命名空间kube-system(默认)中的pods
kubectl get pods -n kube-system
```



#### 1.5.4 配置网络插件

下载calico配置文件

```
cd /opt
mkdir k8s
cd k8s/
curl https://docs.projectcalico.org/manifests/calico.yaml -O
```

修改文件：

```
# 修改的值与我们初始化设置的--pod-network-cidr=10.244.0.0/16一致
CALICO_IPV4POOL_CIDR 属性
```

配置

```
# 查看功能模块
grep image calico.yaml

sed -i 's#docker.io/##g' calico.yaml
# 构建
kubectl apply -f calico.yaml
# 查看是否构建完成
kubectl get po -n kube-system
# 查看构建进度,上面命令查到的名字
kubectl describe po calico-kube-controllers-74dbdc644f-685wq -n kube-system
```

**验证**

```
# 验证
kubectl create deployment nginx --image=nginx

# 暴露端口，设置容器内的端口
kubectl expose deployment nginx --port=80 --type=NodePort

# 查看 pod 以及服务信息，获取映射端口
kubectl get pod,svc

# 访问
curl 192.168.30.100:30959(映射端口)

```

`192.168.30.100:30959`、`192.168.30.101:30959`、`192.168.30.102:30959`

都能访问到nginx服务器。



## 二、Kubernetes 资源管理

> 在kubernetes中，所有的内容都抽象为资源，用户需要通过操作资源来管理kubernetes
>
> 命令行工具kubectl,它有一个问题，在node节点中是没有配置有用户认证信息的，是无法使用kubectl命令
>
> 资源操作：
>
> pods -----po
>
> deployment----deploy
>
> services----svc
>
> namespace-----ns
>
> nodes------no

### 2.1 命令行

#### 2.1.1 在任意节点中使用命令行

1. 将master节点中 `/etc/kubernetes/admin.conf`拷贝到需要运行的服务器的`/etc/kubernetes`目录中。

   ```
   scp /etc/kubernetes/admin.conf root@k8s-node1:/etc/kubernetes
   ```

2. 在对应服务器上配置环境变量

   ```
   echo "export KUBECONIFG=/etc/kubernetes/admin.config" >> ~/.bash_profile
   source ~/.bash_profile
   ```

### 2.2 资源管理方式

* #### 命令式对象管理：直接使用命令去操作资源

  格式：

  ```
  kubectl [command] [type] [name] [flags]
  ------------------------------------------
  comand：指定要对资源执行的操作，例如create、get、delete
  
  type：指定资源类型，比如deployment、pod、service
  
  name：指定资源的名称，名称大小写敏感
  
  flags：指定额外的可选参数
  ```

  ```
  kubectl run nginx-pod --image=nginx:1.17.1 --port=80
  ```

* #### 命令式对象配置：通过命令配置和配置文件去操作资源

  ```
  kubectl create/patch -f nginx-pod.yaml
  kubectl delete -f nginx-pod.yaml
  ```

  **yaml文件**

  ```
  apiVersion: v1
  kind: Namespace
  metadata:
    name: dev
  
  ---
  
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginxpod
    namespace: dev
  spec:
    containers:
    - name: nginx-containers
      image: nginx:latest
  
  ```

  

* #### 声明式对象配置：通过applay命令和配置文件去操作资源

  ```
  kubectl apply -f nginx-pod.yaml
  ```



### 2.3 基本命令

```
# 查看当前状态
systemctl status kubelet 

# 查看node节点
kubectl get nodes

# 查看所有命名空间
kubectl get ns

# 获取指定命名空间kube-system(默认)中的pods
kubectl get pods -n kube-system

# 设置容器规模
kubectl scale depoly --replicas=3 nginx

# 查看pod详细信息
kubectl get po -o wide

# 将容器配置信息以yaml文件形式展示出来
kubectl get deploy nginx -o yaml
```



### 三、深入POD

### 3.1 基础命令

```
# 获取 pod
kubectl get pods 

# 删除 pod
kubectl delete deploy nginx
```

> 由于pods是由deployment创建出来的，所以pod是被deployment所关联的，我们可以删除deployment来删除pod

在`/opt/k8s`创建一个`pods`文件夹，在里面创建pods配置信息

```
# 创建文件
touch nginx-demo.yaml
# 编辑
vim nginx-demo.yaml
```

配置文件

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: nginx-demo
 labels:
  type: app
  test: 1.0.0
 namespace: 'default'
spec:
 containers:
 - name: nginx
   image: nginx:1.7.9
   imagePullPolicy: IfNotPresent
   command:
   - nginx
   - -g
   - 'daemon off;'
   workingDir: /usr/share/nginx/html
   ports:
   - name: http
     containerPort: 80
     protocol: TCP
   env:
   - name: JVM_OPTS
     value: '-Xms128m -Xmx128m'
   resources:   
     requests:
      cpu: 100m
      memory: 128Mi
 restartPolicy: OnFailure
```



#### 3.1.1 pod 配置文件注解

```yaml
# yaml格式的pod定义文件完整内容：
apiVersion: v1        　　 # 必选，版本号，例如v1
kind: Pod                 # 必选，Pod,资源对象类型
metadata:       　　　　　　# 必选，元数据
  name: string              # 必选，Pod名称
  namespace: string         # 必选，Pod所属的命名空间
  labels:       　　　　　　  # 自定义标签
    - name: string      　    # 自定义标签名称
  annotations:              # 自定义注释列表
    - name: string            # 自定义注释列表名称
spec:                     # 必选，Pod中容器的详细定义
  containers:               # 必选，Pod中容器列表
  - name: string              # 必选，容器名称
    image: string             # 必选，容器的镜像名称
    imagePullPolicy: [Always | Never | IfNotPresent]  
                              # 获取镜像的策略
                                # Alawys 下载镜像
                                # IfnotPresent 优先使用本地镜像，否则下载镜像
                                # Nerver 仅使用本地镜像
    command: [string]         # 容器的启动命令列表，如不指定，使用打包时使用的启动命令
    args: [string]            # 容器的启动命令参数列表
    workingDir: string        # 容器的工作目录
    volumeMounts:             # 挂载到容器内部的存储卷配置
    - name: string              # 引用pod定义的共享存储卷的名称，需用volumes[]部分定义的的卷名
      mountPath: string         # 存储卷在容器内mount的绝对路径，应少于512字符
      readOnly: boolean         # 是否为只读模式
    ports:                    # 需要暴露的端口库号列表
    - name: string              # 端口号名称
      containerPort: int        # 容器需要监听的端口号
      hostPort: int             # 容器所在主机需要监听的端口号，默认与Container相同
      protocol: string          # 端口协议，支持TCP和UDP，默认TCP
    env:                      # 容器运行前需设置的环境变量列表
    - name: string              # 环境变量名称
      value: string             # 环境变量的值
    resources:                # 资源限制和请求的设置
      limits:                   # 资源限制的设置
        cpu: string               # Cpu的限制，单位为core数，将用于docker run --cpu-shares参数
        memory: string            # 内存限制，单位可以为Mib/Gib，将用于docker run --memory参数
      requests:                 # 资源请求的设置
        cpu: string               # Cpu请求，容器启动的初始可用数量
        memory: string            # 内存请求，容器启动的初始可用数量
    livenessProbe:            # 对Pod内的容器健康检查的设置，当探测无响应几次后将自动重启该容器，检查方法有exec、httpGet和tcpSocket，对一个容器只需设置其中一种方法即可
      exec:                     # 对Pod容器内检查方式设置为exec方式
        command: [string]         # exec方式需要制定的命令或脚本
      httpGet:                  # 对Pod内个容器健康检查方法设置为HttpGet，需要制定Path、port
        path: string
        port: number
        host: string
        scheme: string
        HttpHeaders:
        - name: string
          value: string
      tcpSocket:                # 对Pod内个容器健康检查方式设置为tcpSocket方式
         port: number
       initialDelaySeconds: 0   # 容器启动完成后首次探测的时间，单位为秒
       timeoutSeconds: 0        # 对容器健康检查探测等待响应的超时时间，单位秒，默认1秒
       periodSeconds: 0         # 对容器监控检查的定期探测时间设置，单位秒，默认10秒一次
       successThreshold: 0
       failureThreshold: 0
       securityContext:
         privileged: false
    restartPolicy: [Always | Never | OnFailure]
                              # Pod的重启策略，
	                              # Always 一旦不管以何种方式终止运行，kubelet都将重启
	                              # OnFailure表示只有Pod以非0退出码退出才重启
	                              # Nerver表示不再重启该Pod
    nodeSelector: obeject     # 设置NodeSelector表示将该Pod调度到包含这个label的node上，以key：value的格式指定
    imagePullSecrets:         # Pull镜像时使用的secret名称，以key：secretkey格式指定
    - name: string
    hostNetwork: false        # 是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
    volumes:                  # 在该pod上定义共享存储卷列表
    - name: string            # 共享存储卷名称 （volumes类型有很多种）
      emptyDir: {}            # 类型为emtyDir的存储卷，与Pod同生命周期的一个临时目录。为空值
      hostPath: string        # 类型为hostPath的存储卷，表示挂载Pod所在宿主机的目录
        path: string          # Pod所在宿主机的目录，将被用于同期中mount的目录
      secret:                 # 类型为secret的存储卷，挂载集群与定义的secre对象到容器内部
        scretname: string
        items:
        - key: string
          path: string
      configMap:              #类型为configMap的存储卷，挂载预定义的configMap对象到容器内部
        name: string
        items:
        - key: string
          path: string
```



### 3.2 探针机制

> LivenessProbe:用于探测容器中的应用是否运行，如果探测失败，kubelet会根据配置的重启策略进行重启，若没有配置，默认认为容器启动成功，不会执行重启策略。



> 需求：在容器其它组件初始化完成之前，都不能启动成功,即不能接收外部数据。
>
> ReadinessProbe：用于探测容器内的程序是否健康，它的返回值如果返回success，那么就认为该容器已经完全启动，并且容器是可以接收外部流量的。



> StartupProbe:当配置了这个探针后，会先禁用其它探针，直到 startupProbe成功后，其它探针才会继续。
>
> 作用：由于有时候不能准确预估应用一定是多长时间启动成功，因此配至爱另外两种方式不方便初始化时长来检测，而配置了这个探针以后，只有在应用启动成功了，才会执行另外两种探针，更方便结合其它探针。



探测方式：

ExecAction：在容器内部执行一个命令，如果返回值为0；则任务容器是健康的。

TCPSocketAction：通过tcp连接监测容器内端口是否开放，如果开放证明容器健康的。

HTTPGetAction：如果接口返回状态码在200~400之间，则健康。

















