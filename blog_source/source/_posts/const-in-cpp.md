---
title: c++中的const总结
date: 2018-06-27 10:36:47
tags:
- c++
- const
categories:
- coding
keywords:
description:

---



最近在看《c++ primer》，里面涉及到很多关于const相关的一些概念，比如顶层const，底层const，const函数重载等，以下是自己的一些总结，作为备忘。



<!--more-->

## const变量

最简单的一个用法，可以用const来定义一个常量，如

```c++
const int val=1024;
```



val是一个常量，定义之后不能被改变。有点类似与使用宏定义，如

```c++
#define VAL 1024
```



常量与宏定义的区别在于：

- 宏定义是被预处理器（preprocessor）处理的，预处理器并不知道类型信息，只是将VAL出现过的地方替换为1024
- const定义的变量是被编译器（compiler）处理的，编译器理解变量的定义，类型。

## 常量指针与指向常量的指针

常量指针表示指针本身是一个常量，意味着不能修改该指针。

指向常量的指针表示指针指向的对象是一个常量，该对象不能够被修改。

```c++
int i = 0;
int j = 1;
int *const p = &i;  //p指针本身是一个常量，不允许改变p的值，顶层const
int const *q = &i;  //q指针指向的内容是一个常量，不允许改变*q的值，底层const
const int *q = &i;  //等价与int const *q = &i
	
p = &j;  //报错，编译无法通过
*q = 2;  //报错，编译无法通过
```

可以通过“从右往左”的结合方式，来判断是属于常量指针还是指向常量的指针。

- 顶层const（top-level const）: 表示该指针本身就是一个const常量
- 底层const（low-level const）:表示该指针指向的内容是一个const常量 



这里还涉及到左值(lvalue)，右值(rvalue)的概念。

- **左值**：能放在赋值语句的左侧，当一个对象被用作左值，用的是对象的身份（在内存中的位置）
- **右值**：不能放在赋值语句的左侧，当一个对象被用作右值，用的是对象的值（内容）



## const修饰函数返回值

看下面一个例子：

```c++
char *foo(){
    return "hello, world";
}

foo()[1] = 'a';
```

编译无问题，但是在运行的时候会报错，因为试图修改字符串字面量导致未定义行为。

```c++
const char *foo(){
    return "hello, world";
}

foo()[1] = "a";  //无法通过编译
```

使用const修饰函数的返回值，可以在编译的时候发现问题。



## const修饰函数形参

先看下面一个例子：

```c++
void foo(int val){
    val = 1;
}

void bar(int &val){
    val = 1;
}

int val = 0;
foo(val);
bar(val);
```

我们知道调用foo时，会将实参拷贝一份赋值给形参，在函数内部修改val的值并不会影响外部的val。而调用bar时，由于传递的是引用，所以外部的val会受到影响。这里存在一种情况，就是如果实参是一个很大的数据结构，在参数传递时，希望避免不需要的拷贝，我们可以传递引用。但是传递引用可能会改变实参的值。这个时候可以使用const来修饰参数。

```c++
void foo(const big_type &val){
    //todo
}
```

使用const可以保证传参时不被拷贝，同时不被函数内部修改。



## const成员函数

先看例子：

```c++
class foo{
public:
    void bar();
private:
    int val;
};

void foo::bar(){
    ++val;
}

foo ifoo;
ifoo.bar();
```

例子很简单，调用bar之后会将ifoo对象中的val值加1。如果bar函数本身并不想去修改val的值，这可以将其定义为const成员函数。

```
...
void bar() const;
...
```

在const成员函数中，无法对对象的数据成员进行修改。



## const函数重载

函数重载是指函数名称一样，但是函数签名不一样。需要const函数重载的原因是**常量对象无法调用非const成员函数**。

```c++
class foo{
public:
	int get_val(){ return val; }
private:
	int val = 0;
};

foo f1;
f1.get_val();  //正确

const foo f2;
f2.get_val();  //报错，f2只能调用const成员函数
```

f2无法调用get_val函数，因为f2为常量对象，只能调用const成员函数。

```c++
class foo{
public:
	int get_val(){ return do_get_val(); }
	int get_val() const{ return do_get_val(); }
private:
	int val = 0;
	int do_get_val() const { return val; }
};
```

我们再添加一个get_val的const重载函数。注意，这个额外定义了一个do_get_val函数来返回val值，在这个简单的例子中可能显得很多余，但是如果需要做一些比较复杂的操作，单独成一个函数可以防止代码冗余，同时如果以后修改也比较方便。



## 参考链接

- [The C++ 'const' Declaration: Why & How](http://duramecho.com/ComputerInformation/WhyHowCppConst.html)
- 《C++ primer》