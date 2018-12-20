---
title: C++ 中的智能指针
date: 2018-12-04 13:58:59
tags:
- c++
categories:
- coding
keywords:
- c++
description:
---

C++ 11中共有四种智能指针(Smart Pointers)：`std::auto_ptr`,`std::unique_ptr`,`std::shared_ptr`,`std::wek_ptr`。其中`std::auto_ptr`是在C++98中就引入的智能指针，在C++11中已经被`std::unique_ptr`所取代。所以本文只讨论剩下的三种智能指针。

<!--more-->

## 为什么要使用智能指针？

不像C#/Java等语言拥有垃圾回收机制，C++必须靠程序员自己申请和释放内存。这样就给内存泄漏带来了机会。智能指针就是为了解决可能的内存泄漏的风险而设计的。在C++中局部变量在离开作用域之后，编译器会自动调用变量的析构函数对其进行析构，即使在发生异常的情况下也可以保证对象被析构。

智能指针其实就是贯彻了[**RAII**](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization)(Resource acquisition is initialization)的思想。将Raw Pointer作为资源托管起来，在离开作用域之后，自动调用Raw pointer的deletor。智能指针的本质就是对象。

下面我们来分别看下三种智能指针的使用场景和一些注意事项。

## std::unique_ptr

当需要使用智能指针时，`std::unique_ptr`应该作为首选，用来表达对资源的专属所有权。

当使用默认deletor时，`std::unique_ptr`跟裸指针尺寸相同，这意味着不会带来memory overhead。一个非空的`std::unique_ptr`总是指向涉及的资源。移动一个`std::unique_ptr`会将所有权将源指针移至目标指针(源指针被置空)。`std::unique_ptr`不支持复制操作。

我们可以用以下方式来创建`std::unique_ptr`。

- 通过C++14 引入的标准make函数`std::make_unique`
- 直接接管newly allocated object
- 创建自己的make函数

举例如下：

```cpp
#include <iostream>
#include <memory>

#define MY_PRINT std::cout<< __FUNCTION__ << std::endl;

class Investment {
public:
	Investment(const char* name) : m_name(new char[128]){
		strcpy_s(m_name, 128, name);
		MY_PRINT;
	}
    
	virtual ~Investment() {
		delete m_name;
		MY_PRINT;
	}

protected:
	char* m_name;
};

class Stock : public Investment {
public:
	Stock(const char* name) : Investment(name){
		MY_PRINT;
	}

	~Stock() {
		MY_PRINT;
	}

};

std::unique_ptr<Investment> makeInvestment() {
	//in c++ 11
	//std::unique_ptr<Stock> stock(new Stock("hello"));
	//return stock;

	return std::unique_ptr<Stock>(new Stock("hello"));  //newly allocated object是一个右值
}

std::unique_ptr<Investment> makeInvestment2() {
	//in c++ 14
	return std::make_unique<Stock>("hello");
}

```

在C++11中，由于没有引入unique_ptr的make函数，我们应该直接从new object接管pointer的控制权。如果使用以下方式来创建unique_ptr，会增加内存泄漏的风险。

```cpp
auto ptr = new Stock("hello");
std::unique_ptr<Investment> pInvest(ptr);  
```

通过左值来初始化一个unique_ptr不是一个好的选择，因为ptr这个裸指针有可能会被delete掉，而pInvest在析构的时候会再次delete 裸指针，这时会造成未定义行为。

Best practice是通过make函数来创建智能指针。如果使用的是C++11，还有一种方式是创建自定义的make函数。如下是一个简单的版本。当然了，也可以直接去标准库中copy make_unique函数的实现代码。

```cpp
template<typename T, typename ... Args>
std::unique_ptr<T> make_unique(Args&&... params) {
	return std::unique_ptr<T>(new T(std::forward<Args>(params)...));
}
```

我们也可以为unique_ptr自定义deletor。有以下两种方式：

- 函数对象，如lambda表达式
- 函数指针

举例如下：

```cpp
//use lambda expression	
auto delInvest = [](Investment* pInvestment) {
		//do something
		delete pInvestment;
	};
	 
	std::unique_ptr<Investment, decltype(delInvest)> ptr(new Stock("hello"), delInvest);

//use function pointer
	void delInv(Investment* pInvestment) {
		delete pInvestment;
	}
	 
	std::unique_ptr<Investment, void(*)(Investment* pInvestment)> ptr2(new Stock("hello"), delInv);
	
	//或者使用decltype
	std::unique_ptr<Investment, decltype(&delInv)> ptr2(new Stock("hello"), delInv);

```

一般来说对于std::unique_ptr，在默认deletor的情况下，unique_ptr和裸指针的大小一样，但是使用自定义的deletor之后，情况变得有所不同。如果自定义的deletor是函数指针，则unique_ptr的大小会至少增加一个函数指针的大小。如果是函数对象，则带来的尺寸变化取决于函数对象中存储了多少状态。无状态的函数对象，如无捕获的lambda表达式不会浪费任何尺寸。 

这意味着一个自定义deletor可以用函数指针也可以使用无捕获的lambda表达式时，lambda表达式是更好的选择。

注意，自定义deletor必须是一个directly-callable object，如果想让一个member function作为deletor的话，可以使用`std::mem_fn`将其转化为函数对象。

```cpp
#include <functional> //for std::mem_fn
#include <memory>

void Investment::release(){  //release is a member function
    //do something
}

std::unique_ptr<Investment, decltype(std::mem_fn(&Investment::release))> pInvest(new Stock("hello"), std::mem_fn(&Investment::release));
```



`std::unique_ptr`可以很方便的转换为`std::shared_ptr`，所以适合作为工厂函数的返回型别。具体可以参考《Modern Effective C++》 item18。

```cpp
std::unique_ptr<std::string> unique = std::make_unique<std::string>("test");
std::shared_ptr<std::string> shared = std::move(unique);

//or
std::shared_ptr<std::string> shared = std::make_unique<std::string>("test");
```

## std::shared_ptr

[](https://herbsutter.com/2013/05/29/gotw-89-solution-smart-pointers/)

todo: 使用场景，用来管理具体共享所有权的资源

引用计数(reference counting)

shared_ptr的大小为裸指针的两倍，看具体的图：控制块

## std::weak_ptr



todo: 使用场景

weak_ptr并不是一种独立的智能指针，是对shared_ptr的一种扩充。用来break cycles，用作object cache

检测是否过期: expired(), lock()

## 几个注意点

### 尽量使用make函数来创建智能指针

### unique_ptr 与 shared_ptr的几个需要注意的地方

- Base 与Derived virtual dtor问题
- custom deletor作为unique_ptr的型别，而对于shared_ptr则不是这样
- custom deletor 中使用函数对象与函数指针的区别

## 参考资料

- scott mayer 《modern effective c++》



## 