---
title: Lua快速入门
categories: 编程
tag: Lua
abbrlink: 71a33465
date: 2024-02-15 21:39:45
---
对Lua基本知识的一个精炼总结, 快速认识Lua.  <!--more-->  
写在前面:  

0. 作者学习Lua目的是更好的使用Neovim, 所以文章内容会比较浅显, 如果您想要使用Lua进行程序设计, 模块开发, 最好能够阅读更详细的教学资料, 详细的了解Lua的一些特性与原理.  
0. 文章没有对函数的使用方法进行解释, 参考手册全部都可以查到.    
0. 本文没有对一些编程通用的基本概念进行讲解(什么是注释, 变量等), 您最好对编程有所认识.  

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
Lua标识符要求:

- 以任意字母,数字和下划线组成  
- 不能以数字开头  
- 不能为关键字  

是大小写敏感的.  

> Lua 关键字:  
> and | break | do | else | elseif | end | false | goto | for | function  
> if | in | local | nil | not | or | repeat | return | then | true | until | while  

## 注释  

### 单行注释
```lua
-- line
```

### 多行注释
```lua
--[[
line1
line2
]]
```
一般来说, 多行注释的结尾大家都习惯写成`--]]`方便打开关闭注释. 另外, 多行注释的`[`和`]`中间是可以添加任意数量个等号来排除被注释内容有注释字符的干扰, 但是前后必须相同.  
```lua
--[===[
line1
line2
--]===]
```

## 类型与值  
Lua中有8种基本类型: nil/boolean/number/string/userdata/function/table/thread.  
使用type函数可以获得类型名称字符串:  
```lua
type(1) -- number
type("lua") -- string
```
下面依次来介绍一下各个类型:  

### Nil
nil类型只有一个值, 也写作nil, 它在lua中代表一种无效值.  

### Boolean
传统的布尔值, 有 true/false 两个值. 不过Lua中对真假的判断并非是指true/false. Lua将nil和false以外的值都视为真. 所以我们描述判断条件时说成真/假, 而使用true/false时仅仅是指bool值.  

### Number
整数小数都属于number类型.  
```lua
4 -- 整数
4.4 -- 小数
4.57e-3 -- 科学计数法
0x3b -- 16进制  
0xa.bp2 -- 16进制的科学计数法
```
Lua中数值的比较是算术值的比较, 所以, 4 == 4.0 为真.  

> Lua分为两种编译模式, 标准Lua的数值类型使用64bit储存, 精简Lua使用32bit储存.  

> Lua5.3以后number在更底层有了整型与浮点型的区别, 整数使用整型存储, 小数使用浮点型存储. Lua中对数值的计算可能存在底层类型的改变, 比如, 1+2=3结果为整型, 而1+2.0=3.0结果为浮点型, 它会先将1转换成浮点型再计算. 那么问题就是, 同样都是64位,整型和浮点型的表示范围是不同的, 由于浮点数的精度无法表示所有的整数, 比如`math.maxinteger + 2.0 == math.maxinterget + 1.0` 会得到true的结果.  
> 所以, 数值范围在[-2^53, 2^53]之间(这是浮点型可以精确表示的整数范围)可以忽略这一机制, 如果可能超出这一范围则必须严格控制每一个值的表示方式.  

### String
Lua中字符串使用"或'包裹都可以, 区别在于使用"或'时的转义.  
```lua
y = "One 'good' \"Day\"" -- One 'good' "Day"
z = 'One \'good\' "Day"' -- One 'good' "Day"
```
`..` 为字符串拼接运算符.  
```lua
y = "hello" .. "lua" -- hello lua
```

#### 转义
Lua支持C风格的转义字符 **\\a,\\b,\\f,\\n,\\r,\\t,\\t,\\\\,\\',\\"**, 此外还支持**\\z** 表示跳过其后的所有空白字符直到遇到第一个非空白字符.  
```lua
h = "hello /z
    world"
print(h) -- hello world
```
可以使用\\ddd(十进制 \\10 )或\xhh(十六进制 \\x41)来表示字符. 使用\\u{h...h}来表示UTF-8字符.  
```lua
a = "\97\98\99" -- abc
b = "\x41\x42\x43" -- ABC
s = "\u{3b1}\u{3b2}\u{3b3}" -- αβγ
```

#### 长字符串
多行字符串语法类似注释,不会对其中的转义字符转义. 与注释相同两侧的[[,]]之间也可以添加等量的=.  
```lua
txt = [[
line1
line2\tline2
]]
```

#### 强制类型转换
Lua会自动在需要字符串的时候将数值转称字符串, 在需要数值时将字符串转成数值.  
```lua
print(10 .. 20) -- "1020"  
a = "10" + 1 -- 11  
```

### Userdata
userdata主要是用来保存C的数据(与外部交互的数据), Lua无法创建或修改这种类型的值. userdata除了赋值和相等性测试以外没有其他预定义的操作.  

### Table
类似数组, 集合, 索引可以是任意类型的值, 同一个table中可以储存不同类型的值, 也可以使用不同类型的索引.  
```lua
a = {} -- 一个空的table
b = a -- a只是table的一个引用可以赋值给其他变量, 此时b与a是同一个table
k = "x"
a[k] = 10 -- 键:"x" 值:10
a[20] = "great" -- 键:20 值:"great"
a["x"] -- 10
a.x = 10 -- 等价于 a["x"] = 10
a[1] -- 未初始化的索引为nil
```
索引使用的是算术值, 也就是2和2.0是同一个键. 当table没有指向它的引用时会自动删除. 我们平时常见的库其实也是table, math.sin 就等价于 math["sin"] 这个键的值为一个function类型的值.  

#### 构造table
table的几种初始化方式:  
```lua
a = {} -- 空构造器, 一个空的table
a = {"a", "b"} -- 初始化列表, 索引为1,2
a = {x=10, y=20} -- 记录式初始化, 等价于 a.x = 10  a.y = 20
```

#### 序列
当一个table只以数值为索引(1,2,3...), 且中间没有空洞, 对这种table叫做序列. 可以使用长度运算符#获取序列的长度. 非序列的table使用#获得的结果不可靠.  
```lua
a = {1,2,3,4,5,6,7,8}
#a -- 8
```

#### 遍历
```lua
t = {"a", "b"}
for k,v in pairs(t) do 
    print(k,v))
end
```
这种方式元素出现顺序是随即的. 使用ipars可以保证顺序, 但是只能用在序列上.  

### Function

#### 函数定义
```lua
function foo(x) return x end
-- 等价于
foo = function(x) return x end
type(foo) -- function
```
定义的含义实际上就是创建一个function类型的值并赋值给一个变量. 函数类型于其他基本类型一样都属于"第一类值", 可以保存在变量或表中.  

#### 参数
调用函数传参时, 多传的实参被抛弃, 不足时补nil.  

#### 返回值
return 返回结果或退出函数(整个脚本其实也可以被看成一个函数). 所有的函数最后一行都有一个隐含的return.  
return 必须是所在代码块的最后一个语句, 否则会报错.  
```lua
a = 10
if a == 10 then
    return
    a = a + 1 -- 有这行会报错
end
```
如果有特殊情况必须如调试需要这么做可以使用do end包裹(制造一个代码块).  
```lua
function foo()
    return -- 这么写会报错
    do return end -- Ok
    -- other statements
end
```

return可以返回多个返回值.  
```lua
function foo(a, b)
    return a, b
end

c, d=foo(10,20)
```
返回多个值的函数单独被调用时抛弃所有返回值, 被当作非最后一个表达式时返回第一个值, 最后一个表达式返回所有值, 多重赋值时, 不够的用Nil  
```lua
function foo2()
    return 1,2
end
foo2() -- 没有返回值
x,y,z = foo2(),20 -- 1,20,nil 返回1个值
x,y,z = foo2() -- 1,2,nil 返回所有值
```
当函数作为参数最后一个参数也返回所有值.  
```lua
print(foo2(),10) -- 返回1个值
print(10,foo2()) -- 返回所有值
```
作为表构造参数时返回所有值.  
```lua
t = {foo2()}
```
用作返回值时也会返回所有值.  
```lua
function foo3()
    return foo2()
end
```
用括号括起来强制返回1个结果.  
```lua
a = (foo2()) -- 1
```

#### 变长参数
变长参数用...来表示.  
```lua
function add (...)
    local a, b, c = ...
    return a
end
```
#### 非全局函数
储存在局部变量或表字段中就是非全局函数. 大部分的库就是这种形式.  
```lua
local foo = function(x,y) return x,y end
-- 等价于
local function foo(x,y) return x,y end

lib = {}
lib.foo = function(x,y) return x,y end
-- 或者
lib = {
    foo = function(x,y) return x,y end
}
-- 或者
function lib.foo(x,y) return x,y end
```
使用局部递归时要注意
```lua
local fact=function(n)
    if n==0 then return 1
    else return n*fact(n-1)
    end
end
```
当解释到fact(n-1)时, 由于局部变量fact还没创建, 解释器将会判断寻找全局的fact. 需要使用先定义局部变量(向前声明)的方式.  
```lua
local fact
fact=function(n)
    if n==0 then return 1
    else return n*fact(n-1)
    end
end
```
实际上Lua展开函数定义语法糖时:  
local function foo(params) body end
会被展开成
local foo; foo = function(params) body end
所以使用这种语法定义递归函数没有问题. 但是, 间接递归的情况下必须使用向前声明的形式.  

### thread

## 变量
变量没有固定的类型, 可以包含任何类型.  
```lua
a = 10  -- number 类型
a = "hello lua"  -- string 类型
```

### 全局变量
Lua中变量默认为全局变量, 可以直接使用, 未初始化的全局变量值为nil. 当一个全局变量被赋值为nil时将回收这个变量.  
```lua
g = 1
a = g -- 1
a = b -- nil
```

### 局部变量  
局部变量要使用local声明, 并且局部变量仅在声明它的代码块内有效.  
```lua
local a = 1;
```

### 作用域  
一个代码块就是一个作用域, 可以是一个控制结构的主体(if else for...),也可以是一个函数的主体, 或者是一个代码段.  
```lua
if a == nil then
    local b = 100;
    print(b)
end
b -- nil 这里已经超出local b的范围

function foo()
    local s = 10
    s = s + 1 --s的作用域内
end
s -- nil local s的作用域外
```

## 运算符  

### 逻辑运算符  
Lua支持的逻辑运算符 **and/or/not** (与/或/非). 它们短路求值.  
and 第一个操作数为false则返回第一个操作数, 否则返回第二个操作数.  
or低一个操作数为true则返回低一个操作数,否则返回第二个操作数.  
not 返回一个相反的boolean.  

### 算术运算符  
常见的加(+)减(-)乘(\*)除(/)模(%)幂(^) 还有一个整除(//).  
除运算是可以得到的是浮点数, 整除是结果向负无穷取整.  

### 关系运算符
**<,>,<=,>=,==** Lua中不等于写成**~=**.  
==,~= 两边可以类型不同, 如果类型不同认为不相等, 类型相同再比较值. 其他运算符两边必须类型相同.  

### 位运算
与(&), 或(|), 左移(<<), 右移(>>), 取反(~), 异或(~).  
其中左右移是逻辑左移(移出的舍弃并补0). 取反和异或符号相同依靠上下文区分(取反是单目运算符,异或是双目).  

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

### 完整I/O模型  
该模型与C语言的文件读写几乎一样了.  
当你向对文件进行读写需要先使用 io.open 打开文件获得文件流, 在使用文件流对象提供的方法(read,write...)来操作文件流, 最后要关闭(close)流.  
```lua
f = io.open("hello.txt","a")
f:write("f:write")
f:close()
```

