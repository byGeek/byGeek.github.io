---
title: 由sizeof引起的问题
date: 2018-11-06 15:48:46
tags:
- c++
categories:
- coding
keywords:
- vtable
- c++
description:
---

最近在看Lippman的《Inside the c++ object model》，书中Lippman说有人发邮件问了他一个问题。

```cpp
class X {};

class Y : public virtual X{};
class Z: public virtual X{};

class A:public Y, public Z{};
```

根据以上定义，使用sizeof运算符分别计算类X,Y,Z,A的所占大小。

<!--more-->

答案公布：

在msvc140和gcc 5.4环境下测试：

```cpp
size_t temp = sizeof(X); //1
temp = sizeof(Y);  //4
temp = sizeof(Z);  //4
temp = sizeof(A);  //8
```

## 为什么sizeof(X)为1？

X是一个"空类"，按理来说sizeof(X)应该是0，因为没有数据需要存储。为何sizeof(X)为1呢？

Lippman给出的解释是，如果空类所占空间为0，则类的两个不同的实例地址就相同，这样的结果显然不是我们想要的。编译器自动给空类插入占用一字节的char数据成员，这样可以保证不同的对象会分配唯一的地址空间。

```cpp
X a, b;
if(&a == &b) cerr << "yipes" <<endl;
```

在《Effective c++》中item 39中，作者提到：freestanding objects必须有Non-zero size，对于大多数编译器，sizeof(X)是1，是因为编译器插入一个char完成的。

在[cppreference](https://en.cppreference.com/w/cpp/language/ebo)可以找到相关的标准规定:

> The size of any [object](https://en.cppreference.com/w/cpp/language/object) or member subobject (unless [[no_unique_address]] -- see below) (since C++20) is required to be at least 1 even if the type is an empty [class type](https://en.cppreference.com/w/cpp/language/class) (that is, a class or struct that has no non-static data members), in order to be able to guarantee that the addresses of distinct objects of the same type are always distinct.

## 为什么sizeof(Y), sizeof(Z)为4？

先来考虑另外一个问题

```cpp
class X{};
class Q: public X{
    private:
    	int data;
}
```

Q继承自空类X，考虑到sizeof(X) = 1, 那么Q类的所占空间是sizeof(int)还是sizeof(int) + 1?

答案是根据Empty base optimization原则，Q所占空间为sizeof(int)。也就是说base class所占空间被“优化”掉了。但是这里我们class Y， Z并没有数据成员，sizeof(Y) == sizeof(Z)  == 4。

这是因为使用了virtual inheritance。一旦跟**virtual**扯上关系，编译器又在背后偷偷的做了一些事情。

为了实现面向对象中的多态特性，大多数编译器都会将一个vptr插入到the most base class中，vptr指向vtable。关于vtable的相关知识，请参考我的这篇[博文](https://bygeek.cn/2018/10/22/vtable-and-object-memory-layout-in-cpp-language/)。正是因为vptr指针占用4个字节，并且基于empty base optimization原则，所以sizeof(Y) = 4。

## 为什么sizeof(A)为8？

因为virtual继承关系，编译器会在Y，Z的memory layout中加入一个vptr指针。在多重继承下，X的memory layout会包含Y,Z的vptr，所以sizeof(A)为8。

## 如看查看memory layout

### MSVC

在msvc下我们可以使用编译器选项`/d1reportSingleClassLayout<classname>`， `/d1reportAllClassLayout`来查看memory layout。打开visual studio 开发者prompt

```
cl.exe your_source_file_path.cpp /d1reportAllClassLayout
```

如果包含的类比较多，可以将输出信息重定向到文件中，方便查看。

```
cl.exe your_source_file_path.cpp /d1reportAllClassLayout > "d:\out.txt"
```

在MSVC140下dump出来的memory layout如下：

```
class X	size(1):
	+---
	+---

class Y	size(4):
	+---
 0	| {vbptr}
	+---
	+--- (virtual base X)
	+---

Y::$vbtable@:
 0	| 0
 1	| 4 (Yd(Y+0)X)
vbi:	   class  offset o.vbptr  o.vbte fVtorDisp
               X       4       0       4 0

class Z	size(4):
	+---
 0	| {vbptr}
	+---
	+--- (virtual base X)
	+---

Z::$vbtable@:
 0	| 0
 1	| 4 (Zd(Z+0)X)
vbi:	   class  offset o.vbptr  o.vbte fVtorDisp
               X       4       0       4 0

class A	size(8):
	+---
 0	| +--- (base class Y)
 0	| | {vbptr}
	| +---
 4	| +--- (base class Z)
 4	| | {vbptr}
	| +---
	+---
	+--- (virtual base X)
	+---

A::$vbtable@Y@:
 0	| 0
 1	| 8 (Ad(Y+0)X)

A::$vbtable@Z@:
 0	| 0
 1	| 4 (Ad(Z+0)X)
vbi:	   class  offset o.vbptr  o.vbte fVtorDisp
               X       8       0       4 0

```

### GCC

在gcc中可以通过编译选项`-fdump-class-hierarchy`来查看。

```bash
g++ main.cpp -fdump-class-hierarchy
```

编程成功后，会生成一个main.cpp.002t.class的文件，搜索想要查看的类的memory layout即可。

载g++ 5.4 ubuntu下dump出来的main.cpp.002t.class文件如下：

```
Class X
   size=1 align=1
   base size=0 base align=1
X (0x0x7f34c09cf5a0) 0 empty

Vtable for Y
Y::_ZTV1Y: 3u entries
0     0u
8     (int (*)(...))0
16    (int (*)(...))(& _ZTI1Y)

VTT for Y
Y::_ZTT1Y: 1u entries
0     ((& Y::_ZTV1Y) + 24u)

Class Y
   size=8 align=8
   base size=8 base align=8
Y (0x0x7f34c08661a0) 0 nearly-empty
    vptridx=0u vptr=((& Y::_ZTV1Y) + 24u)
  X (0x0x7f34c09cf600) 0 empty virtual
      vbaseoffset=-24

Vtable for Z
Z::_ZTV1Z: 3u entries
0     0u
8     (int (*)(...))0
16    (int (*)(...))(& _ZTI1Z)

VTT for Z
Z::_ZTT1Z: 1u entries
0     ((& Z::_ZTV1Z) + 24u)

Class Z
   size=8 align=8
   base size=8 base align=8
Z (0x0x7f34c0866208) 0 nearly-empty
    vptridx=0u vptr=((& Z::_ZTV1Z) + 24u)
  X (0x0x7f34c09cf660) 0 empty virtual
      vbaseoffset=-24

Vtable for A
A::_ZTV1A: 6u entries
0     0u
8     (int (*)(...))0
16    (int (*)(...))(& _ZTI1A)
24    18446744073709551608u
32    (int (*)(...))-8
40    (int (*)(...))(& _ZTI1A)

Construction vtable for Y (0x0x7f34c0866270 instance) in A
A::_ZTC1A0_1Y: 3u entries
0     0u
8     (int (*)(...))0
16    (int (*)(...))(& _ZTI1Y)

Construction vtable for Z (0x0x7f34c08662d8 instance) in A
A::_ZTC1A8_1Z: 3u entries
0     18446744073709551608u
8     (int (*)(...))0
16    (int (*)(...))(& _ZTI1Z)

VTT for A
A::_ZTT1A: 4u entries
0     ((& A::_ZTV1A) + 24u)
8     ((& A::_ZTC1A0_1Y) + 24u)
16    ((& A::_ZTC1A8_1Z) + 24u)
24    ((& A::_ZTV1A) + 48u)

Class A
   size=16 align=8
   base size=16 base align=8
A (0x0x7f34c0878310) 0
    vptridx=0u vptr=((& A::_ZTV1A) + 24u)
  Y (0x0x7f34c0866270) 0 nearly-empty
      primary-for A (0x0x7f34c0878310)
      subvttidx=8u
    X (0x0x7f34c09cf6c0) 0 empty virtual
        vbaseoffset=-24
  Z (0x0x7f34c08662d8) 8 nearly-empty
      subvttidx=16u vptridx=24u vptr=((& A::_ZTV1A) + 48u)
    X (0x0x7f34c09cf6c0) alternative-path
```

在调试的时候还可以通过GDB命令`info vtbl`来查看vtable，下面是在vscode中调试示例。

{% asset_img vtable1.png %}





如果想更多了解C++背后的故事，推荐阅读Lippman的《Inside the C++ Object Model》一书。

## 参考链接

- <https://ofekshilon.com/2010/11/07/d1reportallclasslayout-dumping-object-memory-layout/>

- http://visualgdb.com/gdbreference/commands/info_vtbl
- https://www.gonwan.com/2010/09/20/c-class-layout-using-msvc/
- https://stackoverflow.com/questions/99297/how-are-virtual-functions-and-vtable-implemented
- https://www.v2ex.com/t/500682#reply10
- https://isocpp.org/wiki/faq/pointers-to-members