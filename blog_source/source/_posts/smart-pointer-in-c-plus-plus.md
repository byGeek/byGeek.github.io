---
title: C++ 中的智能指针
date: 2018-12-04 13:58:59
tags:
- c++
categories:
- coding
keywords:
- c++
- smart pointers
description:
---

C++ 11中共有四种智能指针(Smart Pointers)：`std::auto_ptr`,`std::unique_ptr`,`std::shared_ptr`,`std::weak_ptr`。其中`std::auto_ptr`是在C++98中就引入的智能指针，在C++11中已经被`std::unique_ptr`所取代。所以本文主要讨论讨论剩下的三种智能指针。

<!--more-->

### 为什么要使用智能指针？

不像C#/Java等语言拥有垃圾回收机制，C++必须靠程序员自己申请和释放内存。这样就给内存泄漏带来了机会。智能指针就是为了解决可能的内存泄漏的风险而设计的。在C++中局部变量在离开作用域之后，编译器会自动调用变量的析构函数对其进行析构，即使在发生异常的情况下也可以保证对象被析构。

智能指针其实就是贯彻了[**RAII**](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization)(Resource acquisition is initialization)的思想。将Raw Pointer作为资源托管起来，在离开作用域之后，自动调用Raw pointer的deletor。智能指针就是对象。

查看智能指针的头文件: memory，这里摘录下MSVC的实现：

```cpp
	// DECLARATIONS
template<class _Ty>
	struct default_delete;

template<class _Ty,
	class _Dx = default_delete<_Ty>>
	class unique_ptr;

template<class _Ty>
	class shared_ptr;

template<class _Ty>
	class weak_ptr;
```

可以看到三种智能指针被声明为带有模板参数的类，其中unique_ptr带有两个模板参数，shared_ptr和weak_ptr都只有一个模板参数。

下面我们来分别详细来分析下三种智能指针的使用场景和一些注意事项。

### std::unique_ptr

当需要使用智能指针时，`std::unique_ptr`应该作为首选，用来表达对资源的专属所有权。

当使用默认deletor时，`std::unique_ptr`跟裸指针尺寸相同，这意味着不会带来memory overhead。一个非空的`std::unique_ptr`总是指向涉及的资源。移动一个`std::unique_ptr`会将所有权将源指针移至目标指针(源指针被置空)。`std::unique_ptr`不支持复制操作。

#### 创建std::unique_ptr的方法

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

#### 自定义deletor

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

#### 转化为shared_ptr

`std::unique_ptr`可以很方便的转换为`std::shared_ptr`，所以适合作为工厂函数的返回型别。具体可以参考《Modern Effective C++》 item18。

```cpp
std::unique_ptr<std::string> unique = std::make_unique<std::string>("test");
std::shared_ptr<std::string> shared = std::move(unique);

//or
std::shared_ptr<std::string> shared = std::make_unique<std::string>("test");
```

#### auto_ptr vs unique_ptr

在C98标准的时候，由于没有移动语义（move sematic），引入了auto_ptr来表示对资源的所有权。在表示所有权转移的时候，auto_ptr实际是通过拷贝操作来实现的。

```cpp
std::auto_ptr<Stock> ap(new Stock("hello"));
std::auto_ptr<Stock> ap2 = ap;  //compile success, while unique_ptr don't

ap->foo();  //crash because ap is set to NULL
```

通过auto_ptr的拷贝赋值运算符之后，ap已经置为了NULL，无法再使用ap2。

在使用unique_ptr时，由于unique_ptr将copy ctor和copy assign operator声明为delete function，所以编译无法通过。如果想转移资源的所有权，必须使用`std::move`（位于utility头文件）。

```cpp
std::unique_ptr<Stock> up(new Stock("hello"));
std::unique_ptr<Stock> up1 = std::move(up);  //use std::move to explicitly transfer ownership, and up is empty, can not use it again

up.foo();  //crash, can not use up again because ownership has transferred to up1
```

#### unique_ptr 源码剖析

从上面unique_ptr的声明可以看到unique_ptr的类型中第二个模板参数默认时一个`default_delete`。查看其源码发现`default_delete`就是一个定义了函数调用运算符的函数对象:

```cpp
template<class _Ty>
	struct default_delete
	{	// default deleter for unique_ptr
	constexpr default_delete() noexcept = default;
        //...省略部分

	void operator()(_Ty * _Ptr) const noexcept
		{	// delete a pointer
		static_assert(0 < sizeof (_Ty),
			"can't delete an incomplete type");
		delete _Ptr;  //调用delete
		}
	};
```

在函数调用中直接调用delete，delete操作会调用_Ty类型的析构函数。这就是unique_ptr的默认析构所做的事。再看下unique_ptr的具体定义：

```cpp
template<class _Ty,
	class _Dx>	// = default_delete<_Ty>
	class unique_ptr
		: public _Unique_ptr_base<_Ty, _Dx>
	{	// non-copyable pointer to an object
public:
//...省略
}
```

unique_ptr继承自`_Unique_ptr_base<_Ty, _Dx>`，继续看`_Unique_ptr_base<_Ty, _Dx>`的代码，其中包含一个成员变量：

```cpp
_Compressed_pair<_Dx, pointer> _Mypair;
```

这里的pointer就是unique_ptr实际的对象的指针。接着在头文件`xutility`中可以看到`_Compressed_pair`的两个定义：

```cpp
//定义1
template<class _Ty1,
	class _Ty2,
	bool = is_empty_v<_Ty1> && !is_final_v<_Ty1>>
	class _Compressed_pair final
		: private _Ty1
	{	// store a pair of values, deriving from empty first
private:
	_Ty2 _Myval2;

	using _Mybase = _Ty1;	// for visualization
	//...省略
	}

//定义2
template<class _Ty1,
	class _Ty2>
	class _Compressed_pair<_Ty1, _Ty2, false> final
	{	// store a pair of values, not deriving from first
private:
	_Ty1 _Myval1;
	_Ty2 _Myval2;
        //...省略
    }
```

这两个定义的区别直观上来说声明的内部成员变量不同，一个只有一个实际对象的指针（_Ty2）,另外一个不仅有_Ty2，还包含deletor类型变量。换句话说，这个区别影响了unique_ptr指针所占用的内存大小。

`_Ty1`就是`_Dx`，也就是deletor的类型，`_Ty2`也就是unique_ptr实际指向的对象的指针。这里应该是通过重载的机制，让编译器去选择实例化哪个`_Compressed_pair`模板的代码。这个判断条件就是:

```cpp
is_empty_v<_Ty1> && !is_final_v<_Ty1>
```

查询cpp reference可以`is_empty_v`用来标识类是否是一个empty类

> Trait class that identifies whether *T* is an empty class.
>
> An empty class is a class that stores no data, either cv-qualified or not.  

`is_final_v`用来标识是否是final类（使用final关键字声明类）。

至此我们可以看到unique_ptr所占空间的大小跟deletor有关，如果使用默认的deletor，则显然`default_delete`是一个空类，那么unique_ptr跟裸指针具有相同的大小。而如果自定义的deletor（可以理解为function object）包含了其他的data，则unique_ptr的大小是裸指针和function object占用大小之和。

### std::shared_ptr

#### 控制块

和`std::unique_ptr`用来表示对资源的独占性相反，`std::shared_ptr`用来表示对资源的共享。创建的shared_ptr对象共享同一个资源。shared_ptr在内部实现使用引用计数的方式。每个`shared_ptr<T>`对象都包含两个部分，一个是指向T型别对象的指针，一个是指向控制块的指针，如下图：

{% asset_img "shared_ptr memory layout.png" %}

很显然，从shared_ptr的memory layout可以看到，shared_ptr的大小为裸指针的两倍。

注意：**必须保证对同一个资源只创建一个控制块**。因为当控制块中的引用计数为0时，资源被销毁。如果违反了这个约定，资源会被“销毁两次”，也就是说会造成未定义行为。控制块需要动态分配。控制块的创建遵循以下规则：

- std::make_shared总是创建一个控制块
- 从具备专属所有权的指针(即unique_ptr或auto_ptr指针)出发构造一个shared_ptr时，会创建一个控制块。
- 使用裸指针作为实参来调用（创建）shared_ptr时，会创建一个控制块。如果使用shared_ptr或者weak_ptr来创建shared_ptr，则不会创建控制块。

#### 创建std::shared_ptr

```cpp
//Stock和Investment接上面unique_ptr中的定义

//通过make函数构造
std::shared_ptr<Stock> ptr = std::make_shared<Stock>("hello");  //创建一个控制块

//通过传递右值构造
std::shared_ptr<Stock> ptr(new Stock("hello"));  //创建一个控制块

//bad code: 通过裸指针，左值构造
auto raw_ptr = new Stock("hello");
std::shared_ptr<Stock> ptr1(raw_ptr);  //为raw_ptr创建了一个控制块
std::shared_ptr<Stock> ptr2(raw_ptr);  //又创建了一个控制块，在析构时，会造成未定义行为。

std::shared_ptr<Stock> ptr3(ptr1);  //从一个已有的shared_ptr创建另一个shared_ptr，调用shared_ptr的copy ctor，不会再次创建控制块，
```

Best Practice是使用make函数来构造shared_ptr。最后的例子通过裸指针来构造，会为同一个资源创建两个控制块，也就是会有两个引用计数，当一个引用计数为0时，会发生析构，这样会对同一个资源析构两次，造成未定义行为。

#### 自定义deletor

在unique_ptr的类型中，自定义的deletor是作为一个模板参数，所以deletor的型别是作为unique_ptr型别的一部分的，而shared_ptr则不一样。

```cpp
void delete_stock(Stock* s) {
	delete s;
}

auto stockDeletor = [](Stock* s) {
		//logsomething();
		delete s;
	};

//自定义析构
std::shared_ptr<Stock> ptr1(new Stock("hello"), stockDeletor);
std::shared_ptr<Stock> ptr2(new Stock("hello"), delete_stock);

std::vector<std::shared_ptr<Stock>> v{ptr1, ptr2};  //不同的deletor，但是属于同一个类型，可以放在vector容器中，如果是unique_ptr，则不行
```

#### shared_from_this

考虑这样一种情况，如果使用this指针直接来创建shared_ptr，会发生什么。以下代码来自[cpp reference](https://en.cppreference.com/w/cpp/memory/enable_shared_from_this)：

```cpp
#include <memory>
#include <iostream>
 
struct Good: std::enable_shared_from_this<Good> // note: public inheritance
{
    std::shared_ptr<Good> getptr() {
        return shared_from_this();
    }
};
 
struct Bad
{
    std::shared_ptr<Bad> getptr() {
        return std::shared_ptr<Bad>(this);
    }
    ~Bad() { std::cout << "Bad::~Bad() called\n"; }
};
 
int main()
{
    //example 1
    // Good: the two shared_ptr's share the same object
    std::shared_ptr<Good> gp1 = std::make_shared<Good>();
    std::shared_ptr<Good> gp2 = gp1->getptr();
    std::cout << "gp2.use_count() = " << gp2.use_count() << '\n';
 
    //example 2
    // Bad: shared_from_this is called without having std::shared_ptr owning the caller 
    try {
        Good not_so_good;
        std::shared_ptr<Good> gp1 = not_so_good.getptr();
    } catch(std::bad_weak_ptr& e) {
        // undefined behavior (until C++17) and std::bad_weak_ptr thrown (since C++17)
        std::cout << e.what() << '\n';    
    }
 
    //example 3
    // Bad, each shared_ptr thinks it's the only owner of the object
    std::shared_ptr<Bad> bp1 = std::make_shared<Bad>();
    std::shared_ptr<Bad> bp2 = bp1->getptr();
    std::cout << "bp2.use_count() = " << bp2.use_count() << '\n';
} // UB: double-delete of Bad
```

在example 3，使用make函数来创建bp1，这是会创建一个控制块，后面又通过this裸指针又会创建一个控制块，所以会发生double delete。

在标准库中已经提供了`enable_shared_from_this`模板来解决这个问题。让类继承自`enable_shared_from_this`的特化版本，这个模板提供一个`shared_from_this`成员函数，可以使用这个成员函数来构造新的shared_ptr。但是注意，必须当前shared_ptr已经有一个相关联的控制块之后，才可以安全的使用`shared_from_this`，否则也会发生未定义行为。如上面的example 2。



### std::weak_ptr

#### weak_ptr vs shared_ptr

weak_ptr并不是一种独立的智能指针，是对shared_ptr的一种扩充。**weak_ptr不能直接由裸指针来构造，只能通过shared_ptr或者其他的weak_ptr来构造**。cpp reference对weak_ptr的定义：

> `std::weak_ptr` is a smart pointer that holds a non-owning ("weak") reference to an object that is managed by [std::shared_ptr](https://en.cppreference.com/w/cpp/memory/shared_ptr). It must be converted to [std::shared_ptr](https://en.cppreference.com/w/cpp/memory/shared_ptr) in order to access the referenced object.

weak_ptr并不影响指向同一个资源对象的shared_ptr的引用计数，换句话说，weak_ptr不影响对象的共享所有权。前面在shared_ptr中提到的控制块中包含一个弱引用计数，weak_ptr影响的就是这个弱引用计数。当这个弱引用计数为0时，也即没有shared_ptr涉及到这个对象时，weak_ptr失效（过期）。

```cpp
	//call use_count function to get the numbers of shared_ptr instances that
    //shared ownship of the managed object, if 0 then the object has been deleted

	std::shared_ptr<Stock> sp = std::make_shared<Stock>("hello");
	std::cout << sp.use_count() << std::endl;  //1

	std::weak_ptr<Stock> wp = sp;
	std::cout << sp.use_count() << std::endl;  //1
	std::cout << wp.use_count() << std::endl;  //1

	std::weak_ptr<Stock> wp2 = wp;
	std::cout << wp.use_count() << std::endl;  //1
	std::cout << wp2.use_count() << std::endl; //1

	std::shared_ptr<Stock> sp2 = wp.lock();  //use lock to get a shared_ptr instance
	if (sp2) {
		//sp2 not null
	}
	else {
		//sp2 null, mean weak_ptr is expired
	}
```

在weak_ptr中使用`lock`方法来新建一个shared_ptr对象并返回，如果为空，则说明weak_ptr已经过期。也可以使用`expired`来检测weak_ptr是否过期，在多线程的环境中，可能会带来竞争风险。所以还是推荐使用`lock`方法。

#### 使用场景

weak_ptr可以有以下几种使用场景。

- 用作cache（缓存）
- 观察者模式
- 避免shared_ptr指针环路

##### 用作cache（缓存）

```cpp
//use weak_ptr in cache scenario
std::shared_ptr<Widget> loadWidget() {
	return std::make_shared<Widget>();
}

std::shared_ptr<Widget> fast_load_widget(int id) {
	static std::unordered_map<int, std::weak_ptr<Widget>> cache;

	auto sp = cache[id].lock();  //check if exist in cache
	if (!sp) {
		sp = loadWidget();
		cache[id] = sp;  //put to cache
	}

	return sp;
}
```

##### 观察者模式

Subject类有一个vector容器，用于保存observers，当需要通知observer时，先检查observer是否还有效。注意：通过`std::make_shared`创建了一个临时的shared_ptr直接复制给weak_ptr，这个时候shared_ptr即被销毁。在调用`lock`测试weak_ptr是否过期时，这个observer是过期的。所以在测试函数中另外一个observer能收到通知。

```cpp
class Observer {
public:
	void DoStuff() {
		std::cout << "Got notified" << std::endl;
	}
};

class Subject {
public:
	void Notify() {
		for (auto& ob : m_obs) {
			auto sp = ob.lock();  //convert to shared_ptr
			if (sp) {  //test if converted shared_ptr is valid
				sp->DoStuff();
			}
		}
	}

	void AddToObserver(std::weak_ptr<Observer>& ob) {
		m_obs.push_back(ob);
	}

private:
	std::vector<std::weak_ptr<Observer>> m_obs;
};

void test_observer_pattern_using_weak_ptr() {
	Subject s;
	
	auto sp = std::make_shared<Observer>();  //make a shared_ptr obj
	std::weak_ptr<Observer> ob1 = std::make_shared<Observer>();  //make a temp shared_ptr, after the assignment, shared_ptr is deleted
	std::weak_ptr<Observer> ob2(sp);  //only this observer can receive notification!!!

	s.AddToObserver(ob1);  
	s.AddToObserver(ob2);

	s.Notify();
}
```

##### 避免shared_ptr指针环路

{% asset_img avoid_shared_ptr_loop.png %}

A, C都共享B，如果B需要保有一个对A的指针，这时候如果使用shared_ptr的话，会造成shared_ptr 环路，A, B互相保持引用，这样A和B的引用计数都不为0，无法析构，造成内存泄漏。

weak_ptr可以避免这个问题。当A的引用计数为0，即使B保有一个指向A的weak_ptr，不影响A被销毁。



### Make函数的几个注意点

在使用智能指针时，Best Practice都是使用make函数来创建。特别是对于shared_ptr， 因为与之相关的控制块需要动态分配，如果使用make函数，则可以进行一次系统调用来分配内存（包含控制块和shared_ptr对象的内存）。而如果使用先创建裸指针，然后再创建shared_ptr的方法，需要进行两次系统调用来申请内存。这样做带来裸指针安全性的问题，也增加的性能上的开销。

当然了，make函数也不是万能的，如果需要使用自定义的deletor，那么只能通过其他方式来创建智能指针。



本文主要参考了Scott Meyers大师的  ***Effective Modern C++***，可以直接在网络上阅读：[地址在这](https://www.oreilly.com/library/view/effective-modern-c/9781491908419/ch04.html).

### 参考资料

- Scott Meyers ***Effective Modern C++***
- [GotW91: Smart Pointer Parameters](https://herbsutter.com/2013/06/05/gotw-91-solution-smart-pointer-parameters/)
- [top-10-dumb-mistakes-avoid-c-11-smart-pointers](https://www.acodersjourney.com/top-10-dumb-mistakes-avoid-c-11-smart-pointers/)
