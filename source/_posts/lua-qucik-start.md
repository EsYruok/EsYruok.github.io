---
title: Lua快速入门
categories: 编程
tag: Lua
abbrlink: 71a33465
date: 2024-02-15 21:39:45
---
Lua 是一种强大、高效、轻量级、可嵌入的脚本语言, 本文带您快速的认识它.  <!--more-->  
本文需要您懂得一些编程通用的基本概念(什么是注释,什么是变量等). 本文的目标不在于完全掌握Lua, 而是让您进行从不认识到认识的一个过程. 文章内容及其精炼, 适合那些只是想要使用Lua书写一些简单工具脚本, 或是想要看懂别人的一些简单的代码即可的同学. 如果您想使用Lua去做程序设计, 那么建议您可以通过本篇文章全面了解Lua后再详细阅读其他教程资料, 详细了解Lua的内部原理与一些复杂的机制, 帮助您编写出优秀的代码.  
Lua官方网站: [Lua](https://www.lua.org/)  

## 安装Lua  
Lua属于脚本语言, 想要执行Lua需要安装Lua解释器.  
```sh
# ArchLinux
sudo pacman -S lua
```
其他操作系统可以参考官网.  
Lua 解释器有两种使用方式, 交互式与非交互式.  

- 交互式是指直接输入`lua`命令, 会进入一个交互式界面, 它会及时的执行你输入的内容 (了解python的应该非常熟悉)  
- 非交互式是指将你的代码储存在脚本文件中, 再传递给解释器. 例如: `lua hello.lua`  

## 标识符
所有语言都离不开标识符, 它代表着变量名,函数名等一切您要自定义的名称.  
Lua要求标识符是以 **任意字母,数字和下划线组成,并且不能以数字开头,不能为关键字,大小写敏感** 的字符串.  

> Lua 关键字:  
> and break do else elseif  
> end false goto for function  
> if in local nil not  
> or repeat return then true  
> until while  

## 注释  
常规的两种, 单行注释和多行注释.  
```lua
-- 单行注释

--[[
line
多行注释
]]
```
一般来说, 多行注释的结尾大家都习惯写成`--]]`方便打开关闭注释. 另外, 多行注释的两个`[`和`]`中间是可以添加任意数量个等号, 但是前后必须相同.    
```lua
--[===[
line1
line2
--]===]
```

## 类型与值  
lua中有8种基本类型: nil/boolean/number/string/userdata/function/thread/table.  
下面依次来介绍一下:  

#### Nil
nil类型只有一个值, 也写做nil. 它在lua中代表一种无效值.  

#### Boolean
传统的布尔值, 只有 true/false 两个值.  
不过Lua中对真假的判断并非是指的true/false. Lua将nil和false以外的值都视为真. 所以我们对条件判断说成真/假, 而说true/false仅仅是指boolean类型的两个值.    

#### number


## 变量
变量没有固定的类型, 可以包含任何类型.  
```lua
a = 10;
a = "hello lua"
```
全局变量可以直接使用, 未初始化的全局变量值为nil.  



