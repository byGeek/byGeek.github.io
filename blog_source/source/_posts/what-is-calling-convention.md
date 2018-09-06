---
title: 什么是调用惯例
date: 2018-09-05 16:46:37
tags:
- c
categories:
- coding
keywords:
description:
---

调用惯例（Calling Convention）是指函数调用时发生的一些约定。wikipedia定义如下：
> a calling convention is an implementation-level (low-level) scheme for how subroutines receive parameters from their caller and how they return a result. Differences in various implementations include where parameters, return values, return addresses and scope links are placed, and how the tasks of preparing for a function call and restoring the environment afterward are divided between the caller and the callee.
> 

<!--more-->

经常在一些代码中看到：

```c++
void __stdcall foo(int a, int b);
void __cdelc bar(double a, double b);
```

或者在c#中利用P/Invoke一些native代码时：

```csharp
[DllImport("User32.dll", EntryPoint = "SetForegroundWindow", CallingConvention = CallingConvention.StdCall)]
public static extern bool SetForegroundWindow(IntPtr hWnd);
```

代码中的`__stdcall`和`__cdecl`即调用惯例。在csharp中P/Invoke时默认的CallingConvention是StdCall。



## 具体指什么

一般来说，调用惯例一般会规定以下的内容：

- 函数参数的传递顺序和方式

  函数参数的传递可以有很多种方式，最常见的一种是通过栈传递。函数的调用方将参数压入栈中，函数自己再从栈中将参数取出。对于有多个函数参数的函数，调用惯例要规定函数调用将参数压栈的顺序：是从左到右还是从右到左。有些调用惯例还允许使用寄存器传递参数，以提高性能。

- 栈的维护方式

  在函数将参数压栈之后，函数体会被调用，此后需要将被压入栈中的参数全部弹出，以使得栈在函数调用前后保持一致。这个弹出的工作可以由函数的调用方来完成，也可以由函数本身来完成。

  cdecl由函数调用方来完成，stdcall由函数本身来完成。在WIN32 API中，除了变长的参数的函数之后，都是用的调用管理是stdcall。因为变长的函数参数，函数本身并不知道参数的个数，所以只能由函数调用方来处理出栈。

- 名字修饰(name-mangling)的策略

  为了链接的时候对调用管理进行区分，调用惯例要对函数本身的名字进行修饰，不同的调用惯例有不同的名字修饰策略。

  cdecl采用的名字修饰策略是： 下划线+函数名

  stdcall: 下划线+函数名+@+参数的字节数，如函数int func(int a, double b)的修饰名为_func@12



《程序员的自我修养--链接，装载与库》一书中有详细的解释（这本书是对底层的一些概念解释的很清楚，推荐阅读）。摘录一图如下：

{% asset_img calling-convention.png %}



