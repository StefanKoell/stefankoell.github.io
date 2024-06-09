---
title: Why Avalonia
comments: true
date: 2024-01-03
tags: 
    - Avalonia
    - General
    - Accessibility

---

My personal view of things and why I think Avalonia is a great UI framework.

<!--more-->

## Coming from WinForms
I've used WinForms since the very beginning. Before the .NET Framework 1.0 release, I mostly developed in VB5 and VB6. Although I had a bit of a background in C/C++ and even Pascal (Delphi), I preferred Visual Basic for quick prototyping and UI work on Windows. Thinking about that time makes me really feel old.

With the release of .NET Framework 1.0, I started to learn C# and one of my first project was actually [Royal TS](https://www.royalapps.com/blog/more-than-10-years-of-royal-ts-history).

So I still work on the product. It's still WinForms using the [DevExpress](https://www.devexpress.com/) UI components. It's quite modern: using .NET 7, using SVG icons, designing state-of-the-art UI and UX. So, WinForms is still very important to me.

## Looking at WPF
I worked quite a bit with WPF in the past years and in general, I liked the way you can build apps using the MVVM pattern. WPF offers interop capabilities. So you can still use WinForms components.

What made me nervous about WPF was the lacking support of Microsoft. No improvements, no innovations. Even after MS open sourced WPF, it was still dead in the water. This has improved a bit lately but it's still not really encouraging.

## Looking at WinUI, Uno, MAUI
While WPF was really promising, all the successors were not really interesting, for various reasons. It felt like Microsoft isn't able to pull something off like WPF for the "modern" new world. Lack of APIs, no interop, etc. made it a hard sell. The ecosystem never reached a point where WinForms or WPF ever was. All the frameworks are quite limiting, especially in the interop department (more on that later).

## Avalonia
Since WinForms interop was a deal-breaker for me, I basically locked in on WPF but I always kept an eye on [Avalonia](https://www.avaloniaui.net/). Last year, Avalonia released version 11 with lots of great features and improvements, so I thought I re-evaluate the framework again and see if it fits my needs.

- Support for many platforms (Desktop, Mobile and even Web)
- Verify familiar XAML coming from WPF with lots of convenient improvments
- Much greater performance compared to WPF
- Native interop (on many platforms)

### Native Interop
The reason why most other UI frameworks are unusable for me is the lack of WinForms interop. Microsoft used to be very good at backwards compatibility but since WPF, MS never released a framework which allowed you to use WinForms/COM/ActiveX controls. WPF had a WindowsFormsHost control which allowed you to re-use legacy controls/UI. All successors do not have such a control and requires you to re-write everything. If you have the source code and the time, you might be able to pull that off, but in my case, I use a lot of 1st and 3rd party closed-source controls (like RDP ActiveX), it's not really an option.

Avalonia has a [NativeControlHost](https://docs.avaloniaui.net/docs/guides/platforms/android/embed-native-views) similar to what the WindowsFormsHost is to WPF. It's quite impressive because it even works cross-platform. [LibVLCSharp.Avalonia](https://github.com/videolan/libvlcsharp/tree/3.x/src/LibVLCSharp.Avalonia) is a great example of a true cross-platform Avalonia app which embeds the native VLC player on Windows, macOS and Linux.

A couple of months ago I released a nuget package with my own [WinFormsNativeHost](https://github.com/royalapplications/royalapps-community-avalonia) based on the NativeControlHost. It allows better control of the embedded WinForms control lifecycle in an Avalonia view. So the interop requirement seems to be met.

### Wishlist
Having spent a while with Avalonia, I do see some things which needs improvement. Don't get me wrong, Avalonia is a mature platform with a lot of apps already based on it. Besides that, the community is really active and issues are quite often resolved within hours or days. Compared to the progress we saw in the WPF github repo, this really feels like an active project with lots of responsive community members.

In any case, let me address some of the issues I encountered during my brief time with Avalonia.

#### Accessiblity
Often overlooked, but really necessary is good keyboard navigation and screen reader support. Avalonia checks all the boxes in theory but in reality there are still quite a few nasty bugs in the framework which makes it hard to navigate in more complex applications. Cycling through controls with the tab key is not working reliably and also using accelerator keys are not always working - especially when you have more than one with the same defined key. I hope this gets addressed in the upcoming months.

#### Native SVG Support
I admit it, I'm spoiled. Using the WinForms DevExpress components, I am able to leverage SVGs in my UI. This is a big deal in WinForms, especially when you work on DPI aware applications. Managing all the resources you need for the UI (button icons, toolbar glyphs, ...) is extremely easy with their somewhat propriatary implementation with styling support.

SVG has become a defacto-standard when it comes to icons and glyphs in the UI. Also, one of the common rendering backends of Avalonia is based on [Skia](https://skia.org/), similar to what Chrome is using. 

I really wish Avalonia has native SVG support. There are [open source libraries](https://github.com/wieslawsoltes/Svg.Skia) which you can use and even have good Avalonia integration. A couple of weeks ago one of the repo owners switched the license from MIT to GPL and reverted that change a couple of days later. This license change would have created a massive disruption for me and others. I even panicked briefly because the number of good SVG libraries - even commercial ones - is not that great. So yes, having built-in SVG support would be great and would make me feel much safer.

#### 3rd Party Ecosystem
WinForms and WPF has quite an extensive ecosystem of 3rd party components and frameworks. The ecosystem around Avalonia is still small but I can already see interesting stuff. [Actipro](https://www.actiprosoftware.com/docs/controls/avalonia/index) started already working on Avalonia based components. The big components like SyntaxEditor, Ribbon and Docking are still missing but I'm sure they are already working on it.

I guess it's just a matter of time but right now, there's not much to choose from.
