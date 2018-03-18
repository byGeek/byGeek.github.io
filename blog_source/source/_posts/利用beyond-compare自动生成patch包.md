---
title: 利用beyond compare自动生成patch包
date: 2018-03-12 10:33:40
tags:
- CI
categories:
- CI
keywords:
description:
---



## 前言

beyond compare对于程序员来说，可谓是一个不可多得的文件比较工具，试用过winMerge以及一些自带的diff工具之后，还是发现beyond compare界面最为友好，功能也比其他工具强大。



最近有一个需求是能否在每次release之后，可以比较方便的产生各个版本的升级patch包。持续集成的工具(类似jenkins，Travis CI)我没有了解过，不知道是否能有这个功能。但是仅仅对于这个小需求来说，利用beyond compare工具就可以做到。



<!--more-->



## 利用BC GUI生成patch包

1. 选中需要对比的两个文件夹，右键选择compare

2. 在BC中选择diff，只显示有变化的文件

3. 在BC菜单栏中Edit -> Expand all, Edit -> select all files

4. 在选中的文件中右击，选择copy to folder

   ![bc_copy_to_folder](bc_copy_to_folder.png)

5. 在打开的对话框中，选择要生成patch的文件(left/right side)，选择folder structure为base

   ![copy_to](copy_to.png)

6. patch包生成完毕。



## 编写脚本自动化

BC其实是支持脚本运行的，可移步与[此](https://www.scootersoftware.com/v4help/index.html?sample_scripts.html)。同时安装完BC之后，会在安装目录有CHM的帮助文件，具体的一些语法可参考该文件。



1. 编写脚本，将脚本保存为`bc_auto_script.txt`

   ```powershell
   log verbose "c:\bclog.txt"        #表示将脚本的log记录在bclog.txt中
   load "d:\testv1.0" "d:\testv2.0"     #加载需要比较的文件夹
   filter "-*.log;-lib\"             #利用beyong compare中的filter：除去*.log文件以及lib子文件夹，即这些不参与比较
   expand all                        #展开文件，这个命令与beyond compare中的UI的expand all是对应的
   select right.diff.files right.orphan.files   #只选取有差异的文件
   copyto right path:base "D:\diff.zip"        #将有差异的文件拷贝到d:\diff中，注意copyto命令的参数
                                               ##right: 对应上述load命令中的参数，即testv2.0，意思是将testv2.0的差异文件copy出来
                                               ##path:base: 指保留目录结构
                                               ##d:\diff: 输出目录，也可以指明为zip文件：如"d:\diff.zip"，这样最后会生成一个zip包
   ```

2. cmd下执行：`"C:\Program Files (x86)\Beyond Compare 4\Bcompare.exe" /silent "@D:\bc_auto_script.txt"`, BC安装路径， silent参数表示不启动GUI， 后面是在第一部中编写的脚本文件。

3. diff.zip即为生成的patch文件。如果要实现批量的生成patch包，可以编写一个批处理脚本。可参考[这里](https://github.com/byGeek/auto_patch)。



## 参考链接

- [beyond compare sample script](https://www.scootersoftware.com/v4help/index.html?sample_scripts.html)