---
layout: post
title: Why MAUI Blazor Hybrid
date: 2025-04-14T00:00:00.0000000-05:00
categories: []
tags: []
published: true
permalink: 
image: 
---
It can be challenging to choose a UI technology in today's world. Even if you narrow it down to wanting to build "native" apps for phones, tablets, and PCs there are so many options.

In the Microsoft .NET space, there are _still_ many options, including .NET MAUI, Uno, Avalonia, and others. The good news is that these are good options - Uno and Avalonia are excellent, and MAUI is coming along nicely.

At this point in time, my _default_ choice is usually something called a MAUI Hybrid app, where you build your app using Blazor, and the app is hosted in MAUI so it is built as a native app for iOS, Android, Windows, and Mac.

Before I get into why this is my default, I want to point out that I (personally) rarely build mobile apps that "represent the brand" of a company. Take the Marriott or Delta apps as examples - the quality of these apps and the way they work differently on iOS vs Android can literally cost these companies customers. They are a primary contact point that can irritate a customer or please them. This is not the space for MAUI Blazor Hybrid in my view.

## Common Code

MAUI Blazor Hybrid is (in my opinion) for apps that need to have rich functionality, good design, and be _common across platforms_, often including phones, tablets, and PCs. Most of my personal work is building business apps - apps that a business creates to enable their employees, vendors, partners, and sometimes even customers, to interact with important business systems and functionality.

Blazor (the .NET web UI framework) turns out to be an excellent choice for building business apps. Though this is a bit of a tangent, Blazor is my go-to for modernizing (aka replacing) Windows Forms, WPF, Web Forms, MVC, Silverlight, and other "legacy" .NET app user experiences.

The one thing Blazor doesn't do by itself, is create native apps that can run on devices. It creates web sites (server hosted) or web apps (browser hosted) or a combination of the two. Which is wonderful for a lot of scenarios, but sometimes you really need things like offline functionality or access to per-platform APIs and capabilities.

This is where MAUI Hybrid comes into the picture, because in this model you build your Blazor app, and that app is _hosted_ by MAUI, and therefore is a native app on each platform: iOS, Android, Windows, Mac. That means that your Blazor app is installed as a native app (therefore can run offline), and it can tap into per-platform APIs like any other native app.

## Per-Platform

In most business apps there is little (or no) per-platform difference, and so the vast majority of your app is just Blazor - C#, html, css. It is common across all the native platforms, and optionally (but importantly) also the browser.

When you do have per-platform differences, like needing to interact with serial or USB port devices, or arbitrary interactions with local storage/hard drives, you can do that. And if you do that with a little care, you still end up with the vast majority of your app in Blazor, with small bits of C# that are per-platform.

## End User Testing

I mentioned that a MAUI Hybrid app can not only create native apps but that it can also target the browser. This is fantastic for end user testing, because it can be challenging to do testing via the Apple, Google, and Microsoft stores. Each requires app validation, on their schedule not yours, and some have limits on the numbers of test users.

> In .NET 9, the ability to create a MAUI Hyrid that also targets the browser is a pre-built template. Previously you had to set it up yourself.

What this means is that you can build your Blazor app, have your users do a lot of testing of your app via the browser, and once you are sure it is ready to go, then you can do some final testing on a per-platform basis via the stores (or whatever scheme you use to install native apps).

## Rich User Experience

Blazor, with its use of html and css backed by C#, directly enables rich user experiences and high levels of interactivity. The defacto UI language is html/css after all, and we all know how effective it can be at building great experiences in browsers - as well as native apps.

There is a rapidly growing and already excellent ecosystem around Blazor, with open-source and commercial UI toolkits and frameworks available that make it easy to create many different types of user experience, including Material design and others.

From a developer perspective, it is nice to know that learning any of these Blazor toolsets is a skill that spans native and web development, as does Blazor itself.

In some cases you'll want to tap into per-platform capabilities as well. The [MAUI Community Toolkit](https://github.com/CommunityToolkit/Maui) is available and often provides pre-existing abstractions for many per-platform needs. Some highlights include:

* File system interaction
* Badge/notification systems
* Images
* Speech to text

Between the basic features of Blazor, advanced html/css, and things like the toolkit, it is pretty easy to build some amazing experiences for phones, tablets, and PCs - as well as the browser.

## Offline Usage

Blazor itself can provide a level of offline app support if you build a Progressive Web App (PWA). To do this, you create a standlone Blazor WebAssembly app that includes the PWA manifest and worker job code (in JavaScript).

PWAs are quite powerful and are absolutely something to consider as an option for some offline app requirements. The challenge with a PWA is that it is running in a browser (even though it _looks_ like a native app) and therefore is limited by the browser sandbox and the local operating system.

For example, iOS devices place substantial limitations on what a PWA can do and how much data it can store locally. There are commercial reasons why Apple doesn't like PWAs competing with "real apps" in its store, and the end result is that PWAs _might_ work for you, as long as you don't need too much local storage or too many native features.

MAUI Hybrid apps are actual native apps installed on the end user's device, and so they can do anything a native app can do. Usually this means asking the end user for permission to access things like storage, location, and other services. As a smartphone user you are certainly aware of that type of request as an app is installed.

The benefit then, is that if the user gives your app permission, your app can do things it couldn't do in a PWA from inside the browser sandbox.

In my experience, the most important of these things is effectively unlimited access to local storage for offline data that is required by the app.

## Conclusion

This has been a high level overview of my rationale for why MAUI Blazor Hybrid is my "default start point" when thinking about building native apps for iOS, Android, Windows, and/or Mac.

Can I be convinced that some other option is better for a specific set of business and technical requirements? Of course!!

However, having a well-known and very capable option as a starting point provides a short-cut for discussing the business and technical requirements - to determine if each requirement is or isn't already met. And in many cases, MAUI Hybrid apps offer very high developer productivity, the functionality needed by end users, and long-term maintainability.