---
title: visual studio中版本号自动管理
date: 2018-04-04 09:13:35
tags:
- visual studio
categories:
- coding
keywords:
- Automatic Versions
- version
description:

---



在开发中，对版本号进行管理是一个很重要的事情。特别在visual studio中有多个project的时候，每一个project都有自己的version。下面介绍三种方式来实现版本号自动化管理。



<!--more-->

## 前言

先简单介绍下在visual studio中的三个version。

- Assembly Version

  > .net CLR加载DLL的时候使用的版本号，如果加载强签名的dll，需要指定dll的版本号，就是这个Assembly Version

- Assembly File Version

  > 在windows 资源管理器中显示的文件版本号

- Assembly Info Version

  > 在windows 资源管理器中显示的产品版本号

{% asset_img strong_named_assembly.png %}

{% asset_img version.png %}

## 利用VS自带的版本号递增

在project的AssemblyInfo.cs文件中，可以使用通配符来使得版本号自动变化。

```csharp
// Version information for an assembly consists of the following four values:
//
//      Major Version
//      Minor Version 
//      Build Number
//      Revision
//
// You can specify all the values or you can default the Build and Revision Numbers 
// by using the '*' as shown below:
// [assembly: AssemblyVersion("1.0.*")]
[assembly: AssemblyVersion("1.0.*")]
//[assembly: AssemblyFileVersion("1.0.0.0")]  注释掉该行
```

这样每次成功build之后，assembly version ，File version都会变化，其中build number和revision是以某个时间节点为起点，到build时计算出来的值。



在vs中，每个project都有一个AssemblyInfo.cs文件，在多个project之间，可以通过共享文件来共享同一个版本号。

- 首先将所有project的AssemblyInfo.cs文件中的AssemblyVersion和AssemblyFileVersion注释掉

- 在你的启动工程中创建一个SharedInfo.cs，加入以下内容:

  ```csharp
  using System.Reflection;

  [assembly: AssemblyVersion("1.0.*")]
  ```

- 在其他project中将上一步创建的文件通过add as link方式add进来。

  {% asset_img add_as_link.png %}



这样所有project都会共享SharedInfo文件。

## 使用T4模板生成版本号

在AssemblyVersion属性中使用通配符生成的版本号并不是那么“友好”。有的时候仅仅是想让版本号加一而已。使用T4(**Text Template Transformation Toolkit**)模板自动生成代码可以做到这一点。

> In Visual Studio, a *T4 text template* is a mixture of text blocks and control logic that can generate a text file.

> T4 is used by developers as part of an application or tool framework to automate the creation of text files with a variety of parameters. These text files can ultimately be any text format, such as code (for example C#), XML, HTML or XAML.

在vs中创建一个SharedInfo.tt文件(new ->item -> text template)，加入以下内容:

```csharp
<#@ template debug="false" hostspecific="true" language="C#" #>
<#@ import namespace="System.IO" #>
<#@ output extension=".cs" #>
<#
     int major = 0; 
     int minor = 0; 
     int build = 0; 
     int revision = 0; 
  
     try
     {
         using(var f = File.OpenText(Host.ResolvePath("SharedInfo.cs")))
         {
             string maj = f.ReadLine().Replace("//","");
             string min = f.ReadLine().Replace("//","");
             string b = f.ReadLine().Replace("//","");
             string r = f.ReadLine().Replace("//","");
  
             major = int.Parse(maj); 
             minor = int.Parse(min); 
             build = int.Parse(b); 
             revision = int.Parse(r) + 1; 
         }
     }
     catch
     {
         major = 1; 
         minor = 0; 
         build = 0; 
         revision = 0; 
     }
 #>
 //<#= major #>
 //<#= minor #>
 //<#= build #>
 //<#= revision #>
 // 
 // This code was generated by a tool. Any changes made manually will be lost
 // the next time this code is regenerated.
 // 
  
 using System.Reflection;
  
 [assembly: AssemblyFileVersion("<#= major #>.<#= minor #>.<#= build #>.<#= revision #>")]
```

保存之后，会在SharedInfo.tt文件下面生成SharedInfo.cs文件。然后将该文件通过add as link 加入到其他project中。这样每次要生成一个新的版本号之后，需要在SharedInfo.tt文件的右键菜单中`Run Custom tool`，重新生成SharedInfo.cs文件。

当然也可以在使用pre-build事件，自动化这一过程。在project的pre-build事件中加入以下内容：

```bat
set textTemplatingPath="%CommonProgramFiles(x86)%\Microsoft Shared\TextTemplating\$(VisualStudioVersion)\texttransform.exe"
if %textTemplatingPath%=="\Microsoft Shared\TextTemplating\$(VisualStudioVersion)\texttransform.exe" set textTemplatingPath="%CommonProgramFiles%\Microsoft Shared\TextTemplating\$(VisualStudioVersion)\texttransform.exe"
%textTemplatingPath% "$(ProjectDir)SharedInfo.tt"
```

{% asset_img pre_build.png %}

## 使用VS插件Automatic Versions 1

最简单也是最强大的莫过于直接使用[Automatic Versions](https://marketplace.visualstudio.com/items?itemName=PrecisionInfinity.AutomaticVersions)插件了。该VS插件提供了多种策略来实现版本号自动递增，可以支持整个solution和单个project的versioning继承。即可以为整个solution设置versionning的规则，也可以为单个project设置不继承solution的规则，自己按自己的规则。



安装好之后，在vs菜单栏tools -> automatic versions settings，打开设置面板。

{% asset_img automatic_version_setting %}

设置选项参考官方的文档：

> CustomSystem.Version Options
>
> Major, Minor, Build, andRevision can be individually configured to the incrementation type desired:
>
> {% asset_img clip_image001.png %}
>
> In these examples, we usethe date July 29th 2024, 1:30pm
>
> - None -     Do not increment this value.
> - Increment (Always) - Always Increments this value by 1*
> - Increment w/AutoReset - Increments this value by     1, unless a more significant number increases, in which case it will reset     to 0.
> - On Demand (Build New     Version) - Increment this value     by 1 only when a build is initiated using the Build New Version command (on the     context menu of each project). You can set this value on both the Major     and the Minor numbers and use the context menu (see usage below) to     control which one is incremented.
> - On Demand w/Reset (Obsolete) - Use Increment w/AutoReset instead.     This feature may not be supported in future versions of the product.     (Legacy functionality: Increment this value by 1 only when a build is     initiated using the Build New Version command (on the context menu of each     project). Resets all Increment sub-values.)
> - Day Of Year (ddd) - Set this value to DateTime.UtcNow.DayOfYear:     211.
> - Day (dd) -     Set this value to the current day of the month: 29.
> - Month (MM) - Set this value to the current month of the year:     7.
> - Year (yyyy) - Set this value to the current year: 2024.
> - Short Year (yy) - Set this value to 2 digit     current year: 24.
> - Date (yyddd) - Set this value to the current date (yyddd format     - where ddd is the day of year)**: 24211.
> - Date (MMdd) - Set this value to the     current date (MMdd) where MM is a 2-digit month and dd is a 2-digit day of     month: 0729.
> - UTC Time (HHmm) - Set this value to the current time (HHmm     format): 1330.
> - Delta Days (since 1/1/2000) - Set this value to the number of days that have     occurred since January 1, 2000: 8976.
> - UTC Seconds Since Midnight/2 - If you are using a custom     unique time based stamp for your version number, this gives you the most     granularity to a single day. The number has to be divided by 2 so it     doesn't overflow the max value.
>
> *There is an exception to this rule whenIncrement On Demand w/Reset (Obsolete) is used it will reset this value if youchoose Build New Version
>
> **This value will overflow in 2065. Since itis very useful and there are limited alternatives that work as well as thisone, we are continuing to use it and even recommend using it, however, we maychange it's functionality at some point such that it does not overflow in 2065.The overflow itself is not a security issue, however. It will only cause abuild error.
>
> CustomSemantic Version (BETA/PRO)
>
> Semantic Versioning istypically used for 'packaging' the results of a build for use in a manifestfile or as a 'published package' of your product. Therefore Semantic Versioningleverages the other version attributes by letting you choose which of those youwant to use for your Major/Minor numbers, and independently does it's ownincrementation on the Patch and Pre-Release numbers.
>
> {% asset_img clip_image002.png %}
>
> - Major/Minor settings - Set to use     AssemblyVersion, AssemblyFileVersion, or Set Manually. To set manually,     open the AssemblyInfo file (for Full .Net) or the project properties (for     .Net Standard/Core) and manually set the major and minor values.
> - Patch settings - This will increment the     Patch number based on the incrementation settings. Increment Once is     available for when you switch to a 'pre-release' version of the product,     you will typically want the Patch number to Increment Once for the start     of your pre-release cycle. After incrementing once, this setting will     automatically update to None (whereby Pre-Release will be doing the     incrementation).
> - Pre-Release (optional) - Set to alpha, beta,     preview, rc, or N/A (release). If you set a pre-release, then Patch will     automatically change to "increment once" for you and the     prerelease will increment instead starting with alpha, then alpha-01,     alpha-02, etc. When you set this to (N/A release) then Patch will be     automatically changed to Increment w/AutoReset for you.

注意一点：

OnDemand 表示在工程的右键菜单中手动选择build

{% asset_img on_demand.png %}



## 参考链接

- https://jonthysell.com/2017/01/10/automatically-generating-version-numbers-in-visual-studio/
- https://marketplace.visualstudio.com/items?itemName=PrecisionInfinity.AutomaticVersions
- https://weblogs.asp.net/kon/assembly-file-version-auto-increment-magic
- https://docs.microsoft.com/en-us/visualstudio/modeling/code-generation-and-t4-text-templates
