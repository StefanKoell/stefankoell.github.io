---
title: Avalonia's Dispatcher
comments: true
date: 2024-02-29
tags: 
    - Avalonia
---

Lessons learned when using Avalonia's dispatcher and how to avoid choppy progress animations when scheduling jobs on the UI thread.

<!--more-->

## Introduction

Most application frameworks have a *single main thread* (UI thread) per process which handles everything related to UI and rendering. This is also known as **STA** or **Single Threading Apartment**. Concurrency is hard and limiting all UI based operations to a single UI thread keeps applications stable and consistent. Historically, pretty much all application frameworks - even a modern web browser - follow the STA concept.

So, running long and blocking operations on the UI thread will make your application unresponsive. UI updates and input will not be processed during the blocking operation. In .NET with C# there are two concepts which allows you to keep your app responsive:

* [Multi-threading](https://learn.microsoft.com/en-us/dotnet/api/system.threading.thread?view=net-8.0)
* [Task Parallel Library](https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/task-parallel-library-tpl)
* [async/await](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)

When you use multi-threading, the long and blocking operation runs on a different thread and keeps your UI responsive. This may also be the case when using TPL or async/await. This is great but if you want to report back to the UI to update a progress bar or some other UI element, you need to make sure it happens on the main/UI thread.

Similar to WPF, Avalonia features a Dispatcher which provides access to that UI thread. 

The `Dispatcher.UIThread` in [Avalonia](https://docs.avaloniaui.net/docs/guides/development-guides/accessing-the-ui-thread) allows you to conveniently run code on the UI thread using the `Post` (fire and forget) or `InvokeAsync` (awaitable) methods.

## Avoid UI Thread Congestion

Starting a thread in the background and using the Dispatcher to get marshalled back to the UI thread can still cause UI thread congestions. This can happen if you have a tight loop where you call an expensive method on the UI thread. For example, imagine you need to create thousands of objects on the UI thread and the constructor is quite heavy and takes a realtively long time to finish:

```csharp
private void CreateUIObjectsInBackground()
{
  for (int i = 0; i++; i < 10_000)
  {
    Dispatcher.UIThread.Post(() => CreateObject());
  }
}
```

In the above example, the method `CreateUIObjectsInBackground` runs on a seperate (non-UI) thread. The method `CreateObject()` must be executed on the UI thread. The code above looks great and one could argue, everything looks fine. Still, this can make the UI blocking or at least get choppy when it comes to animations. The reason for that is the way the job is queued on the dispatcher queue. The default dispatcher priority can interfere with input and rendering jobs.

To resolve the issue, you can work around that by simply specifying a [`DispatcherPriority`](https://reference.avaloniaui.net/api/Avalonia.Threading/DispatcherPriority/), like this:

```csharp
private void CreateUIObjectsInBackground()
{
  for (int i = 0; i++; i < 10_000)
  {
    Dispatcher.UIThread.Post(() => CreateObject(), DispatcherPriority.Background);
  }
}
```

According to the docs, this will make sure that all non-idle operations (like rendering) are done before the scheduled job passed on to the `Post` method is executed.
