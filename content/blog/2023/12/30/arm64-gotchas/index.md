---
title: ARM64 Gotchas
comments: true
date: 2023-12-30
tags: 
    - Avalonia
    - General
    - Hardware
    - ARM64
    - Parallels
    - Performance
---

macOS on Apple Silicon runs great. It's fast, it's very reliable, it's quiet and it's beautiful. Windows 11 for ARM using Parallels on Apple Silicon also runs great, fast and reliable. There are still gotchas...

<!--more-->

## Dev Box
I've always been a fan of Apple Hardware. It's not cheap but quality and customer service is exceptional. I own two machines: A Mac Studio (M1) with two Studio Displays and a MacBook Pro (M3). On both machines I run macOS as well as Windows and Linux using [Parallels](https://www.parallels.com/eu/).

In the past you could use Bootcamp to run Windows natively on a Mac hardware but since Apple transitioned to their own ARM based chips, this is no longer an option. In addition you are also *forced* to run a version of Windows which is built for ARM chips ([Windows on ARM](https://learn.microsoft.com/en-us/windows/arm/overview)).

I think I will do a dedicated blog post about my development hardware and software with more details in the future but for now, let me say that working on Windows (for ARM) in Parallels on an Apple Silicon hardware for development is working really well, even if you develop a WinForms desktop app and want to build and deploy it for x64 architecture. You can run x64 binaries and even binaries which are mixed (e.g. an ARM64 .NET executable which has a reference to a DLL compiled for the x64 architecture).

## Caveats

### Performance
Let's take [Royal TS for Windows](https://royalapps.com/ts/win/features), which is available as x64 binary and as ARM64 binary. On my M1 Mac Studio running Windows 11 using Parallels, the startup time of the app alone is almost 4 times slower than running the native binary.

| Binary | Startup Time |
|--------|-------------:|
| ARM64 (native) | 6s |
| x64 (on ARM) | 22s|

Running the x64 binary, you definitely feel the slow startup time and some things in the UI will show a short delay when used for the first time but in general you can say the app is quite usable on ARM when running the x64 binary. This is good news if you need to run "legacy apps" which are **not** available natively on ARM64.

Luckily, Royal TS is available on ARM64 natively which makes it really fast and snappy!

### The Good
If you install Windows 11 on ARM and even some dev tools, you will see in the Task Manager that almost everything runs natively on the ARM architecture.

{{< cards >}}
  {{< card link="taskmanager.png" image="taskmanager.png" subtitle="Task Manager" method="Resize" options="600x q80 webp" >}}
{{< /cards >}}

Even apps like Visual Studio, Visual Studio Code or Rider run already natively on ARM64.

### The Bad
When I tried to run an Avalonia demo UI in a web browser, I noticed it was extremely slow and laggy. The app was basically not usable in the browser. After a while I realized, I'm running Google Chrome and guess what? 

> [!CAUTION]
> Google Chrome does not have an ARM64 binary available for Windows!

I never really noticed that Chrome is not running as a native ARM64 binary. Web browsing speed was quite OK but running an Avalonia app in Chrome is painfully slow!

> [!TIP]
> When running Windows 11 on ARM64, make sure you are using Microsoft's Edge browser when testing an Avalonia app in the web browser!

### The Ugly
Quite strange, to be honest. Both, Edge and Google Chrome are using the same Chromium engine under the hood. I don't really know why Google hasn't caught up yet. It's time for them to release an ARM64 version of Chrome.