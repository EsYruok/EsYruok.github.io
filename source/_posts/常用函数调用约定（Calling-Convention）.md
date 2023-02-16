---
title: 常用函数调用约定（Calling Convention）
date: 2023-02-16 21:40:04
tags:
---
介绍windows下的调用约定，常用的有__stdcall，__cdecl，__thiscall，__fastcall以及不常用的__vectorcall，__clrcall。
<!-- more -->

## __stdcall
Win32 API使用的调用约定。由被调用者清理堆栈，参数从右向左压入堆栈。  

## __cdecl
C/C++程序默认的调用约定。由调用者清理堆栈，支持变参函数，参数从右向左压入堆栈。  

## __thiscall
C++类成员函数使用的调用约定（固定参数）。由被调用者清理堆栈，所以不支持变参函数，`this`指针通过`ECX`传递，其余参数从右向左压入堆栈。  
变参类成员函数使用的是`cdecl`调用约定。this指针最后压入堆栈  

## __fastcall
一种尽可能使用寄存器传参的调用约定。  
在x86下，由被调用放清理堆栈，从左到右前两个参数通过`ECX,EDX`传递，其余参数从右向左压入堆栈。可以看成是`cdecl`进行某种变化。  
在x64下，`fastcall`是默认的调用约定，由调用者清理堆栈，支持变参函数，前4个参数如果是整型则使用`RCX,RDX,R8,R9`传递，如果是浮点型则使用`XMM0,XMM1,XMM2,XMM3`传递，其余参数从右向左压入堆栈。  
前4个参数虽然会通过寄存器传递，但是仍然会在堆栈上留有对应的空间（shadow）让函数来存放使用寄存器来传递的参数。所以其余参数在入栈后仍然是在shadow之后。  
注意一下，前4个参数是按固定位置来使用寄存器，不是按照顺序来使用寄存器，可能说的比较抽象，下面放几个微软提供的例子很好理解。  
```c++
func1(int a, int b, int c, int d, int e, int f);
// a in RCX, b in RDX, c in R8, d in R9, f then e pushed on stack

func2(float a, double b, float c, double d, float e, float f);
// a in XMM0, b in XMM1, c in XMM2, d in XMM3, f then e pushed on stack

func3(int a, double b, int c, float d, int e, float f);
// a in RCX, b in XMM1, c in R8, d in XMM3, f then e pushed on stack

func4(__m64 a, __m128 b, struct c, float d, __m128 e, __m128 f);
// a in RCX, ptr to b in RDX, ptr to c in R8, d in XMM3,
// ptr to f pushed on stack, then ptr to e pushed on stack
```
## __vectorcall
`__vectorcall`是微软所特有的调用约定，继承于`fastcall`进行扩展，它们非常相似，只是`__vectorcall`尽可能的使用寄存器传参。  
x64上，由调用者清理堆栈，整数参数尽可能使用`RCX,RDX,R8,R9`传递，`this`指针算第一个整型参数。浮点参数尽可能使用`XMM0~XMM5`传递。其余参数从右向左压入堆栈。栈上同样拥有shadow区域。  
x86上，由被调用者清理堆栈，证书参数尽可能使用`ECX,EDX`传递，`this`指针算第一个整型参数。浮点参数尽可能使用`XMM0~XMM5`传递。其余参数从右向左压入堆栈。  

## __clrcall
这个是.Net平台用的，用来指定函数只能从托管代码调用。实现方式与`__fastcall`几乎相同。