---
title: Choosing Client UI Tech in Early 2020
weblogName: Rockford Lhotka
postDate: 2020-03-30
---
I'm writing this at the end of March in 2020. It is important to note the timeframe, because of all parts of the tech stack for enterprise application development, the UI frameworks and targets change fastest.

I was asked a question on the [CSLA forum](https://github.com/MarimerLLC/cslaforum) about UI technology choice.

> My app would only run on Windows desktop and Android. Which technology should I go for?
Would developing a UI in Blazor suffice for both the platforms?.

I thought the answer would be better as a blog post, so here we go.

## Cross Platform

Before getting into the details, I want to pause and examine the thought of building _any_ app right now that only targets Windows. Or Android. Or iOS.

Neither we, nor our users, really know what the future holds. Personally I tend to err on the side of assuming that most apps will end up needing to be broadly cross-platform.

Sure, there are niche scenarios where the app is part of some larger hardware system that is built assuming a specific type of device. Manufacturing machines and retail devices are examples of physical devices built as peripherals to specific computing hardware. And in that case you can be more confident that your software will only ever run on that computing device, because the whole system would be junked to replace it.

But _business_ software is almost certainly going to need to run on many types of device, supporting many types of operating system.

I call this out, because through the rest of this post I'll tend to lean heavily toward cross-platform solutions rather than single-platform options.

Now onto the answer.

## Smart or Thin Client

Step one is to determine whether your users will be satisfied with a thin client, or would benefit from a smart client experience. Of course this is really a spectrum, including:

1. Pure server-side code with a browser-based terminal experience
1. Server-side code with some JavaScript helpers to provide a slightly richer user experience
1. Smart client code with a robust and interactive user experience

Options 1 and 2 are largely the same, and in the Microsoft world are usually implemented using ASP.NET. Today I'd favor using Razor Pages, which is a nice evolution on top of the less productive MVC. But some folks prefer writing more code and so like MVC.

Either way you get to the same place: server-side code that projects HTML and _maybe_ some JavaScript to create some sort of terminal-style user experience via the browser.

Or you go for a smart client option.

## Smart Client Options

Smart client development has been popular since the early 1990's thanks to Visual Basic, Powerbuilder, and similar technologies. It temporarily faded away in the face of server-side web development, but users constantly push for better user experiences, and we've seen the rebirth of smart client development over the past several years.

### Native Mobile Apps

This started with Objective C and the desire to cash in on the iPhone/iPad gold rush. That led to Java on Android, then Swift and Kotlin, and (for Microsoft folks) Xamarin.

Today the primary options seem to be:

1. Native via Swift, Java, Xamarin, React Native, or Flutter
1. Non-native cross-platform options (my editorial view is these create the crappy apps that are crappy on all devices)
1. PWA style web pages

If you are a .NET developer, the obvious choice here is Xamarin.

However, Blazor fits into this picture too, as I'll discuss later.

### JavaScript Single Page Apps

This started with a wild frenzy of UI frameworks, all scrambling to build on jquery or replace jquery. So I suppose it started with jquery, one way or the other.

Today things have settled down somewhat, with the primary options being:

1. Angular
1. React plus other stuff

No place here for .NET developers, but again, Blazor fits directly into this picture, as I'll discuss later.

### Desktop Apps

Desktop apps have been around since the 1980's, but really became mainstream in the 1990's. Things have evolved over time, but not radically. The primary options today seem to be:

1. Native via Xamarin, UWP, WPF, Windows Forms, GTK+, Objective C, or Swift
1. Semi-native via Electron
1. PWA style web pages

Many folks would think that WPF is the right choice here, and if you still must support Windows 7 users (though Windows 7 is past end of life now), then WPF is the right choice.

But Xamarin is probably the best option, because it can target UWP along with Mac, Linux, Android, and iOS. Even if you think your app only needs to target Windows, it can be hard to predict the future. There's real value in considering Xamarin for Windows for its flexibility alone.

However, Blazor does fit into this picture too, so let's discuss Blazor.

## WebAssembly and Blazor

Around the turn of the millennium (how cool is it to be able to say that???) there was a healthy debate about the future of the browser and client-side development. In summary, the question was whether browsers should basically remain colorful terminals (a la 3270 mainframe terminals), or if they should become a standard cross-platform runtime for client-side code.

The dot-bomb derailed that conversation, leading to a decade of the browser-as-terminal world, until SPAs became a thing. And we've seen the rise of browser-as-platform over the past decade or so.

WebAssembly is really an important step in the browser-as-platform worldview, because it means we can use nearly any programming language to build apps that run in browsers on any device or OS anywhere. 

Blazor is a UI framework based on .NET, that allows us to use HTML/CSS with C# behind the markup (instead of JavaScript).

Blazor can _also_ be used in a server-side setting, where your code runs on the server, with a small JavaScript _widget_ running in the browser. This model makes the browser much closer to a VT terminal experience than to a 3270 or traditional browser experience. In other words, each keystroke, mouse movement, tap, or other user interaction is _immediately_ provided to the server.

The result is that server-side Blazor and client-side Blazor WebAssembly _feel the same_ to end users. And their code is the same for us devs. But resource consumption is different, because server-side Blazor runs your code on the server, and client-side Blazor runs your code (the same code) on the client device.

What's really cool, and important, is that you can build a Blazor app and choose to deploy it as a server-side or client-side app. Same code, you choose the deployment. Even _more important_ is that you can do both at once, so some users use the server-side deployment, and others use the client-side deployment.

I discuss all this in more depth in my [Using CSLA 5: WebAssembly and Blazor](https://store.lhotka.net/using-csla-5-blazor-and-webassembly) book.

### Blazor Compared to Others

The real question is how does Blazor compare to the older options?

### Blazor vs Native or Desktop

Native mobile (Swift, Java, Xamarin) are apps that deploy to the device and have full access to the device hardware and OS. Native Windows, Mac, and Linux apps deploy to the PC and have full access to hardware and OS features.

Blazor apps are not native, and can only access device or PC hardware or the OS features that are available via browser apps. Browser apps can access a number of commonly used features, but if you need features not available from the browser then that rules out the use of Blazor.

### Blazor vs JavaScript SPA

Server-side Blazor provides a SPA-like user experience, without deploying or running substantial code on the client. If some of your user devices are low-powered or slow, server-side Blazor is probably a great option for those users.

Client-side Blazor provides the same SPA-like user experience, but runs the code on the client device. This provides cheaper scaling because all processing isn't centralized on your servers. Any scenario where you'd have chosen Angular or React is a candidate for client-side Blazor.

But here's where the ability of Blazor apps to be deployed server-side _and_ client-side on a per-user basis becomes really nice. You can support low-end devices with server-side Blazor and higher end devices with client-side Blazor, all with a single codebase!

### Blazor vs Electron

Blazor can run inside Electron, which means there really isn't an either/or question here. You can use Electron to host an app built in Blazor. This means your app can access local hardware and OS features of the PC (whether Windows, Mac, or Linux) just like any Electron app.

However, we know that there are experiments being performed with Blazor to enable it to create apps that are hosted in a lighter weight equivalent to Electron. The problem with Electron is that it includes a node web server to support standard web development, and that is a waste with Blazor, because Blazor really doesn't need a web server at all.

So in the long run it seems likely that Blazor will provide a better alternative to Electron. And in the meantime if you want to use Electron for deployment, Blazor is absolutely an option.

### Blazor and Progressive Web Apps

One key drawback is the lack of offline support. When using Blazor for mobile or desktop development is that the user accesses the app via the browser by navigating to a server URL. Even when using client-side Blazor, the app is _deployed_ from a web URL.

Fortunately modern browsers support the concept of a Progressive Web App (PWA). The PWA technology allows a web site to be designed such that it can work offline, and Blazor apps are essentially a form of web site.

As a result, you can build client-side Blazor apps that can deploy as a PWA on the user's device or PC. That app can then run offline.

Remember that it is still a web app, and so it can't access any hardware or OS features not supported by browser-based apps, but this does provide a lot of flexibility overall, because the app can run offline and can store local data within the browser's local data storage mechanism.

## Summary

So this turned out to be quite the discussion. I think it can be summarized somewhat by this diagram.

![]({{ site.url }}/assets/2020-03-30-Choosing-Client-UI-Tech-in-Early-2020/flowchart.png)

Or even more concise (because really, who wants to give their users an old-fashioned terminal-style user experience??):

1. Do you need native hardware/OS access? Use Xamarin
1. Otherwise use Blazor

Again, I'm talking about business apps, with a heavy bias toward cross-platform support of modern platforms. Your business and app requirements may cause a different thought process based on different priorities.

Here's a way to compare the capabilities I've discussed with the broad technologies.

![]({{ site.url }}/assets/2020-03-30-Choosing-Client-UI-Tech-in-Early-2020/compare.png)

Green is good, yellow is doable but perhaps limited or challenging, red is hard or impossible, gray is not applicable.

You can see clearly that Xamarin is the most capable and flexible of the options, but it lacks the simplicity of web deployment. Blazor has web deployment and offers a great deal of capability. And it is a good bet that Blazor will have more capabilities in the future.
