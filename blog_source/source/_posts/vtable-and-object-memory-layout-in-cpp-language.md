---
title: vtable and object memory layout in cpp language
date: 2018-10-22 11:07:44
tags:
- c++
categories:
- coding
keywords:
description:
---



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

