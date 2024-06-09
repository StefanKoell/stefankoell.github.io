---
title: Animations
comments: true
date: 2024-04-06
tags: 
    - Avalonia
    - Styles
    - Animations
---

First steps with Animations in Avalonia.

<!--more-->

## Sample

I created a very simple example below. A standard Avalonia application with only a couple of elements in the `MainWindow.axaml` file. For simplicity, I don't have any view models, only a binding directly to the checkbox to control whether or not to show the **Footer** element.

```xml {linenos=table}
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        mc:Ignorable="d" d:DesignWidth="800" d:DesignHeight="450"
        x:Class="AvaloniaTransitions.MainWindow"
        Title="AvaloniaTransitions">
    <Grid RowDefinitions="*, Auto">
        
        <Border Grid.Row="0" Background="Navy">
            <StackPanel Orientation="Vertical" VerticalAlignment="Top">
                <CheckBox Name="CheckBox" Margin="20">Show Footer</CheckBox>
            </StackPanel>
        </Border>
        
        <Border Grid.Row="1" 
                IsVisible="{Binding #CheckBox.IsChecked}"
                BorderThickness="0 1 0 0"
                BorderBrush="Red"
                Background="DarkRed">
            <TextBlock Margin="20">Footer</TextBlock>
        </Border>
        
    </Grid>
</Window>
```

This is what happens when you toggle the checkbox:
{{< rawhtml >}} 

<video width=450px controls autoplay>
    <source src="SampleWithoutAnimations.mp4" type="video/mp4">
    Your browser does not support the video tag.  
</video>

{{< /rawhtml >}}

## Desired Result
It would be nice if the footer panel would fade in and slide in from the bottom when it's shown and fade out as well as slide out when it's hidden.

### Failed Approaches

#### SplitView
My very first approach was to use the [SplitView](https://docs.avaloniaui.net/docs/reference/controls/splitview) control. Unfortunately this only works for scenarios where the content to be shown is either on the left or right hand side. There's a suggestion to [expand the SplitView control](https://github.com/AvaloniaUI/Avalonia/issues/14346), but it's still not implemented.

#### KeyFrame Animations
My attempts to do this animation using [KeyFrame animations](https://docs.avaloniaui.net/docs/guides/graphics-and-animation/keyframe-animations) all failed at some point. The whole thing was very unreliable and unpredictable. I wasn't able to create the proper animation using KeyFrames to work in all scenarios. If you happen to know how to do it with KeyFrame animations, let me know in the comments.

## Solution
I was able to successfully implement the animation using [Transitions](https://docs.avaloniaui.net/docs/guides/graphics-and-animation/transitions). To make this work, the following needs to be done:

### Style Classes

In Avalonia, there's a very convenient way to apply style classes based on binding values. Take a close look at lines 2 and 3 in the updated **Footer** border code:

```xml {linenos=table}
<Border Grid.Row="1" 
        Classes.show="{Binding #CheckBox.IsChecked}"
        Classes.hide="{Binding !#CheckBox.IsChecked}"
        BorderThickness="0 1 0 0"
        BorderBrush="Red"
        Background="DarkRed">
        <TextBlock Margin="20">Footer</TextBlock>
</Border>
```

Whenever the IsChecked property from the CheckBox is true, the `show` class is applied to the border. When the IsChecked property is false, the `hide` class is applied to the border. If you open the **Dev Tools** using `F12`, you can see the classes being applied when you toggle the check box.

{{< callout type="info" >}}
  Also note that the binding to the `IsVisible` property has been removed from the border. We deal with that later. So right now, the **Footer** is always visible.
{{< /callout >}}

### Fade-In and Fade-Out

Now that we have those two classes applied to the border depending on the checked state of our checkbox, we need to create two styles and one transition to make the border fade-in and fade-out.

```xml {linenos=table}
<Border Grid.Row="1" 
        Classes.show="{Binding #CheckBox.IsChecked}"
        Classes.hide="{Binding !#CheckBox.IsChecked}"
        BorderThickness="0 1 0 0"
        BorderBrush="Red"
        Background="DarkRed">
    <Border.Styles>
        <Style Selector="Border.show">
            <Setter Property="Opacity" Value="1" />
        </Style>
        <Style Selector="Border.hide">
            <Setter Property="Opacity" Value="0" />
        </Style>
    </Border.Styles>
    <Border.Transitions>
        <Transitions>
            <DoubleTransition Property="Opacity" Duration="0:0:1" />
        </Transitions>
    </Border.Transitions>
    <TextBlock Margin="20">Footer</TextBlock>
</Border>
```

- In lines 8 and 9 we create a style which selects the border with the `show` class applied and set the `Opacity` property value to `1`.
- In lines 11 and 12 we create a style for the border with the`hide` class applied and set the `Opacity` property value to `0`.
- Lastly, in lines 15 through 19 we create a `DoubleTransition` for the `Opacity` property. For demonstration, I used `0:0:1` (1 second). In a real world application you may want this animation to be more subtle and set it to `0:0:0.2` (200 ms).

{{< callout type="info" >}}
  If you run the sample with the code above, you will notice that the **Footer** will fade-in and fade-out but the border area is always visible (we removed the `IsVisible` binding). So right now, the border will always occupy the content in the grid resulting in a black area at the bottom where the footer is shown.
{{< /callout >}}

### Handle Visibility

The challange we have now is that we need to set the `IsVisible` to false whenever the `Opacity` is `0` and set `IsVisible` to true whenever the `Opacity` is greater than `0`. This can be achieved with another `Style`:

```xml {linenos=table}
<Border Grid.Row="1" 
        Classes.show="{Binding #CheckBox.IsChecked}"
        Classes.hide="{Binding !#CheckBox.IsChecked}"
        BorderThickness="0 1 0 0"
        BorderBrush="Red"
        Background="DarkRed">
    <Border.Styles>
        <Style Selector="Border.show">
            <Setter Property="Opacity" Value="1" />
        </Style>
        <Style Selector="Border.hide">
            <Setter Property="Opacity" Value="0" />
        </Style>
        <Style Selector="Border[Opacity=0]">
            <Setter Property="IsVisible" Value="False" />
        </Style>
    </Border.Styles>
    <Border.Transitions>
        <Transitions>
            <DoubleTransition Property="Opacity" Duration="0:0:1" />
        </Transitions>
    </Border.Transitions>
    <TextBlock Margin="20">Footer</TextBlock>
</Border>
```

Lines 14 through 16 demonstrate how to create a style like that.

### Sliding Animation

Sliding animations can be done using `RenderTransforms` and the corresponding `translate` methods. In our case we only need the `translateY` methods.

```xml {linenos=table}
<Border Grid.Row="1" 
        Classes.show="{Binding #CheckBox.IsChecked}"
        Classes.hide="{Binding !#CheckBox.IsChecked}"
        BorderThickness="0 1 0 0"
        BorderBrush="Red"
        Background="DarkRed">
    <Border.Styles>
        <Style Selector="Border.show">
            <Setter Property="Opacity" Value="1" />
            <Setter Property="RenderTransform" Value="translateY(0px)" />
        </Style>
        <Style Selector="Border.hide">
            <Setter Property="Opacity" Value="0" />
            <Setter Property="RenderTransform" Value="translateY(75px)" />
        </Style>
        <Style Selector="Border[Opacity=0]">
            <Setter Property="IsVisible" Value="False" />
        </Style>
    </Border.Styles>
    <Border.Transitions>
        <Transitions>
            <DoubleTransition Property="Opacity" Duration="0:0:0.2" />
            <TransformOperationsTransition Property="RenderTransform" Duration="0:0:0.2" />
        </Transitions>
    </Border.Transitions>
    <TextBlock Margin="20">Footer</TextBlock>
</Border>
```
In lines 10 and 14 I added another `Setter` for the `show` and `hide` class of our border. The property `RenderTransform` should have a `translateY` offset of 75px on the y-axis when hidden and no offset (0px) when shown.
In line 23 we need to add a `TransformOperationsTransition` for the `RenderTransform` property.

That's it. The result (with shorter timings) looks like this:
{{< rawhtml >}} 

<video width=450px controls autoplay>
    <source src="SampleWithAnimations.mp4" type="video/mp4">
    Your browser does not support the video tag.  
</video>

{{< /rawhtml >}}

## Conclusion
Note, that I intentionally set those background colors for demonstration purposes to show the impact on the layout when handling the `IsVisible` property. The effect that the **Footer** bottom area is hidden after the animation may not be that visible depending on your layout.

So one improvement to make this smoother would be to not deal with the IsVisible property the way I did and simply just let the **Footer** seemlessly slide-in and out. Not sure how this can be done. Another thing which is not ideal is the hardcoded `translateY` value (75px). A solution where just the desired height of the **Footer** is taken would be much better. Also not sure how this can be accomplished.

I'm still learing Avalonia and there's a good chance that there are other ways to accomplish a similar effect. If you have suggestions or different solutions on how to implement these animations, let me know in the comments.