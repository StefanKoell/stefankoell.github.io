---
title: More Mica
comments: true
date: 2025-09-06
tags: 
    - Avalonia
    - Styles
    - XAML
    - Mica
---


Updated sample app to support custom accent colors and system accent colors using the free [Actipro](https://www.actiprosoftware.com/products/controls/avalonia) theme for Avalonia.

<!--more-->

## Introduction

This is a continuation of my previous blog post about using [Mica](/blog/2024/10/27/mica) with [Avalonia](https://avaloniaui.net/). This time, I extended my sample app to include the Actipro theme. Actipro offers a completely free, polished, high quality, Avalonia theme which provides a lot of functionality out of the box - like different UI densities or color themes. Actipro also offers paid components (Pro Products) like Ribbon, Docking, User Prompts, and many more. Full disclosure: I'm a paying customer. This blog post has not been sponsored in any way by Actipro. I also want to point out that the provided sample does not require any of the paid components.

Here's a short video of the sample app in action:
{{< rawhtml >}} 

<video width=450px controls autoplay>
    <source src="MoreMica.mp4" type="video/mp4">
    Your browser does not support the video tag.  
</video>

{{< /rawhtml >}}

The updated sample application can be found on my [github repository](https://github.com/StefanKoell/Misc/tree/main/src/AvaloniaMica)

## Key Elements of the Sample Code

Most of the code should be self-explanatory but I wanted to highlight a couple of things to better understand how it works.

Actipro's theme (`ModernTheme`) is quite feature rich and a lot of things in their theme are generated. There are also a lot of APIs and hooks you can use to tweak the theme generation. Version 25.2.1 allows you to create your own `ColorPaletteFactory` and handle the creation of `ColorRamps` by yourself. The documentation for the Actipro components is also excellent and explains how their theme generator and color ramps work.

### Custom ColorPaletteFactory

I created my own factory class (`ColorPaletteFactory.cs`) which is derived from the `DefaultColorPaletteFactory`. I created my own enum with a value `System` and another value `Custom` in addition to all color ramps, supported by Actipro.

### Using Custom Factory

In the `App.axaml` simply make sure you put your own `ColorPaletteFactory` in your `ThemeDefinition` as shown in line 9:

```xml {linenos=table, hl_lines=[9]}
<actipro:ModernTheme Includes="NativeColorPicker">
    <actipro:ModernTheme.Definition>
        <generation:ThemeDefinition
            UseAccentedSwitches="True" 
            ToggleSwitchAppearanceKind="Solid" 
            ToggleSwitchHasFarAffinity="True" 
            AccentColorRampName="Red">
            <generation:ThemeDefinition.ColorPaletteFactory>
                <avaloniaMica:ColorPaletteFactory />
            </generation:ThemeDefinition.ColorPaletteFactory>
        </generation:ThemeDefinition>
    </actipro:ModernTheme.Definition>
</actipro:ModernTheme>
```

### Overriding the Create method

In addition to the out-of-the-box supported color ramps, there are two "modes" which should provide a custom color ramp. If one of the two modes (`System` or `Custom`) is set in the `AccentColor` property, we need to replace one of the existing ramps with a new ramp containing the new and custom color shades. In my example, I decided to replace the `Red` color ramp with the custom color ramp:

```csharp {linenos=table, hl_lines="14-15"}
public override ColorPalette Create()
{
    var palette = base.Create();
    var color = AccentColor switch
    {
        AccentColor.System => GetSystemAccentColor(),
        AccentColor.Custom => CustomAccentColor,
        _ => null,
    };

    if (color is null) 
        return palette;
    
    palette.Ramps.Remove(Hue.Red);
    palette.Ramps.Add(CreateColorRamp(Hue.Red, color.Value));

    return palette;
}
```

I'm simply calling the `CreateColorRamp` method with the color I get from the system (or custom color) to create the ramp. This is a method from the base class. I'm not sure if this is the correct approach but from what I can see in my tests, this seems to work pretty well. I was told that the shade 50 should always be white (`#ffffff`).

### Setting the Accent Color

The method (`SetAccentColor`) is called whenever you change one of the accent color defining properties. The code below shows that a dynamic resource `UIWindowBorderColorActive` is updated with the proper color value (line 98) and the `AccentColorRampName` is also set. The `GetColorRampName` method ensures that the name `Red` is returned for the custom modes and the `App.Theme.RefreshResources();` call (line 11) ensures that the `Create()` method of the `DefaultColorFactory` is called.

```csharp {linenos=table}
private void SetAccentColor(AccentColor accentColor)
{
    if (App.Theme?.Definition is null)
        return;
    
    Application.Current?.Resources["UIWindowBorderColorActive"] = BorderAccentColorEnabled 
        ? GetColor(accentColor)
        : Colors.Transparent;

    App.Theme.Definition.AccentColorRampName = GetColorRampName(accentColor);
    App.Theme.RefreshResources();
}
```

### Monitor System's Accent Color Changes

The factory class also has a method which sets up and event handler when the OS color values have changed:

```csharp {linenos=table}
public void StartSystemAccentColorWatcher()
{
    App.MainWindow?.PlatformSettings?.ColorValuesChanged += (_, _) =>
    {
        if (AccentColor != AccentColor.System)
            return;
        SetAccentColor(AccentColor);
    };
}
```

## Working with the new Factory

### Convenience Stuff

The following properties were added to my application class:

```csharp {linenos=table}
internal partial class App : Application
{
    public static TopLevel? MainWindow { get; private set; }
    public static ModernTheme? Theme => ModernTheme.TryGetCurrent(out var theme) ? theme : null;
    public static ColorPaletteFactory? ColorPaletteFactory => Theme?.Definition?.ColorPaletteFactory as ColorPaletteFactory;
...
```

With this in place, I can easily access the new `ColorPaletteFactory` from my view models.

### Changing the Accent Color

The `MainWindowViewModel.cs` file demonstrates how to set the properties on the `ColorPaletteFactory` to change the accent color accordingly.

Setting the `AccentColor` property on the factory:

```csharp {linenos=table}
App.ColorPaletteFactory.AccentColor = AccentColor.System;
App.ColorPaletteFactory.AccentColor = AccentColor.Custom;
App.ColorPaletteFactory.AccentColor = AccentColor.Red;
App.ColorPaletteFactory.AccentColor = AccentColor.Orange;
...
```

Setting the `CustomAccentColor` to an arbitrary color will change the `AccentColor` property to `AccentColor.Custom` and also sets the custom color accordingly:

```csharp
App.ColorPaletteFactory.CustomAccentColor = Colors.Coral;
```

That's it! The rest of the view model and factory are mostly helper and convenience methods.