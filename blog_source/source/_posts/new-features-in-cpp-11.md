---
title: C++ 11带来的新特性
date: 2018-12-04 09:56:55
tags:
- c++
categories:
- coding
- repost
keywords:
- modern c++
description:
---

本文转载自Herb Sutter的[blog](https://herbsutter.com/elements-of-modern-c-style/)。Herb Sutter是C++标准委员会的主席，他在本文中主要讲述了C++ 11 带来的新的一些feature，同时建议尽量使用Modern C++ style编程。

同时我建议阅读Scott Mayers的《Effective Modern C++》一书。

原文如下。

<!--more-->

The C++11 standard offers [many useful new features](http://www2.research.att.com/~bs/C++0xFAQ.html). This page focuses specifically and only on those features that make C++11 really feel like a new language compared to C++98, because:

- They change the styles and idioms you’ll use when writing C++ code, often including the way you’ll design C++ libraries. For example, you’ll see more smart pointer parameters and return values, and functions that return big objects by value.
- They will be used so pervasively that you’ll probably see them in most code examples. For example, virtually every five-line modern C++ code example will say “auto” somewhere.

Use the other great C++11 features too. But get used to these ones first, because these are the pervasive ones that show why C++11 code is clean, safe, and fast – just as clean and safe as code written in any other modern mainstream language, and with C++’s traditional to-the-metal performance as strong as ever.

Notes:

- Like Strunk & White, this page is deliberately focused on brief summary guidance. It is not intended to provide exhaustive rationale and pro/con analysis; that will go into other articles.
- This is a living document. See the end for a list of the main changes and additions over time.

### auto

Use auto wherever possible. It is useful for two reasons. First, most obviously it’s a convenience that lets us avoid repeating a type name that we already stated and the compiler already knows.

```cpp
// C++98
map<int,string>::iterator i = m.begin();
double const xlimit = config["xlimit"];
singleton& s = singleton::instance();
 
// C++11
auto i = begin(m);
auto const xlimit = config["xlimit"];
auto& s = singleton::instance();
```

Second, it’s more than just a convenience when a type has an unknown or unutterable name, such as the type of most lambda functions, that you couldn’t otherwise spell easily or at all.

```cpp
// C++98
binder2nd< greater > x = bind2nd( greater(), 42 );
 
// C++11
auto x = [](int i) { return i > 42; };
```

Note that using auto doesn’t change the code’s meaning. The code is still statically typed, and the type of every expression is already crisp and clear; the language just no longer forces us to redundantly restate the type’s name.

Some people are initially afraid of using auto here, because it may feel like not (re)stating the type we want could mean we’ll get a different type by accident. If you want to explicitly *enforce a type conversion*, that’s okay; state the target type. The vast majority of the time, however, just use auto; it will rarely be the case that you get a different type by mistake, and even in those cases the language’s strong static typing means the compiler will usually let you know because you’ll be trying to call a member function the variable doesn’t have or otherwise use it as something that it isn’t.

### Smart pointers: No delete

Always use the standard smart pointers, and non-owning raw pointers. Never use owning raw pointers and delete, except in rare cases when implementing your own low-level data structure (and even then keep that well encapsulated inside a class boundary).

If you know you’re the only owner of another object, use unique_ptr to express unique ownership. A “new T” expression should immediately initialize another object that owns it, typically a unique_ptr. A classic example is the Pimpl Idiom (see [GotW #100](https://herbsutter.com/gotw/_100/)):

```cpp
// C++11 Pimpl idiom: header file
class widget {
public:
    widget();
    // ... (see GotW #100) ...
private:
    class impl;
    unique_ptr<impl> pimpl;
};
 
// implementation file
class widget::impl { /*...*/ };
 
widget::widget() : pimpl{ new impl{ /*...*/ } } { }
// ...
```

Use shared_ptr to express shared ownership. Prefer to use make_shared to create shared objects efficiently.

```
// C++98
widget* pw = new widget();
:::
delete pw;
 
// C++11
auto pw = make_shared<widget>();
```

Use weak_ptr to break cycles and express optionality (e.g., implementing an object cache).

```cpp
// C++11
class gadget;
 
class widget {
private:
    shared_ptr<gadget> g; // if shared ownership
};
 
class gadget {
private:
    weak_ptr<widget> w;
};
 
```

If you know another object is going to outlive you and you want to observe it, use a (non-owning) raw pointer.

```cpp
// C++11
class node {
 vector<unique_ptr<node>> children;
 node* parent;
public:
 :::
};
```

### nullptr

Always use nullptr for a null pointer value, never the literal 0 or the macro NULL which are ambiguous because they could be either an integer or a pointer.

```cpp
// C++98
int* p = 0;
 
// C++11
int* p = nullptr;
```

### Range for

The range-based for loop is a much more convenient way to visit every element of a range in order.

```cpp
// C++98
for( vector<int>::iterator i = v.begin(); i != v.end(); ++i ) {
    total += *i;
}
 
// C++11
for( auto d : v ) {
    total += d;
}
```

### Nonmember begin and end

Always use nonmember begin(x) and end(x) (not x.begin() and x.end()), because begin(x) and end(x) are extensible and can be adapted to work with all container types – even arrays – not just containers that follow the STL style of providing x.begin() and x.end() member functions.

If you’re using a non-STL collection type that provides iteration but not STL-style x.begin() and x.end(), you can often write your own non-member begin(x) and end(x) overloads for that type and then you can traverse collections of that type using the same coding style above as for STL containers. The standard has set the example: C arrays are such a type, and the standard provides begin and end for arrays.

```cpp
vector<int> v;
int a[100];
 
// C++98
sort( v.begin(), v.end() );
sort( &a[0], &a[0] + sizeof(a)/sizeof(a[0]) );
 
// C++11
sort( begin(v), end(v) );
sort( begin(a), end(a) );
```

### Lambda Functions and Algorithms

Lambdas are a game-changer and will frequently change the way you write code to make it more elegant and faster. Lambdas make the existing STL algorithms roughly 100x more usable. Newer C++ libraries increasingly are designed assuming lambdas as available (e.g., PPL), and some even require you to write lambdas to use the library at all (e.g., C++ AMP).

Here’s one quick example: Find the first element in v that’s >x and <y. In C+11, the simplest and cleanest code is to use a standard algorithm.

```cpp
// C++98: write a naked loop (using std::find_if is impractically difficult)
vector<int>::iterator i = v.begin(); // because we need to use i later
for( ; i != v.end(); ++i ) {
    if( *i > x && *i < y ) break;
}
 
// C++11: use std::find_if
auto i = find_if( begin(v), end(v), [=](int i) { return i > x && i < y; } );
```

Want a loop or similar language feature that’s not actually in the language? No sweat; just write it as a template function (library algorithm), and thanks to lambdas you can use it with *almost* the same convenience as if it were a language feature, but with more flexibility because it really is a library and not a hardwired language feature.

```cpp
// C#
lock( mut_x ) {
    ... use x ...
}
 
// C++11 without lambdas: already nice, and more flexible (e.g., can use timeouts, other options)
{
    lock_guard<mutex> hold { mut_x };
    ... use x ...
}
 
// C++11 with lambdas, and a helper algorithm: C# syntax in C++
// Algorithm: template<typename T> void lock( T& t, F f ) { lock_guard hold(t); f(); }
lock( mut_x, [&]{
    ... use x ...
});
```

Get familiar with lambdas. You’ll use them a lot, and not just in C++ – they are already widely available and pervasively used in several popular mainstream languages. A good place to start is my talk [Lambdas, Lambdas Everywhere](https://herbsutter.com/2010/10/30/pdc-languages-panel-andshortened-lambdas-talk/) at PDC 2010.

### Move / &&

Move is best thought of as an optimization of copy, though it also enables other things like perfect forwarding.

Move semantics change the way we design our APIs. We’ll be designing for return by value a lot more often.

```cpp
// C++98: alternatives to avoid copying
vector<int>* make_big_vector(); // option 1: return by pointer: no copy, but don't forget to delete
:::
vector<int>* result = make_big_vector();
 
void make_big_vector( vector<int>& out ); // option 2: pass out by reference: no copy, but caller needs a named object
:::
vector<int> result;
make_big_vector( result );
 
// C++11: move
vector<int> make_big_vector(); // usually sufficient for 'callee-allocated out' situations
:::
auto result = make_big_vector(); // guaranteed not to copy the vector
```

Enable move semantics for your type when you can do something more efficient than copy.

### Uniform Initialization and Initializer Lists

What hasn’t changed: When initializing a local variable whose type is non-POD or auto, continue using the familiar = syntax without extra { } braces.

```cpp
// C++98 or C++11
int a = 42;        // still fine, as always
 
// C++ 11
auto x = begin(v); // no narrowing or non-initialization is possible
```

In other cases, including especially everywhere that you would have used ( ) parentheses when constructing an object, prefer using { } braces instead. Using braces avoids several potential problems: you can’t accidentally get narrowing conversions (e.g., float to int), you won’t occasionally accidentally have uninitialized POD member variables or arrays, and you’ll avoid the occasional C++98 surprise that your code compiles but actually declares a function rather than a variable because of a declaration ambiguity in C++’s grammar – what Scott Meyers famously calls “C++’s most vexing parse.” There’s nothing vexing about the new style.

```cpp
// C++98
rectangle       w( origin(), extents() );   // oops, declares a function, if origin and extents are types
complex<double> c( 2.71828, 3.14159 );
int             a[] = { 1, 2, 3, 4 };
vector<int>     v;
for( int i = 1; i <= 4; ++i ) v.push_back(i);
 
// C++11
rectangle       w   { origin(), extents() };
complex<double> c   { 2.71828, 3.14159 };
int             a[] { 1, 2, 3, 4 };
vector<int>     v   { 1, 2, 3, 4 };
```

The new { } syntax works pretty much everywhere:

```cpp
// C++98
X::X( /*...*/ ) : mem1(init1), mem2(init2, init3) { /*...*/ }
 
// C++11
X::X( /*...*/ ) : mem1{init1}, mem2{init2, init3} { /*...*/ }
```

Finally, sometimes it’s just convenient to pass function arguments without a type-named temporary:

```cpp
void draw_rect( rectangle );
 
// C++98
draw_rect( rectangle( myobj.origin, selection.extents ) );
 
// C++11
draw_rect( { myobj.origin, selection.extents } );
```

The only place where I prefer not to write the braces is on simple initialization of a non-POD variable, like *auto x = begin(v);* , where it would make the code needlessly ugly because I know it’s a class type, so I know I don’t need to worry about accidental narrowing conversions, and modern compilers already routinely perform the optimization to elide the extra copy (or the extra move, if the type is move-enabled).

### And More

There’s more to modern C++. [Much more.](http://www2.research.att.com/~bs/C++0xFAQ.html) And in the future I plan to write more in-depth pieces about these and other features of C++11 we’ll get to know and love.

But for now, this is the list of must-know features. These features form the core that defines modern C++ style, that make C++ code look and perform the way it does, that you’ll see used pervasively in nearly every piece of modern code you’ll see or write… and that make modern C++ the clean, and safe, and fast language that our industry will continue relying on heavily for years to come.

### Major Change History

2011-10-30: Added C# lock example to lambdas. Reordered smart pointers to introduce unique_ptr first.

2011-11-01: Added uniform initialization.



