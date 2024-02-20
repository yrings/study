## 第三章 Redis 命令

> Redis 根据命令所操作的对象不同，可以分为三大类：对Redis进行基础性操作的命令，对Key的操作命令、对Value的操作命令。

### 3.1 Redis 基础命令

* 心跳命令

  `ping`

* 设置获取键值对

  ```
  set key value
  get key
  ```

  （set设置的数值都是字符串，redis专有的字符串，后面会说）

* 选择数据库

  ```
  select <数据库id>
  ```

* 获取数据库大小

  ```
  dbsize
  ```

* 删除数据库

  ```
  flushdb
  flushall
  ```

* 退出客户端

  ```
  exit
  quit
  ```

### 3.2 Key 操作命令

> Redis 中存储的数据整体是一个Map，其key为String类型，而value 则可以是String、Hash表、List、Set等类型

* **keys**

  格式：KEYS pattern

  功能：查找所有符合给定模式pattern的key，pattern为正则表达式

  说明：KEYS的速度非常快，但是在它是单线程阻塞的，在一个大的数据库中使用它可能会阻塞当前服务器。所以使用scan命令代替

* **exists**

  格式：EXISTS key 

  功能：检查给定 key 是否存在。 

  说明：若 key 存在，返回 1 ，否则返回 0 。

* **del**

  格式：DEL key [key ...] 

  功能：删除给定的一个或多个 key 。不存在的 key 会被忽略。 

  说明：返回被删除 key 的数量。

* **rename**

  格式：RENAME key newkey  功能：将 key 改名为 newkey。 

  说明：当 key 和 newkey 相同，或者 key 不存在时，返回一个错误。当 newkey 已经 存在时， RENAME 命令将覆盖旧值。改名成功时提示 OK ，失败时候返回一个错误。

* **move**

  格式：MOVE key db 

  功能：将当前数据库的 key 移动到给定的数据库 db 当中。 

  说明：如果当前数据库(源数据库)和给定数据库(目标数据库)有相同名字的给定 key ， 或者 key 不存在于当前数据库，那么 MOVE 没有任何效果。移动成功返回 1 ，失败 则返回 0

* **type**

  格式：TYPE key

  功能：返回key所存储的值的类型

  说明：一共有六种

  * none（key不存在）
  * string（字符串）
  * list（列表）
  * set（集合）
  * zset（有序集合）
  * hash（哈希表）

* **expire 与 pexpire**

  格式：EXPIRE key seconds 

  功能：为给定 key 设置生存时间。当 key 过期时(生存时间为 0)，它会被自动删除。 expire 的时间单位为秒，pexpire 的时间单位为毫秒。在 Redis 中，带有生存时间的 key 被称为“易失的”(volatile)。 

  说明：生存时间设置成功返回 1。若 key 不存在时返回 0 。rename 操作不会改变 key 的生存时间。

* **ttl与pttl**

  格式：TTL key 

  功能：TTL, time to live，返回给定 key 的剩余生存时间。 

  说明：其返回值存在三种可能： 

  * 当 key 不存在时，返回 -2 。 
  * 当 key 存在但没有设置剩余生存时间时，返回 -1 。 
  * 否则，返回 key 的剩余生存时间。ttl 命令返回的时间单位为秒，而 pttl 命令 返回的时间单位为毫秒。

* **persist**

  格式：PERSIST key 

  功能：去除给定 key 的生存时间，将这个 key 从“易失的”转换成“持久的”。 

  说明：当生存时间移除成功时，返回 1；若 key 不存在或 key 没有设置生存时间，则 返回 0。

* **randomkey**

  格式：RANDOMKEY 

  功能：从当前数据库中随机返回(不删除)一个 key。 

  说明：当数据库不为空时，返回一个 key。当数据库为空时，返回 nil

* **☆scan**

  格式：SCAN cursor [MATCH pattern] [COUNT count] [TYPE  type]

  

  功能：都是用于增量遍历集合中的元素。

   cursor：本次迭代开始的游标。 

  pattern ：本次迭代要匹配的 key 的模式。 

  count ：本次迭代要从数据集里返回多少元素，默认值为 10 。 

  type：本次迭代要返回的 value 的类型，默认为所有类型。

  说明：还有与它功能类似的命令

  * SCAN用于遍历当前数据库中的键
  * SSCAN用于遍历集合键中的元素
  * HSACN用于遍历哈希键中的键值对
  * ZSCAN用于遍历有序集合中的元素

  > 相较于keys命令的优点：
  >
  > 1. scan命令的时间复杂度虽然也是O(N)，但它是分次进行的，不会阻塞线程。
  > 2. scan命令提供了limit参数，可以控制每次返回结果的最大条数。

​		**关于游标的增长规律：把它转化成二进制可以看出来它是在高位逐步加一的，和我们平常的低位的方法是相反的，这是考虑到了数据库增容的问题。**



### 3.3 String型Value操作命名

> Redis 存储数据的 Value 可以是一个 String 类型数据。String 类型的 Value 是 Redis 中最基 本，最常见的类型。String 类型的 Value 中可以存放任意数据，包括数值型，甚至是二进制 的图片、音频、视频、序列化对象等。一个 String 类型的 Value 最大是 512M 大小

#### set

* 格式 SET key value  [EX seconds| PX miliseconds] [NX] [XX]

*  功能：SET 除了可以直接将 key 的值设为 value 外，还可以指定一些参数。 

  EX seconds：为当前 key 设置过期时间，单位秒。等价于 SETEX 命令。 

  PX milliseconds：为当前 key 设置过期时间，单位毫秒。等价于 PSETEX 命令。 

  NX：指定的 key 不存在才会设置成功，用于添加指定的 key。等价于 SETNX 命令。 

  XX：指定的 key 必须存在才会设置成功，用于更新指定 key 的 value。

* 说明：如果 value 字符串中带有空格，则该字符串需要使用双引号或单引号引起来，否 则会认为 set 命令的参数数量不正确，报错。

#### setex 与 psetex

* 格式：SETEX/PSETEX key seconds value 
* 功能：set expire，其不仅为 key 指定了 value，还为其设置了生存时间。setex 的单位为 秒，psetex 的单位为毫秒。 
* 说明：如果 key 已经存在， 则覆写旧值。该命令类似于以下两个命令，不同之处是， SETEX 是一个原子性操作，关联值和设置生存时间两个动作会在同一时间内完成，该命 令在 Redis 用作缓存时，非常实用。 SET key value EXPIRE key seconds # 设置生存时间

#### setnx

* 格式：SETNX key value 
* 功能：SET if Not eXists，将 key 的值设为 value ，当且仅当 key 不存在。若给定的 key 已经存在，则 SETNX 不做任何动作。成功，返回 1，否则，返回 0。 
* 说明：该命令等价于 set key value nx

#### getset

* 格式：GETSET key value 
* 功能：将给定 key 的值设为 value ，并返回 key 的旧值。 
* 说明：当 key 存在但不是字符串类型时，返回一个错误；当 key 不存在时，返回 nil 。

#### mset 与  msetnx

* 格式：MSET/MSETNX key value [key value ...] 
* 功能：同时设置一个或多个 key-value 对。 
* 说明：如果某个给定 key 已经存在，那么 MSET 会用新值覆盖原来的旧值，如果这不 是你所希望的效果，请考虑使用 MSETNX 命令：它只会在所有给定 key 都不存在的情 况下进行设置操作。MSET/MSETNX 是一个原子性(atomic)操作，所有给定 key 都会在同 一时间内被设置，某些给定 key 被更新而另一些给定 key 没有改变的情况不可能发生。 该命令永不失败

####  mget

* 格式：MGET key [key ...] 
* 功能：返回所有(一个或多个)给定 key 的值。 
* 说明：如果给定的 key 里面，有某个 key 不存在，那么这个 key 返回特殊值 nil 。 因此，该命令永不失败。

#### ☆append

* 格式：APPEND key value 
* 功能：如果 key 已经存在并且是一个字符串， APPEND 命令将 value 追加到 key 原 来的值的末尾。如果 key 不存在， APPEND 就简单地将给定 key 设为 value ，就像 执行 SET key value 一样。 
* 说明：追加 value 之后，返回 key 中字符串的长度。

#### incr 与 decr

* 格式：INCR key 或 DECR key 
* 功能：increment，自动递增。将 key 中存储的数字值增一。 decrement，自动递减。将 key 中存储的数字值减一。 
* 说明：如果 key 不存在，那么 key 的值会先被初始化为 0，然后再执行增一/减一操作。 如果值不能表示为数字，那么返回一个错误提示。如果执行正确，则返回增一/减一后 的值。

####  incr 与 decr

* 格式：INCR key 或 DECR key 
* 功能：increment，自动递增。将 key 中存储的数字值增一。 decrement，自动递减。将 key 中存储的数字值减一。 
* 说明：如果 key 不存在，那么 key 的值会先被初始化为 0，然后再执行增一/减一操作。 如果值不能表示为数字，那么返回一个错误提示。如果执行正确，则返回增一/减一后 的值。

####  incrbyfloat

* 格式：INCRBYFLOAT key increment 
* 功能：为 key 中所储存的值加上浮点数增量 increment 。 
* 说明：与之前的说明相同。没有 decrbyfloat 命令，但 increment 为负数可以实现减操作 效果

#### strlen 

* 格式：STRLEN key 
* 功能：返回 key 所储存的字符串值的长度。 
* 说明：当 key 储存的不是字符串值时，返回一个错误；当 key 不存在时，返回 0 。 

#### getrange 

* 格式：GETRANGE key start end 
* 功能：返回 key 中字符串值的子字符串，字符串的截取范围由 start 和 end 两个偏移 量决定，包括 start 和 end 在内。 
* 说明：end 必须要比 start 大。支持负数偏移量，表示从字符串最后开始计数，-1 表示 最后一个字符，-2 表示倒数第二个，以此类推

#### setrange 

* 格式：SETRANGE key offset value 
* 功能：用 value 参数替换给定 key 所储存的字符串值 str，从偏移量 offset 开始。 
* 说明：当 offset 值大于 str 长度时，中间使用零字节\x00 填充，即 0000 0000 字节填充； 对于不存在的 key 当作空串处理。

#### 典型应用场景

* 数据缓存

  Redis 作为数据缓存层，MySQL 作为数据存储层。应用服务器首先从 Redis 中获取数据， 如果缓存层中没有，则从 MySQL 中获取后先存入缓存层再返回给应用服务器。

* 计数器

  在Redis中写入一个value为数值型的key作为平台计数器、视频播放计数器等。每个有效客户端访问一次或视频没播放一次，都是直接修改Redis的计数器，然后再以异步方式持久化到其它数据库。类似的要多次统计的数据都像如此操作。

* 共享Session

  对于一个分布式应用系统，如果将类似用户登录信息这样的 Session 数据保存在提供登 录服务的服务器中，那么如果用户再次提交像收藏、支付等请求时可能会出现问题：在提供 收藏、支付等服务的服务器中并没有该用户的 Session 数据，从而导致该用户需要重新登录。 对于用户来说，这是不能接受的。 此时，可以将系统中所有用户的 Session 数据全部保存到 Redis 中，用户在提交新的请 求后，系统先从 Redis 中查找相应的 Session 数据，如果存在，则再进行相关操作，否则跳 转到登录页面。这样就不会引发“重新登录”问题。

* 限速器

  现在很多平台为了防止 DoS（Denial of Service，拒绝服务）攻击，一般都会限制一个 IP 不能在一秒内访问超过 n 次。而 Redis 可以可以结合 key 的过期时间与 incr 命令来完成限速 功能，充当限速器。 注意，其无法防止 DDoS（Distributed Denial of Service，分布式拒绝服务）攻击。

  ```java
  // 客户端每提交一次请求，都会执行下面的代码
  // 等价于 set 192.168.192.55 1 ex 60 nx
  // 指定新 ip 作为 key 的缓存过期时间为 60 秒
  Boolean isExists = redis.set(ip, 1, “EX 60”, “NX”);
  if(isExists != null || redis.incr(ip) <= 5) {
   // 通过
  } else {
   // 限流
  }
  ```


### 3.4 Hash 型 Value操作命令

> Redis 存储数据的Value可以是一个Hash类型。Hash类型也称为Hash表、字典等。
>
> Hsah表就是一个映射表Map，也就是键值对构成，为了与整体的key进行区别，这里的键称filed，值称为value。注意，Redis的Hash表中的field-value对均为String类型。

#### 3.4.1 hset 

* 格式：HSET key field value 
* 功能：将哈希表 key 中的域 field 的值设为 value 。 
* 说明：如果 key 不存在，一个新的哈希表被创建并进行 HSET 操作。如果域 field 已 经存在于哈希表中，旧值将被覆盖。如果 field 是哈希表中的一个新建域，并且值设置 成功，返回 1 。如果哈希表中域 field 已经存在且旧值已被新值覆盖，返回 0 。 

#### 3.4.2 hget 

* 格式：HGET key field 
* 功能：返回哈希表 key 中给定域 field 的值。 
* 说明：当给定域不存在或是给定 key 不存在时，返回 nil 。 

#### 3.4.3 hmset 

* 格式：HMSET key field value [field value ...] 
* 功能：同时将多个 field-value (域-值)对设置到哈希表 key 中。 
* 说明：此命令会覆盖哈希表中已存在的域。如果 key 不存在，一个空哈希表被创建并 执行 HMSET 操作。如果命令执行成功，返回 OK 。当 key 不是哈希表(hash)类型时， 返回一个错误。 

#### 3.4.4 hmget 

* 格式：HMGET key field [field ...] 
* 功能：按照给出顺序返回哈希表 key 中一个或多个域的值。 
* 说明：如果给定的域不存在于哈希表，那么返回一个 nil 值。因为不存在的 key 被当 作一个空哈希表来处理，所以对一个不存在的 key 进行 HMGET 操作将返回一个只带 有 nil 值的表

#### 3.4.5 hgetall 

* 格式：HGETALL key 
* 功能：返回哈希表 key 中所有的域和值。 
* 说明：在返回值里，紧跟每个域名(field name)之后是域的值(value)，所以返回值的长度 是哈希表大小的两倍。若 key 不存在，返回空列表。若 key 中包含大量元素，则该命 令可能会阻塞 Redis 服务。所以生产环境中一般不使用该命令，而使用 hscan 命令代替。 

#### 3.4.6 hsetnx 

* 格式：HSETNX key field value 
* 功能：将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在。 
* 说明：若域 field 已经存在，该操作无效。如果 key 不存在，一个新哈希表被创建并 执行 HSETNX 命令。 

#### 3.4.7 hdel 

* 格式：HDEL key field [field ...] 
* 功能：删除哈希表 key 中的一个或多个指定域，不存在的域将被忽略。 
* 说明：返回被成功移除的域的数量，不包括被忽略的域。 

#### 3.4.8 hexits 

* 格式：HEXISTS key field 
* 功能：查看哈希表 key 中给定域 field 是否存在。 
* 说明：如果哈希表含有给定域，返回 1 。如果不含有给定域，或 key 不存在，返回 0 。 

#### 3.4.9 hincrby 与 hincrbyfloat 

* 格式：HINCRBY key field increment 
* 功能：为哈希表 key 中的域 field 的值加上增量 increment 。hincrby 命令只能增加整 数值，而 hincrbyfloat 可以增加小数值。 
* 说明：增量也可以为负数，相当于对给定域进行减法操作。如果 key 不存在，一个新 的哈希表被创建并执行 HINCRBY 命令。如果域 field 不存在，那么在执行命令前，域 的值被初始化为 0。对一个储存字符串值的域 field 执行 HINCRBY 命令将造成一个错误。 

#### 3.4.10 hkeys 与 hvals 

* 格式：HKEYS key 或 HVALS key 
* 功能：返回哈希表 key 中的所有域/值。 
* 说明：当 key 不存在时，返回一个空表。 

#### 3.4.11 hlen 

* 格式：HLEN key 
* 功能：返回哈希表 key 中域的数量。 
* 说明：当 key 不存在时，返回 0 。 

#### 3.4.12 hstrlen 

* 格式：HSTRLEN key field 
* 功能：返回哈希表 key 中， 与给定域 field 相关联的值的字符串长度（string length）。 
* 说明：如果给定的键或者域不存在， 那么命令返回 0 。 

#### 3.4.13 应用场景

**Hash 型 Value 非常适合存储对象数据。key 为对象名称，value 为描述对象属性的 Map， 对对象属性的修改在 Redis 中就可直接完成。其不像 String 型 Value 存储对象，那个对象是 序列化过的，例如序列化为 JSON 串，对对象属性值的修改需要先反序列化为对象后再修改， 修改后再序列化为 JSON 串后写入到 Redis。**

### 3.5 List 型 Value 操作命令

> Redis 存储数据的Value 可以说是一个String 列表类型数据。即该列表中的每个元素均为String类型数据。列表中的数据会按照插入顺序进行排序。不过，**该列表的底层实际是一个无头结点的双向列表，所以对列表表头与表尾的操作性能较高，但对中间元素的插入与删除的操作的性能较差。**



#### 3.5.1 lpush/rpush 

* 格式：LPUSH key value [value ...] 或 RPUSH key value [value ...] 
* 功能：将一个或多个值 value 插入到列表 key 的表头/表尾（表头在左表尾在右） 
* 说明：如果有多个 value 值，对于 lpush 来说，各个 value 会按从左到右的顺序依次插 入到表头；对于 rpush 来说，各个 value 会按从左到右的顺序依次插入到表尾。如果 key 不存在，一个空列表会被创建并执行操作。当 key 存在但不是列表类型时，返回一个 错误。执行成功时返回列表的长度。 

#### 3.5.2 llen 

格式：LLEN key  功能：返回列表 key 的长度。  说明：如果 key 不存在，则 key 被解释为一个空列表，返回 0 。如果 key 不是列表 类型，返回一个错误。 

#### 3.5.3 lindex 

* 格式：LINDEX key index 
* 功能：返回列表 key 中，下标为 index 的元素。列表从 0 开始计数。 
* 说明：如果 index 参数的值不在列表的区间范围内(out of range)，返回 nil 。 

#### 3.5.4 lset 

* 格式：LSET key index value 
* 功能：将列表 key 下标为 index 的元素的值设置为 value 。 
* 说明：当 index 参数超出范围，或对一个空列表（key 不存在）进行 LSET 时，返回一 个错误。 

#### 3.5.5 lrange 

* 格式：LRANGE key start stop 
* 功能：返回列表 key 中指定区间[start, stop]内的元素，即包含两个端点。 
* 说明：List 的下标从 0 开始，即以 0 表示列表的第一个元素，以 1 表示列表的第二个 元素，以此类推。也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列 表的倒数第二个元素，以此类推。超出范围的下标值不会引起错误。如果 start 下标比 列表的最大下标 还要大，那么 LRANGE 返回一个空列表。如果 stop 下标比最大下标 还要大，Redis 将 stop 的值设置为最大下标。 

#### 3.5.6 lpushx 与 rpushx 

* 格式：LPUSHX key value 或 RPUSHX key value 
* 功能：将值 value 插入到列表 key 的表头/表尾，当且仅当 key 存在并且是一个列表。 
* 说明：当 key 不存在时，命令什么也不做。若执行成功，则输出表的长度。 

#### 3.5.7 linsert 

* 格式：LINSERT key BEFORE|AFTER pivot value
* 功能：将值 value 插入到列表 key 当中，位于元素 pivot 之前或之后。 
* 说明：当 pivot 元素不存在于列表中时，不执行任何操作，返回-1；当 key 不存在时， key 被视为空列表，不执行任何操作，返回 0；如果 key 不是列表类型，返回一个错误； 如果命令执行成功，返回插入操作完成之后，列表的长度。 

#### 3.5.8 lpop / rpop 

* 格式：LPOP key [count] 或 RPOP key [count] 
* 功能：从列表 key 的表头/表尾移除 count 个元素，并返回移除的元素。count 默认值 1 
* 说明：当 key 不存在时，返回 nil 

#### 3.5.9 blpop / brpop 

* 格式：BLPOP key [key ...] timeout 或 BRPOP key [key ...] timeout 
* 功能：BLPOP/BRPOP 是列表的阻塞式(blocking)弹出命令。它们是 LPOP/RPOP 命令的阻 塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被 BLPOP/BRPOP 命令阻 塞，直到等待 timeout 超时或发现可弹出元素为止。当给定多个 key 参数时，按参数 key 的先后顺序依次检查各个列表，弹出第一个非空列表的头元素。timeout 为阻塞时长， 单位为秒，其值若为 0，则表示只要没有可弹出元素，则一直阻塞。 
* 说明：假如在指定时间内没有任何元素被弹出，则返回一个 nil 和等待时长。反之，返 回一个含有两个元素的列表，第一个元素是被弹出元素所属的 key ，第二个元素是被 弹出元素的值。 

#### 3.5.10 rpoplpush 

* 格式：RPOPLPUSH source destination 
* 功能：命令 RPOPLPUSH 在一个原子时间内，执行以下两个动作： 
  * 将列表 source 中的最后一个元素(尾元素)弹出，并返回给客户端。 
  * 将 source 弹出的元素插入到列表 destination ，作为 destination 列表的的头元素。 如果 source 不存在，值 nil 被返回，并且不执行其他动作。如果 source 和 destination 相同，则列表中的表尾元素被移动到表头，并返回该元素，可以把这种特殊情况视作列 表的旋转(rotation)操作。 

#### 3.5.11 brpoplpush 

* 格式：BRPOPLPUSH source destination timeout 
* 功能：BRPOPLPUSH 是 RPOPLPUSH 的阻塞版本，当给定列表 source 不为空时， BRPOPLPUSH 的表现和 RPOPLPUSH 一样。当列表 source 为空时， BRPOPLPUSH 命令 将阻塞连接，直到等待超时，或有另一个客户端对 source 执行 LPUSH 或 RPUSH 命令 为止。timeout 为阻塞时长，单位为秒，其值若为 0，则表示只要没有可弹出元素，则 一直阻塞。
* 说明：假如在指定时间内没有任何元素被弹出，则返回一个 nil 和等待时长。反之，返 回一个含有两个元素的列表，第一个元素是被弹出元素的值，第二个元素是等待时长。 

#### 3.5.12 lrem 

* 格式：LREM key count value 
* 功能：根据参数 count 的值，移除列表中与参数 value 相等的元素。count 的值可以 是以下几种： 
  * count > 0 : 从表头开始向表尾搜索，移除与 value 相等的元素，数量为 count 。 
  * count < 0 : 从表尾开始向表头搜索，移除与 value 相等的元素，数量为 count 的 绝对值。
  * count = 0 : 移除表中所有与 value 相等的值。 

* 说明：返回被移除元素的数量。当 key 不存在时， LREM 命令返回 0 ，因为不存在 的 key 被视作空表(empty list)。 

#### 3.5.13 ltrim 

* 格式：LTRIM key start stop 
* 功能：对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指 定区间之内的元素都将被删除。 
* 说明：下标(index)参数 start 和 stop 都以 0 为底，也就是说，以 0 表示列表的第一 个元素，以 1 表示列表的第二个元素，以此类推。也可以使用负数下标，以 -1 表示 列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。当 key 不是列表 类型时，返回一个错误。如果 start 下标比列表的最大下标 end ( LLEN list 减去 1 )还要 大，或者 start > stop ， LTRIM 返回一个空列表，因为 LTRIM 已经将整个列表清空。 如果 stop 下标比 end 下标还要大，Redis 将 stop 的值设置为 end 。 

#### 3.5.14 应用场景 

Value 为 List 类型的应用场景很多，主要是通过构建不同的数据结构来实现相应的业务 功能。这里仅对这些数据结构的实现方式进行总结，不举具体的例子。 

（1） 栈 

通过 lpush + lpop 可以实现栈数据结构效果：先进后出。通过 lpush 从列表左侧插入数 据，通过 lpop 从列表左侧取出数据。当然，通过 rpush + rpop 也可以实现相同效果，只不过 操作的是列表右侧 

（2） 队列 

通过 lpush + rpop 可以实现队列数据结构效果：先进先出。通过 lpush 从列表左侧插入 数据，通过 rpop 从列表右侧取出数据。当然，通过 rpush + lpop 也可以实现相同效果，只不 过操作的方向正好相反。 

（3） 阻塞式消息队列 

通过 lpush + brpop 可以实现阻塞式消息队列效果。作为消息生产者的客户端使用 lpush 从列表左侧插入数据，作为消息消费者的多个客户端使用 brpop 阻塞式“抢占”列表尾部数 据进行消费，保证了消费的负载均衡与高可用性。brpop 的 timeout 设置为 0，表示只要没 有数据可弹出，就永久阻塞。 

（4） 动态有限集合 

通过 lpush + ltrim 可以实现有限集合。通过 lpush 从列表左侧向列表中添加数据，通过 ltrim 保持集合的动态有限性。像企业的末位淘汰、学校的重点班等动态管理，都可通过这 种动态有限集合来实现。当然，通过 rpush + ltrim 也可以实现相同效果，只不过操作的方向 正好相反。



### 3.6 Set型Value操作命令

> Redis存储数据的Value可以是一个Set集合，且集合中的每一个元素均String类型。Set与List非常相似，但不同之处是Set中的元素具有无序性与不可重复性，而List则是具有有序和可重复性。
>
> Redis中的Set集合与java中的Set集合的实现相似，其底层都是value 为null的hash表。正因如此，才会引发无序性与不可重复性。

#### 3.6.1 sadd 

* 格式：SADD key member [member ...]  功能：将一个或多个 member 元素加入到集合 key 当中，已经存在于集合的 member  元素将被忽略。  说明：假如 key 不存在，则创建一个只包含 member 元素作成员的集合。当 key 不 是集合类型时，返回一个错误。 

#### 3.6.2 smembers 

* 格式：SMEMBERS key  功能：返回集合 key 中的所有成员。  说明：不存在的 key 被视为空集合。若 key 中包含大量元素，则该命令可能会阻塞 Redis 服务。所以生产环境中一般不使用该命令，而使用 sscan 命令代替。 

#### 3.6.3 scard 

* 格式：SCARD key  功能：返回 Set 集合的长度  说明：当 key 不存在时，返回 0 。 

#### 3.6.4 sismember 

* 格式：SISMEMBER key member  功能：判断 member 元素是否集合 key 的成员。  说明：如果 member 元素是集合的成员，返回 1 。如果 member 元素不是集合的成 员，或 key 不存在，返回 0 。 

#### 3.6.5 smove 

* 格式：SMOVE source destination member  功能：将 member 元素从 source 集合移动到 destination 集合。  说明：如果 source 集合不存在或不包含指定的 member 元素，则 SMOVE 命令不执 行任何操作，仅返回 0 。否则， member 元素从 source 集合中被移除，并添加到 destination 集合中去，返回 1。当 destination 集合已经包含 member 元素时， SMOVE 命令只是简单地将 source 集合中的 member 元素删除。当 source 或 destination 不 是集合类型时，返回一个错误。 

#### 3.6.6 srem 

* 格式：SREM key member [member ...] 
* 功能：移除集合 key 中的一个或多个 member 元素，不存在的 member 元素会被忽 略，且返回成功移除的元素个数。 
* 说明：当 key 不是集合类型，返回一个错误。 

#### 3.6.7 srandmember 

* 格式：SRANDMEMBER key [count] 
* 功能：返回集合中的 count 个随机元素。count 默认值为 1。 
* 说明：若 count 为正数，且小于集合长度，那么返回一个包含 count 个元素的数组， 数组中的元素各不相同。如果 count 大于等于集合长度，那么返回整个集合。如果  count 为负数，那么返回一个包含 count 绝对值个元素的数组，但数组中的元素可能 会出现重复。 

#### 3.6.8 spop 

* 格式：SPOP key [count] 
* 功能：移除并返回集合中的 count 个随机元素。count 必须为正数，且默认值为 1。
* 说明：如果 count 大于等于集合长度，那么移除并返回整个集合。 

#### 3.6.9 sdiff / sdiffstore 

* 格式：SDIFF key [key ...] 或 SDIFFSTORE destination key [key ...] 
* 功能：返回第一个集合与其它集合之间的差集。差集，difference。 
* 说明：这两个命令的不同之处在于，sdiffstore 不仅能够显示差集，还能将差集存储到指 定的集合 destination 中。如果 destination 集合已经存在，则将其覆盖。不存在的 key 被 视为空集。 

#### 3.6.10 sinter / sinterstore 

* 格式：SINTER key [key ...] 或 SINTERSTORE destination key [key ...] 
* 功能：返回多个集合间的交集。交集，intersection。 
* 说明：这两个命令的不同之处在于，sinterstore 不仅能够显示交集，还能将交集存储到 指定的集合 destination 中。如果 destination 集合已经存在，则将其覆盖。不存在的 key  被视为空集。 

#### 3.6.11 sunion / sunionstore 

* 格式：SUNION key [key ...] 或 SUNIONSTORE destination key [key ...] 
* 功能：返回多个集合间的并集。并集，union。 
* 说明：这两个命令的不同之处在于，sunionstore 不仅能够显示并集，还能将并集存储到 指定的集合 destination 中。如果 destination 集合已经存在，则将其覆盖。不存在的 key  被视为空集。 

#### 3.6.12 应用场景 

Value 为 Set 类型的应用场景很多，这里对这些场景仅进行总结。 

（1） 动态黑白名单 

例如某服务器中要设置用于访问控制的黑名单。如果直接将黑名单写入服务器的配置文 件，那么存在的问题是，无法动态修改黑名单。此时可以将黑名单直接写入 Redis，只要有 客户端来访问服务器，服务器在获取到客户端 IP后先从 Redis的黑名单中查看是否存在该 IP， 如果存在，则拒绝访问，否则访问通过。

（2） 有限随机数 

有限随机数是指返回的随机数是基于某一集合范围内的随机数，例如抽奖、随机选人。 通过 spop 或 srandmember 可以实现从指定集合中随机选出元素。 

（3） 用户画像 

社交平台、电商平台等各种需要用户注册登录的平台，会根据用户提供的资料与用户使 用习惯，为每个用户进行画像，即为每个用户定义很多可以反映该用户特征的标签，这些标 签就可以使用 sadd 添加到该用户对应的集合中。这些标签具有无序、不重复特征。 同时平台还可以使用 sinter/sinterstore 根据用户画像间的交集进行好友推荐、商品推荐、 客户推荐等。



### 3.7 有序Set型Value操作命令

> Redis 存储数据的Value 可以是一个有序Set，这个有序Set中的每个元素均为String类型。、
>
> 有序Set与Set的不同之处是，有序Set中的每一个元素都有一个分值score，Redis会根据score的值对集合进行有小到大的排序。其与Set集合要求相同，元素不能重复，但元素的score可以重复。由于该类型的所有命令均是字母z开头，所以该Set也称为ZSet。

。。。

### 3.8 benchmark 测试工具

#### 3.8.1 简介

在Redis安装完毕后会自动安装一个`redis-benchmark`测试工具，用于测试Redis的命令的性能。

可以通过`redis-benchmark -help`命令可以查看到其用法。

#### 3.8.2 测试 1 

```
redis-benchmakrk -h 127.0.0.1 -p 6379 -c 100 -n 100000 -d 8
```

（1） 命令解析 以上命令中选项的意义： 

* -h：指定要测试的 Redis 的 IP，若为本机，则可省略 

* -p：指定要测试的 Redis 的 port，若为 6379，则可省略 

* -c：指定模拟有客户端的数量，默认值为 50 

* -n：指定这些客户端发出的请求的总量，默认值为 100000 

* -d：指定测试 get/set 命令时其操作的 value 的数据长度，单位字节，默认值为 3。在测 试其它命令时该指定没有用处。 

  以上命令的意义是，使用 100 个客户端连接该 Redis，这些客户端总共会发起 100000 个请求，set/get 的 value 为 8 字节数据。

  

（2） 测试结果分析 该命令会逐个测试所有 Redis 命令，每个命令都会给出一份测试报告，每个测试报告由 四部分构成：

测试环境报告，延迟百分比分布（这是按照百分比进行的统计报告：每完成一次剩余测试量的 50%就给出一个统计数据），延迟的累积分布，总述报告。

#### 3.8.3 测试 2

```
redis-benchmark -t set,lpush,sadd -c 100 -n 100000 -q
```

* -t 指定要测试的命名，用逗号分隔，不能有空格
* -q 指定仅给出总述报告

### 3.9 简单动态字符串 SDS

> 无论是 Redis 的 Key 还是 Value，其基础数据类型都是字符串。例如，Hash 型 Value 的 field 与 value 的类型、List 型、Set 型、ZSet 型 Value 的元素的类型等都是字符串。虽然 Redis 是使用标准 C 语言开发的，但并没有直接使用 C 语言中传统的字符串表示，而是自定义了一 种字符串。这种字符串本身的结构比较简单，但功能却非常强大，称为简单动态字符串， Simple Dynamic String，简称 SDS。 
>
> **注意，Redis 中的所有字符串并不都是 SDS，也会出现 C 字符串。C 字符串只会出现在字符串“字面常量”中，并且该字符串不可能发生变更。 **
>
> 例如：返回结果、redisLog(REDIS_WARNNING, “sdfsfsafsafds”)

**可以用命令查看数据在内存中的存储形式,即底层的实现：**

```
object encoding <key>
//会返回"embastr",这个就表示用的是SDS存储形式
```

#### SDS的结构

SDS 不同于 C 字符串。C 字符串本身是一个以双引号括起来，以空字符’\0’结尾的字符序 列。但 SDS 是一个结构体，定义在 Redis 安装目录下的 src/sds.h 中：

```c
struct sdshdr {
// 字节数组，用于保存字符串
char buf[];
// buf[]中已使用字节数量，称为 SDS 的长度
int len;
// buf[]中尚未使用的字节数量
int free;
}
```

例如执行 SET country “China”命令时，键 country 与值”China”都是 SDS 类型的，只不过 一个是 SDS 的变量，一个是 SDS 的字面常量。”China”在内存中的结构如下：

![](image\屏幕截图 2023-09-14 221134.png)

通过以上结构可以看出，**SDS 的 buf 值实际是一个 C 字符串，包含空字符’\0’共占 6 个字 节。但 SDS 的 len 是不包含空字符’\0’的。**

#### SDS 的优势

> C 字符串使用 Len+1 长度的字符数组来表示实际长度为 Len 的字符串，字符数组最后以 空字符’\0’结尾，表示字符串结束。这种结构简单，但不能满足 Redis 对字符串功能性、安全 性及高效性等的要求。

* **（1）防止“字符串长度获取”性能瓶颈**

  对于 C 字符串，若要获取其长度，则必须要通过遍历整个字符串才可获取到的。对于超长字符串的遍历，会成为系统的性能瓶颈。

   但，由于 SDS 结构体中直接就存放着字符串的长度数据，所以对于获取字符串长度需要 消耗的系统性能，与字符串本身长度是无关的，不会成为 Redis 的性能瓶颈

* **（2）保障二进制安全**

  C 字符串中只能包含符合某种编码格式的字符，例如 ASCII、UTF-8 等，并且除了字符串 末尾外，其它位置是不能包含空字符’\0’的，否则该字符串就会被程序误解为提前结束。而 在图片、音频、视频、压缩文件、office 文件等二进制数据中以空字符’\0’作为分隔符的情况 是很常见的。故而在 C 字符串中是不能保存像图片、音频、视频、压缩文件、office 文件等 二进制数据的。 

  但 SDS 不是以空字符’\0’作为字符串结束标志的，其是通过 len 属性来判断字符串是否 结束的。所以，对于程序处理 SDS 中的字符串数据，无需对数据做任何限制、过滤、假设， 只需读取即可。数据写入的是什么，读到的就是什么。

* **（3）减少内存再分配的次数**

  SDS 采用了**空间预分配策略**与**惰性空间释放策略**来避免内存再分配问题。 

  空间预分配策略是指，每次 SDS 进行空间扩展时，程序不但为其分配所需的空间，还会 为其分配额外的未使用空间，以减少内存再分配次数。而额外分配的未使用空间大小取决于 空间扩展后 SDS 的 len 属性值。 

  * 如果 len 属性值小于 1M，那么分配的未使用空间 free 的大小与 len 属性值相同。 
  * 如果 len 属性值大于等于 1M ，那么分配的未使用空间 free 的大小固定是 1M。 

  SDS 对于空间释放采用的是惰性空间释放策略。该策略是指，SDS 字符串长度如果缩短， 那么多出的未使用空间将暂时不释放，而是增加到 free 中。以使后期扩展 SDS 时减少内存 再分配次数。 如果要释放 SDS 的未使用空间，则可通过 `sdsRemoveFreeSpace()`函数来释放。

* **（4）兼容C函数**

  Redis 中提供了很多的 SDS 的 API，以方便用户对 Redis 进行二次开发。为了能够兼容 C 函数，SDS 的底层数组 buf[]中的字符串仍以空字符’\0’结尾。 

  现在要比较的双方，一个是 SDS，一个是 C 字符串，此时可以通过 C 语言函数 strcmp(sds_str->buf，c_str)

  Redis还提供有供二次开发的SDS的操作函数。

### 3.10 集合的底层实现原理

> Redis 中对于 Set 类型的底层实现，直接采用了 hashTable。但对于 Hash、ZSet、List 集 合的底层实现进行了特殊的设计，使其保证了 Redis 的高性能。

#### 两种实现的选择

> 对于Hash与ZSet集合，其底层的实现实际有两种：压缩列表zipList，与跳跃列表skipList。这两种实现对于用户来说是透明的，但是用户写入不同的数据，系统会自动使用不同的实现。
>
> 只有同时满足以下配置文件redis.conf中相关集合元素数量阈值与元素大小阈值两个条件，使用的就是压缩列表zipList，否则就是用skipList。例如，对于ZSet集合中这两个条件下：
>
> * 集合元素个数小于redis.conf中的zset-max-ziplist-ventries属性的值，默认为128
> * 每个集合元素大小都小于redis.conf中zset-max-ziplist-value属性的值，默认为64字节



#### zipList

![](image\屏幕截图 2023-09-19 193036.png)

**（1）什么是zipList**

> zipList，通常称为压缩列表，是一个经过特殊编码的用于存储字符串或整数的双向链表。 其底层数据结构由三部分构成：head、entries 与 end。这三部分在内存上是连续存放的。

**（2）head**

head又由三部分构成：

* **zlbytes：**占 4 个字节，用于存放zipList列表整体数据结构所占的字节数，包括zlbytes本身的长度
* **zltail：**占 4 个字节，用于存放zipList中最后一个entry在整个数据结构中的偏移量（字节）。可以快速定位列表的尾entry位置，以方便操作。
* **zllen：**占 2 字节，用于存放列表包含的entry 个数。其只有16位，所以zipList最多可以含有entry的个数为2^16 - 1 = 65535 个

**（3）entries**

​	entries 是真正的列表，由很多的列表元素 entry 构成。由于不同的元素类型、数值的不 同，从而导致每个 entry 的长度不同。 

每个 entry 由三部分构成： 

* **prevlength：**该部分用于记录上一个 entry 的长度，以实现逆序遍历。默认长度为 1 字节， 只要上一个 entry 的长度<254 字节，prevlength 就占 1 字节，否则其会自动扩展为 3 字 节长度。 
* ***encoding：**该部分用于标志后面的 data 的具体类型。如果 data 为整数类型，encoding固定长度为 1 字节。如果 data 为字符串类型，则 encoding 长度可能会是 1 字节、2 字 节或 5 字节。data 字符串不同的长度，对应着不同的 encoding 长度。 
* **data：**真正存储的数据。数据类型只能是整数类型或字符串类型。不同的数据占用的字 节长度不同。 

**（4） **end 

end 只包含一部分，称为 zlend。占 1 个字节，值固定为 255，即二进制位为全 1，表示 一个 zipList 列表的结束。



#### listPack

> **对于ziplist，实现复杂，为了逆序遍历，每个entry中包含前一个entry的长度这样会导致在ziplist中间修改或插入entry时，需要进行级联更新。在高并发的写操作场景下会极度降低Redis的性能，为了实现更紧凑、更快的解析，更简单的实现，重写实现了ziplist，并命名为listPack**
>
> 在Redis7.0中，已将zipList全部替换成立listPack，但还是保留了zipList相关属性

![](image\屏幕截图 2023-09-20 104256.png)

**（1）listPack底层实现**

listPack 也是一个经过特殊编码的用于存储字符串或整数的双向链表。其底层数据结构 也由三部分构成：head、entries 与 end，且这三部分在内存上也是连续存放的。 

listPack与zipList的重大区别在head与每个entry的结构上，表示列表结束的end与zipList 的 zlend 是相同的，占一个字节，且 8 位全为 1。

**（2）head**

head 由两部分构成： 

* **totalBytes：**占 4 个字节，用于存放 listPack 列表整体数据结构所占的字节数，包括 totalBytes 本身的长度。 
* **elemNum：**占 2 字节，用于存放列表包含的 entry 个数。其意义与 zipList 中 zllen 的相同。 与 zipList 的 head 相比，没有了记录最后一个 entry 偏移量的 zltail。

**（3）entries**

entries 也是 listPack 中真正的列表，由很多的列表元素 entry 构成。由于不同的元素类 型、数值的不同，从而导致每个 entry 的长度不同。但与 zipList 的 entry 结构相比，listPack 的 entry 结构发生了较大变化。 其中最大的变化就是没有了记录前一个 entry 长度的 prevlength，而增加了记录当前 entry 长度的 element-total-len。

而这个改变仍然可以实现逆序遍历，但却避免了由于在列表 中间修改或插入 entry 时引发的级联更新。 

每个 entry 仍由三部分构成： 

* **encoding：**该部分用于标志后面的 data 的具体类型。如果 data 为整数类型，encoding 长度可能会是 1、2、3、4、5 或 9 字节。不同的字节长度，其标识位不同。如果 data 为字符串类型，则 encoding 长度可能会是 1、2 或 5 字节。data 字符串不同的长度，对 应着不同的 encoding 长度。 
* **data：**真正存储的数据。数据类型只能是整数类型或字符串类型。不同的数据占用的字 节长度不同。 
* **element-total-len：**该部分用于记录当前 entry 的长度，用于实现逆序遍历。由于其特殊 的记录方式，使其本身占有的字节数据可能会是 1、2、3、4 或 5 字节。



#### ☆skipList

**（1）什么是skipList**

​	skipList，跳跃列表，简称跳表，是一种**随机化**的数据结构，基于并联的链表，实现简单，查找效率较高。简单来说跳表也是链表的一种，不过在查找元素时，有较高的效率。

**（2）skipList原理**

对于一个有序链表，如果要查找某个数据，需要从头开始查找，直到查到对应的元素，这样的查找效率较低，相同的，要插入数据时效率也不够高。

**这时为了提高查询效率，在偶数节点上增加一个指针，让其指向下一个偶数节点。**

![](image\屏幕截图 2023-09-20 105357.png)

这样所有偶数节点又构成了一个新的链表，**即高层链表**。

此时想要查找或插入某个元素时，先在高层链表查找元素，找到目标节点的前一个较大节点，然后再回到原链表中查找。

如果还想要提高查找效率，可以再构建高层链表。

**但是此时存在的问题是：由于固定序号的元素拥有固定层级，所以在元素增删的情况下，会导致整体元素层级大调整，然后重新进行层级划分，会大大降低系统性能。**

为了优化算法，避免前面的问题，实际上skipList采用了**随机分配层级方式**。

即在确定了总层级后，每添加一个新的元素时，会自动为其随机分配的一个层级。这种随机性就解决了节点序号和层级间的固定关系的问题。

> skipList 指的就是除了最下面第 1 层链表之外，它会产生若干层稀疏的链表，这些链表 里面的指针跳过了一些节点，并且越高层级的链表跳过的节点越多。在查找数据的时先在高 层级链表中进行查找，然后逐层降低，最终可能会降到第 1 层链表来精确地确定数据位置。 在这个过程中由于跳过了一些节点，从而加快了查找速度。



#### quickList

![](image\屏幕截图 2023-09-20 110453.png)

**（1）什么是quickList？**

> 快速列表，本身是一个双向无循环链表，它的**每一个节点都是一个zipList。**从redis3.2版本开始，对于List的底层实现，使用quickList替代了zipLIst和linkedList。
>
> quickList 本质上是 zipList 和 linkedList 的混合体。其将 linkedList 按段切分，每一段使 用 zipList 来紧凑存储若干真正的数据元素，多个 zipList 之间使用双向指针串接起来。当然， 对于每个 zipList 中最多可存放多大容量的数据元素，在配置文件中通过 list-max-ziplist-size 属性可以指定。



**（2）检索操作**

为了更深入的理解 quickList 的工作原理，通过对检索、插入、删除等操作的实现分析来 加深理解。

 对于 List 元素的检索，都是以其索引 index 为依据的。quickList 由一个个的 zipList 构成， 每个 zipList 的 zllen 中记录的就是当前 zipList 中包含的 entry 的个数，即包含的真正数据元素 的个数。**根据要检索元素的 index，从 quickList 的头节点开始，逐个对 zipList 的 zllen 做 sum 求和，直到找到第一个求和后 sum 大于 index 的 zipList，那么要检索的这个元素就在这个 zipList 中。**

**（3）插入操作**

​	由于 zipList 是有大小限制的，所以在 quickList 中插入一个元素在逻辑上相对就比较复杂 一些。假设要插入的元素的大小为 insertBytes，而查找到的插入位置所在的 zipList 当前的大 小为 zlBytes，那么具体可分为下面几种情况： 

* 情况一：当 insertBytes + zlBytes <= list-max-ziplist-size 时，直接插入到 zipList 中相应位置 即可 

* 情况二：当 insertBytes + zlBytes > list-max-ziplist-size，且元素将要插入的位置位于该 zipList 的首部位置，此时需要查看该 zipList 的前一个 zipList 的大小 prev_zlBytes。 

  * 若 insertBytes + prev_zlBytes<= list-max-ziplist-size 时，直接将元素插入到前一个 zipList 的尾部位置即可 

  * 若 insertBytes + prev_zlBytes> list-max-ziplist-size 时，直接将元素自己构建为一个新 的 zipList，并连入 quickList 中 

* 情况三：当 insertBytes + zlBytes > list-max-ziplist-size，且插入的位置位于该 zipList 的尾部位置，此时需要查看该 zipList 的后一个 zipList 的大小 next_zlBytes。 

  * 若 insertBytes + next_zlBytes<= list-max-ziplist-size 时，直接将元素插入到后一个 zipList 的头部位置即可 
  * 若 insertBytes + next_zlBytes> list-max-ziplist-size 时，直接将元素自己构建为一个新 的 zipList，并连入 quickList 中 

* 情况四：当 insertBytes + zlBytes > list-max-ziplist-size，且插入的位置位于该 zipList 的中间位置，则将当前 zipList 分割为两个 zipList 连接入 quickList 中，然后将元素插入到分割后的前面 zipList 的尾部位置

**（4）删除操作**

​	对于删除操作，只需要注意一点，在相应的 zipList 中删除元素后，该 zipList 中是否还有 元素。如果没有其它元素了，则将该 zipList 删除，将其前后两个 zipList 相连接



#### key 与 value 中元素的数量

> 前面讲述的 Redis 的各种特殊数据结构的设计，不仅极大提升了 Redis 的性能，并且还 使得 Redis 可以支持的 key 的数量、集合 value 中可以支持的元素数量可以非常庞大。 
>
> * Redis 最多可以处理 2^32个 key（约 42 亿），并且在实践中经过测试，每个 Redis 实例至 少可以处理 2.5 亿个 key。 
> * 每个 Hash、List、Set、ZSet 集合都可以包含 2^32 个元素。



### 3.12 HyperLogLog 操作命令

> HyperLogLog 是 Redis 2.8.9 版本中引入的一种新的数据类型，其意义是 hyperlog log，超 级日志记录。该数据类型可以简单理解为一个 set 集合，集合元素为字符串。但实际上 HyperLogLog 是一种基数计数概率算法，通过该算法可以利用极小的内存完成独立总数的统 计。其所有相关命令都是对这个“set 集合”的操作。 
>
> HyperLogLog 算法是由法国人 Philippe Flajolet 博士研究出来的，Redis 的作者 Antirez 为了纪念 Philippe Flajolet 博士对组合数学和基数计算算 法分析的研究，在设计 HyperLogLog 命令的时候使用了 Philippe Flajolet 姓名的英文首字母 PF 作为前缀。遗憾的是 Philippe Flajolet 博士于 2011 年 3 月 22 日因病在巴黎辞世。 HyperLogLog 算法是一个纯数学算法，我们这里不做研究



### 3.13 BitMap 操作命令

> BitMap 是 Redis 2.2.0 版本中引入的一种新的数据类型。该数据类型本质上就是一个仅 包含 0 和 1 的二进制字符串。而其所有相关命令都是对这个字符串二进制位的操作。用于描 述该字符串的属性有三个：key、offset、bitValue。 
>
> * key：BitMap 是 Redis 的 key-value 中的一种 Value 的数据类型，所以该 Value 一定有其对 应的 key。 
> * offset：每个 BitMap 数据都是一个字符串，字符串中的每个字符都有其对应的索引，该 索引从 0 开始计数。该索引就称为每个字符在该 BitMap 中的偏移量 offset。这个 offset 的值的范围是[0，2 32 -1]，即该 offset 的最大值为 4G-1，即 4294967295，42 亿多。 
> * bitValue：每个 BitMap 数据中都是一个仅包含 0 和 1 的二进制字符串，每个 offset 位上 的字符就称为该位的值 bitValue。bitValue 的值非 0 即 1。



#### 应用场景

​	由于 offset 的取值范围很大，所以其一般应用于大数据量的二值性统计。例如平台活跃 用户统计（二值：访问或未访问）、支持率统计（二值：支持或不支持）、员工考勤统计（二 值：上班或未上班）、图像二值化（二值：黑或白）等。 

​	不过，对于数据量较小的二值性统计并不适合 BitMap，可能使用 Set 更为合适。当然， 具体多少数据量适合使用 Set，超过多少数据量适合使用 BitMap，这需要根据具体场景进行 具体分析。 

​	例如，一个平台要统计日活跃用户数量。 如果使用 Set 来统计，只需上线一个用户，就将其用户 ID 写入 Set 集合即可，最后只需 统计出 Set 集合中的元素个数即可完成统计。即 Set 集合占用内存的大小与上线用户数量成 正比。假设用户 ID 为 m 位 bit 位，当前活跃用户数量为 n，则该 Set 集合的大小最少应该是 m*n 字节。

​	如果使用 BitMap 来统计，则需要先定义出一个 BitMap，其占有的 bit 位至少为注册用 户数量。只需上线一个用户，就立即使其中一个 bit 位置 1，最后只需统计出 BitMap 中 1 的 个数即可完成统计。即 BitMap 占用内存的大小与注册用户数量成正比，与上线用户数量无 关。假设平台具有注册用户数量为 N，则 BitMap 的长度至少为 N 个 bit 位，即 N/8 字节。 

​	何时使用 BitMap 更合适？令 m*n 字节 = N/8 字节，即 n = N/8/m = N/(8*m) 时，使用 Set 集合与使用 BitMap 所占内存大小相同。以淘宝为例，其用户 ID 长度为 11 位(m)，其注 册用户数量为 8 亿(N)，当活跃用户数量为 8 亿/(8*11) = 0.09 亿 = 9*106 = 900 万，使用 Set 与 BitMap 占用的内存是相等的。但淘宝的日均活跃用户数量为 8 千万，所以淘宝使用 BitMap 更合适。

### 3.14 Geospatial 简介









### 3.15 发布 / 订阅命令

> 发布/订阅，即 pub/sub，是一种消息通信模式：发布者也称为消息生产者，生产和发送 消息到存储系统；订阅者也称为消息消费者，从存储系统接收和消费消息。这个存储系统可 以是文件系统 FS、消息中间件 MQ、数据管理系统 DBMS，也可以是 Redis。整个消息发布者、 订阅者与存储系统称为消息系统。 
>
> 消息系统中的订阅者订阅了某类消息后，只要存储系统中存在该类消息，其就可不断的 接收并消费这些消息。当存储系统中没有该消息后，订阅者的接收、消费阻塞。而当发布者 将消息写入到存储系统后，会立即唤醒订阅者。当存储系统放满时，不同的发布者具有不同 的处理方式：有的会阻塞发布者的发布，等待可用的存储空间；有的则会将多余的消息丢失。 
>
> 当然，不同的消息系统消息的发布/订阅方式也是不同的。例如 RocketMQ、Kafka 等消 息中间件构成的消息系统中，发布/订阅的消息都是以主题 Topic 分类的。而 Redis 构成的消 息系统中，发布/订阅的消息都是以频道 Channel 分类的。

#### subscribe

* 格式：SUBSCRIBE channel [channel ...]
* 功能：Redis 客户端通过一个subscribe命令可以同时订阅任意数量的频道。在输出了订阅了主题后，命令处于阻塞状态，等待相关频道的消息。

#### psubscribe

* 格式：PSUBSCRIBE pattern [pattern ...]
* 功能：订阅一个或多个符合给定模式的频道
* 说明：这里的模式只能使用通配符 *。

#### publish

* 格式：PUBLISH channel message
* 功能：Redis 客户端通过一条publish 命令可以发布一个频道的消息。返回值为接收到该消息的订阅者数量。

#### uusubscribe

* 格式：UNSUBSCRIBE [channel [channel] ...]
* 功能：Redis 客户端退订指定的频道，如果没有指定，则会退订所有频道，会返回信息告知客户端退订的频道

#### punsubscribe

* 格式：PUNSUBSCRIBE [pattern [pattern ...]]
* 功能：退订一个或多个符合给定模式的频道
* 说明：只能用通配符*

#### pubsub 

* 格式：PUBSUB  [argument *argument …]]
* 功能：PUBSUB 是一个查看订阅与发布系统状态的内省命令集，它由数个不同格式的子 命令组成，下面分别介绍这些子命令的用法。

#### （1） pubsub channels 

* 格式：PUBSUB CHANNELS [pattern] 
* 功能：列出当前所有的活跃频道。活跃频道指的是那些至少有一个订阅者的频道。 
* 说明：pattern 参数是可选的。如果不给出 pattern 参数，将会列出订阅/发布系统中的 所有活跃频道。如果给出 pattern 参数，那么只列出和给定模式 pattern 相匹配的那些活 跃频道。pattern 中只能使用通配符*。

#### （2） pubsub numsub 

* 格式：PUBSUB NUMSUB [channel-1 … channel-N] 
* 功能：返回给定频道的订阅者数量。不给定任何频道则返回一个空列表。 

#### （3） pubsub numpat 

* 格式：PUBSUB NUMPAT 
* 功能：查询当前 Redis 所有客户端订阅的所有频道模式的数量总和



### 3.16 Redis 事务

​	Redis 的事务本质是一组命令的批处理。这组命令在执行过程中会被顺序地、全部执行完毕，只要没有出现语法错误，这组命令在执行期间是不会被中断。



####  Redis 事务特性

> ACID, 是用来声明数据库事务的四大特性，即原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability）

Redis 的事务仅保证了数据的一致性，不具有像 DBMS 一样的 ACID 特性。 

* 这组命令中的某些命令的执行失败不会影响其它命令的执行，不会引发回滚。即不具备 原子性。 
* 这组命令**通过乐观锁机制实现了简单的隔离性**。没有复杂的隔离级别。 
* 这组命令的执行结果是被写入到内存的，是否持久取决于 Redis 的持久化策略，与事务 无关。



#### Redis 事务实现

三个命令：

* muti：开启事务
* exec：执行事务
* discard：取消事务



#### Redis 事务异常处理

（1）语法错误

当事务中的命令出现语法错误，则在exec执行时会被取消

（2）执行异常

如果事务中的命令没有语法错误，但执行过程中出现异常，该异常不会影响其它命令的执行。



#### Redis 事务隔离机制

（1）为什么需要隔离机制

在并发场景下可能会出现多个客户端对同一数据进行修改的情况。

例如：有两个客户端 C 左与 C 右，C 左需要申请 40 个资源，C 右需要申请 30 个资源。 它们首先查看了当前拥有的资源数量，即 resources 的值。它们查看到的都是 50，都感觉资 源数量可以满足自己的需求，于是修改资源数量，以占有资源。但结果却是资源出现了“超 卖”情况。

（2）隔离的实现

Redis 通过 watch 命令再配合事务实现了多线程下的执行隔离

**WATCH：监视Key改变，用于实现乐观锁。如果监视的Key的值改变，事务最终会执行失败。**

（3）实现原理

1. 当某一客户端对 key 执行了 watch 后，系统就会为该 key 添加一个 version 乐观锁，并初始化 version。例如初始值为1.0
2. 此时客户端A将该key的修改语句写入到了事务的命令队列中，虽然还没有执行，但其将该key修改前的value值与version进行了读取并保存到了当前的客户端的缓存中。即version的初值1.0
3. 这时，客户端B中在客户端A的事务中的写命令还未被执行时，客户端B修改了key的值，同时并增加了version的值，例如使其变成了version2.0，记录到了key的信息中
4. 在客户端A在执行exec，执行事务中的命令。不过在执行到对key的写命令时，该命令首先对当前客户端缓存中保存的version值与当前key信息中的version值进行比较，如果不一致，则会取消执行该写操作。

