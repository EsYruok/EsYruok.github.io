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
`__vectorcall`是微软所特有的调用约定，它比`fastcall`使用更多的寄存器，它的宗旨是尽可能的使用寄存器传参，使用SSE指令加快计算速度。`__vectorcall`只支持含有`Streaming SIMD Extensions 2 (SSE2)`及以上的CPU上。`__vectorcall`可以传递整型（整数，指针，引用，大小合适的结构体联合体等），向量类型（浮点型，SIMD向量类型），HVA type。  
x64上，是对x64`fastcall`的扩展，由调用者清理堆栈，前四个整数类型参数依照位置对应使用`RCX,RDX,R8,R9`传递，`this`指针被视为第一个整数参数。前六个向量类型依照位置对应使用`SSE0-5`寄存器（XMM/YMM）。其余参数从右向左压入堆栈。栈上同样拥有shadow区域。  
```c++
typedef struct {
   __m128 array[2];
} hva2;    // 2 element HVA type on __m128

typedef struct {
   __m256 array[4];
} hva4;    // 4 element HVA type on __m256

// Example 1: All vectors
// Passes a in XMM0, b in XMM1, c in YMM2, d in XMM3, e in YMM4.
// Return value in XMM0.
__m128 __vectorcall
example1(__m128 a, __m128 b, __m256 c, __m128 d, __m256 e) {
   return d;
}

// Example 2: Mixed int, float and vector parameters
// Passes a in RCX, b in XMM1, c in R8, d in XMM3, e in YMM4,
// f in XMM5, g pushed on stack.
// Return value in YMM0.
__m256 __vectorcall
example2(int a, __m128 b, int c, __m128 d, __m256 e, float f, int g) {
   return e;
}

// Example 3: Mixed int and HVA parameters
// Passes a in RCX, c in R8, d in R9, and e pushed on stack.
// Passes b by element in [XMM0:XMM1];
// b's stack shadow area is 8-bytes of undefined value.
// Return value in XMM0.
__m128 __vectorcall example3(int a, hva2 b, int c, int d, int e) {
   return b.array[0];
}

// Example 4: Discontiguous HVA
// Passes a in RCX, b in XMM1, d in XMM3, and e is pushed on stack.
// Passes c by element in [YMM0,YMM2,YMM4,YMM5], discontiguous because
// vector arguments b and d were allocated first.
// Shadow area for c is an 8-byte undefined value.
// Return value in XMM0.
float __vectorcall example4(int a, float b, hva4 c, __m128 d, int e) {
   return b;
}

// Example 5: Multiple HVA arguments
// Passes a in RCX, c in R8, e pushed on stack.
// Passes b in [XMM0:XMM1], d in [YMM2:YMM5], each with
// stack shadow areas of an 8-byte undefined value.
// Return value in RAX.
int __vectorcall example5(int a, hva2 b, int c, hva4 d, int e) {
   return c + e;
}

// Example 6: HVA argument passed by reference, returned by register
// Passes a in [XMM0:XMM1], b passed by reference in RDX, c in YMM2,
// d in [XMM3:XMM4].
// Register space was insufficient for b, but not for d.
// Return value in [YMM0:YMM3].
hva4 __vectorcall example6(hva2 a, hva4 b, __m256 c, hva2 d) {
   return b;
}
```
x86上，由被调用者清理堆栈，前两个整数参数按相应位置使用`ECX,EDX`传递，`this`指针被视作第一个整型参数。前六个向量参数按照相应位置使用`SSE0-5`寄存器（XMM/YMM）。其余参数从右向左压入堆栈。  
```c++
typedef struct {
   __m128 array[2];
} hva2;    // 2 element HVA type on __m128

typedef struct {
   __m256 array[4];
} hva4;    // 4 element HVA type on __m256

// Example 1: All vectors
// Passes a in XMM0, b in XMM1, c in YMM2, d in XMM3, e in YMM4.
// Return value in XMM0.
__m128 __vectorcall
example1(__m128 a, __m128 b, __m256 c, __m128 d, __m256 e) {
   return d;
}

// Example 2: Mixed int, float and vector parameters
// Passes a in ECX, b in XMM0, c in EDX, d in XMM1, e in YMM2,
// f in XMM3, g pushed on stack.
// Return value in YMM0.
__m256 __vectorcall
example2(int a, __m128 b, int c, __m128 d, __m256 e, float f, int g) {
   return e;
}

// Example 3: Mixed int and HVA parameters
// Passes a in ECX, c in EDX, d and e pushed on stack.
// Passes b by element in [XMM0:XMM1].
// Return value in XMM0.
__m128 __vectorcall example3(int a, hva2 b, int c, int d, int e) {
   return b.array[0];
}

// Example 4: HVA assigned after vector types
// Passes a in ECX, b in XMM0, d in XMM1, and e in EDX.
// Passes c by element in [YMM2:YMM5].
// Return value in XMM0.
float __vectorcall example4(int a, float b, hva4 c, __m128 d, int e) {
   return b;
}

// Example 5: Multiple HVA arguments
// Passes a in ECX, c in EDX, e pushed on stack.
// Passes b in [XMM0:XMM1], d in [YMM2:YMM5].
// Return value in EAX.
int __vectorcall example5(int a, hva2 b, int c, hva4 d, int e) {
   return c + e;
}

// Example 6: HVA argument passed by reference, returned by register
// Passes a in [XMM1:XMM2], b passed by reference in ECX, c in YMM0,
// d in [XMM3:XMM4].
// Register space was insufficient for b, but not for d.
// Return value in [YMM0:YMM3].
hva4 __vectorcall example6(hva2 a, hva4 b, __m256 c, hva2 d) {
   return b;
}
```
## __clrcall
`__clrcall`是.Net当中的调用规范，这个是在C++中使用.Net时用的，用来指定函数只能从托管代码调用。由调用者平衡堆栈，参数由左向右入栈。  
关于这类调用约定的传参和平栈方式未在微软的文档找到，是通过其他资料查阅，准确性有待验证。  

参考文档：[Argument Passing and Naming Conventions](https://learn.microsoft.com/en-us/cpp/cpp/argument-passing-and-naming-conventions?view=msvc-170)


---