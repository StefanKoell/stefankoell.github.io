---
title: Lifecycle
comments: true
date: 2024-05-01
tags: 
    - Avalonia
---

Lifecycle and Weak References

<!--more-->

## Introduction

In March, I blogged about [markup extensions](blog/2024/03/10/markup-extensions/) to show how easy it is to make XAML authoring easier and more convenient. I created a couple of extensions and my last adventure turned out to be more challanging. I tried to create a markup extension to help with localization. In general, I wanted to have something like this in the XAML code:

```xml
<TextBlock Text="{ext:UIString Text=Some localizable string}" />
```

Creating a markup extension which returns the correct string from a localization service is quite easy and implemented with a few lines of code. 

**However, it's a bit more challanging when you want to automatically translate and update the string during runtime when the language has changed.**

## Markup Extension Lifecycle

When the XAML is processed, the markup extension class is instantiated for each occurrance and the `ProvideValue` method is invoked. When you just return the translated text, you will not be able to change that text when the language has changed. This is because right after processing the markup extension - shortly after `ProvideValue` was executed, the extension will be 'out of scope' and collected by the garbage collected. You can easily test this by adding a [finalizer](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/finalizers) to your markup extension and put a breakpoint in it.

Markup extensions are short-lived and not really kept around to update values during runtime. This is a good thing because you want to avoid leaks and it's actually quite hard to figure out when to clean up things - especially in those markup extensions since you don't have much to work with. So, what can we do?

## Translator Class with Binding

Instead of returning the translated text, let's build somethig which implements `INotifyPropertyChanged` and return a binding to the translated string. This way, we can use a [WeakReferenceMessenger](https://learn.microsoft.com/en-us/dotnet/communitytoolkit/mvvm/messenger) to notify our translator class that the language has changed. The translator class can update the value and through the binding the UI should be updated at runtime. It's important to use the **WeakReferenceMessenger** to avoid memory leaks.

Here's a basic implementation of the translator class:

```csharp {linenos=table}
using AvaMarkupExtensionTranslate.Messages;
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Messaging;

namespace AvaMarkupExtensionTranslate.Utilities;

public partial class Localize : ObservableObject
{
    private readonly string _text;
    [ObservableProperty] private string? _value;

    public Localize(string text)
    {
        _text = text;
        WeakReferenceMessenger.Default.Register<LanguageChangedMessage>(this, OnLanguageChanged);
        OnLanguageChanged(this, null);
    }

    private static void OnLanguageChanged(object recipient, LanguageChangedMessage? message)
    {
        var self = (Localize)recipient;
        self.Value = LocalizationService.Translate(self._text);
    }
}
```

{{< callout type="info" >}}
  Note, that I'm using `ObservableObject` and the `WeakReferenceManager` from the [MVVM Community Toolkit](https://learn.microsoft.com/en-us/dotnet/communitytoolkit/mvvm/)
{{< /callout >}}

In the markup extension we just create the instance of our translator class `Localize` and return a binding to the `Value` property:

```csharp {linenos=table}
using System;
using System.Collections.Generic;
using System.Diagnostics.CodeAnalysis;
using System.Runtime.CompilerServices;
using Avalonia;
using Avalonia.Controls;
using Avalonia.Data;
using AvaMarkupExtensionTranslate.Utilities;

namespace AvaMarkupExtensionTranslate.MarkupExtensions;

public class UIStringExtension
{
    private static readonly ConditionalWeakTable<Control, Dictionary<AvaloniaProperty, Localize>> References = new ();
    
    public string Text { get; init; } = string.Empty;
    
    [SuppressMessage("Usage", "CA1801:Review unused parameters", Justification = "Markup extension contract")]
    public object? ProvideValue(IServiceProvider serviceProvider)
    {
        var localize = new Localize(Text);
        var binding = new Binding
        {
            Source = localize,
            Path = nameof(Localize.Value)
        };
        
        return binding;
    }
}
```

So far so good, right? Except, there's a tiny problem: if you run the code as shown, you will notice that after switching the language a couple of times, it will just stop working.

## Lifecycle Issue #2

The problem is, that the instance of the `Localize` class which listens to the message when the language changes (also by using weak references in the messenger), will be collected by the garbage collector. You can easily test this again by putting a finalizer in the class and see that it is invoked soon after it has been instantiated.

So all these instances are short lived as they are never directly referenced by any other instance and therefore marked for collection. On one side, we don't want to (and also can't really) refer to those instances from anywhere as they are created by the markup extension and you actually want to have short lived instances - or to put it more precisely **keeping them alive as long as they are needed.** On the other side, we somehow need to tie the lifetime of those instances to the lifetime of the controls which actually need them for the translation.

## Solution

After racking my brains for quite some time, a friend of mine actually found an elegant solution: [ConditionalWeakTable](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.conditionalweaktable-2?view=net-8.0).

```csharp {linenos=table}
using System;
using System.Collections.Generic;
using System.Diagnostics.CodeAnalysis;
using System.Runtime.CompilerServices;
using Avalonia;
using Avalonia.Controls;
using Avalonia.Data;
using Avalonia.Markup.Xaml;
using AvaMarkupExtensionTranslate.Utilities;
using Microsoft.Extensions.DependencyInjection;

namespace AvaMarkupExtensionTranslate.MarkupExtensions;

public class UIStringExtension
{
    private static readonly ConditionalWeakTable<Control, Dictionary<AvaloniaProperty, Localize>> References = new ();
    
    public string Text { get; init; } = string.Empty;
    
    [SuppressMessage("Usage", "CA1801:Review unused parameters", Justification = "Markup extension contract")]
    public object? ProvideValue(IServiceProvider serviceProvider)
    {
        var provideTarget = serviceProvider.GetService<IProvideValueTarget>();
        if (provideTarget?.TargetObject is not Control control || 
            provideTarget.TargetProperty is not AvaloniaProperty avaloniaProperty)
            return null;
        
        if (!References.TryGetValue(control, out var props))
        {
            props = new Dictionary<AvaloniaProperty, Localize>();
            References.Add(control, props);
        }
        
        if (!props.TryGetValue(avaloniaProperty, out var localize))
        {
            localize = new Localize(Text);
            props.Add(avaloniaProperty, localize);
        }

        var binding = new Binding
        {
            Source = localize,
            Path = nameof(Localize.Value)
        };
        
        return binding;
    }
}
```
Luckily, the `ProvideValue` method passes on an instance of the `IServiceProvider` - which can be used to determine the `TargetObject` (the actual control the markup extension is used on) and even the `TargetProperty`. Both can and must be used to keep the corresponding `Localize` instance alive and in memory as long as the control is alive. In line 16, we keep an instance (static) of the `ConditionalWeakTable`.

Lines 23 through 38 is used to determine control and property as well as dealing with storing the `Localize` instance in the `ConditionalWeakTable`.

If the control is out of scope, meaning that Avalonia removed it and it isn't used anywhere else, the `Localize` instances will also be removed eventually.

A couple of profiling sessions showed that this works pretty well. No leaks detected so far. You can check out a full running sample on my [github repo](https://github.com/StefanKoell/Misc/tree/main/src/AvaMarkupExtensionTranslate).

If there are other ways to do this, let me know in the comments.