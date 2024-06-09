---
title: Markup Extensions and Styles
comments: true
date: 2024-06-09
tags: 
    - Avalonia
    - Styles
---

Caveats when using Markup Extensions and Styles.


<!--more-->

## Introduction

I already blogged about markup extensions [here](/blog/2024/03/10/markup-extensions/) and [here](/blog/2024/05/01/lifecycle/). In this post I want to show potential pitfalls and scenarios where applying styles may not work when using markup extensions.

## Name Collision

If you create a [Markup Extension](https://docs.avaloniaui.net/docs/concepts/markupextensions#creating-markupextensions) for one of your own classes, be careful with naming and namespace you use. In my previous blog posts I showed how to create a markup extension for something like the SymbolIcon. This worked great for me. A couple of months later, the author of the SymbolIcon extended his library and provided a markup extension out-of-the-box. I thought that was great and kicked out my own to replace it with the built-in one. So far, so good. Everything worked fine until I wanted to do some custom styling using a style selector. I received weird error messages that the property I wanted to style was not available.

You may get an error like this:
```
Error AVLN2000 Avalonia: Unable to resolve suitable regular or attached property ... on type FluentIcons.Avalonia:FluentIcons.Avalonia.SymbolIconExtension Line 16, position 17.
```

... or this:
```
Error AVLN3000 Avalonia: FontSize is not an AvaloniaProperty
```

What the docs don't tell you (but I mentioned it in my previous markup extension blog post):
There's a hidden naming convention: **If you name your markup extension `SymbolIconExtension`, you can actually use the `{SymbolIcon ...}` without the `Extension` suffix.**

So, in the end, the issue was:
- The class `SymbolIcon` and the class `SymbolIconExtension` was in the same namespace
- The XAML compiler got confused because the style selector `<Style Selector="fluentIcon|SymbolIcon">` was actually trying to set property values on the `SymbolIconExtension` instead of the resulting `SymbolIcon`.

{{< callout type="info" emoji="💡" >}}
Either move the `SymbolIconExtension` to a different namespace or use a different name for the extension to avoid these name conflicts.
{{< /callout >}}



## Local Values

When using **StyledProperties**, the static registration allows you to set a default value. When you use a markup to create the control and explicitly set properties in the markdown extension, they are actually set **locally** and **local** values always precede applied values from styles (unless [defined otherwise](https://docs.avaloniaui.net/docs/guides/styles-and-resources/setter-precedence)).

Let me try to illustrate this using my own markup extension I wrote:

```csharp {linenos=table}
using System.Diagnostics.CodeAnalysis;
using FluentIcons.Avalonia;
using FluentIcons.Common;

namespace SomeNamespace.Extensions;

public sealed class SymbolIconExtension
{
    public Symbol Symbol { get; init; }
    public bool IsFilled { get; set; }
    public double FontSize {get; set; } = 20D;

    public SymbolIconExtension() { }
    public SymbolIconExtension(Symbol symbol) : this(symbol, false)
    {
        Symbol = symbol;
    }
    public SymbolIconExtension(Symbol symbol, bool isFilled)
    {
        Symbol = symbol;
        IsFilled = isFilled;
    }
    
    [SuppressMessage("Usage", "CA1801:Review unused parameters", Justification = "Markup extension contract")]
    public SymbolIcon ProvideValue(IServiceProvider serviceProvider) =>
        new()
        {
            Symbol = Symbol,
            IsFilled = IsFilled,
            FontSize = FontSize
        };
}
```

Look at the line 61 and 80. The markup extension has a default value of 20 for the FontSize and in line 80 we always set that value to the FontSize property of the SymbolIcon. In this case it is a local value (similar to when you set that value in the XAML directly). Writing a style like this:

```xml
<Style Selector="fluentIcon|SymbolIcon">
    <Setter Property="FontSize" Value="32" />
</Style>
```

will not be effective.

To allow styles to be applied, consider the following:

```csharp {linenos=table}
using System.Diagnostics.CodeAnalysis;
using FluentIcons.Avalonia;
using FluentIcons.Common;

namespace SomeNamespace.Extensions;

public sealed class SymbolIconExtension
{
    public Symbol Symbol { get; init; }
    public bool? IsFilled { get; set; }
    public double? FontSize {get; set; }

    public SymbolIconExtension() { }
    public SymbolIconExtension(Symbol symbol) : this(symbol, false)
    {
        Symbol = symbol;
    }
    public SymbolIconExtension(Symbol symbol, bool isFilled)
    {
        Symbol = symbol;
        IsFilled = isFilled;
    }
    
    [SuppressMessage("Usage", "CA1801:Review unused parameters", Justification = "Markup extension contract")]
    public SymbolIcon ProvideValue(IServiceProvider serviceProvider)
    {
        var symbolIcon = newSymbolIcon()
        {
            Symbol = Symbol
        };
        
        if (IsFilled is not null)
            symbolIcon.IsFilled = IsFilled.Value;

        if (FontSize is not null)
            symbolIcon.FontSize = FontSize.Value;

        return symbolIcon;
    }
}
```

So, if not specified in the markup extension, these values will be set to the default values defined in the static styled property definition and allows you to set custom styles. That's why it's also a bad idea to set default values in the constructor because it ends up being a local value. **If you derive from a control and need a different default value for an existing styled property, use the new keyword and define your own styled property on the derived class.**