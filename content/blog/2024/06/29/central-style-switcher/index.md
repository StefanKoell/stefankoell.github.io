---
title: Central Style Switcher
comments: true
date: 2024-06-29
tags: 
    - Avalonia
    - Attached Properties
    - Dynamic Resources
    - Styles
---

I needed a way to enable or disable animations by applying and overriding animation-related styles of various controls during runtime. This is what I came up with.

<!--more-->

## Introduction

There are a couple of ways to implement that but most of them involved a lot of changes in existing XAML. If you use *style classes*, for example, you need to go into all your XAML files and *decorate* all the controls with these classes and write styles for them. In larger applications with lots of XAML this can be quite tedious, error prone and you always have to think about setting those classes when you write new XAML.

In this example, I simply want to have one **central boolean** in a configuration class, let's call it `AnimationsEnabled` which controls whether or not animations should be shown. I then would create a number of styles for various controls which overrides certain properties to **disable animations** when that **boolean is false**.

Turns out, it's not that straight forward but here's a way to do it without touching a lot of XAML:
1. Create a custom, attached boolean property on every control.
1. Control the state of that attached property centrally using a dynamic resource.
1. Create a style which selects the control with that attached property for a specific condition.


## Attached Property

Using an attached property allows you to extend existing objects with additional properties. Think of it like extension methods but for properties. In my use case I need a boolean on all controls which reflects the current user preference, like are animations enabled or not. 

Creating attached properties is really easy. Check out the official Avalonia docs for [Attached Properties](https://docs.avaloniaui.net/docs/guides/custom-controls/how-to-create-attached-properties). 

Here's the attached property called `AnimationsEnabled`:
```csharp {linenos=table}
public class UIGlobal : AvaloniaObject
{
    public static readonly AttachedProperty<bool> AnimationsEnabledProperty = AvaloniaProperty.RegisterAttached<UIGlobal, Control, bool>("AnimationsEnabled", true, true);
    public static void SetAnimationsEnabled(Control obj, bool value) => obj.SetValue(AnimationsEnabledProperty, value);
    public static bool GetAnimationsEnabled(Control obj) => obj.GetValue(AnimationsEnabledProperty);
}
```

Looks really simple and easy but here are some points to be aware of:
- The class which hosts the `AttachedProperty` must inherit `AvaloniaObject`
- The generic `RegisterAttached<UIGlobal, Control, bool>` method takes three types:
    - UIGlobal is the **TOwner** - usually the class you created the attached property in.
    - Control is the **THost** which tells the attached property what types should have this attached property. Note that I used a very broad type `Control` here. So every control in your app will have this attached property available. You can narrow it down, if you only want to have the `Button` class as the host of the property. You can also check out the *Dev Tool* to see what kind of attached properties are already used for certain controls.
    - The last type is the **TValue** which is a boolean because style selectors' property match syntax can only handle boolean values AFAIK.
- Also worth mentioning is the last boolean argument of the `RegisterAttached` method named `inherits` which is set to `true`. In this case, the attached property will then by default just inherit its value from the container element in the hierarchy of controls - unless specified otherwise by setting a local value. So to change the value, you just need a style selector for the `Window` to set all the properties of all the containing controls.


## Dynamic Resource

In order to make the whole thing work, we also need a way to put the state of the user preference in the resource dictionary. Most of the time you declare dynamic resources in XAML but it's also [quite easy](https://docs.avaloniaui.net/docs/guides/styles-and-resources/resources#consuming-resources-from-code) to access the resource dictionary by code. So all you need to do, whenever the user changes the preferences and enables or disables animations in some setting screen, is something like this:

```csharp
App.Current.Resources["AnimationsEnabled"] = _userPreferences.Animations;
```

The above example just shows how to update the resource. In this example, the `_userPreferences.Animations` is just a sample and this, of course, depends on how you store your user's preferences and update the UI once the user changes the preferences. You may also think about the name/key of the resource (AnimationsEnabled) and maybe prefix it or just ensure that it doesn't interfere with an existing resource name from a theme you use.


## Style Selectors

[Style Selector Syntax](https://docs.avaloniaui.net/docs/reference/styles/style-selector-syntax) in Avalonia is really powerful and offers great flexibility. For my particular use case, I'm interested in the following topics:
- [:is(SomeControl) Syntax](https://docs.avaloniaui.net/docs/reference/styles/style-selector-syntax#include-derived-classes) allows you to target all controls including inherited controls from that specified class.
- [Property Match Syntax](https://docs.avaloniaui.net/docs/reference/styles/style-selector-syntax#include-derived-classes) allows you to do boolean conditions on properties of the **object/control** your selector targets. 


### Set Attached Property Value

Since we now have the `AnimationsEnabled` attached property and the dynamic resource holding the boolean value for that, we can now create a style which applies the value to the attached property on every control:

```xml
<Style Selector=":is(Window)">
    <Setter Property="(common:UIGlobal.AnimationsEnabled)" Value="{DynamicResource AnimationsEnabled}" />
</Style>
```

- Note that the selector is using the `:is(Window)` expression. Since we registered the attached property with `inherits: true`, we don't need to select `Control` here. It will get the correct value through inheritance.
- Also note that, as described in the docs, the property match syntax needs the parentheses to indicate it's an attached property we want to set.

> [!WARNING]
> If you want to test the styles, do not rely too much on the Dev Tool. I noticed that the value of the attached property changes one time for the observed control but then it doesn't. This seems to be an issue with the Dev Tool itself - not with the style. So to do proper testing, use some real styles you apply to see if the change of the setting through the dynamic resource is applied.

### Use Attached Property Value

Here's an example for a grid splitter style which uses a `BrushTransition` but only when `AnimationsEnabled` is set to **true**.

```xml {linenos=table}
<Style Selector="GridSplitter">
    <Style Selector="^[(common|UIGlobal.AnimationsEnabled)=True]">
        <Setter Property="Transitions">
            <Transitions>
                <BrushTransition Property="Background" Duration="0:0:0.5" />
            </Transitions>
        </Setter>
    </Style>

    <Setter Property="Background" Value="Transparent" />
    <Style Selector="^:pointerover">
        <Setter Property="Background" Value="{DynamicResource UIBrushHigh3}" />
    </Style>
</Style>
```

The nested style applying the brush transition starting at `line 2` will only be applied if the AnimationsEnabled attached property is true.

## Conclusion

I think this is a good way to handle styles depending on some centrally stored configuration. Using this I can use a global setting to apply conditional styles on **any** control. Implementing something like a **High Contrast** option should be easy with a technique like this. The main advantage for me using the above is, I don't have to touch existing XAML and have a "central style sheet" for these conditions. I'm sure there are other ways to something similar, maybe [Behaviors](https://github.com/AvaloniaUI/Avalonia.Xaml.Behaviors), or [Mixins](https://github.com/AvaloniaUI/Avalonia/blob/d4d322654e025ac5c10fc88c34a0a400a407d179/src/Avalonia.Controls/Mixins/PressedMixin.cs#L28). If you have solved this differently, let me know. I'm always interested to see how people are solving problems like this.

{{< callout type="error" emoji="♥️" >}}
Thanks to the great Avalonia community on GitHub and Telegram. Those folks are a tremendous help and inspiration and always patient as well!
{{< /callout >}}



