---
title: c/c++ 中的变长参数
date: 2018-09-28 16:29:09
tags:
- c/c++
categories:
- coding
keywords:
description:
---

## 一个bug引起的思考

最近碰到一个bug，是在一个log模块，在使用`vsprintf_s`函数时发生access deny错误。奇怪的是在Debug模式下没有问题，切换到Release模式下就会重现。我把问题**简单化后的代码**如下：

<!--more-->

```cpp
#include "stdafx.h"
#include <stdio.h>
#include <string>
#include <stdarg.h>
#include <iostream>

void test_var_args(const char* format, ...)
{
	va_list args;
	va_start(args, format);

	char buf[512];
	int len = vsprintf_s(buf, 512, format, args);
	//buf[len] = '\0';
	printf(buf);
	va_end(args);
}

void test_var_args() {
	std::string s("robert");
	//const char* str = "robert";
	test_var_args("%s age is %d", s, 25);

	std::cout << std::endl;
}


int main()
{
	test_var_args();
	getchar();
    return 0;
}
```

代码很简单。使用vs2017 在debug模式下编译，成功运行，当然输出的字符串有问题，先暂时忽略。在release下直接报如下的错误

{% asset_img access_violation.png %}

当然了，因为上面的代码是简化后的代码，所以直接看输出就能定位到是变长参数带来的内存访问问题。

我在debug实际代码的过程中，开始并没有发现问题的根源。而是纠结在为什么debug模式能work而release不行。于是面向stackoverflow编程([Common reasons for bugs in release version not present in debug mode](https://stackoverflow.com/questions/1762088/common-reasons-for-bugs-in-release-version-not-present-in-debug-mode))。在试过了修改release模式下的配置，使得尽量与debug下一致（比如使用相同的Runtime Library，不启用优化等），无果。后来看到一个答案，大致意思是， 除了性能上的影响之外，debug和release模式配置无影响，debug working而release not working的原因很可能是bug其实一直都在，只是在debug下没有注意到一些细节，应该去分析代码。

再次去debug下分析代码，单步调试到vsprintf_s，发现格式化输出的字符串有一个变量明显不对(为什么开始没有注意到！！！)，类似上面的简单化代码的输出效果。定位到问题就好了，这个应该是变长参数传递的锅。再次面向stackoverflow编程([Trouble with va_list c++](https://stackoverflow.com/questions/36478609/trouble-with-va-list-c))。

>  you can't portably extract `std::string` from variadic function arguments. Only trivial types are fully supported, and for strings you have to use `char*`. `std::string` is not trivial, because it has non-trivial constructor and destructor. Some compilers do support non-trivial types as arguments for such functions, but others do not, so you shouldn't try this.
The last, but not the least: variadic functions have no place in C++ world, even for assignments.

大致意思是在可变参数传递中只支持简单类型(trivial types), `std::string`传递会有问题。果不其然，查看call stack的调用关系，其中有一步，变长参数传递中，有一个参数直接传递的`std::string`类型。将其修改调用string的`c_str()`方法，问题解决。



这个问题解决了，心里有了另外的问题，变长参数是如何实现的呢？

## c中的变长参数实现

C语言中提供了变长函数声明，使用省略号`...`(ellipses)表示参数是可变的。通过位于`stdarg.h`头文件中定义的三个宏`va_start, va_end, va_args`和一个类型`va_list`来实现。先看个例子。

```c
#include <stdarg.h>

int test_sum(int num, ...) {
	va_list list;
	va_start(list, num);
	int i = 0;
	int sum = 0;
	while (i < num)
	{
		int value = va_arg(list, int);
		sum += value;
		++i;
	}
	va_end(list);
	return sum;
}

int main(){
    int sum = 0;
	sum =test_sum(1, 1);
	sum =test_sum(3, 1, 2, 3);
	sum =test_sum(3, 1, 2);
	sum =test_sum(4, 1, 2, 3, 4);
}
```

其中`test_sum`定义中第一个形参为后面的参数个数，第二个形参为可变参数。之所以需要第一个形参，是因为在`while`循环中去通过`va_arg`去取得参数。那么问题来了，函数的参数是如何传递的呢？

可参考我的另外一边文章: [什么是调用惯例](https://bygeek.cn/2018/09/05/what-is-calling-convention/)。
在C中通过cdecl方式,参数是从右至左的顺序入栈。以`test_sum(3,1,2,3)`调用为例，参数入栈后结果示意。注意，栈是高地址向低地址增长的。

address | stack
--- | ---
high_addr | 3
 | 2
 | 1
 | 3
low_addr(top) | ret_addr

下面来看下几个宏定义的声明和定义。摘自MSDN
```C
type va_arg(
   va_list arg_ptr,
   type
);
void va_copy(
   va_list dest,
   va_list src
); // (ISO C99 and later)
void va_end(
   va_list arg_ptr
);
void va_start(
   va_list arg_ptr,
   prev_param
); // (ANSI C89 and later)
void va_start(
   arg_ptr
);  // (deprecated Pre-ANSI C89 standardization version)
```

定义摘录自`stdarg.h`和`vadefs.h`。
```c
typedef char* va_list;

#define va_start __crt_va_start
#define va_arg   __crt_va_arg
#define va_end   __crt_va_end
#define va_copy(destination, source) ((destination) = (source))

#define __crt_va_start_a(ap, v) ((void)(ap = (va_list)_ADDRESSOF(v) + _INTSIZEOF(v)))
#define __crt_va_arg(ap, t)     (*(t*)((ap += _INTSIZEOF(t)) - _INTSIZEOF(t)))
#define __crt_va_end(ap)        ((void)(ap = (va_list)0))
```

从源码可以看出：
- `va_list`只是`char*`的别称而已。
- `va_start`的实现有使用了`_INTSIZEOF`, `_ADDRESSOF`两个宏定义。先不管，望文生义结合`va_start`的声明，可以推测`va_start`是通过可变参数中前一个参数的地址得到第一个可变参数地址。
- `va_arg`通过`va_list`指针获取下一个参数的地址，并转化为相应的type
- `va_end` 将`va_list`置为空。


接下来看下`_INTSIZEOF`, `_ADDRESSOF`两个宏定义。
```c
#define _INTSIZEOF(n)          ((sizeof(n) + sizeof(int) - 1) & ~(sizeof(int) - 1))

#ifdef __cplusplus
    #define _ADDRESSOF(v) (&const_cast<char&>(reinterpret_cast<const volatile char&>(v)))
#else
    #define _ADDRESSOF(v) (&(v))
#endif
```

- `_ADDRESSOF`很明显就是取地址
- `_INTSIZEOF`是对int类型所占字节数进行对齐。

举个例子
```c
void test_INTSIZEOF() {
	int size = 0;
	size = _INTSIZEOF("hello");  //8
	size = _INTSIZEOF("h"); //4
	size = sizeof("hello");  //6
	size = sizeof("h");  //2
}
```

字符串结尾的'\0'字符也会被记为一个字节长度。由例子很容易理解`_INTSIZEOF`宏的作用。
那么问题又来了，为什么要做这个字节对齐处理呢？简单的说，因为参数在入栈的时候就是这样存放的，更详细的理解，请参考wiki: [Data structure alignment](https://en.wikipedia.org/wiki/Data_structure_alignment)


其实变长参数我们遇到最多的应该是`printf`了。在格式化一些输出时，printf是如何知道参数的个数呢？在上面的演示中，我们是手动传递了参数个数，作为第一个参数的。

答案就在于格式化字符中。例如`"%s age is %d"`,通过这个格式化字符串，可以知道需要两个参数, 类型分别是"%s"与"%d"。



## c++中变长参数实现
根据stackoverflow上的回答
> The last, but not the least: variadic functions have no place in C++ world, even for assignments.

c语言中对变长参数的实现在C++中是有更modern的实现的.所以我有对C++中的实现做了一番总结.主要参考了《C++ PRIMER 5TH Edition》。以下基于C++ 11 标准。

### 使用initializer_list
对于可变参数是同一类型，可以使用C++11新增的一个类型`initialize_list`。举例如下：
```c++
#include <initializer_list>

int sum_initializer_list(std::initializer_list<int> list) {
	auto beg = list.begin();
	auto sum = 0;
	for (; beg != list.end(); ++beg) {
		sum += *beg;
	}
	return sum;
}

void test_sum_initilizer_list() {
	sum_initializer_list({1,2,3});
	sum_initializer_list({1,2,3,4});
}
```
### 使用可变参数模板template<typename... Args>
在C++中可变参数被称为参数包(parameter package), 存在两种：模板参数包，表示零个或多个模板参数；函数参数包，表示零个或多个函数参数。举例如下：
```c++
//Args 是模板参数包，args是函数参数包
template<typename T, typename... Args>
void foo(const T& t, const Args&... args);
```

可变参数函数通常是递归的。第一部调用处理包中的第一个实参，然后用剩余实参调用自身。举个例子：
```c++
template<typename T>
void test_vaargs(const T& t) {
	std::cout << t << std::endl;
}

template<typename T, typename... args>
void test_vaargs(const T& t, const args&... param) {
	std::cout << "sizeof...(args): " << sizeof...(args) << std::endl;
	std::cout << "sizeof...(param): " <<sizeof...(param) << std::endl;
	std::cout << "print: " << t << std::endl;
	test_vaargs(param...);
}

//call
test_vaargs(1, 2, "robert");
```

当我们需要知道包中有多少元素时，可以使用`sizeof...`运算符。
对于一个参数包，在模板实例化的时候，是对包里的每一个元素进行扩展。包扩展（package expend）是对省略号前面的pattern进行扩展。比如上面的foo声明。

```c++
//declaration
template<typename T, typename... Args>
void foo(const T& t, const Args&... args);

//test
std::string str("robert"); int a=1; double b=0.1;
foo(a, b, str);
foo(b, "robert", a);
```
编译器通过类型推断，分别实例化生成对应的声明：
```c++
//参数包const Args&... 被扩展，可以理解为此时的pattern为const Args&
void foo(const int&, const double&, const std::string&);  //注意string类型推断为string
void foo(const double&, const char[7]&, const int&);  //字符串字面量类型推断为const char[len]&
```



## 心得

在troobleshooting的过程中还是要多注意细节，多观察一些预期的变量是否正确。当然了，c/c++的知识还要加强啊。