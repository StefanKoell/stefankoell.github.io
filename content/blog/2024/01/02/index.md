---
title: Avalonia's FontManager
comments: true
date: 2024-01-02
tags: 
    - Avalonia
---

This post is about using Avalonia's FontManager class.

<!--more-->

## Using Custom Fonts
There are a couple of samples demonstrating how to ship an app with a custom, embedded font (like a Google Font): [Avalonia Blog Post](https://avaloniaui.net/Blog/elevate-your-application-s-ui-with-custom-fonts) and [Avalonia Documentation](https://docs.avaloniaui.net/docs/guides/styles-and-resources/how-to-use-fonts).

I wasn't able to find a sample showing how to create a simple font picker with the fonts installed on the system.

## Creating a Font Picker
This is quite simple as it is using a standard ComboBox control which lists all the installed system fonts.

### The View Model

```c# {linenos=table}
internal partial class FontPickerViewModel : ObservableObject
{
    [ObservableProperty] 
    private ObservableCollection<FontFamily> _items = new();

    public FontPickerViewModel()
    {
        foreach (var font in FontManager.Current.SystemFonts.OrderBy(f => f.Name))
        {
            Items.Add(font);
        }
    }
}
```

{{< callout type="info" >}}
  I'm using the [MVVM Community Toolkit](https://github.com/CommunityToolkit/dotnet?tab=readme-ov-file) to create a class and properties implementing INotifyPropertyChanged.
{{< /callout >}}

The `Items` collection is of type `Avalonia.Media.FontFamily` and the `FontManager` class from the same namespace provides convenient access to all the installed system fonts.

### The View

The view is also quite simple. The item template of the ComboBox simply contains a TextBlock control which displays the name in the corresponding font. 

```xml {linenos=table}
    <ComboBox ItemsSource="{Binding Items}" >
        <ComboBox.ItemTemplate>
            <DataTemplate x:DataType="FontFamily">
                <TextBlock 
                    Text="{Binding Name}"
                    FontFamily="{Binding .}"
                    TextTrimming="CharacterEllipsis" 
                    />
            </DataTemplate>
        </ComboBox.ItemTemplate>
    </ComboBox>
```

If you only want to show the font names *without rendering them in the corresponding font*, simply remove line 6. 

I tested this on Windows and Linux using Avalonia 11.0.6.