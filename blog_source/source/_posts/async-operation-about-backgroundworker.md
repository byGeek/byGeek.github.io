---
title: 从BackgroundWorker说起
date: 2018-08-13 15:33:58
tags:
- csharp
categories:
- coding
keywords:
description:
---



BackgroundWorker用于在另一个线程里执行一些操作。以前经常在UI thread启动一个workder对象，去完成一个耗时操作。然后在其ProgressChanged事件的EventHandler中去更新UI. 在.net 中，创建UI control和access的线程必须是同一个线程，否则操作是非法的。在文档中可以查到在ProgressChanged的EventHandler中是可以安全的更新UI的。那么在BackgroundWorker中是如何做的呢？



<!--more-->

首先，BackgroundWorker的DoWork的EventHandler肯定需要在另外的线程中运行，而要想在安全的更新UI，必须要Invoke到UI thread。所以一开始我觉得应该是在BackgroundWorker中调用了Invoke操作。查看其reference code:

```c#
private readonly WorkerThreadStartDelegate  threadStart;
        private readonly SendOrPostCallback operationCompleted;
        private readonly SendOrPostCallback progressReporter;
 

public BackgroundWorker()
        {
            threadStart        = new WorkerThreadStartDelegate(WorkerThreadStart);
            operationCompleted = new SendOrPostCallback(AsyncOperationCompleted);
            progressReporter   = new SendOrPostCallback(ProgressReporter);
        }

public void RunWorkerAsync(object argument)
        {
            if (isRunning)
            {
                throw new InvalidOperationException(SR.GetString(SR.BackgroundWorker_WorkerAlreadyRunning));
            }
 
            isRunning = true;
            cancellationPending = false;
            
            asyncOperation = AsyncOperationManager.CreateOperation(null);
            threadStart.BeginInvoke(argument,
                                    null,
                                    null);
        }
```



可以看到在BackgroundWorker中是通过delegate的BeginInvoke来实现异步操作的。所以Backgroundworker适用于执行一些轻量级的小任务。



上面代码中创建了一个`AsyncOperation`对象，在BackgroundWorker中ReportProgress就是使用该对象调用Post方法将progressReporter传递出去。progressReprorter会触发ProgressChanged事件。

```c#
public void ReportProgress(int percentProgress, object userState)
        {
            if (!WorkerReportsProgress)
            {
                throw new InvalidOperationException(SR.GetString(SR.BackgroundWorker_WorkerDoesntReportProgress));
            }
            
            ProgressChangedEventArgs args = new ProgressChangedEventArgs(percentProgress, userState);
 
            if (asyncOperation != null)
            {
                asyncOperation.Post(progressReporter, args);
            }
            else
            {
                progressReporter(args);
            }
        }
```



进入到`AsyncOperationManager`源代码中：

```c#
public static AsyncOperation CreateOperation(object userSuppliedState) {
            return AsyncOperation.CreateOperation(userSuppliedState, SynchronizationContext);
        }

public static SynchronizationContext SynchronizationContext {
            get {
                if (SynchronizationContext.Current == null) {
                    SynchronizationContext.SetSynchronizationContext(new SynchronizationContext());
                }
 
                return SynchronizationContext.Current;
            }
```

这里面使用到了一个对象`SynchronizationContext`, 同步上下文对象。看起来上面的调用的Post方法就是调用`SynchronizationContext`对象的Post方法:

```c#
public virtual void Post(SendOrPostCallback d, Object state)
        {
            ThreadPool.QueueUserWorkItem(new WaitCallback(d), state);
        }
```

是直接利用线程池来执行progressReporter 这个callback。但是我们注意到这个方法是一个virtual方法，而且在`AsyncOperation`中获取`SynchronizationContext`对象先判断`SynchronizationContext.Current`是否为null，如果是则新建一个`SynchronizationContext`对象，这种情况下是直接调用virtual方法的，也即使用线程池来执行callback。否则使用`SynchronizationContext.Current`对象来执行Post。那么这个Current对象又是什么呢？



查阅了一些资料，特别是MSDN上的这篇[博客](https://blogs.msdn.microsoft.com/pfxteam/2012/06/15/executioncontext-vs-synchronizationcontext/).

> SynchronizationContext is just an abstraction, one that represents a particular environment you want to do some work in.  As an example of such an environment, Windows Forms apps have a UI thread (while it’s possible for there to be multiple, for the purposes of this discussion it doesn’t matter), which is where any work that needs to use UI controls needs to happen.  For cases where you’re running code on a ThreadPool thread and you need to marshal work back to the UI so that this work can muck with UI controls, Windows Forms provides the Control.BeginInvoke method.  You give a delegate to a Control’s BeginInvoke method, and that delegate will be invoked back on the thread with which that control is associated. 



`SynchoronizationContext`提供了具体上下文的抽象，提供了Post和Send方法接口，比如在一个worker thread里，你想update UI，则必须marshel到UI thread。

在windows form中我们使用control.BeginInvoke
在wpf中我们使用Dispatcher.BeginInvoke

那么SynchronizationContext就是对这个操作做了统一的封装。
提供了一个virtual的Post方法。

当实现类为WindowsFormSynchronizationContext （即在windowsfrom中）使用control.BeginInvoke

当实现类为DispatcherSynchronizationContext（即在wpf中）使用Dispatcher.BeginInvoke

SynchronizationContext还提供一个同步的Send方法。
在windows form中具体实现是： control.Invoke
在wpf中具体实现是： Dispatcher.Invoke



我们找到`WindowsFormSynchronizationContext`的源代码中的Post方法：

```c#
public override void Post(SendOrPostCallback d, Object state) {
            Debug.Assert(controlToSendTo != null, "Should always have the marshaling control by this point");
 
            if (controlToSendTo != null) {
                controlToSendTo.BeginInvoke(d, new object[] { state });
            }
        }
```

可以看到确实使用control.BeginInvoke来将操作marshel到UI thread。



关于`SynchronizationContext`，还可以参考这篇文章:  [It's All About the SynchronizationContext](https://msdn.microsoft.com/magazine/gg598924.aspx)