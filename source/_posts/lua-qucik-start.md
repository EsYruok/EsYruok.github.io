---
title: Lua快速入门
categories: 编程
tag: Lua
abbrlink: 71a33465
date: 2024-02-15 21:39:45
---
对Lua基本知识的一个精炼总结, 快速认识Lua.  <!--more-->  
写在前面:  

0. 作者学习Lua目的是更好的使用Neovim, 所以文章内容更不会太深, 如果您想要使用Lua进行程序设计, 最好能够阅读更详细的教学资料, 详细的了解Lua的一些特性与原理.  
0. 文章没有对库函数的使用方法进行具体的解释说明, 因为依靠参考手册全部都可以查到.    
0. 文章没有对一些编程通用的基本概念进行讲解(什么是注释, 变量等), 您最好对编程有所认识.  

Lua官方网站: [Lua](https://www.lua.org/)  
Lua参考手册: [Reference manuals](https://www.lua.org/manual/)

## 安装Lua  
要使用Lua需要安装解释器(得到lua命令).  

### ArchLinux
```sh
sudo pacman -S lua
```

### Windows
在[LuaBinaries](https://luabinaries.sourceforge.net/)页面选择版本, 下载二进制文件解压并添加环境变量即可.  

> Lua 解释器有两种使用方式, 交互式与非交互式.  
>
> - 交互式是指控制台直接输入`lua`命令, 进入一个交互式界面, 它会即时的执行你输入的内容.  
> - 非交互式是指将代码储存在脚本文件中, 再传递给解释器执行. 例如: `lua hello.lua`  

## 标识符
Lua标识符规定:

- 以任意字母,数字和下划线组成  
- 不能以数字开头  
- 不能为关键字  
- 大小写敏感  

> Lua 关键字:  
> and | break | do | else | elseif | end | false | goto | for | function  
> if | in | local | nil | not | or | repeat | return | then | true | until | while  

## 注释  

### 单行注释
以`--`为起始.  
```lua
-- line
```

### 多行注释
以`--[[`为起始, 到`]]`结束.  
```lua
--[[
line1
line2
--]]
```
通常,多行注释的结尾大家都习惯写成`--]]`, 当想要打开这个多行注释时只需要在起始位置添加一个-变成`---[[`(单行注释)即可.  
另外如果多行注释的内容中存在`]]`字符会被误判成结束位置,可以在起始和结束的括号中间添加相同的任意数量等号来排除干扰.  
```lua
--[===[
line1
line2
--]===]
```

## 变量
Lua中的变量没有固定的类型, 可以包含任何类型的值. 关于数据类型在稍后讲解.  

### 全局变量
Lua中变量默认为全局变量, 无需声明可以直接使用, 未初始化的全局变量值为nil. 当一个全局变量被赋值为nil时将回收这个变量.  
```lua
g --> nil
g = 1
g --> 1
```

### 局部变量  
局部变量要使用local关键字声明, 并且仅在它的作用域内有效.  
```lua
local a = 1;
```

### 作用域  
作用域范围就是一个代码块, 可以是一个控制结构的主体(if else for...),也可以是一个函数的主体, 或者是一个代码段(所在文件).  

> 在稍后会介绍控制结构,函数等内容.

```lua
if a == nil then -- then 代码块开始的地方
    local b = 100 -- 从声明的位置开始有效
    print(b) -- 一直到代码块结束前都为b的作用域
end -- then 代码块结束的地方
b -- nil 这里已经超出local b的范围, 使用的b属于未初始化的全局变量所以为nil

function foo()
    local s = 10 -- s作用域开始的地方
    s = s + 1 --s的作用域内
end -- foo函数主体结束的位置, 也是local s作用域结束的地方
s -- nil local s的作用域外, s属于未初始化的全局变量
```

> 另外还有一个叫非局部变量, 这是在闭包上下文当中所产生的概念, 不在这里解释.  

## 类型与值  
Lua中有8种基本类型: **nil/boolean/number/string/userdata/function/table/thread**  
使用type函数可以获得类型名称字符串:  
```lua
type(1) -- number
type("lua") -- string
```
下面依次来介绍一下各个类型:  

### Nil
nil类型只有一个值, 也写作nil, lua主要用它来与其他类型的值进行区分, 它在lua中代表一种无效值.  

### Boolean
传统的布尔值, 有true/false两种值.  
要注意Lua中对条件判断的真假并非是指true/false. Lua将nil和false以外的值都视为真(重点注意0也算真). 所以避免混淆, 当我们描述判断条件时说真/假, 而true/false是指bool值.  

### Userdata
userdata主要是用来保存C的数据(与外部交互的数据), Lua无法创建或修改这种类型的值. userdata除了赋值和相等性测试以外没有其他预定义的操作. 

### Number
Lua5.3开始, number类型在更底层分为两种:被成为integer的64位整和被成为float的双精度浮点型.(精简Lua模式下使用的是32位整型和浮点型.  

Lua支持10进制与16进制两种形式,可以使用科学计数法, 具有小数或者指数的数值会被当作浮点型值, 否则会被当作整型值. 使用函数math.type可以区分整形值和浮点型值.  
```lua
type(4) -- number
math.type(4) -- integer
type(4.4) -- number
math.type(4.4) -- float
4.57e-3 -- float
0x3b -- integer  
0xa.bp2 -- float
```

由于它们都属于number类型, 所以具有相同算术值的整型和浮点型是相等的.  
```lua
1 == 1.0 -- true
```

浮点型能够精确表示的整型值范围是[-2^53,2^53]之间, 在这个范围内可以忽略这一机制, 如果可能超出这一范围则必须严格控制每一个值的表示方式.

### String
Lua中的字符串使用byte数组来存储, 单论存储方面Lua并且不关心这些字节就行以何种方式编码. 但是, Lua中的String库是建立在8bit一个字符的基础上, 以及提供了一个utf8库. 所以在可能的情况下最好使用UTF-8编码.  

#### 字符串常量
Lua中使用一对双引号或单引号都可以声明字符串常量.唯一区别在于,使用双引号声明的字符串中出现单引号时单引号不用转义,使用单引号声明的字符串中出现双引号时双引号不用转义.    
```lua
a = "line"
b = 'line two'
```

#### 操作符
使用`#`可以获得字符串所占用的字节数(注意有些编码下字节数不等于字符数).  
```lua
a = "hello"
print(#a) -- 5
```

`..` 为字符串连接运算符.  
```lua
y = "hello" .. "lua" -- hello lua
```

#### 转义
Lua支持C风格的转义字符 **\\a,\\b,\\f,\\n,\\r,\\t,\\t,\\\\,\\',\\"**, 此外Lua5.2后还支持**\\z** 表示跳过其后的所有空白字符直到遇到第一个非空白字符.  
```lua
h = "hello /z
    world"
print(h) -- hello world
```
还可以使用\\ddd(d为至多3个的10进制数字)或\xhh(h必须为2个16进制数字组成)来表示字符. Lua5.3后还能使用\\u{h...h}来表示任意UTF-8字符(h为16进制字节码).  
```lua
s = "\u{3b1}\u{3b2}\u{3b3}" -- αβγ
```

#### 长字符串
多行字符串语法类似注释,使用一对方括号来声明长字符串常量.其中的转义序列不会被转义. 如果多行注释的第一个字符是换行符, 那么这个换行符会被忽略. 与注释相同可以在两侧的方括号之间也可以添加等量的任意数量的=来排除干扰.  
```lua
txt = [[
<html>
hellp
</html>
]]
```
对于非文本的常量最好不要乱用长字符串, 使用10进制或16进制的转义序列进行表示.  

#### 强制类型转换
Lua会自动将在需要数值的地方出现的字符串转换成数值, 在需要字符串的地方出现的字符串转为字符串.  
```lua
print(10 .. 20) -- "1020"  
a = "10" + 1 -- 11  
```
不过建议不要完全寄希望与自动转换. 可以使用函数tonumber和tostring显示的进行转换更稳妥.  

### Table
table是Lua中最主要的一种数据结构,它可以用来表示数组,集合,包等很多数据结构.table中的每一项以键值对的形式存在,可以使用任意类型的值作为索引.  
表是一种动态分配的对象,并且永远是匿名的,它与变量没有绑定关系, 程序使用指向表的引用进行操作. 对于一个表而言,当程序中不再有指向它的引用时就会被回收.  

#### 表构造器
创建table的几种方法:  

1. 空构造器  
```lua
a = {}
```

2. 列表式  
这种初始化方式索引为number 1,2....  
```lua
a = {"a","b"}
```

3. 记录式  
```lua
a = {x=10, y=20}
-- 等价于
a.x = 10
a.y = 20
-- 等价于
a["x"] = 10
a["y"] = 10
```

4. 通用式
一种通用的方式,使用方括号指定索引,列表式记录式只是它的一种特殊情况.  
```lua
a = {[1] = "a", [2] = "b"} -- 等价于上面的列表式
b = {["x"] = 10, ["y"] = 20} -- 等价于上面的记录式
```

#### 索引
table可以使用除nil外的任意类型的值进行索引.不过要注意以下几点:  

1. 未初始化的元素为nil
2. 使用.来进行访问表成员时,a.x等价于a["x"]

#### 数组与序列
使用整型作为索引就可以将table当作数组使用. 数组的索引是从1开始的, Lua中的很多机制都是从1开始.  
序列是指由键由n个正整数{1...n},并且所有值都不为nil的表.可以使用#获取序列的长度.    
```lua
a = {1,2,3,4,5,6,7,8}
#a -- 8
```

#### 遍历
对于所有类型的table,都可以使用pairs遍历,但是元素出现的顺序是随即的.  

> 关于for循环的结构在后面有讲

```lua
for k,v in pairs(t) do 
    print(k,v)
end
```
对于数组或序列可以使用ipairs保证顺序.  
```lua
for k,v in ipairs(t) do
    print(k,v)
end
```
对于序列还可以使用for循环.  
```lua
for k = 1, #t do
    print(k, t[k])
end
```

### Function
Lua中函数与其他类型的值一样属于"第一类值", 即可以保存在变量中, 表中, 也可以作为函数的参数和返回值.  
一个函数的定义需要具有函数名,参数和函数体. 调用函数时使用的参数可以与定义时的参数个数不一致,Lua会通过抛弃多余参数和将不足参数设为nil的方式来调整参数个数.  
```lua
function name(param) body end
```

#### 返回值
return 语句用来返回函数结果或结束函数的运行. 所有函数最后都有一个隐含的return.  
Lua允许返回多个值, 使用多重赋值的方式来获取多个结果.  
```lua
function foo() return 1,2 end
a,b = foo()
```
Lua会根据函数的被调用情况调整返回值的数量.  

1. 被当作单独一条语句调用时所有返回值都被丢弃
2. 当函数被作为表达式调用时,将只保留函数的第一个返回值.
```lua
a = 10 + foo() -- 11
```
3. 当函数是一系列表达式中最后一个表达式时,返回所有返回值. 一系列表达式是指:  
- 多重赋值,多的返回值会被抛弃,不足的使用nil补充  
```lua
x,y = foo() -- x = 1, y = 2
x,y = foo(),100 -- x = 1, y = 200
```
- 当参数传入函数调用  
```lua
print(foo()) -- 1 2
print(foo(),"hi") -- 1 "hi"
```
- 表构造器  
```lua
t = {foo()}
```
- return语句  
```lua
function foo2()
    return foo()
end
```
用括号括起来强制返回1个结果.  
```lua
a = (foo(1,2)) -- 1
```

Lua语法规定, return必须是所在代码块的最后一个语句, 因为return以后的语句无法执行.  
```lua
a = 10
if a == 10 then
    return
    a = a + 1 -- 如果有这行会报错
end
```

但在某些时候在代码块中间使用return很有用(比如调试), 可以显示的使用包含return的do(制造一个代码块).  
```lua
function foo()
    return -- 这么写会报错
    do return end -- Ok
    -- other statements
end
```
#### 变长参数
参数列表使用`...`来表示参数是可变长的.  
```lua
function add (...)
    local a, b, c = ...
    return a
end
```
在函数内使用变长参数时,要么使用多重赋值,要么利用table.pack函数将所有数值打包成table.  

#### 全局函数
Lua中最常见的定义函数的方式是:  
```lua
function foo(x) return x end
```
这种形式默认是保存在全局变量中. 它实际上是一种语法糖, 它等价于:  
```lua
foo = function(x) return x end
```
Lua中所有函数都是匿名的, 当我们讨论函数名, 比如print, math.sin等, 都是指保存该函数的变量. 

#### 第一类值  
相对于全局函数, 定义局部函数的方式是:  
```lua
local function foo2 (x) return x end
```
它是一种语法糖, 会被展开成:  
```lua
local foo2; foo2 = function(x) return x end
```
但是使用间接递归调用时, 不要使用语法糖, 要明确的向前声明, 否则会有问题.  
```lua
local f -- 向前声明
local function g()
	some code() f() some code()
end

function f() 
	some code() g() some code()
end
```

也可以将函数储存在表中(Lua中大部分的库就是使用这种形式):  
```lua
Lib = {}
Lib.foo = function(x,y) return x + y end
Lib.goo = function(x,y) return x - y end
-- 或者
Lib = {
	foo = function(x,y) return x + y end
	goo = function(x,y) return x - y end
}
-- 或者
Lib = {}
function Lib.foo (x,y) return x + y end
function Lib.goo (x,y) return x - y end
```

函数作为参数:  
```lua
network = {
	{name = "grauna", IP = "201.26.30.34"},
	{name = "lua",IP = "201.26.30.35"},
}
table.sort(network, function (a,b) return (a.name > b.name) end)
```

函数作为返回值:
```lua
foo = function() return print end
```

#### 词法定界
这是Lua函数的一种特性, 指一个被其他函数B包含的函数A, 被包含的函数A可以访问包含其的函数B的所有局部变量.  
```lua
function newCounter()
	local count = 0
	return function()
			count = count + 1
			return count
		   end
end
```
在返回的函数中, 可以使用的count是newCounter的局部变量, 内层函数中count也叫非局部变量, 也叫upvalue.  
而返回的其实也并非是一个函数而是一个闭包, 闭包就是函数外加能够使函数访问upvalue的机制. 再看下面的使用:    
```lua
c1 = newCounter()
c2 = newCounter()
print(c1()) -- 1
print(c1()) -- 2
print(c2()) -- 1
print(c1()) -- 3
```
两次调用返回的是两个不同的闭包, 它们拥有不同upvalue(count). 所以互不影响.  

### Thread

## 运算符  

### 逻辑运算符  
Lua支持的逻辑运算符 **and/or/not** (与/或/非). 遵循短路求值.  

- and: 第一个操作数为假则返回第一个操作数, 否则返回第二个操作数.  
- or: 低一个操作数为真则返回低一个操作数,否则返回第二个操作数.  
- not: 返回一个相反的boolean.  

### 算术运算符  
Lua支持, 加(+)减(-)乘(\*)除(/)模(%)幂(^)整除(//).整除是指对得到的商向负无穷取整.  
其中,除了除法与幂运算以外, 如果两个操作数都是整形值,那么结果也是整型值,否则结果是浮点型. 而除法与幂运算结果一定为浮点型.  

### 关系运算符
**<,>,<=,>=,==,~=**.  
== 用作相等性测试,~= 用作不等性测试, 这两个运算符两边可以类型不同, 如果类型不同认为不相等, 类型相同再比较值. 其他运算符两边必须类型相同.  

### 位运算
与(&), 或(|), 左移(<<), 右移(>>), 取反(~), 异或(~).  
其中左右移是逻辑左移(移出的舍弃并补0). 注意取反和异或符号相同依靠上下文区分(取反是单目运算符,异或是双目).  

### 运算符优先级  
从上到下, 由高到低.  

> ^(幂)  
> -, #, ~, not(一元运算符)  
> \*, /, //, %  
> ..(字符串连接)  
> <<, >>(位移)  
> &(按位与)  
> ~(按位异或)  
> |(按位或)  
> <, >, <=, >=, ~=, ==  
> and  
> or  

## 控制语句  
if分支语句, while, repeat, for 循环语句. 主要认识一下语法.  

### if
```lua
if a < 0 then
    a = 0
else if a >  100 then
    a = 100
else
    a = a + 1
end
```

### while  
```lua
while a[i] do
    print(a[i])
    i = i + 1
end
```

### repeat
```lua
repeat
    local line = io.read()
until line
```
repeat循环的判断条件属于repeat作用域范围内.  

### 数值型for
数值型for, 格式为: for var = exp1, exp2, exp3 do.  
var 是for自动声明的变量, 作用域仅限for  
exp1 为起始值, exp2 为结束值, exp3 为步长如果省略默认为1.  

```lua
for v = 0, 10, 1 do
    print(v)
end
```
### 泛型for
结合迭代器(pairs, ipairs, os.lines等)使用的for.  
```lua
for k, v in pairs(t) do
    print(k,v)
end
```

### break
break可以从控制语句的结构中跳出, 并且Lua没有continue.  


### goto
注意标签的写法和使用就行.  
```lua
a = 1
::redo::
    print(a)
    a = a + 1
    if a == 10 then
        return
    end
goto redo
```

## 文件读写  
文件读写功能来自IO库, 提供了两种模型, 简单I/O模型与完整I/O模型.  

### 简单I/O模型  
每个程序都有两个流, 输入流和输出流. I/O库将输入流默认初始化进程的标准输入(stdin), 将输出流初始化为进程的标准输出(stdout).  
简单I/O模型就是使用 io.input 和 io.output函数改变使用的输入输出到指定文件, 再使用io.read, io.write等函数对流进行读写.  
```lua
io.output("hello.txt")
io.write("write some text")
```
主要是io库各个函数怎么调用.  

### 完整I/O模型  
该模型与C语言的文件读写几乎一样了.  
当你向对文件进行读写需要先使用 io.open 打开文件获得文件流, 在使用文件流对象提供的方法(read,write...)来操作文件流, 最后要关闭(close)流.  
```lua
f = io.open("hello.txt","a")
f:write("f:write")
f:close()
```
主要是io库和file对象各个方法调用.  

