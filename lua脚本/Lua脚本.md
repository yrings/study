# Lua脚本语言

## Lua入门

**Lua简介**

> Lua是一个有标准C语言开发的、开源的、可扩展的、轻量级的、弱类型的、解释型脚本语言。

**在Linux上安装Lua**

将压缩文件解压：

`tar -zxvf lua-5.4.6.tar.gz -C /opt/apps`

对安装文件进行编译并测试：

`make linux test`

最后安装：

`make install`

查看lua的安装目录：

`whereis lua`

**Lua的执行方式**

1. lua打开lua程序编辑
2. vim编辑一个脚本文件，例如`demo.lua`，然后`lua demo.lua`执行
3. 将脚本文件变成一个可执行文件，先在 `demo.lua`第一行上加上`#!/usr/bin/lua`表示要用什么程序来执行，然后更改文件权限`chmod 755 demo.lua`,然后`./demo.lua`来执行。

**在windos上安装Lua**

> 官网：https://github.com/rjpcomputing/luaforwindows/releases

配置：

在Options下的Global Option File打开配置文件。

第10行：可以调节字体大小。

第302行：将`code.page=65001`注释打开，将`code.page=0`注释后即可支持中文。

快捷键：

`ctrl + D`复制当前行

`ctrl + L`剪切当前行

`ctrl + T`上下行位子互换

`ctrl + Q`注释

`F5` 运行



## 基本语法

### 数据结构

> Lua中有8种类型，分别为：nil，boolean，number，string，userdata，function，threa和table，通过type()函数可以查看一个数据的类型，例如，type(nil)的结果为nil；



| 数据类型 | 描述                                                         |
| :------- | ------------------------------------------------------------ |
| nil      | 只有值 nil 属于该类，表示一个无效值，与 Java 中的 null 类似。但在条件 表达式中相当于 false。 |
| boolean  | 包含两个值：false 和 true。                                  |
| number   | 表示双精度类型的实浮点数。                                   |
| string   | 字符串，由一对双引号或单引号括起来。当一个字符串包含多行时，可以 在第一行中以[[开头，在最后一行中以]]结尾，那么在[[与]]括起来的这多行 内容就是一个字符串。换行符为字符串”\n”。 |
| table    | 类似于 Java 中的数组，但比数组的功能更强大，更灵活。         |
| function | 由 C 或 Lua 编写的函数。                                     |
| thread   | **协同线程**，是**协同函数**的执行体，即正在执行的协同函数。 |
| userdata | 一种用户自定义数据，用于表示一种由应用程序或 C/C++ 语言库所创建 的类型，可以将任意 C/C++ 的任意数据类型的数据存储到 Lua 变量中 调用。 |



### 标识符

**（1）保留字**

> Lua 常见的保留字共有 22 个。不过，除了这 22 个外，Lua 中还定义了很多的**内置全局变量**，这些内置全局变量的一个共同特征是，以**下划线开头后跟全大写字母**。所以我们在定 义自己的标识符时不能与这些保留字、内置全局变量重复

![](images\屏幕截图 2023-10-10 200948.png)



字符串连接用`..`



**算术运算符**

![](images\屏幕截图 2023-10-10 201529.png)



### 函数

> Lua 中函数的定义是以 function 开头，后跟函数名与参数列表，以 end 结尾。其可以没 有返回值，也可以一次返回多个值

**（1）固定参函数**

​	Lua 中的函数在调用时与 Java 语言中方法的调用是不同的，其不要求实参的个数必须与 函数中形参的个数相同。如果实参个数少于形参个数，则系统自动使用 nil 填充；如果实参 个数多于形参个数，多出的将被系统自动忽略

**（2）可变参函数**

​	在函数定义时不给出具体形参的个数，而是使用三个连续的点号。在函数调用时就可以 向该函数传递任意个数的参数，函数可以全部接收。

```lua
-- 定义一个普通函数，包含可变参
function f(...)
	local a,b,c,d = ...
	print(a,b,c,d)
    print(...)
end

print("传递五个")
f(10,20,30，40,50)
```

**（3）可返回多个值**

​	Lua 中的函数一次可以返回多个值，但需要有多个变量来同时接收

```lua
-- 定义一个普通函数，返回两个值
function f(a,b)
	local sum = a + b
	local mul = a * b
	return sum,mul
end

m,n = f(3,5)
print(m,n)
```

**（4）函数作为参数**

​	Lua 的函数中，允许函数作为参数。而作为参数的函数，可以是已经定义好的普通函数， 也可以是匿名函数。

```lua
-- 定义两个普通函数
function sum(a,b)
	return a + b
end

function mul(a,b)
	return a * b
end

--定义一个函数，其参数为另一个函数
function f(m,n,fun)
	local result = fun(m,n)
	print(result)
end

--普通调用
f(3,5,sum)
f(3,5,mul)
--匿名函数调用
f(3,5,function (a,b)
    	return a -b
       end
    )
```

### 流程控制语句

**（1）if 语句**

​	Lua 提供了 if…then 用于表示条件判断，其中 if 的判断条件可以是任意表达式。Lua 系统 将 false 与 nil 作为假，将 true 与非 nil 作为真，**即使是 0 也是真**。 需要注意，Lua 中的 if 语句的判断条件可以使用小括号括起来，也可以不使用

**（2）if 嵌套语句**

​	Lua 中提供了专门的关键字 elseif 来做 if 嵌套语句。注意，不能使用 else 与 if 两个关键 字的联用形式，即不能使用 else if 来嵌套 if 语句。

```lua
a = 5
if a>0 then
    print("num > 0")
elseif a == 0 then
    print("num == 0")
else 
    print("num < 0")
end
```

### 循环控制语句

> Lua 提供了四种循环控制语句：while...do 循环、repeat...until 循环、数值 for 循环，及 泛型 for 循环。同时，Lua 还提供了 break 与 goto 两种循环流程控制语句

**（1）while..do**

​	只要while中的条件成立就一直循环

```lua
a =3
while a > 0 do
	print(a)
	a = a - 1
end
```

**（2）repeat...until**

​	until中的条件成立了，循环就要停止（先执行，再判断）

```lua
a = 3
repeat
	print(a)
	a = a + 1
until a >= 0
```

**（3）数值 for**

​	这种 for 循环只参用于循环变量为数值型的情况。其语法格式为： 

```lua
for var=exp1, exp2, exp3 do 
    循环体 
end

for i = 10,50,10 do
	print(i)
end
```

​	var 为指定的循环变量，exp1 为循环起始值，exp2 为循环结束值，exp3 为循环步长。 

​	步长可省略不写，默认为 1。每循环一次，系统内部都会做一次当前循环变量 var 的值与 exp2 的比较，如果 var 小于等于 exp2 的值，则继续循环，否则结束循环

**（4）泛型 for**

​	泛型 for 用于遍历 table 中的所有值，其需要与 Lua 的迭代器联合使用。后面 table 学习 时再详解

**（5）break**

​	break 语句可以提前终止循环。其只能用于循环之中

**（6）goto**

​	goto 语句可以将执行流程无条件地跳转到指定的标记语句处开始执行，注意，是开始 执行，并非仅执行这一句，而是从这句开始后面的语句都会重新执行。当然，该标识语句在 第一次经过时也是会执行的，并非是必须由 goto 跳转时才执行。 

语句标记使用一对双冒号括起来，置于语句前面。goto 语句可以使用在循环之外。 注意，Lua5.1 中不支持双冒号的语句标记

```lua
function f(a)
    ::flag:: print("======")
    if a>1 then
        print(a)
        a = a - 1
        goto flag
    end
end
```

## Lua 语法进阶

### table

**（1）数组**

​	使用 table 可以定义一维、二维、多维数组。不过，需要注意，Lua 中的数组索引是从 1 开始的，且无需声明数组长度，可以随时增加元素。当然，同一数组中的元素可以是任意类 型。

```lua
-- 定义一个数组
cities = {"北京","上海","深圳"}
cities[4] = "广州"

for i = 1,4 do
	print("cities["..i.."]="..cities[i])
end
-- 定义一个二维数组
arr = {}

for i=1,3 do
    arr[i] = {}
    for j=1,2 do
        arr[i][j] = i * j
    end
end

for i=1,3 do
    for j=1,2 do
        print(arr[i][j])
    end
end
```

**（2）map**

​	使用 table 也可以定义出类似 map 的 key-value 数据结构。其可以定义 table 时直接指定 key-value，也可单独指定 key-value。而访问时，一般都是通过 table 的 key 直接访问，也可 以数组索引方式来访问，此时的 key 即为索引。

```lua
-- 定义一个map
emp = {name = "张三",age = 23,dapart = "销售部"}
emp.office = "2nd floor"
emp["gender"] = "男"

print(emp.office,emp["name"])

a = 3
b = 5
emp["emp_"..a] = true
emp[a + b] = "hello"

print(emp.emp_3)
print(emp[8])
```

**（3）混合结构**

​	Lua 允许将数组与 key-value 混合在同一个 table 中进行定义。key-value 不会占用数组的 数字索引值

```lua
-- 定义一个数组、map混合结构
emps = {
	{name = "张三", age = 23},
	{name = "liu", age = 24},
	{name = "wang", age = 25},
	{name = "zhao", age = 26},
}

for i=1,4 do
	print(emps[i].name..":"..emps[i].age)
end
```

**（4）table 操作函数**

* **table.concat()**

  ​	该函数用于将指定的 table **数组元素**进行字符串连接。连接从 start 索引位置到 end索引位置的所有数组元素, 元素间使用指定的分隔符 sep 隔开。如果 table 是一个混合结构， 那么这个连接与 key-value 无关，仅是连接数组元素

  **注意：当数组中元素之间有间隔，将会连接到间隔之前的元素**

  ```lua
  emp = {"北京",name="张三",age=23,"上海",depart="市场部","广州","深圳"}
  
  print(table.concat(emp))
  print(table.concat(emp,",",2,3))
  ```

* **table.unpack()**

  拆包。该函数返回指定 table 的数组中的从第 i 个元素到第 j 个元素值。i 与 j 是可 选的，默认 i 为 1，j 为数组的最后一个元素,Lua5.1 不支持该函数

  ```lua
  arr = {"bj","sh","gz","sz"}
  table.unpack(arr)
  a,b = table.unpack(arr,2,3)
  ```

* **table.pack()**

  `table.pack (…) `

  打包。该函数的参数是一个可变参，其可将指定的参数打包为一个 table 返回。这 个返回的 table 中具有一个属性 n，用于表示该 table 包含的元素个数。Lua5.1 不支持该函数。

* **table.maxn()**

  该函数返回指定 table 的数组中的最大索引值，即数组包含元素的个数

* **table.insert()**

  `table.insert (table, [pos,] value):`

  该函数用于在指定 table 的数组部分指定位置 pos 插入值为 value 的一个元素。其 后的元素会被后移。pos 参数可选，默认为数组部分末尾

* **table.remove()**

  `table.remove (table [, pos]) `

  该函数用于删除并返回指定 table 中数组部分位于 pos 位置的元素。其后的元素会被前移。pos 参数可选，默认删除数组中的最后一个元素。

* **table.sort()**

​	`table.sort(table [,fun(a,b)]) `

该函数用于对指定的 table 的数组元素进行**升序排序**，也可按照指定函数 fun(a,b) 中指定的规则进行排序。fun(a,b)是一个用于比较 a 与 b 的函数，a 与 b 分别代表数组中的两 个相邻元素。 

注意： 

* 如果 arr 中的元素既有字符串又有数值型，那么对其进行排序会报错。 
* 如果数组中多个元素相同，则其相同的多个元素的排序结果不确定，即这些元素的索引谁排前谁排后，不确定。 
* 如果数组元素中包含 nil，则排序会报错

**实现倒序：**

```lua
cities = {"bj北京","sh上海","gz广州","sz深圳","tj天津"}

table.sort(cities, function(a,b)
    				return a>b
        		end
    )
print(table.concat(cities,","))
```



### 迭代器

> Lua 提供了两个迭代器 pairs(table)与 ipairs(table)。这两个迭代器通常会应用于泛型 for 循环中，用于遍历指定的 table。
>
> 这两个迭代器的不同是： 
>
> * ipairs(table)：仅会迭代指定 table 中的数组元素。 
> * pairs(table)：会迭代整个 table 元素，无论是数组元素，还是 key-value。

```lua
emp = {"北京",name="张三",age=23,"上海",depart="市场部","广州","深圳"}

-- 遍历emp中的所有数组元素
for i,v in ipairs(emp) do
	print(i,v)
end

-- 遍历emp中的所有元素
for k,v in pairs(emp) do
	print(k,v)
end
```



### 模块

> 模块是Lua中特有的一种数据结构。从 Lua 5.1 开始，Lua 加入了标准的模块管理机制， 可以把一些公用的代码放在一个文件里，以 API 接口的形式在其他地方调用，有利于代码 的重用和降低代码耦合度。 
>
> 模块文件主要由 table 组成。在 table 中添加相应的变量、函数，最后文件返回该 table 即可。如果其它文件中需要使用该模块，只需通过 **require** 将该模块导入即可。



**（1） 定义一个模块**

​	模块是一个 lua 文件，其中会包含一个 table。一般情况下该文件名与该 table 名称相同， 但其并不是必须的

```lua
-- 声明一个模块
rectangle = {}

-- 为模块添加一个变量
rectangle.pi = 3.14

--为模块添加函数（求周长）
function rectangle.perimeter(a,b)
	return (a + b) * 2
end

--以匿名函数的方式为模块添加一个函数（求面积）
rectangle.area = function(a,b)
	return a*b
end

return rectangle
```

**（2）使用模块**

​	这里要用到一个函数 require(“文件路径”)，其中文件名是不能写.lua 扩展名的。该函数 可以将指定的 lua 文件静态导入（合并为一个文件）。不过需要注意的是，该函数的使用可 以省略小括号，写为 require “文件路径”。 

​	require()函数是有返回值的，返回的就是模块文件最后 return 的 table。可以使用一个变 量接收该 table 值作为模块的别名，就可以使用别名来访问模块了。

```lua
-- 导入一个模块
require "rectangle"
-- 访问模块
print(rectangle.pi)
print(rectangle.perimeter(3,5))
print(rectangle.area(3,5))
```

**（3）无关参数**

​	模块文件中一般定义的变量与函数都是模块 table 相关内容，但也可以定义其它与 table 无关的内容。这些全局变量与函数就是普通的全局变量与函数，与模块无关，但会随着模块 的导入而同时导入。所以在使用时可以直接使用，而无需也不能添加模块名称



### 元表与元方法

> 元表，即Lua中普通table的元数据表，而元方法则是元表中定义的普通表的**默认行为**，Lua中的每个普通table都可为其定义一个元表，用于拓展该普通table的行为功能。
>
> 例如， 对于 table 与数值相加的行为，Lua 中是没有定义的，但用户可通过为其指定元表来扩展这 种行为；再如，用户访问不存在的 table 元素，Lua 默认返回的是 nil，但用户可能并不知道 发生了什么。此时可以通过为该 table 指定元表来扩展该行为：给用户提示信息，并返回用 户指定的值。
>
> **就比如说在java中重写类中的toString()方法**

**（1）关键函数**

* `setmetatable(table,metatable)`:将 metatable指定为普通表table的元表
* `getmetatable(table)`：获取指定普通表 table 的元表

**（2）__index元方法（两个下划线）**

> 当用户在对table进行读取访问时，如果访问的**数组索引**或者**key**不存在，那么系统就会调用元表的`__index`方法。如果 重写的_ _index 元方法是函数，且有返回值，则直接返回；如果没有返回值，则返回 nil

```lua
emp = {"北京",nil,name="张三",age=23,"上海",depart="市场部","广州","深圳"}
print(emp.x)
-- 声明一个元表
meta = {}
-- 将原始表与元表相关联
setmetatable(emp,meta)
--  有返回值
meta.__index = function(tab,key)
	return "通过 【"..key.."】 访问的值不存在"
end
print(emp.x)
print(emp[2])

------输出结果---------
nil
通过 【x】 访问的值不存在
通过 【2】 访问的值不存在
```

**该重写的方法可以是一个函数，也可以是另一个表。如果在原始表中若找不到，则会到元素指定的普通表中查找**



**（3）__newindex元方法**

> 当用户为 table 中一个不存在的索引或 key 赋值时，就会自动调用元表的_ _newindex 元 方法。

```lua
emp = {"北京",nil,name="张三",age=23,"上海",depart="市场部","广州","深圳"}
print(emp[5])
-- 声明一个元表
meta = {}
-- 将原始表与元表相关联
setmetatable(emp,meta)
-- 无返回值的情况
function meta.__newindex(tab,key,value)
	print("新增的key为"..key..",value值为"..value)
	-- 将新增的key-value写入到原始表
	rawset(tab,key,value)
end

emp.x = "天津"
```

**该重写的方法可以是一个函数，也可以是另一个表。元表指定的另一个普通表的作用是，暂存新增加的数据。**

**（4）运算符方法**

> 如果要为一个表扩展加号（+），减号（-）等运算功能，则可重写相应的元方法。
>
> ​	例如，要为一个table扩展加号运算功能，则可以重写该table元表的__add元方法。

```lua
emp = {"北京",name="张三",age=23,"上海",depart="市场部","广州","深圳"}

-- 声明一个元表
meta = {
	__add = function(tab,num)
	-- 遍历tab的所有元素
	for k,v in pairs(tab) do
		-- 若value为数值类型，则做算术加法
		if type(v) == "number" then
			tab[k] = v + num
		-- 若value为string，则做字符串拼接
		elseif type(v) == "string" then
			tab[k] = v..num
		end

	end

	-- 返回变化过的tab
	return tab
end
}
--  将原始表与元表相关联
setmetatable(emp,meta)

empsum = emp + 5

for k,v in pairs(empsum) do
	print(k..":"..v)
end

-------输出结果-------
1:北京5
2:上海5
3:广州5
4:深圳5
depart:市场部5
name:张三5
age:28
```

其它运算符元方法：

| 元方法   | 说明           | 元方法 | 说明         |
| -------- | -------------- | ------ | ------------ |
| __add    | 加             | __band | 按位与，&    |
| __sub    | 减             | __bor  | 按位或，\|   |
| __mul    | 乘             | __bxor | 按位异或，~  |
| __div    | 除             | __bont | 按位非，~    |
| __mod    | 取模，%        | __shl  | 按位左移，<< |
| __pow    | 次幂, ^        | __shr  | 按位右移，>> |
| __unm    | 取反, -        | __eq   | 等于， ==    |
| __idiv   | 取整除法，//   | __lt   | 小于         |
| __concat | 字符串连接，.. | __le   | 小于等于     |
| __len    | 字符串长度， # |        |              |

**（5）__tostring 元方法**

> 直接输出一个table，其输出的内容为类型与table  的存放地址。如果想让其输出 table 中的内容，可重写 
>
> __tostring 元方法。

```lua
emp = {"北京",name="张三",age=23,"上海",depart="市场部","广州","深圳"}

-- 声明一个元表
meta = {
	__add = function(tab,num)
	....
end,
	__tostring = function(tab)
		str = ""
		-- 字符串拼接
		for k,v in pairs(tab) do
			str = str .. " " ..k..":"..v
		end
		return str

	end
}
--  将原始表与元表相关联
setmetatable(emp,meta)
empsum = emp + 5

print(emp)
print(empsum)
```

**注意：在table中写入两个方法之间要加逗号！**

**（6）__call 元方法**

> 当将一个table 以函数形式来使用时，系统会自动调用重写的__call元方法。该用法是可以简化对table的相关操作，将对table的操作与函数直接结合。

```lua
-- 将原始表与匿名元表相关联
setmetatable(emp,{
	__call = function(tab,num,str)
	-- 遍历table
	for k,v in pairs(tab) do
		-- 若value为数值类型，则做算术加法
		if type(v) == "number" then
			tab[k] = v + num
		-- 若value为string，则做字符串拼接
		elseif type(v) == "string" then
			tab[k] = v..str
		end
	end
	return tab
	end
})
newemp = emp(5,"-hello")
for k,v in pairs(newemp) do
	print(k..":"..v)
end

-------输出结果--------
1:北京-hello
2:上海-hello
3:广州-hello
4:深圳-hello
depart:市场部-hello
name:张三-hello
age:28
```

**（7）元表单独定义**

为了便于管理与复用，可以将元素单独定义为一个文件。该文件中仅可定义一个元表， 且一般文件名与元表名称相同。 

若一个文件要使用其它文件中定义的元表，只需使用 require “元表文件名”即可将元表 导入使用。 

如果用户想扩展该元表而又不想修改元表文件，则可在用户自己文件中重写其相应功能 的元方法即可。

### 面向对象

> Lua 中通过 table 与 fcuntion 可以创建出一个简单的 Lua 对象：table 为 Lua 对象赋予属 性，通过 function 为 Lua 对象赋予行为，即方法

**（1）简单对象的创建**

Lua 中通过 table 与 fcuntion 可以创建出一个简单的 Lua 对象：table 为 Lua 对象赋予属 性，通过 function 为 Lua 对象赋予行为，即方法

```lua
-- 创建一个名称为animal的对象
animal = {name = "tom",age = 5}

--  为animal添加方法
--~ animal.bark = function(self,voice)
--~ 	print(self.name.."在"..voice.."叫")
--~ end

-- 该方式会自动传入self
function animal:bark(voice)
	print(self.name.."在"..voice.."叫")
end

animal:bark("喵喵")
animal2 = animal

-- 将animal置空
animal.name = "jerry"
animal:bark("汪汪")

```

**（2）类的创建**

Lua 中使用 table、function 与元表可以定义出类：使用一个表作为基础类，使用一个 function 作为该基础类的 new()方法。在该 new()方法中创建一个空表，再为该空表指定一个 元表。该元表重写_ _index 元方法，且将基础表指定为重写的_ _index 元方法。由于 new() 中的表是空表，所以用户访问的所有 key 都会从基础类（表）中查找

```lua
-- 创建一个类
Animal = {name = "no_name", age = 0}

function Animal:bark(voice)
	print(self.name.."在"..voice.."叫")
end

-- 为该类添加一个无参构造器
function Animal:new()
	-- 创建一个空表
	local a = {}
	-- 为新表指定元表尾当前基础表
	-- 在新表中找不到的key，会从self基础类表中查找
	setmetatable(a,{__index = self})
	return a
end

-- 为该类添加一个带参构造器（table的情况）
function Animal:new(obj)
	local a = ""
	a = obj or {}

	setmetatable(a,{__index = self})

	return a
end

animal = Animal:new()
animal2 = Animal:new()
animal3 = Animal:new({type="老鼠"})

animal.name = "tom"
animal.age = 9
animal.type = "猫"

function animal2:skill()
	return "小老鼠偷油"
end

print(animal2.name..":"..animal2.age..":"..animal2.skill())
print(animal3.type)
```

### 

### 协同线程与协同函数

**（1）协同线程**

> Lua 中有一种特殊的线程，称为 coroutine，协同线程，简称协程。其可以在运行时暂停 执行，然后转去执行其它线程，然后还可返回再继续执行没有执行完毕的内容。即可以“走 走停停，停停再走走”。 
>
> 协同线程也称为协作多线程，在 Lua 中表示独立的执行线程。**任意时刻只会有一个协程执行**，而不会出现多个协程同时执行的情况。 
>
> 协同线程的类型为 thread，其启动、暂停、重启等，都需要通过函数来控制。下表是用 于控制协同线程的基本方法

```lua
-- 创建一个协同线程实例
crt = coroutine.create(
	function(a,b)
		print(a,b,a+b)
		-- 获取正在执行的协同线程，类型为thread
		tr = coroutine.running()
		print(tr)
		-- 查看系统线程实例的类型
		print(type(tr))
		-- 查看状态
		print(coroutine.status(tr))
		-- 将当前线程实例挂起
		coroutine.yield()
		print("重新返回协同线程")
	end
)
-- 启动协同线程实例
coroutine.resume(crt,3,5)
-- 查看crt的类型
print("main-"..type(crt))
-- 查看状态
print("main-"..coroutine.status(crt))
-- 继续执行协同线程
coroutine.resume(crt)
-- 查看状态
print("main-"..coroutine.status(crt))
```



```lua
-- 创建一个协同线程实例
crt = coroutine.create(
	function(a,b)
		print(a,b)
		-- 将当前线程实例挂起
		coroutine.yield(a*b,a/b)
		print("重新返回协同线程")
		-- 返回两个值
		return a+b,a-b
	end
)
-- 启动协同线程，返回三个值
-- 第一个返回值为系统线程启动成功状态
-- 第二个返回值为内置函数的返回值
success, result1, result2 = coroutine.resume(crt,12,3)

print(success, result1, result2)

success,result1,result2 = coroutine.resume(crt)

print(success, result1, result2)

-----输出结果---------
12	3
true	36	4
重新返回协同线程
true	15	9
```

**（2）协同函数**

> 协同线程可以单独创建执行，也可以通过协同函数的调用启动执行。使用 coroutine 的 wrap()函数创建的就是协同函数，其类型为 function。 
>
> 由于协同函数的本质就是函数，所以协同函数的调用方式就是标准的函数调用方式。只 不过，协同函数的调用会启动其内置的协同线程

```lua
-- 创建一个协同函数
cf = coroutine.wrap(
	function (a,b)
		print(a,b)

		-- 获取当前协同函数创建的协同线程
		tr = coroutine.running()
		print("tr的类型为："..type(tr))
		-- 挂起当前的协同线程
		coroutine.yield(tr,a+1,b+1)
		print("重新返回到协同线程")
		return a+b,a*b
	end

)
-- 调用协同函数，启动协同线程
cftr,result1,result2 = cf(3,5)
print(result1,result2)
print("cf的类型为："..type(cf))

--重启挂起的协同线程
success,result1,result2 = coroutine.resume(cftr)
print(result1,result2)
--------输出结果--------
tr的类型为：thread
4	6
cf的类型为：function
重新返回到协同线程
8	15
```

### 文件IO



**（1）静态函数**

```lua
-- 以只读方法打开一个文件
file = io.open("test.properties","r")
-- 指定要读取的文件为file
io.input(file)
-- 读取一行数据
line = io.read("*l")
while line ~= nil do
	print(line)
	line = io.read("*l")
end
io.close(file)


-- 以追加方式打开文件
file = io.open("test.properties" , "a")
-- 指定要写入的文件为file
io.output(file)
-- 写入一行数据
io.write("\ngender=male")
io.close(file)
```

**（2）实例函数**

```lua
-- 以只读方法打开一个文件
file = io.open("test.properties","r")
-- 读取一行数据
line = file:read("*l")
while line ~= nil do
	print(line)
	line = file:read("*l")
end
file:close(file)


-- 以追加方式打开文件
file = io.open("test.properties" , "a")
-- 写入一行数据
file:write("\ntest=7777")
file:close(file)
```

*  **file:seek() **

  【格式】file:seek ([whence [, offset]]) 

  【解析】该函数用于获取或设置文件读写指针的当前位置。位置从 1 开始计数，除文件最后 一行外，每行都有行结束符，其会占两个字符位置。位置 0 表示文件第一个位置的前面位置。 当 seek()为无参时会返回读写指针的当前位置。参数 whence 的值有三种，表示将指针 定位的不同位置。而 offset 则表示相对于 whence 指定位置的偏移量，offset 的默认值为 0， 为正表示向后偏移，为负表示向前偏移。 

  * set：表示将指针定位到文件开头处，即 0 位置处 
  * cur：表示指针保持当前位置不变，默认值 
  * end：表示将指针定位到文件结尾

```lua
-- 只读方式打开
file = io.open("test2.properties", "r")

pos = file:seek()
print("当前位置："..pos)
-- 读取一行数据
line = file:read("*l")
print(line)

pos = file:seek()
print("当前位置："..pos)

pos = file:seek("set",2)
print("移动后的当前位置："..pos)
-- 读取一行数据
line = file:read("*l")
print(line)
```









