---
title: Object Lifecycle
comments: true
date: 2024-07-15
tags: 
    - Avalonia
---


Here's the order of events when dealing with Avalonia UI components. I constantly forget it and often need it. To my future self...


<!--more-->

## Introduction

In this post, I only look at the virtual methods of a control you can override. A lot of those virtual methods have corresponding events and the order of the events should match the order of the virtual method calls.

## Creation

The following events/virtual methods are called when an object is created and shown in Avalonia:

- ctor
- OnAttachedToLogicalTree
- OnInitialized
- OnDataContextBeginUpdate
- OnDataContextChanged
- OnDataContextEndUpdate
- OnAttachedToVisualTree
- OnApplyTemplate
- OnLoaded

## Destruction

When an object is destroyed, the following virtual methods are called:

- OnDetachedFromLogicalTree
- OnDetachedFromVisualTree
- OnUnloaded

## OnPropertyChanged

The virtual method `OnPropertyChanged` will be invoked multiple times for each property that changed at different stages. AFAIK there's no guaranteed order of `OnPropertyChanged` calls during binding - at least there's no documentation on that and if you see a certain order, it could change in the future. This one bit me because I had properties which depended on each other and the order was crucial. The recommendation is to only "collect" all the data needed for the control and use one of the lifetime events to decide what to do.