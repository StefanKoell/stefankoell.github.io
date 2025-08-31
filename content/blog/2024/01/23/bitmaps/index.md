---
title: Bitmaps
comments: true
date: 2024-01-23
tags: 
    - Avalonia
    - Images
    - Bitmaps
    - WinForms
---

Are you using Bitmaps in your application? Make sure you keep an eye on your memory.

<!--more-->

## History Lesson

I used (and still use) WinForms. Displaying images in a WinForms application is quite common. You can set images on menu items, toolbar items, buttons, etc. Back in the good old days, handling images wasn't a big deal most of the time. You put images into a resource file, get a nice wrapper to access the image and all is good. Even though the `System.Drawing.Image` class had a `Dispose()` method, it was mostly not so important to care about the lifetime of an image. When you start the app, images are loaded through the resources, and shown/displayed while executing the application. Disposing of the image mostly caused unwanted side effects if you wanted to re-use images.

Times are changing. Nowadays, apps need to support a light and dark mode. High DPI support is also often required or desired. So, if you do dynamic image generation for different color variations or image sizes, it can get messy. You have to be careful when and how you create an image and when you dispose images not in used anymore to avoid memory leaks.

In WinForms, images created from a stream, must also keep the stream open for the lifetime of the image. I created my own `ManagedImage` class which can keep a reference to the underlying `Stream` as well as the `Image`. If I have to *re-generate* images because of a DPI or color theme changes, I can clean up and re-create all the images I need.

*Modern* WinForms app may not use bitmaps at all. There are ways to completely avoid the usage of bitmap based images with proper SVG support, for example. Depending on the framework and implementation, SVG images, may not need extra care about lifecycle management, disposal, etc. Check out [DevExpress' WinForms](https://docs.devexpress.com/WindowsForms/117631/Common-Features/Graphics-Performance-and-High-DPI/How-To-Draw-and-Use-SVG-Images) components which support SVG in almost all places. 

## Consequences

As a consequence, not cleaning up correctly can lead to nasty memory leaks. When in doubt, do some profiling to find those leaks. Also worth mentioning, if the underlying stream is closed for whatever reason, your app will crash. See: [MSDN](https://learn.microsoft.com/en-us/dotnet/api/system.drawing.image.fromstream?view=dotnet-plat-ext-8.0#system-drawing-image-fromstream(system-io-stream))

## What about Avalonia?

In Avalonia (and other XAML based framework) you can actually do a lot without ever needing bitmap based images. The `Image` control allows you to set an image `Source` (which must implement the `IImage` interface). So, you have a couple of choices:

- Set the source to a `DrawingImage`. In this case you usually define the XAML for the DrawingImage and put it into a resource dictionary. Then you pull in the resource using a dynamic resource binding using the resource key. 
- Set the source to an `SvgImage`. There's a great [nuget package](https://www.nuget.org/packages/Avalonia.Svg/) / [Github repo](https://github.com/wieslawsoltes/Svg.Skia) created and maintained by 
Wiesław Šoltés which allows you to use SVG images directly with the `<Image ... />` control. A [markup extension](https://github.com/wieslawsoltes/Svg.Skia?tab=readme-ov-file#image-control) allows you to load SVG images directly from your Avalonia resources/
- Set the source to a bitmap based image.

In the first two cases you don't really have to care about memory usage because it is all managed internally. Neither a DrawingImage nor an SVG image even has a `Dispose()` method on it.

If you are using bitmap based images, you have to think about where the image comes from and how it is used in your application during it's lifetime. **The `Image` control does nothing with the image source when the control is removed from the visual tree.** 

> [!WARNING]
> If you source needs disposal, make sure you clean up after the image control or image is not used anymore. In my profiling tests, I also found out that setting the `Source` property explicitly to `null` is necessary. It looked like the garbage collector didn't really clear out the images when this is not done.

## Image Cache

In all my apps, I usually create an ImageCache class or some sort of image manager. Having a central place where you can fetch images from, provide caching and also have some hooks to invalidate the cache (including cleaning up old images not in use anymore) is very convenient and served me well.