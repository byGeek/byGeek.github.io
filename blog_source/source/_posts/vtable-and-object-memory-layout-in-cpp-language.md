---
title: vtable and object memory layout in cpp
date: 2018-10-22 11:07:44
tags:
- c++
categories:
- coding
keywords:
description:
---





在C语言中，data和function是独立的entity，换句话说data和function是没有直接关联的。所以说使用C编程是面向过程编程，使用一个个的function来操作外在的data。而在C++中，通过Class将data和function集合在一起，形成一个Object。通过面向对象编程的三大特点：1)封装2)继承3)多态 来实现更好的灵活性。

<!--more-->

```cpp

//c style： declare data structure and function seperately
//data
typedef struct _point{
    float x;
    float y;
}Point_t;

//function
void print_point(const Point_t* ptr){
    printf("%f %f", ptr->x, ptr->y);
}

//c++ style: use class
class Point{
public:
    Point(float x, float y): m_x(x), m_y(y){}
    ~Point() = default;
   
    void print(){
        printf("%f %f", m_x, m_y);
    }
    float x(){return m_x;}
    float y(){return m_y;}
    
private:
    float m_x;
    float m_y;
}
```

## c++ Object Memory Model

在上述代码中，Point类的大小跟C语言中的Point 结构体的大小是一致的，在64bit机器上使用sizeof运算符得到大小都是8 byte。考虑下面的一个例子：

```cpp
class A{
    int a;
    float f;
    char c1;
    char c2;
    char d[4];
}
```

一个对象所占的内存空间是一块连续的地址，且后声明的数据成员具有更高的地址。



## what is vtable

- static array usually the first slot in c++ object model
- class-specific
- vptr, vptr is assigned in constructor by compiler generated code, that is why member initialization list is different with initialization in constructor body.
- polymorphic can be implemented differently between compilers. But generally implement using vtable
- use sample code in linux, use gdb to show vtable: -exec info vtbl objectname

## object memory model

- alignment

## member function

- member function is just a tag in code segment, is not include in object memory layout

## single inheritance and multi-inheritance

- pointer fix needed in multi-inheritance





## notes

- [pointers-to-members](https://isocpp.org/wiki/faq/pointers-to-members)
- PPT material

