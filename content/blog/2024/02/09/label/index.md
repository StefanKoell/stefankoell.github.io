---
title: Taming the Label Control
comments: true
date: 2024-02-09
tags: 
    - Avalonia
---

There's a Label control in Avalonia but it's weird.

<!--more-->

## About the Label control

If you care about accessibility in your app, you should take a look at the [Label](https://docs.avaloniaui.net/docs/reference/controls/detailed-reference/label) control. There's not much to read in the docs (as of this writing). The two sentences in the docs give you a clue when to use it though.

### When to use a Label control?

This is quite simple: if you care about accessibility and keyboard support, make use of the Label control.

### Features

The Label control inherits from `ContentControl`, so the only property worth mentioning is the `Target` property which accepts an `IInputElement`` (e.g. a ComboBox or TextBox). 

What the docs do not mention is that the Content can (or should) be a `string` for the label text which can also contain an underscore `_` character in front of the character you want to mark as **accelerator** or **access** key. The character will be highlighted (underlined) when you press the `ALT` key on your keyboard.

Here's a simple example:
```xml
<StackPanel Orientation="Vertical" VerticalAlignment="Center">
    <StackPanel Orientation="Horizontal" HorizontalAlignment="Center" Margin="10">
        <Label Content="Click _Me:" Target="TextBox1" VerticalAlignment="Center" />
        <TextBox x:Name="TextBox1" MinWidth="100" VerticalAlignment="Center" />
    </StackPanel>
    <StackPanel Orientation="Horizontal" HorizontalAlignment="Center" Margin="10">
        <Label Content="Or M_e:" Target="TextBox2" VerticalAlignment="Center" />
        <TextBox x:Name="TextBox2" MinWidth="100" VerticalAlignment="Center" />
    </StackPanel>
    <StackPanel Orientation="Horizontal" HorizontalAlignment="Center" Margin="10">
        <Label Content="Or _Me Again:" Target="TextBox3" VerticalAlignment="Center" />
        <TextBox x:Name="TextBox3" MinWidth="100" VerticalAlignment="Center" />
    </StackPanel>
</StackPanel>
```

In the XAML you simply provide a name (x:Name) for your input element (TextBox) and use this name in the `Target` attribute of the label control. Also notice that I used the underscore character in front of the desired access key.

When you run this application, you get something like this:
{{< cards >}}
  {{< card link="label-01.png" image="label-01.png" subtitle="Screenshot" method="Resize" options="600x q80 webp" >}}
{{< /cards >}}

A couple of notes:
* The screenshot shows the form after the `ALT` key has been pressed. The characters marked with an underscore prefix are underlined. Note that the underlined characters are not shown initially.
* Clicking a label or pressing `ALT + Access Key` will transfer the focus to the control specified in the `Target` attribute of the label.

## Taming the Label control

So far so good, right? Not quite. As it turns out the label control is somewhat limited in terms of configuration and behavior. You might expect properties and flexibility similar to a `TextBlock` control. Sorry to disappoint. In my particular case, I needed to change the text wrapping behavior similar to what the TextBlock is offering: `TextWrapping="Wrap"`. Unfortunately, this is not possible - or at least not that easily.

### Failed attempts

A quick search pointed me to [that post](https://github.com/AvaloniaUI/Avalonia/issues/5143) which suggested to just put a TextBlock control into the Label's content:
```xml
<Label Target="TextBox4">
    <TextBlock Text="Click _Me:" TextWrapping="Wrap" />
</Label>
<TextBox x:Name="TextBox4" MinWidth="100" VerticalAlignment="Center" />
```

In general this is a valid approach since the Label is a ContentControl. The setup as shown above works - well, at least partially.

* You can click the TextBlock inside the Label and the focus is transferred to the TextBox4
* The TextWrapping works as expected
* The big problem though is, you loose the `Access Key` functionality completely

So, if the Label control would expose the same functionality and features as the TextBlock control, it would be much more useful.

Also, if the Label control would provide a way to **manually** set the access key, it would at least be possible to make this work somehow and process the label text manuelly. I haven't found a way to do that.

The benefit would be that you can really use a different control within the Label. One example would be to put Markdown in it or use Inline Runs to have partially bold text, different colors or so.

One can always hope and maybe the Label control will grow up some day and bring more features to the table...

### Solution

Anyway, hoping for features to be included in the future doesn't really solve my issue now. Don't get me wrong, the Avalonia repo is very active and a lot of fixes and enhancements are submitted and accepted every day. My impression is that accessibility is currently not the main focus of the maintainers. I understand that there are more pressing issues and other priorities right now but I do hope that this will get on their radar sooner or later.

### Avalonia DevTools is your friend!

Hit `F12` while debugging the app to open the Avalonia DevTools and locate the Label control in the **Visual Tree**:

{{< cards >}}
  {{< card link="label-02.png" image="label-02.png" subtitle="Avalonia DevTools" method="Resize" options="600x q80 webp" >}}
{{< /cards >}}

As you can see, the Label's ContentPresenter is putting the provided string (from the Label's `Content` attribute) into a control called `AccessText`. In the **Attached Properties** section you can see that there are TextBlock related properties available, including `TextWrapping`.

### Applying a style

Turns out that applying a style to the AccessText control can alter the TextWrapping behavior:

```xml
<Label Content="Click _Me:" Target="TextBox1" VerticalAlignment="Center">
  <Label.Styles>
    <Style Selector="AccessText">
      <Setter Property="(TextPresenter.TextWrapping)" Value="Wrap" />
    </Style>
  </Label.Styles>
</Label>
```

The above sample applies the style for text wrapping to a specific label instance. You can, of course, write styles which are applied to all AccessText instances or use classes to pick and choose. You should also be able to change other properties from the TextBlock which are not directly exposed in the Label control.

## Issues

I'm quite happy with the outcome and learned a lot. There is still one issue with the Label control or more precisly with handling **Access Keys**:

If you look at the app's screenshot again, you will notice that `TextBox1` and `TextBox3` has the same access key. In this situation, the access key will only focus the first control ('TextBox1`). 

**The desired behavior should be to cycle through all controls with the same access key. This has been the default behavior in Win32 apps (WinForms) as well as WPF IIRC.**

There's already an open [GitHub issue](https://github.com/AvaloniaUI/Avalonia/issues/13160) since Oct. 2023 with no fix. The issue not only applies to menu items but also controls with labels and access keys. As mentioned before, priorities seem to be elsewhere at the moment. Keeping fingers crossed this will be resolved soon.