# 第五章 Redis 主从集群

> 为了避免Redis的单点故障问题，我们可以搭建一个Redis集群，将数据备份到集群的其它节点上。
>
> Redis 支持三种集群方案：
>
> * 主从复制模式
> * Sentinel（哨兵）模式
> * Cluster 模式

## 一、主从复制集群搭建

> Redis 的主从集群是一个“一主多从”的读写分离集群。集群中的 Master 节点负责处理 客户端的读写请求，而 Slave 节点仅能处理客户端的读请求。
>
> 只所以要将集群搭建为读写分离模式，主要原因是，对于数据库集群，写操作压力一般都较小，压力大多数来自于读操作 请求。所以，只有一个节点负责处理写操作请求即可。

### 伪集群搭建与配置

> 在采用单线程 IO 模型时，为了提高处理器的利用率，一般会在一个主机中安装多台 Redis， 构建一个 Redis 主从伪集群。
>
> 当然，搭建伪集群的另一个场景是，在学习 Redis，而学习用 主机内存不足以创建多个虚拟机。 下面要搭建的读写分离伪集群包含一个 Master 与两个 Slave。它们的端口号分别是：6380、 6381、6382。

**（1）复制redis.conf**

​	在redis安装目录中新建一个`cluster`目录，将`redis.conf`文件复制到`cluster`目录中。里面包含	`redis.conf`用来设置每个Redis节点相同的公共属性。

**（2）修改 redis.conf**

​	**masterauth**

![](image\屏幕截图 2023-10-09 193922.png)

​	因为我们要搭建主从集群，所有每个主机都有可能会是Master，所以最好不要设置密码验证属性 `requirepass`。

​	如果真需要设置，则要每个主机密码都一样。此时每个 配置文件中都要设置两个完全相同的属性：`requirepass` 与 masterauth。其中 `requirepass` 用 于指定当前主机的访问密码，而 masterauth 用于指定当前 slave 访问 master 时向 master 提 交的访问密码，用于让 master 验证自己身份是否合法

**repl-disable-tcp-nodelay**

该属性用于设置是否禁用 TCP 特性 `tcp-nodelay`。设置为 yes 则禁用 `tcp-nodelay`，此时 master 与 slave 间的通信会产生延迟，但使用的 TCP 包数量会较少，占用的网络带宽会较小。 相反，如果设置为 no，则网络延迟会变小，但使用的 TCP 包数量会较多，相应占用的网络 带宽会大。 

>  `tcp-nodelay`：为了充分复用网络带宽，TCP 总是希望发送尽可能大的数据块。为了达到 该目的，TCP 中使用了一个名为 Nagle 的算法。 
>
> Nagle 算法的工作原理是，网络在接收到要发送的数据后，并不直接发送，而是等待着 数据量足够大（由 TCP 网络特性决定）时再一次性发送出去。这样，网络上传输的有效数据 比例就得到了大大提升，无效数据传递量极大减少，于是就节省了网络带宽，缓解了网络压 力。 
>
> `tcp-nodelay` 则是 TCP 协议中 Nagle 算法的开头。
>
> **Nagle算法**：当流量大时延迟会很小，故一般禁用

**（3）新建 redis6380.conf**

```conf
include redis.conf
pidfile /var/run/redis_6380.pid
port 6380
dbfilename dump6380.rdb
appendfilename appendonly6380.aof
replica-priority 90

# logfile access6380.log
```

`include redis.conf`将包含`redis.conf`里面的属性，然后会将本文件里面写的属性代替

`replica-priority`属性，是哨兵用于来选举master的依据，越小优先级越高，如果是0，就不能成为master

`info replication`c查看当前Redis服务的状态，查看role角色

再创建两个分别是redis6381.conf，redis6382.conf

**（4）启动三台redis**

**（5）设置主从关系**

`slaveof 127.0.0.1 6380`

通过命令查看状态信息

`info replication`

### 分级管理

> 若Redis主从集群中的Slave较多时，它们的数据同步过程会对Master形成较大的性能压力。此时可以对这些Slave进行分级管理。
>
> 设置方式很简单，只需要让低级别 Slave 指定其 slaveof 的主机为其上一级 Slave 即可。 不过，上一级 Slave 的状态仍为 Slave，只不过，其是更上一级的 Slave。 
>
> 例如，指定 6382 主机为 6381 主机的 Slave，而 6381 主机仍为真正的 Master 的 Slave

#### 容灾冷处理

> 在 Master/Slave 的 Redis 集群中，若 Master 出现宕机怎么办呢？
>
> 有两种处理方式，
>
> 一种是通过手工角色调整，使 Slave 晋升为 Master 的冷处理；
>
> 一种是使用哨兵模式，实现 Redis 集群的高可用 HA，即热处理。 
>
> 无论 Master 是否宕机，Slave 都可通过 slaveof no one 将自己由 Slave 晋升为 Master。如 果其原本就有下一级的 Slave，那么，其就直接变为了这些 Slave 的真正的 Master 了。而原 来的 Master 也会失去这个原来的 Slave。



### 主从复制的原理

当一个Redis节点（slave节点）接收到类似`slaveof 127.0.0.1 6380`的指令后直至其可以从master持续复制数据。

![](image\QQ图片20231009200833.jpg)

**总结：**

**（1）保存master地址**

当slave接收到slaveof指令后，slave会立即将新的master的地址保存下来。

**（2）建立连接**

slave中维护着一个定时任务该定时任务会尝试着与该 master 建立 socket 连接。如果 连接无法建立，则其会不断定时重试，直到连接成功或接收到 slaveof no one 指令

**（3）slave发送ping命令**

连接建立成功后，slave 会发送 ping 命令进行首次通信。如果 slave 没有收到 master 的 回复，则 slave 会主动断开连接，下次的定时任务会重新尝试连接

**（4）对slave身份验证**

如果 master 收到了 slave 的 ping 命令，并不会立即对其进行回复，而是会先进行身份 验证。如果验证失败，则会发送消息拒绝连接；如果验证成功，则向 slave 发送连接成功响 应。

**（5）master持久化**

首次通信成功后，slave 会向 master 发送数据同步请求。当 master 接收到请求后，会 fork 出一个子进程，让子进程以异步方式立即进行持久化。

**（6）数据发送**

持久化完毕后 master 会再 fork 出一个子进程，让该子进程以异步方式将数据发送给 slave。slave 会将接收到的数据不断写入到本地的持久化文件中。 

在 slave 数据同步过程中，master 的主进程仍在不断地接受着客户端的写操作，且不仅 将新的数据写入到了 master 内存，同时也写入到了同步缓存。当 master 的持久化文件中的 数据发送完毕后，master 会再将同步缓存中新的数据发送给 slave，由 slave 将其写入到本地 持久化文件中。数据同步完成

**（7）slave恢复内存数据**

当 slave 与 master 的数据同步完成后，slave 就会读取本地的持久化文件，将其恢复到 本地内存，然后就可以对外提供读服务了

**（8）持续增量复制**

在 slave 对外提供服务过程中，master 会持续不断的将新的数据以增量方式发送给 slave， 以保证主从数据的一致性。

### 数据同步演变过程

**（1）sync同步**

Redis 2.8 版本之前，首次通信成功后，slave 会向 master 发送 sync 数据同步请求。然后 master 就会将其所有数据全部发送给 slave，由 slave 保存到其本地的持久化文件中。这个过 程称为**全量复制**。 

但这里存在一个问题：在全量复制过程中可能会出现由于网络抖动而导致复制过程中断。 当网络恢复后，slave 与 master 重新连接成功，此时 slave 会重新发送 sync 请求，然后会从 头开始全量复制。 

由于全量复制过程非常耗时，所以期间出现网络抖动的概率很高。而中断后的从头开始 不仅需要消耗大量的系统资源、网络带宽，而且可能会出现长时间无法完成全量复制的情况

**（2）psync同步**

在Redis2.8版本之后，全量复制采用了**psync（Partial Sync ,不完全同步）**同步策略。当 全量复制过程出现由于网络抖动而导致复制过程中断时，当重新连接成功后，复制过程可以 **“断点续传”**。即从断开位置开始继续复制，而不用从头再来。这就大大提升了性能。 为了实现 psync，整个系统做了三个大的变化：

1. **复制偏移量**

   系统为每个要传送数据进行了编号，该编号从 0 开始，每个字节一个编号。该编号称为 复制偏移量。参与复制的主从节点都会维护该复制偏移量

   ![](image\屏幕截图 2023-10-09 210134.png)

   master 每发送过一个字节数据后就会进行累计。统计信息通过 `info replication` 的 `master_repl_offset `可查看到。同时，slave 会定时向 master 上报其自身已完成的复制偏移量 给 master，所以 master 也会保存 slave 的复制偏移量 offset。

   slave在接收到master的数据后，也会累计接收到的偏移量。统计信息通过`info replication` 的 `slave_repl_offset `可查看到

2. **主节点复制ID**

   当 master 启动后就会动态生成一个长度为 40 位的 16 进制字符串作为当前 master 的复 制 ID，该 ID 是在进行数据同步时 slave 识别 master 使用的。通过 info replication 的 `master_replid` 属性可查看到该 ID

   **但是端口加ip地址也能标识啊，为什么还要设置一个复制ID呢？**

   设置复制ID的主要作用其实是安全性。假设没有复制ID，而master在主从复制过程中重启了，主从复制还没有完成，**这时psync同步过程会认为是网络原因导致的复制中断**，而会进行“断点续传”，但是由于master重启了，会导致缓存区的部分数据丢失，可能会使得主从复制的数据不完整。

   而此时如果有复制ID，由于服务启动时的复制ID都不一样，所以在重新复制时，发现复制ID变化了，就能知道master重启了，就采用全量复制，而不是“断点续传”的方式。

3. **复制积压缓冲区**

   当 master 有连接的 slave 时，在 master 中就会创建并维护一个队列 `backlog`，默认大小 为 **1MB**，该队列称为复制积压缓冲区。master 接收到了写操作数据不仅会写入到 master 主存，写入到 master 中为每个 slave 配置的发送缓存，而且还会写入到复制积压缓冲区。其作 用就是用于保存最近操作的数据，以备“断点续传”时做数据补偿，防止数据丢失。

   队列backlog默认大小是与配置文件中的maxmemory是共用的。



**（3）数据同步过程**

![](image\屏幕截图 2023-10-09 210336.png)



psync 是一个由 slave 提交的命令，其格式为 `psync <master_replid> <repl_offset>`，表示 当前 slave 要从指定的 master 中的 repl_offset+1 处开始复制。repl_offset 表示当前 slave 已经 完成复制的数据的 offset。该命令保证了“断点续传”的实现。 

在第一次开始复制时，slave 并不知道 master 的动态 ID，并且一定是从头开始复制，所 以其提交的 psync 命令为 `PSYNC ? -1`。即 `master_replid` 为问号(?)，repl_offset 为-1。 

如果复制过程中断后 slave 与 master 成功连接，则 slave 再次提交 psyn 命令。此时的 psyn 命令的 repl_offset 参数为其前面已经完成复制的数据的偏移量。

 其实，并不是slave提交了psyn命令后就可以立即从master处开始复制，而是需要master 给出响应结果后，根据响应结果来执行。master 根据 slave 提交的请求及 master 自身情况会 给出不同的响应结果。响应结果有三种可能： 

* FULLRESYNC  ：告知 slave 当前 master 的动态 ID 及可以开始 全量复制了，这里的 repl_offset 一般为 0 
* CONTINUE：告知 slave 可以按照你提交的 repl_offset 后面位置开始“续传”了 
* ERR：告知 slave，当前 master 的版本低于 Redis 2.8，不支持 psyn，你可以开始全量复 制了

**（4）存在的问题**

* 在psync的同步过程中，若slave重启，在slave内存中保存的master的动态ID与续传offer都会消失“断点续传”将无法进行，从而只能进行全量复制，导致资源浪费。 
* 在 psync 数据同步过程中，master 宕机后 slave 会发生“易主”，从而导致 slave 需要从 新 master 进行全量复制，形成资源浪费。

**（5）psync同步的改进**

**A、解决 slave 重启问题 **

针对“slave 重启时 master 动态 ID 丢失问题”，改进后的 psync 将 master 的动态 ID 直接 写入到了 slave 的持久化文件中。 slave 重启后直接从本地持久化文件中读取 master 的动态 ID，然后向 master 提交获取复 制偏移量的请求。master 会根据提交请求的 slave 地址，查找到保存在 master 中的复制偏移 量，然后向 slave 回复 FULLRESYNC  ，以告知 slave 其马上要开始 发送的位置。然后 master 开始“断点续传”。 

**B、 解决 slave 易主问题 slave **

易主后需要和新 master 进行全量复制，本质原因是新 master 不认识 slave 提交的 psync 请求中“原 master 的动态 ID”。如果 slave 发送 `PSYNC <原 master_replid>  <repl_offer>`命令，新master能够识别出该slave要从原master复制数据，而自己的数据也都是从该master 复制来的。那么新 master 就会明白，其与该 slave“师出同门”，应该接收其“断点续传”同步请求。 

而新 master 中恰好保存的有“原 master 的动态 ID”。由于改进后的 psync 中每个 slave 都在本地保存了当前 master 的动态 ID，所以当 slave 晋升为新的 master 后，其本地仍保存 有之前 master 的动态 ID。而这一点也恰恰为解决“slave 易主”问题提供了条件。通过 master 的 info replicaton 中的 `master_replid2` 可查看到。如果尚未发生过易主，则该值为 40 个 0。

**（6）无盘操作**

Redis 6.0 对同步过程又进行了改进，提出了“无盘全量同步”与“无盘加载”策略，避 免了耗时的 IO 操作。 

* 无盘全量同步：master 的主进程 fork 出的子进程直接将内存中的数据发送给 slave，无 需经过磁盘。 

* 无盘加载：slave 在接收到 master 发送来的数据后不需要将其写入到磁盘文件，而是直 接写入到内存，这样 slave 就可快速完成数据恢复。

在之前master都会现将数据持久化到本地磁盘再加载到内存中，slave中也是如此。

从内存复制到内存，用socket传输。磁盘不好时，网络带宽性能好时，可以选择无磁盘化复制。

不足：当内存复制到内存的过程中，内存是只读的状态，有高并发的写操作时，会把大量的数据缓存存放在复制积压缓冲区，内存复制到内存的同步操作完成后才会把该区域内的写操作缓存写入内存中。

对于这个同步过程是会导致内存只读，直到内存复制到内存的写操作完成。而相比于之前的写入磁盘，再由磁盘传输到一个slave内存的过程，可能会导致性能的下降。因为master写到磁盘后就完成了同步。



**（7）共享复制积压缓冲区：**

Redis 7.0 版本对复制积压缓冲区进行了改进，让各个 slave 的发送缓冲区共享复制积压 缓冲区。这使得复制积压缓冲区的作用，除了可以保障数据的安全性外，还作为所有 slave 的发送缓冲区，充分利用了复制积压缓冲区。

理论上是将所有的发送缓冲区和复制积压缓冲区合并成一个共享的缓存，但是由于在读操作时导致的只读，所以还要开辟一个新的缓存来缓存要写入共享复制积压缓冲区的数据。

## 二、Sentinel 哨兵机制

Sentinel默认的端口号是26379

当master发生变化时，sentinel配置文件和redis配置文件都会发生变化

> WARO（Write All Read one）机制：
>
> 当Client请求向某副本写数据（更新数据），只有当所有的副本都更新成功，这次写操作才会成功。
>
> 所以看出来①写操作很脆弱，只要有一个副本更新失败，本次操作就会不成功。②读操作很简单，因为始终保证了数据的一致性，所以只需要读取一个副本数据即可。
>
> 牺牲了更新服务的可用性，而最大程度地增强了读服务的可用性，而Quorum就是更新服务与读服务之间的一种折中。
>
> Quorum机制：
>
> 假设有N个副本，更新操作write 在W个副本中更新成功之后，才认为此次更新操作write 成功。称成功提交的更新操作对应的数据为：“成功提交的数据”。对于读操作而言，至少需要读R个副本才能读到此次更新的数据。**其中，W+R>N ，即W和R有重叠。**一般，W+R=N+1。



### Redis  高可用集群的搭建

> 下面搭建一个 “一主二从三哨兵”的高可用的伪集群，即这些角色全部安装运行在一台主机。

| 角色     | 端口号 |
| -------- | ------ |
| master   | 6380   |
| slave    | 6381   |
| slave    | 6382   |
| sentinel | 26380  |
| sentinel | 26381  |
| sentinel | 26382  |

**（1）复制 sentinel.conf**

​	将 Redis 安装目录中的 sentinel.conf 文件复制到cluster目录中。该配置文件中用于存放一些sentinel集群中的一些公共配置。

**（2）修改 sentinel.conf**

**A、sentinel monitor**

该配置用于指定 Sentinel 要监控的 master 是谁，并为 master 起了一个 名字。该名字在后面很多配置中都会使用。

同时指定 Sentinel 集群中决定该 master“客观下线状态”判断的法定 sentinel 数量**`<quorum>`**。**`<quorum>`**的另一个用途与 sentinel 的 Leader 选举有关。要求中至少要有 max(quorum, sentinelNum/2+1)个 sentinel 参与， 选举才能进行。 

**这里将该配置注释掉，因为要在后面的其它配置文件中设置，如果不注释就会出现配置 冲突。**

**B、sentinel auth-pass**

如果 Redis 主从集群中的主机设置了访问密码，那么该属性就需要指定 master 的主机名 与访问密码。以方便 sentinel 监控 master

**（3）新建 sentinel26380.conf**

​	在 Redis 安装目录下的 cluster 目录中新建 sentinel26380.conf 文件作为 Sentinel 的配置文 件，并在其中键入如下内容：

```conf
include sentinel.conf
pidfile /var/run/sentinel_26380.pid
port 26380
sentinel monitor mymaster <ip地址> 6380 2

# logfile access26380.log
```

​	最后的 2 是参数 quorum 的值，quorum 有两个用途。一个是只有当 quorum 个 sentinel 都认为当前 master 宕机了才能开启故障转移。另一个用途与 sentinel 的 Leader 选举有关。 要求中至少要有 max(quorum, sentinelNum/2+1)个 sentinel 参与，选举才能进行

为三个sentinel配置相应的配置文件。

### Redis 高可用集群的启动

**（1）启动并关联 Redis 集群**

​	先启动三台 Redis, 然后再通过 slaveof 关联它们。

`redis-cli -p 6381 slaveof 127.0.0.1 6380`

**（2）启动 Sentinel 集群**

两种启动命令：

* `redis-sentinel sentinel26380.conf`
* `redis-server sentinel23680.conf --sentinel`

然后通过客户端连接Sentinel查看Sentinel的信息

`redis-cli -p 26380 info sentinel`

### ☆Sentinel 优化配置

> 在公共的 sentinel.conf 文件中，还可以通过修改一些其它属性的值来达到对 Sentinel 的 配置优化



**（1）sentinel down-after-milliseconds**

​	 每个 Sentinel 会通过定期发送 ping 命令来判断 master、slave 及其它 Sentinel 是否存活。 如果 Sentinel 在该属性指定的时间内没有收到它们的响应，那么该 Sentinel 就会主观认为该 主机宕机。默认为 30 秒



**（2）sentinel parallel-syncs**

​	该属性用于指定，在故障转移期间，即老的 master 出现问题，新的 master 刚晋升后， 允许多少个 slave 同时从新 master 进行数据同步。默认值为 1 表示所有 slave 逐个从新 master 进行数据同步

​	当sentinel parallel-syncs配置设置过大时，会导致系统的可用性降低，过小会导致数据一致性问题。

**（3）sentinel failover-timeout**

指定故障转移的超时时间，默认时间为 3 分钟。该超时时间的用途很多

`sentinel failover-timeout <master-name> <milliseconds>`

1. 同一个sentinel对同一个master两次failover（故障转移）之间的间隔时间。
2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为正确的master那里同步数据时间
3. 当想要取消一个正在进行的failover所需要的时间。
4. 当进行failover时，配置所有slaves指向新的master所需的最大时间。即使超过了指定时间，slaves依然会被正确配置指向master，但不就不按照`parallel-syncs`所配置的规则来了。



**（4）sentinel deny-scripts-reconfig**

​	指定是否可以通过命令 sentinel set 动态修改 notification-script 与 client-reconfig-script 两 个脚本。默认是不能的。这两个脚本如果允许动态修改，可能会引发安全问题



动态修改配置：

​	通过 redis-cli 连接上 Sentinel 后，通过 sentinel set 命令可动态修改配置信息。例如，下 面的命令动态修改了 sentinel monitor 中的 quorum 的值

| 参数                    | 示例                                                         |
| ----------------------- | ------------------------------------------------------------ |
| quorum                  | sentinel set mymaster quorum 2                               |
| down-after-milliseconds | sentinel set mymaster down-after-milliseconds 50000          |
| failover-timeout        | sentinel set mymaster failover-timeout 300000                |
| parallel-syncs          | sentinel set mymaster parallel-syncs 3                       |
| notification-script     | sentinel set mymaster /var/redis/notify.sh                   |
| client-reconfig-script  | sentinel set mymaster client-reconfig-script /var/redis/reconfig.sh |
| auth-pass               | sentinel set mymaster auth-pass 111                          |



### 哨兵机制原理

#### 三个定时任务

**（1）info 任务**

​	每个 Sentinel 节点每 10 秒就会向 Redis 集群中的每个节点发送 info 命令，以获取最新的 Redis 拓扑结构。

**（2）心跳任务**

​	每个Sentinel节点每1秒就会向所有Redis节点及其它Sentinel节点发送一条ping命令， 以检测这些节点的存活状态。该任务是判断节点在线状态的重要依据

**（3）发布/订阅任务**

**对于每个Sentinel节点是如何知道其它Sentinel节点是信息的呢？**

通过发布/订阅定时任务，每个Sentinel在启动的时候都会去订阅集群中所有Redis节点中的`_ _sentinel_ _:hello`主题信息，当Redis节点中该主题的信息发生了变化，就会立即通知所有订阅者。

启动后，每个Sentinel节点**每 2 秒**就会向` _ _sentinel_ _:hello`主题发送信息，**该信息是当前Sentinel对每个Redis节点在线状态的判断结果及当前Sentinel节点的信息。**

此时该主题信息发生了变化，就会将信息通知到所有订阅者，所以其它Sentinel节点就获取到了该Sentinel节点的信息。

主要完成以下三项工作： 

* 如果发现有新的 Sentinel 节点加入，则记录下新加入 Sentinel 节点信息，并与其建立连接。 
* 如果发现有 Sentinel Leader 选举的选票信息，则执行 Leader 选举过程。 
* 汇总其它 Sentinel 节点对当前 Redis 节点在线状态的判断结果，作为 Redis 节点客观下线的判断依据。

#### Redis 节点下线判断

**（1） 主观下线**

​	 每个 Sentinel 节点每秒就会向每个 Redis 节点发送 ping 心跳检测，如果 Sentinel 在 down-after-milliseconds 时间内没有收到某 Redis 节点的回复，则 Sentinel 节点就会对该 Redis 节点做出“下线状态”的判断。这个判断仅仅是当前 Sentinel 节点的“一家之言”，所以称 为主观下线。 

**（2） 客观下线**

​	 当 Sentinel 主观下线的节点是 master 时，该 Sentinel 节点会向每个其它 Sentinel 节点发 送 `sentinel is-master-down-by-addr` 命令，以询问其对 master 在线状态的判断结果。这些 Sentinel 节点在收到命令后会向这个发问 Sentinel 节点响应 0（在线）或 1（下线）。当 Sentinel 收到超过 quorum 个下线判断后，就会对 master 做出客观下线判断



### Sentinel Leader 选举算法

> 当 Sentinel 节点对 master 节点做出客观下线的判断后，会由 Sentinel Leader 来完成后续的故障转移，即 Sentinel 集群中的节点不是对等的，是存在 Leader 和 follower的。
>
> Sentinel 集群的 leader 选举是通过Raft算法实现的。

### master选择算法

在进行故障转移时，Sentinel Leader 需要从所有 Redis 的 Slave 节点中选择出新的 Master。 其选择算法为： 

1. 过滤掉所有主观下线的，或心跳没有响应 Sentinel 的，或 `replica-priority` 值为 0 的 Redis 节点 
2. 在剩余 Redis 节点中选择出 `replica-priority` 最小的的节点列表。如果只有一个节点，则 直接返回，否则，继续 
3. 从优先级相同的节点列表中选择**复制偏移量（`slave_repl_offset`的值）**最大的节点。如果只有一个节点，则直接返回，否则，继续 。
4. 从复制偏移值量相同的节点列表中选择**动态 ID **最小的节点返回

### 故障转移过程

Sentinel Leader 负责整个故障转移过程，经历了如上步骤：

* Sentinel Leader 根据 master 选择算法选择出一个 slave 节点作为新的 master 
*  Sentinel Leader 向新 master 节点发送 slaveof no one 指令，使其晋升为 master 
* Sentinel Leader 向新 master 发送 info replication 指令，获取到 master 的动态 ID 
* Sentinel Leader 向其余 Redis 节点发送消息，以告知它们新 master 的动态 ID 
* Sentinel Leader 向其余 Redis 节点发送 slaveof  指令，使它们成为新 master 的 slave 
* Sentinel Leader 从所有 slave 节点中每次选择出 `parallel-syncs` 个 slave 从新 master 同步数 据，直至所有 slave 全部同步完毕 
* 故障转移完

### 节点上线

不同的节点类型，其上线的方式也是不同的。 

**（1） 原 Redis 节点上线 **

​	无论是原下线的 master 节点还是原下线的 slave 节点，只要是原 Redis 集群中的节点上 线，只需启动 Redis 即可。因为每个 Sentinel 中都保存有原来其监控的所有 Redis 节点列表， Sentinel 会定时查看这些 Redis 节点是否恢复。如果查看到其已经恢复，则会命其从当前 master 进行数据同步。 

​	不过，如果是原 master 上线，在新 master 晋升后 Sentinel Leader 会立即先将原 master 节点更新为 slave，然后才会定时查看其是否恢复。 

**（2） 新 Redis 节点上线 **

​	如果需要在 Redis 集群中添加一个新的节点，其未曾出现在 Redis 集群中，则上线操作 只能手工完成。即添加者在添加之前必须知道当前 master 是谁，然后在新节点启动后运行 slaveof 命令加入集群。 

**（3） Sentinel 节点上线 **

​	如果要添加的是 Sentinel 节点，无论其是否曾经出现在 Sentinel 集群中，都需要手工完成。即添加者在添加之前必须知道当前 master 是谁，然后在配置文件中修改 sentinel monitor 属性，指定要监控的 master。然后启动 Sentinel 即可。



### CAP 定理

* #### 概念

  > 在分布式系统中有个CAP理论，指的是在一个分布式系统中， Consistency（一致性）、 Availability（可用性）、Partition tolerance（分区容错性），三者不可兼得。
  >
  > 一致性（C）：在分布式系统中的所有数据备份，在同一时刻是否同样的值。（等同于所有节点访问同一份最新的数据副本）
  >
  > 可用性（A）：在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求。（对数据更新具备高可用性）
  >
  > 分区容忍性（P）：以实际效果而言，分区相当于对通信的时限要求。**系统如果不能在时限内达成数据一致性**，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择。
  >
  > 对于P（分区容忍性）而言，是实际存在 从而无法避免的。因为，分布系统中的处理不是在本机，而是网络中的许多机器相互通信，故网络分区、网络通信故障问题无法避免。因此，只能尽量地在C 和 A 之间寻求平衡。对于数据存储而言，**为了提高可用性（Availability），采用了副本备份**，比如对于HDFS，默认每块数据存三份。某数据块所在的机器宕机了，就去该数据块副本所在的机器上读取（从这可以看出，数据分布方式是按“数据块”为单位分布的）

  > 实时一致性：要求数据内容一旦发生更新，客户端立即可以访问到最新的数据。所以，在集群环境下该特性是无法实现的，只存在于单机环境内。
  >
  > 最终一致性：数据内容发生更新后，经过一小段时间后，客户端可以访问到最新的数据。
  >
  > 强一致性：也称严格一致性。要求客户端访问到的一点是更新过的新数据。
  >
  > 弱一致性：允许客户端从集群不同节点访问到的数据是不一致的。

  分布式一致性算法-Paxos、Raft

* #### 定理

  ​	CAP 定理的内容是：对于分布式系统，网络环境相对是不可控的，出现网络分区是不可 避免的，因此系统必须具备分区容错性。但系统不能同时保证一致性与可用性。即要么 CP， 要么 AP。

* #### BASE 理论

  BASE 是 Basically Available（基本可用）、Soft state（软状态）和 Eventually consistent（最终一致性）三个短语的简写。

  BASE 理论的核心思想是：即使无法做到强一致性，但每个系统都可以根据自身的业务特点，采用适当的方式来使系统达到最终一致性。

  **（1）基本可用**

  ​	基本可用是指分布式系统在出现不可预知故障的时候，允许损失部分可用性。

  **（2）软状态**

  ​	软状态，是指允许系统数据存在的中间状态，并认为该中间状态的存在不会影响系统的整体可用性，即允许系统主机间进行数据同步的过程存在一定的时延。软状态，其实就是一种灰度状态，过度状态。

  **（3）最终一致性**

  ​	最终一致性强调的是系统中所有的数据副本，在经过一段时间的同步后，最终能够达到 一个一致的状态。因此，最终一致性的本质是需要系统保证最终数据能够达到一致，而不需 要保证系统数据的实时一致性。

* #### CAP应用

  ##### CP应用：Zookeeeper、Nacos

  ##### AP应用：Consul、Redis、Eureka、Nacos

### ☆Raft算法

* #### 基础

  Raft算法是一种通过对**日志复制管理**来达到集群节点一致性的算法。这个日志复制管理发生在集群节点中的 Leader 与 Follower 之间。 Raft 通过选举出的 Leader 节点负责管理日志复制过程，以实现各个节点间数据的一致性。

* #### 角色、任期及角色转变

  ![](image\屏幕截图 2023-09-22 222320.png)

  

  在 Raft 中，节点有三种角色：

  * Leader：唯一负责处理客户端写请求的节点；也可以处理客户端读请求；同时负责日志复制工作。

  * Candidate：Leader 选举的候选人，其可能会成为 Leader。是一个选举中的过程角色

  * Follower：可以处理客户端读请求；负责同步来自于 Leader 的日志；当接收到其它 Cadidate 的投票请求后可以进行投票；当发现 Leader 挂了，其会转变为 Candidate 发起 Leader 选举

* #### Leader 选举

  **（1）选举**

  ​	若 follower 在心跳超时范围内没有接收到来自于 leader 的心跳，则认为 leader 挂了。此时其先会使自身的 term 增一。然后 follower 会完成以下步骤

  * 此时若接收到其它 candidate 的投票请求，则会将选票投给这个candidate
  * 由 follower 转变为 candidate
  * 若之前尚未投票，则向自己投一票
  * 向其他节点发出投票请求，然后等待响应

  **（2）投票**

  ​	follower 在接收到投票请求后，**会重置`election timeout`选举超时**,其会根据以下情况来判断是否投票： 

  * 发来投票请求的 candidate 的 term 不能小于我的 term 
  * 在我当前 term 内，我的选票还没有投出去 
  * 若接收到多个 candidate 的请求，将采取 first-come-first-served 方式投票

  当收到新的Leader发出的心跳通知，就会将自己的term加一

  **（3）等待响应**

  ​	当一个 Candidate 发出投票请求后会等待其它节点的响应结果。这个响应结果可能有三 种情况： 

  * 收到过半选票，成为新的 leader。然后会将消息广播给所有其它节点，以告诉大家我是 新的 Leader 了 
  * 接收到别的 candidate 发来的新 leader 通知，比较了新 leader 的 term 并不比自己的 term 小，则自己转变为 follower 
  * 经过一段时间后，没有收到过半选票，也没有收到新 leader 通知，则重新发出选举

  **（4）选举时机**

  ​	在很多时候，Leader 真的挂了，follower几乎同时能够感知到，所以它们几乎会同时变成 candidate 发起选举。此时就很有可能导致票数分配过于分散，不能超过半数，也就无法选举出 leader。

  ​	为了减小这种可能性，Raft 算法其采用了**randomized election timeouts策略**。。其会为这些 Follower 随机分配一个选举发起时间 `election timeout`，这个 timeout 在 150-300ms 范围内。只有到达了 election timeout 时间的 Follower 才能转变为 candidate， 否则等待。那么 election timeout 较小的 Follower 则会转变为 candidate 然后先发起选举，一 般情况下其会优先获取到过半选票成为新的 leader

  

* #### 数据同步

​	在 Leader 选举出来的情况下，通过日志复制管理实现集群中各节点数据的同步。

**（1）状态机**

​	Raft 算法一致性的实现，是基于日志复制状态机的。状态机的最大特征是，不同 Server 中的状态机若当前状态相同，然后接受了相同的输入，则一定会得到相同的输出

**（2）处理流程**

​	![](image\屏幕截图 2023-10-31 114022.png)

1. leader 在接收到 client 的写操作请求后，leader 会将数据与 term 封装为一个 box，并随 着下一次心跳发送给所有 followers，以征求大家对该 box 的意见。同时在本地将数据封 装为日志 
2. follower 在接收到来自 leader 的 box 后首先会比较该 box 的 term 与本地记录的曾接受过 的 box 的最大 term，只要不比自己的小就接受该 box，并向 leader 回复同意。同时会将 该 box 中的数据封装为日志。 
3. 当 leader 接收到过半同意响应后，会将日志 commit 到自己的状态机，状态机会输出一 个结果，同时日志状态变为了 committed 
4. 同时 leader 还会通知所有 follower 将日志 commit 到它们本地的状态机，日志状态变为 了 committed 
5. 在 commit 通知发出的同时，leader 也会向 client 发出成功处理的响应

**（3）AP 支持**

​	Log 由 term index、log index 及 command 构成。为了保证可用性，各个节点中的日志可 以不完全相同，但 leader 会不断给 follower 发送 box，以使各个节点的 log 最终达到相同。 即 raft 算法不是强一致性的，而是最终一致的













Stable Leader失效的Leader

在Follower等待election timeouts时间时，如果收到candidate的投票请求，就会向请求对象投票并且重置election timeouts时间，然后term加一

> Raft 算法工作原理：http://thesecretlivesofdata.com/raft
>
> [Raft 分布式共识算法动画演示 (kailing.pub)](http://www.kailing.pub/raft/index.html)
