---
title: How will WebAssembly replace frameworks like Angular and React?
weblogName: Rockford Lhotka
postDate: 2019-12-19
---
# How will WebAssembly replace frameworks like Angular and React?

WebAssembly is a way to run compiled code in a browser.

Angular and React are UI frameworks.

These are entirely different things.

So the real question is whether someone creates a UI framework for some language that compiles to WebAssembly, and how long that will take to mature.

Right now, for example, there are two rapidly maturing UI frameworks that use .NET and C# running in the browser: [Blazor](https://blazor.net) and [Uno](https://platform.uno).

Blazor uses Razor syntax to create the UI. Razor syntax is an extension to HTML and CSS, and this means that your existing HTML/CSS skills directly apply, but underneath the markup are powerful navigation, data binding, and other features. Plus all the power of .NET and C#.

Uno uses XAML syntax to create the UI. This means that if you know XAML from WPF, UWP, or Xamarin, all those skills apply. XAML is much better thought out than HTML, but isn’t as popular, and so it lacks the widespread access of free markup you can “borrow” from across the web.

I would hope that UI frameworks will emerge from the Go and Rust communities, and perhaps from other languages that can compile into WebAssembly.

In any case, WebAssembly itself won’t replace Angular or React, because WebAssembly is not a UI framework. But UI frameworks built using other languages, such as C#, Go, or Rust, will almost certainly compete with Angular and React.