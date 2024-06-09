---
title: Development Hardware and Tools
comments: true
date: 2024-01-07
tags: 
    - Avalonia
    - General
    - Hardware
---

What am I using for development in terms of hardware and software (as of Jan, 2024).

<!--more-->

## Hardware
In terms of hardware, I guess it strongly depends what kind of application you write and what kind of platforms you target. Nowadays you can do a lot with virtualization. If you develop a web application which runs on a docker container based on some Linux distro, you can pick whatever you want. Virtualization, docker, Windows Subsystem for Linux, just to name a bunch, allows you to run almost everything on everything. I know, this is a very simplified view and the devil often is in the details.

However, if you are targeting macOS, iOS or any other Apple platform, you actually need a Mac. I know there are "options" (Hackintosh or running macOS in a VM) but in my experience, these "hacks" are causing more problems and headaches than it actually tries to resolve. 

Then there's always the option to get a decent PC and a cheap Mac Mini or something, just for building the stuff. This is of course a viable option but keep in mind that tending to that dedicated build machine (keeping it updated/patched) is also an effort.

The way I set things up is a bit pragmatic but it does get the job done and I'm really happy with it. I basically run macOS on a Mac with Docker and with Parallels running Windows and Linux. This allows me to target "everthing" from one box.

**Development Hardware:**
* Mac Studio (M1 Max, 64GB RAM), 2 Studio Displays
* MacBook Pro 14" (M3 Pro, 36GB RAM)

**Test Hardware:**
* AMD Ryzen9
* Surface Laptop 3
* iPad Pro M1
* iPhone 13 Pro

## Software

These are the main tools I'm using:
* [JetBrains (Rider)](https://www.jetbrains.com/rider/)
* [Visual Studio Code](https://code.visualstudio.com/)
* [Fork](https://git-fork.com/) is still my favorite git client for macOS and Windows

For graphical work, I mainly use [Affinity Photo](https://affinity.serif.com/en-us/photo/) and [Affinity Designer](https://affinity.serif.com/en-us/designer/). Both of these tools are very affordable and run on Windows and on macOS.

### JetBrains (Rider)
In terms of software, I did something similar with the hardware. Luckily I'm able to do so. I switched to [JetBrains](https://www.jetbrains.com/) tools a while back and most of the tools I need for my work are available on Windows, macOS and even Linux. The .NET/C# IDE Rider, for example, *can run on all those operating systems - even on ARM64*.

Having used Visual Studio for a long time before I switched, it was a bit of a transition but I'm telling you the truth that I really don't miss Visual Studio at all.

### Parallels
Since I'm using Mac hardware, Parallels is probably the best way to run Windows on a Mac. There are other options (vmware, for example) but Parallels was there from the very beginning and it even works great on Apple Silicon running Windows 11 on ARM. There are a [couple of things you should be aware of when running Windows on ARM](/blog/2023/12/30/arm64-gotchas/).

What I really find impressive is how good the experience is with Windows 11 on ARM on a Mac is. Performance is extremely good. Build and run times are very fast and everything runs extremely smooth and reliable. I know there's actually ARM based PC hardware out there. From what I've seen, the hardware and OS still doesn't run as reliable as on Intel/AMD hardware. I don't have any numbers but I've seen quite a few devs on social media complaining that there Windows on ARM is acting up and reinstalling/repairing the OS isn't as smooth as you want it to be. I'm running my ARM based setup since the Mac Studio has been released, so almost two years now, and I've never had any issues on the Mac or Windows side of things. Quite an achievement! (also knowcking on wood)

I also use Parallels to run the ARM version of Ubuntu with Rider. High DPI display support is still not great so I may try different distros in the future to see if they can better handle that.

### Font
My development font of choice is also from JetBrains: [JetBrains Mono](https://www.jetbrains.com/lp/mono/).
