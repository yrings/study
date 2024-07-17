# MySQL

## 一、SQL基本知识

### 1.1 三大范式

第一范式(1NF)：每个列都不可以再拆分。
第二范式(2NF)：在第一范式的基础上，非主键列完全依赖于主键，而不能是依赖于主键的一部分。
第三范式(3NF)：在第二范式的基础上，非主键列只依赖于主键，不依赖于其他非主键。



### 1.2 数据类型





### LIKE 子句

> like 提供了强大的模糊查询能力

```sql
SELECT column1, column2, ...
FROM table_name
WHERE column_name LIKE pattern;
```

#### 通配符

* **%百分号通配符:表示零个或多个字符**

  ```sql
  SELECT * FROM customers WHERE last_name LIKE 'S%';
  ```

  所有姓氏以'S'开头的用户

* **_下划线通配符：通配符表示一个字符**

  ```sql
  SELECT * FROM products WHERE product_name LIKE '_a%';
  ```

  选择产品第二个字符为‘a’的所有产品

* **不区分大小写**

  ```sql
  SELCET * FROM employees WHERE last_name LIKE 'smi%' COLLATE utf8mb4_general_ci;
  ```



### UNION 操作符

> UNION 操作符用于连接两个以上的 SELECT 语句的结果组合到一个结果集合，**并除去重复的行**。
>
> UNION 操作符**必须由两个或多个 SELECT 语句组成**，**每个 SELCET 语句的列数和对应的位置和数据类型必须相同**

**语法：**

```sql
SELECT column1, column2, ...
FROM table1
WHERE condition1
UNION
SELECT column1, column2, ...
FROM table2
WHERE condition2
[ORDER BY column1, column2, ...];
```

**例子：**

* 不去除重复行（因此性能较好），UNION ALL

  ```sql
  SELECT city FROM customers
  UNION ALL
  SELECT city FROM suppliers
  ORDER BY city;
  ```



### ORDER BY

> 用来设定你想按什么排序返回搜索结果

**语法：**

```sql
SELECT column1, column2, ...
FROM table_name
ORDER BY column1 [ASC | DESC], column2 [ASC | DESC], ...;
```

**例子：**

* **使用表达式排序**

  ```sql
  SELECT product_name, price * discount_rate AS discounted_price
  FROM products
  ORDER BY discounted_price DESC;
  ```

  discounted_price是sql语句前面定义的表达式

* **使用NULLS FIRST 或者 NULLS LAST 处理NULL 值**

  ```sql
  SELECT product_name, price
  FROM products
  ORDER BY price DESC NULLS LAST;
  ```



### GROUP BY 分组

> 根据一个或多个列队结果集进行分组。在分组的列上我们可以使用 COUNT、SUM、AVG等函数。
>
> 这个语句在 SQL 查询语句中用于汇总和分析数据的重要工具，尤其在处理大量数据时，它能够提供有用的汇总信息。

**语法：**

```sql
SELECT column1, aggregate_function(column2)
FROM table_name
WHERE condition
GROUP BY column1;
```

**注意**

- `GROUP BY` 子句通常与聚合函数一起使用，因为分组后需要对每个组进行聚合操作。
- `SELECT` 子句中的列通常要么是分组列，要么是聚合函数的参数。
- 可以使用多个列进行分组，只需在 `GROUP BY` 子句中用逗号分隔列名即可。





#### 使用 WITH ROLLUP 和 coalesce

`WITH ROLLUP` 可以实现在分组统计数据基础上再进行相同的统计（SUM,AVG,COUNT…）

```sql
mysql> SELECT name, SUM(signin) as signin_count FROM  employee_tbl GROUP BY name WITH ROLLUP;
+--------+--------------+
| name   | signin_count |
+--------+--------------+
| 小丽 |            2 |
| 小明 |            7 |
| 小王 |            7 |
| NULL   |           16 |
+--------+--------------+
4 rows in set (0.00 sec)
```

其中记录 NULL，表示所有人的登陆次数

```java
select coalesce(a,b,c);
```

如果 a == null，则选择 b；如果 b==null,则选择 c；如果 a!=null,则选择 a；如果 a b c 都为 null ，则返回为 null



### JOIN 连接的使用

> JOIN 在两个或多个表中查询数据。

**分类：**

JOIN 按照功能大致分为如下三类：

- **INNER JOIN（内连接,或等值连接）**：获取两个表中字段匹配关系的记录。
- **LEFT JOIN（左连接）：**获取左表所有记录，即使右表没有对应匹配的记录,也会显示出这条记录。
- **RIGHT JOIN（右连接）：** 与 LEFT JOIN 相反，用于获取右表所有记录，即使左表没有对应匹配的记录。

**INNER JOIN：**

```sql
SELECT column1, column2, ...
FROM table1
INNER JOIN table2 ON table1.column_name = table2.column_name;
```



### Limit

`LIMIT` 和 `OFFSET` 子句通常和`ORDER BY` 语句一起使用，当我们对整个结果集排序之后，我们可以 `LIMIT`来指定只返回多少行结果 ,用 `OFFSET`来指定从哪一行开始返回。你可以想象一下从一条长绳子剪下一小段的过程，我们通过 `OFFSET` 指定从哪里开始剪，用 `LIMIT` 指定剪下多少长度。

limit查询

```
SELECT column, another_column, … 
FROM mytable WHERE condition(s)
ORDER BY column ASC/DESC 
LIMIT num_limit OFFSET num_offset;
```

OFFSET 不会从自己所在索引开始遍历，在下一个



### 类型转换

CAST(expression AS TYPE) 函数可以将任何类型的值转换为具有指定类型的值,利用该函数可以直接在数据库层处理部分因数据类型引起的问题。
以下为该函数支持的数据类型

支持的 TYPE 类型	描述
BINARY	二进制型
CHAR	字符型
DATE	日期，格式为 ‘YYYY-MM-DD’
DATETIME	日期加具体的时间，格式为 ‘YYYY-MM-DD HH:MM:SS’
TIME	时间，格式为 ‘HH:MM:SS’
DECIMAL	float 型
SIGNED	int 型
UNSIGNED	无符号int
下面对几种转换进行示例讲解
说明：示例中的固定值可以换为实际的查询的表的字段，例如：id

```sql
1、固定值转为BINARY 二进制型

SELECT CAST( 1231 AS BINARY ) AS result 

运行结果：1231 

2、int类型值转为CHAR 字符型

SELECT CAST(1995 AS CHAR) as result

运行结果："1995"

3、固定时间字符串转为DATE 日期，格式为 'YYYY-MM-DD’;

SELECT CAST('2019-08-29 16:50:21' as date) as result

运行结果：2019-08-29;

4、固定时间字符串转为DATETIME 日期加具体的时间，格式为 'YYYY-MM-DD HH:MM:SS’

SELECT CAST('2019-08-29 16:50:21' as DATETIME) as result

运行结果：2019-08-29 16:50:21

5、固定时间字符串转为TIME 时间，格式为 'HH:MM:SS’

SELECT CAST('2019-08-29 16:50:21' as TIME) as result

运行结果：16:50:21

6、float型值通过DECIMAL 获取精度

SELECT CAST(220.23211231 AS DECIMAL(10, 3)) AS result 

运行结果：220.232;

7、固定字符串转为SIGNED int 型

SELECT CAST("12321" AS SIGNED  ) AS result 

运行结果：12321;

8、固定字符串转为UNSIGNED 无符号int

SELECT CAST("12321" AS UNSIGNED   ) AS result 

运行结果：12321
```



### 正则表达式

>  MySQL 中使用`REGEXP` 和 ` RLIKE`**操作符来进行正则表达式匹配。
>
> 两个字符的作用完全相同，可以互相替换。



**正则表达式匹配的字符类**

- `.`：匹配任意单个字符。
- `^`：匹配字符串的开始。
- `$`：匹配字符串的结束。
- `*`：匹配零个或多个前面的元素。
- `+`：匹配一个或多个前面的元素。
- `?`：匹配零个或一个前面的元素。
- `[abc]`：匹配字符集中的任意一个字符。
- `[^abc]`：匹配除了字符集中的任意一个字符以外的字符。
- `[a-z]`：匹配范围内的任意一个小写字母。
- `\d`：匹配一个数字字符。
- `\w`：匹配一个字母数字字符（包括下划线）。
- `\s`：匹配一个空白字符。
- `{n}`：n是一个非负整数，匹配确定的n次；例如，'o{2}' 不能匹配 "Bob" 中的 'o'，但是能匹配 "food" 中的两个 o。
- `{n,m}`：和上面的类似但是n <= m,表示最少匹配n次，且最多匹配m次。



### 事务

一般来说，事务是必须满足4个条件（ACID）：：原子性（**A**tomicity，或称不可分割性）、一致性（**C**onsistency）、隔离性（**I**solation，又称独立性）、持久性（**D**urability）。

- **原子性：**一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
- **一致性：**在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。
- **隔离性：**数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。
- **持久性：**事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。



#### 隔离性级别

* **读未提交（Read UnCommitted/RU）**

  一个事务可以读取到另一个事务未提交的数据。

  **脏读**：如果另一个事务发起了回滚，会导致数据一致性问题。

* **读已提交（Read Committed/RC）**

  一个事务提交之后，它做的变更才会被其它事务看到

  **不可重复读**：同一事务先后读取同一个数据，在这期间数据发送了变化，两次读取的结果不一致。
  
* **可重复读（Repeatable Read）**（MySQL的InnoDB默认的隔离级别）

  一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。当然在可重复读隔离级别下，未提交变更对其他事务也是不可见的。

  **幻读**：同一事务按照某个条件先后两次查询数据库，两次查询结果的条数不同，这种现象称为幻读。不可重复读与幻读的区别可以通俗的理解为前者是数据变了，后者是数据的行数变了。

* **可串行化**



### 索引

> MySQL 索引是一种数据结构，用于加快数据库查询的速度和性能。
>
> 索引可以大大提高检索速度。

索引分单列索引和组合索引：

* 单列索引：即一个索引只包含单个列，一个表可以有多个单列索引
* 组合索引：即一个索引包含多个列

索引虽然能够提高查询性能，但也需要注意以下几点：

- 索引需要占用额外的存储空间。
- 对表进行插入、更新和删除操作时，索引需要维护，可能会影响性能。
- 过多或不合理的索引可能会导致性能下降，因此需要谨慎选择和规划索引。



**语法：**

**普通索引：**

```sql
CREATE INDEX index_name
ON table_name (column1 [ASC|DESC], column2 [ASC|DESC], ...);

ALTER TABLE table_name
ADD INDEX index_name (column1 [ASC|DESC], column2 [ASC|DESC], ...);
```

在创建表结构时，直接指定索引：

```sql
CREATE TABLE table_name (
  column1 data_type,
  column2 data_type,
  ...,
  INDEX index_name (column1 [ASC|DESC], column2 [ASC|DESC], ...)
);
```

**唯一索引：**

> 普通索引允许被索引的数据列包含重复的值。比如说，因为人有可能同名，所以同一个姓名在同一个“员工个人资料”数据表里可能出现两次或更多次。
> 如果能确定某个数据列将只包含彼此各不相同的值，在为这个数据列创建索引的时候就应该用关键字UNIQUE把它定义为一个唯一索引。**这么做的好处：**
>
> **一是简化了MySQL对这个索引的管理工作，这个索引也因此而变得更有效率；**
>
> **二是MySQL会在有新记录插入数据表时，自动检查新记录的这个字段的值是否已经在某个记录的这个字段里出现过了；如果是，MySQL将拒绝插入那条新记录。也就是说，唯一索引可以保证数据记录的唯一性。事实上，在许多场合，人们创建唯一索引的目的往往不是为了提高访问速度，而只是为了避免数据出现重复。**

```sql
CREATE UNIQUE INDEX index_name
ON table_name (column1 [ASC|DESC], column2 [ASC|DESC], ...);
```



### 视图





## 二、配置

### Mysql查看连接数（连接总数、活跃数、最大并发数）

```sql
show variables like '%max_connection%'; # 查看最大连接数
set global max_connections=1000;    #重新设置最大连接数

mysql> show status like 'Threads%';
+-------------------+-------+
| Variable_name   | Value |
+-------------------+-------+
| Threads_cached  | 32  |
| Threads_connected | 10  |
| Threads_created  | 50  |
| Threads_rejected | 0   |
| Threads_running  | 1   |
+-------------------+-------+
5 rows in set (0.00 sec)
```

Threads_connected ：这个数值指的是打开的连接数.

Threads_running ：这个数值指的是激活的连接数，这个数值一般远低于connected数值.

Threads_connected 跟show processlist结果相同，表示当前连接数。准确的来说，Threads_running是代表当前并发数

查询数据库当前设置的最大连接数

```sql
mysql> show variables like '%max_connection%';
+-----------------------+-------+
| Variable_name     | Value |
+-----------------------+-------+
| extra_max_connections |    |
| max_connections    | 2512 |
+-----------------------+-------+
2 rows in set (0.00 sec)
```



在/etc/my.cnf里面设置数据库的最大连接数

[mysqld]

max_connections = 1000

MySQL服务器的线程数需要在一个合理的范围之内，这样才能保证MySQL服务器健康平稳地运行。Threads_created表示创建过的线程数，通过查看Threads_created就可以查看MySQL服务器的进程状态。

```sql
mysql> show global status like ``'Thread%'``;``
+-------------------------+-------+``
| Variable_name      | Value |``
+-------------------------+-------+``
| Threadpool_idle_threads | 0   |``
| Threadpool_threads   | 0   |``
| Threads_cached     | 29  |``
| Threads_connected    | 13  |``
| Threads_created     | 50  |``
| Threads_rejected    | 0   |``
| Threads_running     | 1   |``
+-------------------------+-------+``
7 rows ``in` `set` `(0.00 sec)
```

如果我们在MySQL服务器配置文件中设置了thread_cache_size，当客户端断开之后，服务器处理此客户的线程将会缓存起来以响应下一个客户而不是销毁(前提是缓存数未达上限)。

Threads_created表示创建过的线程数，如果发现Threads_created值过大的话，表明MySQL服务器一直在创建线程，这也是比较耗资源，可以适当增加配置文件中thread_cache_size值，查询服务器thread_cache_size的值：

```sql
mysql> show variables like ``'thread_cache_size'``;
+-------------------+-------+``
| Variable_name   | Value |``
+-------------------+-------+``
| thread_cache_size | 100  |``
+-------------------+-------+``
1 row ``in` `set` `(0.00 sec)
```

命令：show processlist;
如果是root帐号，你能看到所有用户的当前连接。如果是其它普通帐号，只能看到自己占用的连接。
show processlist命令只列出前100条，如果想全列出请使用show full processlist;
mysql> show processlist;

命令：show status;

mysql>show status like '%变量名%';

变量名如下：
Aborted_clients 由于客户没有正确关闭连接已经死掉，已经放弃的连接数量。
Aborted_connects 尝试已经失败的MySQL服务器的连接的次数。
Connections 试图连接MySQL服务器的次数。
Created_tmp_tables 当执行语句时，已经被创造了的隐含临时表的数量。
Delayed_insert_threads 正在使用的延迟插入处理器线程的数量。
Delayed_writes 用INSERT DELAYED写入的行数。
Delayed_errors 用INSERT DELAYED写入的发生某些错误(可能重复键值)的行数。
Flush_commands 执行FLUSH命令的次数。
Handler_delete 请求从一张表中删除行的次数。
Handler_read_first 请求读入表中第一行的次数。
Handler_read_key 请求数字基于键读行。
Handler_read_next 请求读入基于一个键的一行的次数。
Handler_read_rnd 请求读入基于一个固定位置的一行的次数。
Handler_update 请求更新表中一行的次数。
Handler_write 请求向表中插入一行的次数。
Key_blocks_used 用于关键字缓存的块的数量。
Key_read_requests 请求从缓存读入一个键值的次数。
Key_reads 从磁盘物理读入一个键值的次数。
Key_write_requests 请求将一个关键字块写入缓存次数。
Key_writes 将一个键值块物理写入磁盘的次数。
Max_used_connections 同时使用的连接的最大数目。
Not_flushed_key_blocks 在键缓存中已经改变但是还没被清空到磁盘上的键块。
Not_flushed_delayed_rows 在INSERT DELAY队列中等待写入的行的数量。
Open_tables 打开表的数量。
Open_files 打开文件的数量。
Open_streams 打开流的数量(主要用于日志记载）
Opened_tables 已经打开的表的数量。
Questions 发往服务器的查询的数量。
Slow_queries 要花超过long_query_time时间的查询数量。
Threads_connected 当前打开的连接的数量。
Threads_running 不在睡眠的线程数量。
Uptime 服务器工作了多长时间，单位秒。　
