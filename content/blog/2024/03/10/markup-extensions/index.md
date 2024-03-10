---
title: Markup Extensions
comments: true
date: 2024-03-10
tags: 
    - Avalonia
---

Give your XAML a massive boost with markup extension.

<!--more-->

## What are Markup Extensions

XAML Frameworks, including Avalonia, ship with a couple of [built-in markup extensions](https://docs.avaloniaui.net/docs/reference/mark-up-extensions). It's part of the tooling and provides a convenient way to refer to other objects and values in XAML. `{Binding}`, `{DynamicResource}` and `{StaticResource}` are probably the most commonly used markup extensions you use every day, without really knowing what's going on behind the scenes.

Best thing about markup extensions is, it can be extended and you can build your own. Second best thing is, it's quite easy.

## Why Markup Extensions

You may wonder, when and why do you want to do your own markup extensions. There are actually quite a few useful scenarios. You can create extensions where you pass in a string and that string get's localized, for example. You could do calculations for sizes, like [Actipro](https://www.actiprosoftware.com/docs/controls/avalonia/themes/user-interface-density) is offering for different density settings. Markup extensions can be very powerful and can also help you with streamlining your XAML code.

## Creating your own Markup Extensions

Here's an example for a markup extension I needed recently and thought it would be a good way to demonstrate how it works.

### Using SymbolIcon

I came across the [FluentIcons](https://github.com/davidxuang/FluentIcons) project which allows you to use the vast library of fluent icons provided by [Microsoft](https://github.com/microsoft/fluentui-system-icons) using a simple XAML syntax. Once you added the nuget package, you can write XAML to draw one of the icons in your layout:

```xml
<Window xmlns:ic="using:FluentIcons.Avalonia">
    <ic:SymbolIcon Symbol="ArrowLeft" IsFilled="True" />
</Window>
```

`SymbolIcon` (the class from the FluentIcons.Avalonia library) derives from `IconElement` (from Avalonia.Controls namespace) - which basically is a control.

I recently ported the [SettingsCard](https://learn.microsoft.com/en-us/dotnet/communitytoolkit/windows/settingscontrols/settingscard) control from the WinUI controls to Avalonia and wanted to stay as close to the original API as possible. The SettingsCard has properties like Description, Header but also a property called `HeaderIcon` with the type `IconElement`. This means I can simply provide a `SymbolIcon` from FluentIcons to the `HeaderIcon` property, like this:

```xml
<settings:SettingsCard Header="Some header text">
  <settings:SettingsCard.HeaderIcon>
    <fluentIcons:SymbolIcon Symbol="Bug" />
  </settings:SettingsCard.HeaderIcon>
  <TextBox />
</settings:SettingsCard>
```

This works great but it's kind of cumbersome with the nesting of the HeaderIcon element. You could provide a string through your ViewModel and have a converter provide the SymbolIcon class but that means you have to provide all the icon names in the view model. If you want to set the IsFilled property, Foreground color or size, it gets even more complicated.

Let's solve that using a custom markup extension.

### Writing a Markup Extension

```csharp
using System.Diagnostics.CodeAnalysis;
using FluentIcons.Avalonia;
using FluentIcons.Common;

namespace SomeNamespace.Extensions;

public sealed class SymbolIconExtension
{
    public Symbol Symbol { get; init; }
    public bool IsFilled { get; set; }

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
        };
}
```

A couple of things to mention:
* A custom markup extension **must** have a `ProvideValue(IServiceProvider serviceProvider)` method.
* This method needs to return the appropriate type you expect the markup extension to return.
* Your class can have multiple constructors. An empty constructor allows you to specify the properties by name in the markup. Make sure your property setter (if not nullable) is `init`.
* Since the `Symbol` constructor argument/property is an enum, you even have intellisense in the XAML.

### Using the Markup Extensions

To XAML is now a bit simpler with the markup extension:

```xml
<settings:SettingsCard 
  Header="Some header text" 
  HeaderIcon="{controlExtensions:SymbolIcon Symbol=Bug, IsFilled=False}">
  <TextBox />
</settings:SettingsCard>
```

There is a bit of magic going on in the background. 

The name of the extension to use in XAML depends on your class name which implements the mandatory **ProvideValue** method. 
* If the class name ends with "...Extension", like `SomeCLassNameExtension`, you need to use `{controlExtensions:SomeCLassName ...}` and omit the Extension suffix. 
* If the class name does not end with "...Extension", like `SomeClassNameExt`, you need to use the full name, like `{controlEtensions:SomeClassNameExt ...}`.

## Caveats

If your markup extension accepts a string and your string contains certain characters, make sure you do proper escaping. Like this: 
```xml
<SomeControl Text="{extension:TextExtension Text=Here\'s some text containing a \'\,\' character}" />
```