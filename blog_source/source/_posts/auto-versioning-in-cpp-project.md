---
title: 在VS C++工程中使用Auto versionning
date: 2018-12-17 16:12:33
tags:
- C++
categories:
- coding
keywords:
- auto versionning
description:
---

[前面](https://bygeek.cn/2018/04/04/automatic-versionning-in-visual-studio/)我已经总结了在csharp中如何auto versionning来管理Assembly的版本号。本文总结一下在C++下如何方便的管理DLL的版本号。

<!--more-->

### 遇到的问题

首先明确一下本文要解决的问题：

> 在一个C++ solution中实现DLL或者EXE共享同一个版本号。

既然要共享同一个版本号，那么最简单的类似csharp中的共享同一个AssemblyInfo文件了。csharp工程可以通过Add as Link方式将一个文件共享给其他project。在C++ project中自然也可以通过Add Existing File来实现这个目的。不过在VS2015之后Visual Studio支持了Shared Project Template。那么可以将version的信息放在Shared Project中，这样修改起来也方便。

### 解决办法

在C++ project中version信息是放在Resource.rc中的。

假设现在我们的代码结构是这样的：

> solution
>
> +-- project1
>
> +-- project2
>
> +-- sharedproject

首先我们给project1和project2工程建立Resource文件。

> Project-> Add->Resource->Version->New

右击生成的Resource.rc文件, 选择View Code，滑动到Version部分。可以看到Version信息。

接下来在sharedproject中建立一个头文件verson.h

```c++
#define STRINGIZE2(s) #s   //stringizing operator
#define STRINGIZE(s) STRINGIZE2(s)

#define VERSION_MAJOR               1
#define VERSION_MINOR               0
#define VERSION_REVISION            0
#define VERSION_BUILD               0

//#define VER_FILE_DESCRIPTION_STR    "Description"  //decription和productionname各自独立，需要单独定义
#define VER_FILE_VERSION            VERSION_MAJOR, VERSION_MINOR, VERSION_REVISION, VERSION_BUILD
#define VER_FILE_VERSION_STR        STRINGIZE(VERSION_MAJOR)        \
                                    "." STRINGIZE(VERSION_MINOR)    \
                                    "." STRINGIZE(VERSION_REVISION) \
                                    "." STRINGIZE(VERSION_BUILD)    \

//#define VER_PRODUCTNAME_STR         "c_version_binary"
#define VER_PRODUCT_VERSION         VER_FILE_VERSION
#define VER_PRODUCT_VERSION_STR     VER_FILE_VERSION_STR
#define VER_ORIGINAL_FILENAME_STR   VER_PRODUCTNAME_STR ".exe"
#define VER_INTERNAL_NAME_STR       VER_ORIGINAL_FILENAME_STR
#define VER_COPYRIGHT_STR           "Copyright (C) 2011"

#ifdef _DEBUG
  #define VER_VER_DEBUG             VS_FF_DEBUG
#else
  #define VER_VER_DEBUG             0
#endif

#define VER_FILEOS                  VOS_NT_WINDOWS32
#define VER_FILEFLAGS               VER_VER_DEBUG
#define VER_FILETYPE                VFT_APP
//注意，这里需要有一个空行，否则在resource中include这个头文件会报"unexpected end of file"错误

```

然后分别在project1和project2的Resource 属性中将Additional Include Directory将version.h的路径加进去。

> Project Property ->Resource -> General -> Additional Include Directories

注意是Resource选项卡，不是C++选项卡中的设置。

然后分别对project1和project2的Resource文件做如下操作：

- include version.h，定义description和product name

  ```cpp
  // Microsoft Visual C++ generated resource script.
  //
  #include "resource.h"
  
  #define VER_PRODUCTNAME_STR         "product_name_here"
  #define VER_FILE_DESCRIPTION_STR    "description_here"
  
  #include "version.h"
  ```

- 替换version section

  ```cpp
  /////////////////////////////////////////////////////////////////////////////
  //
  // Version
  //
  VS_VERSION_INFO VERSIONINFO
   FILEVERSION        VER_FILE_VERSION
   PRODUCTVERSION     VER_PRODUCT_VERSION
   FILEFLAGSMASK      0x3fL
   FILEFLAGS          VER_FILEFLAGS
   FILEOS             VER_FILEOS
   FILETYPE           VER_FILETYPE
   FILESUBTYPE        0x0L
  BEGIN
      BLOCK "StringFileInfo"
      BEGIN
          BLOCK "040904b0"
          BEGIN
              VALUE "FileDescription",  VER_FILE_DESCRIPTION_STR "\0"
              VALUE "FileVersion",      VER_FILE_VERSION_STR "\0"
              VALUE "InternalName",     VER_INTERNAL_NAME_STR "\0"
              VALUE "LegalCopyright",   VER_COPYRIGHT_STR "\0"
              VALUE "OriginalFilename", VER_ORIGINAL_FILENAME_STR "\0"
              VALUE "ProductName",      VER_PRODUCTNAME_STR
              VALUE "ProductVersion",   VER_PRODUCT_VERSION_STR "\0"
          END
      END
      BLOCK "VarFileInfo"
      BEGIN
          VALUE "Translation", 0x409, 1200
      END
  END
  ```

Build一下工程，并修改version.h头文件，看project1和project2的版本号是否是version里设置的版本号。

下次如果要修改版本号，就不用一个个去改每个工程的resource.rc文件了，直接修改version.h即可。

### 进阶

上面的方法需要每次都手动修改version信息，可以利用visual studio中的Build Event自动将version.h信息更新。Share Project Template不支持VS project Build Event，可以将其换正常的Project类型。

具体请参考Code Project的一篇文章：[Automatic Build Versioning in Visual Studio](https://www.codeproject.com/Articles/10313/Automatic-Build-Versioning-in-Visual-Studio).

### 参考链接

- [Versioning a Native C/C++ Binary with Visual Studio](https://www.zachburlingame.com/2011/02/versioning-a-native-cc-binary-with-visual-studio/)
- [unexpected end of file found](http://www.cplusplus.com/forum/windows/64819/)

