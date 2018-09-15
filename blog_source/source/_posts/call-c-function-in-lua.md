---
title: call c function in lua
date: 2018-09-11 09:14:18
tags:
- lua
categories:
- coding
keywords:
description:
---



Code in [here](https://github.com/byGeek/CallCFunctionInLua).



<!--more-->

This project use Lua5.2.4, visual studio 2017.  You can find the lua binary file located in `TestLua/lua`folder.



To create a C library and export function to lua. Follow this steps:

- Define your function with following signature

  ```c
  typedef int (*lua_CFunction)(lua_State* L)
  ```

  example

  ```c
  int myadd(lua_State* L);
  ```

- Construct a `luaL_Reg` array

  ```c
  const struct luaL_Reg mylib[] ={
      {"myadd", myadd},
      {NULL, NULL}
  }
  ```

  and `luaL_Reg` is defined in `lauxlib.h`

  ```c
  typedef struct luaL_Reg {
    const char *name;
    lua_CFunction func;
  } luaL_Reg;
  ```

So the first is the function name which will be used in lua, and the second is the function address in c library. Note that the last element in this array must be `{NULL, NULL}`.

- Define a `luaopen_DLLName` function

  ```c
  int luaopen_TestLua(lua_State *L){
      luaL_newlib(L, mylib);
      return 1;
  }
  ```

  This function name should be match pattern `luaopen_dllname`, dllname is your dll's name

- Final step, Use this c library in your lua code.

  ```lua
  -- test.lua
  
  local t = require "TestLua"
  print(t.myadd(1, 2))
  ```

  when you run this `test.lua` script in lua environment, it should output correct answer. Remerber to use **same lua exe version** to run this script.

  Note that the module name is **case sensitive** because internally lua interpreter will call `luaopen_Modulename` function which exported by your c library.



Another note, you should export those symbols in c lib mentioned in the first three step and use `extern "C" `to make it follow the C standard if you are using c++.