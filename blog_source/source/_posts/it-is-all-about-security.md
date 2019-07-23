---
title: windows 开发遇到的一些权限问题
date: 2019-05-15 10:21:39
tags:
- windows
categories:
- coding
keywords:
- privilege
description:
---

最近碰到一个问题，折腾了很久。因为troubleshooting的过程中沟通不畅（种种原因暂且不表），导致像个无头苍蝇一样debug。最后发现是个windows权限的问题。好了，直接说问题。

<!--more-->

### 问题

问题简化为以下代码，是在windows下使用PIPE来进行进程间通信。首先看下server 端代码：

```c++
#include <Windows.h>
#include <stdio.h>

typedef struct _message {
	int id;
	char data[8];
} Message;

int main() {

	HANDLE h = INVALID_HANDLE_VALUE;
	const char* pipename = "\\\\.\\pipe\\mypipe";
	const int BUFSIZE = 1024;
	const int DEFAULT_TIMEOUT = 100;
	char buf[1024];

	h = CreateNamedPipe(
		pipename,
		PIPE_ACCESS_DUPLEX,/*| FILE_FLAG_OVERLAPPED*/
		PIPE_TYPE_MESSAGE | PIPE_READMODE_MESSAGE,
		PIPE_UNLIMITED_INSTANCES,
		BUFSIZE,
		BUFSIZE,
		DEFAULT_TIMEOUT,
		NULL
	);

	if (h == INVALID_HANDLE_VALUE) {
		exit(0);
	}

	if (ConnectNamedPipe(h, NULL)) {
		DWORD read = 0;
		if (ReadFile(h, buf, sizeof(Message), &read, NULL)
			&& read == sizeof(Message)) {
			Message* msg = reinterpret_cast<Message*>(buf);
			printf("get message from client: id: %d, data: %s\n", msg->id, msg->data);

			printf("ready to echo back!\n");

			DWORD written = 0;
			if (WriteFile(h, msg, sizeof(Message), &written, NULL)
				&& written == sizeof(Message)) {
				printf("echo back completed!\n");
			}
			else {
				printf("WriteFile failed: %d\n", GetLastError());
				exit(0);
			}
		}
		else {
			printf("ReadFile failed: %d\n", GetLastError());
			exit(0);
		}
	}
	else {
		printf("ConnectNamedPipe failed: %d\n", GetLastError());
		exit(0);
	}

	CloseHandle(h);

	getchar();
	exit(0);
}
```

代码很简单，使用WIN32 API `CreateNamedPipe`创建一个命名管道，然后等待pipe client来建立连接。收到client的message之后再echo回去。

再看下client端代码：

```c++
HANDLE h = INVALID_HANDLE_VALUE;
	const char* pipename = "\\\\.\\pipe\\mypipe";
	const int BUFSIZE = 1024;
	const int DEFAULT_TIMEOUT = 100;

	h = CreateFile(pipename,
		GENERIC_READ | GENERIC_WRITE,
		0,
		NULL,
		OPEN_EXISTING,
		0,
		NULL
	);
	if (h == INVALID_HANDLE_VALUE) {
		printf("open pipe failed: %d\n", GetLastError());
		exit(0);
	}

	Message m = { 1, "hello" };
	DWORD written = 0;
	if (WriteFile(h, &m, sizeof(Message), &written, NULL)
		&& written == sizeof(Message)) {

		printf("client write pipe finished\n");

		char buf[1024];
		DWORD read = 0;
		if (ReadFile(h, buf, sizeof(Message), &read, NULL)
			&& read == sizeof(Message)) {
			Message* msg = reinterpret_cast<Message*>(buf);
			printf("received message: id: %d, data: %s\n", msg->id, msg->data);
		}
		else {
			printf("read pipe failed: %d\n", GetLastError());
			exit(0);
		}

	}
	else {
		printf("Write Pipe failed: %d\n", GetLastError());
		exit(0);
	}

	CloseHandle(h);
	exit(0);
```

client代码也很简单，发起pipe connect，之后发送一个简单的message，然后收到server回复后退出。

但是在实际的生产环境中，发现client端无法连接到server端。根据GetLastError返回值是5，对应的error message是“Access Deny”。那肯定是权限问题嘛。果然，server端是使用Administrator运行的，而client端只有standard user的权限，因为Windows vista中加入的[Mandatory Integrity Control](<https://en.wikipedia.org/wiki/Mandatory_Integrity_Control>)(强制性完整性控制)，导致low integrity level的对象无法modify或者delete high integrity level的对象。

### 什么是Integrity Level

[MSDN](<https://docs.microsoft.com/en-us/previous-versions/dotnet/articles/bb625957(v=msdn.10)>)上如是说：

> The integrity level is a representation of the trustworthiness of running application processes and objects, such as files created by the application. The integrity mechanism provides the ability for resource managers, such as the file system, to use pre-defined policies that block processes of lower integrity, or lower trustworthiness, from reading or modifying objects of higher integrity. The integrity mechanism allows the Windows security model to enforce new access control restrictions that cannot be defined by granting user or group permissions in access control lists (ACLs).

抛开定义，首先先visualize一下Integrity level。我们使用[process explorer](<https://docs.microsoft.com/en-us/sysinternals/downloads/process-explorer>)工具来查看，如果没有这一栏，可以在View->Select Columns->Process Image tab中勾选Integrity level。

{% asset_img 01_show_integrity_level.png %}

我们可以看到Integrity level(以下简写为IL)分为几个等级：low, medium, high, system。根据[MSDN](<https://docs.microsoft.com/en-us/windows/desktop/secauthz/mandatory-integrity-control>):

> Windows defines four integrity levels: low, medium, high, and system. Standard users receive medium, elevated users receive high. Processes you start and objects you create receive your integrity level (medium or high) or low if the executable file's level is low; system services receive system integrity. Objects that lack an integrity label are treated as medium by the operating system; this prevents low-integrity code from modifying unlabeled objects. Additionally, Windows ensures that processes running with a low integrity level cannot obtain access a process which is associated with an app container.

标准用户得到meduim IL，这意味着标准用户启动的程序或创建的内核对象都拥有medium IL，除非在程序或对象中指定其为low IL。特权用户得到high IL。系统服务得到system IL。任何其他没有声明IL 标签的，系统默认其为medium IL。windows系统会保证低IL的对象无法读写/访问高IL的对象。

摘录一张《windows via c++》第四章最后一节的图：

{% asset_img 02_integrity_level_example.png %}

> When a piece of code tries to access a kernel object, the system compares the integrity level of the
> calling process with the integrity level associated to the kernel object. If the latter is higher than the
> former, modify and delete actions are denied. Notice that this comparison is done before checking
> ACLs. So, even though the process would have the right privileges to access the resource, the fact
> that it runs with an integrity level lower than the one required by the resource denies the requested
> access to the object.

简单来说，就是当访问一个内核对象的时候，系统会比较调用进程的IL和与内核对象关联的IL，如果内核对象关联的IL更高，则无法对其进行修改或者删除操作。一般可以进行读操作。

所以在上面的demo中，由于pipe server使用的是管理员用户运行的，则其创建的PIPE 内核对象拥有high 级别的IL。pipe client 使用标准用户运行，去访问pipe 内核对象时，由于只有medium 级别的IL，所以发生access deny的错误。

除了在进程间访问内核对象提供保护之外，IL还用在windows 的用户界面中，用于防止low IL的进程去修改high IL的界面。我们知道，windows提供了SendMessage和PostMessage来在窗口之间发送消息。窗体可以根据收到的消息来更新UI等。当一个low IL的进程通过SendMessage、PostMessage、SendInput发送消息给high IL的窗体时，系统会屏幕掉这类消息， 这个时候SendInput会返回0，并且GetLastError也不会显示出异常。这种机制称为**User Interface Privilege Isolation**（UIPI）。

这里再多说一句，一些杀毒软件，如360安全卫士会主动拦截SendMessage发出的消息。曾经就遇到过bug排查一天，最后发现是这个问题。

关于详细内容，可以参考《windows via c++》中第四章的内容，中文书名为《windows 核心编程》。

### 什么是UAC

### 什么事ACL

### 参考文档

- [What's New in User Account Control]([https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd446675%28v%3dws.10%29](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd446675(v%3dws.10)))

- [Windows Security Survival Guide](<https://social.technet.microsoft.com/wiki/contents/articles/2275.windows-security-survival-guide.aspx>)
- [Exploring the Windows Security Survival Guide – Integrity](<https://blogs.technet.microsoft.com/yuridiogenes/2011/04/13/exploring-the-windows-security-survival-guide-integrity/>)
- [What is the Windows Integrity Mechanism?](<https://docs.microsoft.com/en-us/previous-versions/dotnet/articles/bb625957(v=msdn.10)>)