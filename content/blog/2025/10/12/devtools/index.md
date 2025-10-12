---
title: DevTools - Two is better than one
comments: true
date: 2025-10-12
tags: 
    - Avalonia
    - Debugging
---


Avalonia Accelerate customers get a great, new DevTool but what if you want to use the old one as well? You can actually have both.

<!--more-->

## Introduction

[Avalonia](https://github.com/avaloniaui/avalonia) is an (MIT) open source UI framework which also includes a [DevTool](https://docs.avaloniaui.net/docs/guides/implementation-guides/developer-tools) to inspect UI elements, events, styles, etc.

The company which maintains the UI framework, also has a commercial offering called [Accelerate](https://avaloniaui.net/accelerate) which offers a brand new [DevTool](https://docs.avaloniaui.net/accelerate/tools/dev-tools/getting-started).

## Setup both DevTools

To setup both DevTools in your project, you can actually register the DevTools with different `Gesture` configurations.

### Register the Legacy Developer Tools

As mentioned in the docs, you can get the *Legacy Developer Tools* by adding the `Avalonia.Diagnostics` package and call `this.AttachDevTools();`. So, if you hit the `F12` key, the *Legacy DevTool* should open.

### Register the Accelerate Developer Tools

For the new DevTool, you need the `AvaloniaUI.DiagnosticsSupport` package. But for this, we call the `AttachDeveloperTools` method slightly differently:

```csharp {linenos=table}
#if DEBUG
        // dev tools package is only available in DEBUG configuration
        this.AttachDeveloperTools(options =>
        {
            options.Gesture = KeyGesture.Parse("F11");
        });
#endif
```

Hitting the `F11` key is now opening the new DevTool. Of course, you can use whatever key you want to invoke the new one.

> [!NOTE]
> Also as mentioned in the docs, make sure the package reference and the call to attach the DevTool has a condition for `DEBUG` builds only.

## Why?

You may be wondering why you still want to use the old DevTool? The answer is quite simple: some stuff in the new tool is not yet working well. Since it's a new tool, it still needs some time to iron out some kinks. If you are working with the Overlay layer or the Adorner layer, the new DevTool may not work well yet (at least at the time of this writing). Also freezing popups doesn't seem to work for me in the new one, so the old one still has some value.

I'm sure there will be a time when I can remove the old DevTool completely.