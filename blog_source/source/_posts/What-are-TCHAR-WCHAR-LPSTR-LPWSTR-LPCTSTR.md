---
title: What are TCHAR, WCHAR, LPSTR, LPWSTR, LPCTSTR
date: 2018-06-19 10:34:56
tags:
- c++
categories:
- learning
keywords:
- c++ basic
description:
---



前段时间看到一篇关于C++中TCHAR，LPSTR的基础知识的文章，写的非常清晰易懂，看完之后觉得解决了以前一直郁结在心中的一些问题。推荐去看下原文：[What are TCHAR, WCHAR, LPSTR, LPWSTR, LPCTSTR ](https://www.codeproject.com/Articles/76252/What-are-TCHAR-WCHAR-LPSTR-LPWSTR-LPCTSTR-etc)。本文简单归纳总结下。

<!--more-->

## 缘由

字符可以使用一个byte或2个byte来表示。传统的ANSI字符是由一个byte表示，但是一个byte只能编码256个字符，显然无法满足所有的字符（比如说中文字符等）。所以后来创造出来unicode编码，可以用来表示任何字符。（Unicode编码只是一个统称，UTF-8，UTF-16，UTF-32都属于Unicode编码。）



## TCHAR宏

在visual c++ 编译器中原生支持char(8 bits)和wchar_t(16 bits)类型。在代码中，我们应该使用更通用的类型来表示字符和字符串，已获得更好的移植性和健壮性。TCHAR类型就是这样定义出来的。查看TCHAR.h头文件，可以看到：

```cpp
#ifdef _UNICODE
typedef wchar_t TCHAR;
#else
typedef char TCHAR;
#endif
```

通过条件编译，TCHAR只是wchar_t或char的别名。在visual studio中，当将character set 设置为"Use Unicode Character Set "时，会自动定义`_UNICODE`这个symbol，所以TCHAR类型就会被翻译成wchar_t。当设置为"Use Multi-Byte Character Set"，TCHAR类型会被翻译为char。

{% asset_img TCHAR.png %}

同样的，避免直接使用string的一些库函数如`strlen,strcpy,strcat`或者`wcslen, wcscpy, wcscat`，而应该使用`tcslen,_tcscpy,_tcscat`。原有的函数原型和TCHAR.H头文件中的声明:

```cpp
size_t strlen(const char*);

size_t wcslen(const wchar_t* );  //wide character string length

//tchar.h中的声明
size_t _tcslen(const TCHAR* );

#ifdef _UNICODE
#define _tcslen wcslen 
#else
#define _tcslen strlen
#endif
```



## SetWindowTextW vs SetWindowTextA

在查阅msdn中，经常可以看到windows的API分为两个版本，其实就是char与wchar_t的区别导致的。

```cpp
// WinUser.H
#ifdef UNICODE
#define SetWindowText  SetWindowTextW
#else
#define SetWindowText  SetWindowTextA
#endif // !UNICODE
```

通过宏定义，将这些区别隐藏起来，客户端只需要直接调用SetWindowText。注意SetWindowText只是一个宏定义，在dll中并不存在真正的SetWindowText函数入口！

```cpp
HMODULE hDLLHandle;
FARPROC pFuncPtr;

hDLLHandle = LoadLibrary(L"user32.dll");

pFuncPtr = GetProcAddress(hDLLHandle, "SetWindowText");
//pFuncPtr will be null, since there doesn't exist any function with name SetWindowText !
```

在user32.dll中导出的是``SetWindowTextA`和`SetWindowTextW`，所以去获取`SetWindowText`函数的入口地址返回为NULL。



## _T宏, _TEXT宏

我们经常使用直接使用双引号标记一个string。但是这种方式定义的字符串是ANSI-string，意味着每个字符用一个byte来表示。

```cpp
"This is ANSI String. Each letter takes 1 byte."
```

如果要表示Unicode-string，需要在字符串前加入`L`前缀。如：

```cpp
L"This is Unicode string. Each letter would take 2 bytes, including spaces."
```

这样表示的所有字符都将使用两个byte来表示。

事实上，在TCHAR.h头文件中定义了两个宏来隐藏硬编码string导致的不同。

```cpp
// SIMPLIFIED
#ifdef _UNICODE 
 #define _T(c) L##c
 #define TEXT(c) L##c
#else 
 #define _T(c) c
 #define TEXT(c) c
#endif
```

\#\#符号是[token pasting operator](<https://docs.microsoft.com/en-us/cpp/preprocessor/token-pasting-operator-hash-hash> )，用来宏定义参数中作为连接符号，将两个token连接在一起。

```cpp
"ANSI String"; // ANSI
L"Unicode String"; // Unicode

_T("Either string, depending on compilation"); // ANSI or Unicode
// or use TEXT macro, if you need more readability
```



## WCHAR, LPSTR, LPWSTR, LPCSTR, LPCWSTR, LPTSTR, LPCTSTR

这些宏定义在winnt.h文件中定义。在MSDN中[data type](https://msdn.microsoft.com/en-us/library/windows/desktop/aa383751%28v=vs.85%29.aspx)中有详细说明。通常来说：

- **LP** - Long Pointer
- **C** - Const
- **STR** - String
- **WSTR** - Wide character String
- **T**- TCHAR

### WCHAR

`typedef wchar_t WCHAR;    // wc,   16-bit UNICODE character`

### LPSTR

`typedef char *LPSTR;  //A pointer to a null-terminated string of 8-bit Windows (ANSI) characters.`

### LPWSTR

`typedef wchat_t *LPWSTR;  //A pointer to a null-terminated string of 16-bit Unicode characters. `

### LPCSTR

`typedef const char *LPCSTR; //A pointer to a constant null-terminated string of 8-bit Windows (ANSI) characters.`

### LPCWSTR

`typedef const wchat_t *LPCWSTR; //A pointer to a constant null-terminated string of 16-bit Unicode characters. `

### LPTSTR

```cpp
#ifdef UNICODE
 typedef LPWSTR LPTSTR;
#else
 typedef LPSTR LPTSTR;
#endif
```

### LPCTSTR

```cpp
#ifdef UNICODE
 typedef LPCWSTR LPCTSTR; 
#else
 typedef LPCSTR LPCTSTR;
#endif
```



## Example

经过上面一番讨论，在TCHAR，LPSTR, LPWSTR，LPTSTR的使用过程中，需要注意不要混用，否则容易出现一些“莫名其妙的错误”。如下：

```cpp
int main()
{
    TCHAR name[] = "Saturn";
    int nLen; // Or size_t

    lLen = strlen(name);
}
```

使用ANSI char set编译通过，但是使用charset为unicode编译出错。

修改后的code：

```cpp	
TCHAR name[] = _T("Saturn");
size_t len;
len = _tclen(name);
```

同时在申请内存的时候也要注意sizeof的类型。

```cpp
LPTSTR pBuffer;
pBuffer = new TCHAR[128];  //申请一个128字符的空间，实际大小可能是128bytes或者256bytes
pBuffer = (TCHAR *)malloc(128 *sizeof(TCHAR));
```





