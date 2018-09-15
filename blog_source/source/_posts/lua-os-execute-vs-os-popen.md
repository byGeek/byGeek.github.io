---
title: 'lua: os.execute vs os.popen'
date: 2018-09-13 17:34:52
tags:
categories:
keywords:
description:
---



最近遇到一个lua上问题：在一个c++工程中(进程A)，执行lua脚本，由该脚本去启动一个另外的进程B。在lua脚本执行完毕时，在C++工程中调用`lua_close`来退出lua环境时，线程stuck在该函数中。



<!--more-->

## 找到问题

使用Process Explorer查看C++工程的进程，发现进程B属于进程A的子进程。所以问题很明显了，因为子进程B未退出，所以进程A无法退出，调用`lua_close`时lua 解析器等待子进程退出，造成stuck问题。

下面来看下lua reference(lua 5.1)中提供的启动外部程序的API

## 查看API

### os.execute

> This function is equivalent to the C function `system`. It passes `command` to be executed by an operating system shell. It returns a status code, which is system-dependent. If `command` is absent, then it returns nonzero if a shell is available and zero otherwise.

os.execute只是将command传递给shell。查看lua源码(lua 5.2)

```c
static int os_execute (lua_State *L) {
  const char *cmd = luaL_optstring(L, 1, NULL);
  int stat = system(cmd);
  if (cmd != NULL)
    return luaL_execresult(L, stat);
  else {
    lua_pushboolean(L, stat);  /* true if there is a shell */
    return 1;
  }
}
```

可以看到直接调用的c标准库中的`system`。

### io.popen

> Starts program `prog` in a separated process and returns a file handle that you can use to read data from this program (if `mode` is `"r"`, the default) or to write data to this program (if `mode` is `"w"`).
>
> This function is system dependent and is not available on all platforms.

看功能说明可以知道，io.popen可以创建一个进程，并可以往该进程的IO读写数据。

io.popen源码如下：

```c
static int io_popen (lua_State *L) {
  const char *filename = luaL_checkstring(L, 1);
  const char *mode = luaL_optstring(L, 2, "r");
  LStream *p = newprefile(L);
  p->f = lua_popen(L, filename, mode);
  p->closef = &io_pclose;
  return (p->f == NULL) ? luaL_fileresult(L, 0, filename) : 1;
}
```

lua_popen是一个宏定义，最终调用c标准库的popen函数，该函数用来创建一个calling program与command的pipe，说白了就是创建一个子进程。

## 结论

在lua脚本中将启用外部程序的的API由io.popen换成os.execute。

