---
layout: post
title: Should I Migrate to .NET 5
date: 2020-11-11T00:00:00.0000000-06:00
categories: []
tags: []
{}
---
Now that [Microsoft has released .NET 5](https://devblogs.microsoft.com/dotnet/announcing-net-5-0/), you are probably wondering whether you should start using .NET 5, and perhaps more importantly whether you should migrate existing code to .NET 5.

We've had some good discussion around this within [Magenic](https://magenic.com), with some folks arguing against the use of .NET 5, and others taking a more nuanced view.

I thought I'd summarize my thoughts.

## Understanding LTS Releases

Microsoft now uses an LTS model (long term support) for .NET releases. The LTS model has been used by Linux (for example) for a long time, and it is a good model. The basic idea is that _some_ releases are LTS and so have multi-year long term support. Other, intermediate, releases are _only supported until the next release_, so not for years.

.NET Core 3.0 was _not_ LTS, and .NET Core 3.1 _is_ LTS. So this week all support for .NET Core 3.0 ended, because .NET 5 came out. However, .NET Core 3.1 continues to be supported, because it is the LTS release.

.NET 5 is _not and LTS release_, which means that it'll go off support when .NET 6 comes out in about a year.

In summary, if your org is a low-risk type org, you should plan to only use LTS releases, and you should ignore intermediate releases. This way you'll only need to switch from one .NET release to the next every 2-3 years instead of more frequently.

However, the intermediate releases often provide compelling features, fixes, and so might be worth using, if your acceptance of risk allows it.

There's also the tech debt consideration. Moving from LTS to LTS every 2-3 years means that each upgrade cycle will be relatively large, because you'll need to deal with 2-3 years of platform/framework changes you've ignored, so the effort might be quite high.

Upgrading to each intermediate release spreads that work out over time. I don't think it _reduces_ the effort necessarily, but it avoids a "big bang" upgrade at the 2-3 year mark. Sometimes it can be hard to get budget (money, time) for a big upgrade, and it can be easier to "amortize" the upgrade effort by updating to the intermediate releases on a more frequent basis.

I don't think there's a universal "right answer" here. It depends a lot on you and your org, and tolerance for risk, budgeting, appetite for big updates every few years vs smaller updates every few months, and related factors.

I also think the specific parts and features of .NET you are using should be evaluated, as .NET 5 provides more benefits for some types of app, and less for others. You also need to consider where you are today.

## Where You Are Today

Thinking about where you might be today, the first consideration is whether you are dealing with an existing codebase or are create a new greenfield codebase.

### Greenfield Codebase

If you are just starting to build your app today, your choices are .NET Core 3.1 (LTS) or .NET 5 (current). In my view this comes down to your long term strategy based on the factors I discussed earlier in this post.

For orgs that plan to only target LTS releases, clearly you'll want to build your new app targeting .NET Core 3.1, with a plan to upgrade to .NET 6 in about a year.

Orgs that want to use intermediate releases should target .NET 5, with a plan to upgrade to .NET 6 in about a year.

I'd also suggest you read through the low, medium, and high value app scenarios I discuss later in this post, as that might inform your decision.

### Existing Codebase

The harder decision is with existing brownfield codebases. In this scenario your existing code might be targeting anything from .NET Framework 2.0 to 4.8, or if you are lucky perhaps it already targets some version of .NET Core.

#### Code Targeting .NET Framework 4.6 or Older

If your current code targets .NET Framework 4.6 or older, the upgrade process may be substantial. A lot depends on the architecture of the app and the discipline followed to adhere to that architecture and coding standards over the life of the app so far.

An app that has a clean, layered architecture, with good discipline around separation of concerns, will require some effort to upgrade. Such an architecture means that the upgrade effort will be _mostly_ focused on the UI and data access layers.

Apps that have a less clean architecture, or where there's been a lack of discipline around seperation of concerns over time may need major work, or even a complete rewrite to move forward.

#### Code Targeting .NET Framework 4.7.2 and Newer

Code that targets .NET Framework 4.7.2 _and that has a layered architecture with separation of concerns_ is generally pretty easy to upgrade. Much of the non-UI code in such apps can be moved from .NET Framework to .NET Standard 2.0, and that allows the code to be used by your existing .NET Framework app _and also_ by the new .NET Core 3.1 or .NET 5 app code.

In this scenario, .NET Standard 2.0 is the key to success, because it is the bridge between .NET Framework 4.7.2+ and modern .NET. A .NET Standard 2.0 project can be referenced and used by both "flavors" of .NET (as well as Xamarin and UWP).

> ℹ For the most part, you don't need to worry about .NET Standard 2.1. That version of .NET Standard _is not compatible_ with .NET Framework, and is really only useful for Xamarin and pre-.NET 5 Blazor apps. When .NET 6 comes out it will no longer be relevant for Xamarin apps either.

Apps without a good architecture or clean separation of concerns are still going to be highly problematic, and may require major work or a rewrite.

#### Code Targeting .NET Core

Apps that target .NET Core (including projects targeting .NET Standard), are the easiest to upgrade, because you've already got a largely modern codebase.

If your code targets something older than .NET Core 3.1 you need to upgrade, at least, to 3.1. .NET Core 3.0 just became unsupported.

Whether you choose to upgrade existing .NET Core code to .NET 5, or just to .NET Core 3.1, is a decision driven by your LTS strategy and the other factors I discussed earlier in this post.

At this point I'll discuss what I think of as low, medium, and high value scenarios for upgrading to .NET 5.

## Low-Value App Scenarios

When I think of apps where upgrading to .NET 5 provides the least value, these are what come to mind:

* Server-side web apps (not running in containers)
* Server-side web APIs (not running in containers)
* Apps that rely heavily on WCF or .NET Remoting
* Apps that rely heavily on Windows Workflow
* Apps that are tightly coupled to Windows
* Apps that rely on advanced features of .NET Framework such as AppDomains

### Server-side Web

There aren't that many compelling changes in .NET 5 for ASP.NET server-side web development, so if your current app is Web Forms, or even MVC, and isn't running in containers, the value of an upgrade may not offset the effort.

### WCF, Remoting, and Other Not-Supported Tech

Apps that use Web Forms, WCF, Remoting, Windows Workflow, AppDomains, and other features that don't even exist in .NET 5 (or .NET Core) will require major effort, probably including rearchitecting and rewriting, to "upgrade".

Whether to upgrade such apps is more of a business decision than a technical one. This sort of decision comes down to an assessment of the business value of the app, the cost of running the app is-is, possibly modernizing to .NET Framework 4.8 and running it "forever" at that level, vs the cost of rewriting to modernize and the potential benefits of such a rewrite.

## Medium-Value App Scenarios

Other app scenarios are, in my view, less obvious. These often rely on you evaluating your tolerance for risk, and willingness to tackle technical debt now or later. These include:

* Xamarin apps
* UWP apps
* Windows Forms apps
* WPF apps

### Xamarin

The big updates for Xamarin will be in .NET 6, with Xamarin becoming the foundation for the new Maui cross-platform UI framework. This means that upgrading your existing Xamarin apps to .NET 5 and Xamarin.Forms 5 is a risk/benefit assessment you should make based on the factors I discussed earlier in this post. There's some value in upgrading, but the _high value_ upgrade will be in about a year.

### UWP

There are some enhancements around UWP in .NET 5 and the latest Visual Studio. However, UWP continues to run in its own "flavor" of .NET that isn't really .NET Core or .NET 5, and so is in its own world in many ways.

If you are heavily invested in UWP you really need to take a look at the [Uno Platform](https://platform.uno). I realize UWP is from Microsoft, but it seems to me that the truly vibrant community and innovation around UWP is from the Uno community, where UWP has been brought to iOS, Android, and WebAssembly.

Were I the product owner or technical lead for a UWP app, I'd be seriously evaluating whether that is the technology that enables my app to reach more users without rewriting to Xamarin (Maui) or Blazor.

### Windows Forms and WPF

Windows Forms and WPF is more nuanced. Today, .NET Core 3.1 has support for Windows Forms and WPF, and the support is substantially better in .NET 5. A lot of orgs have major enterprise apps written in Windows Forms or WPF, and have been looking for a way to keep these apps current and modern for years.

I argue you have four very valid options for such apps going forward:

* Maintain them in .NET Framework 4.8 "forever"
* Upgrade them to .NET 5 and then .NET 6
* Rewrite them to Xamarin
* Rewrite them to Blazor

I organized that list from least effort/risk to highest effort. I also think it is from least value to highest value, though you might argue whether it is better to achieve cross-platform reach via Xamarin (Maui) or Blazor.

If your app will never need to support anything other than Windows devices, I think your strategy should probably revolve around eventually upgrading your .NET Framework app to .NET 5+. Whether you start that process now or in the future depends on your acceptance of risk, and ability to invest in the app. Microsoft has provided some good tooling to help the migration though, and I recommend you evaluate that tooling.

Many users today expect apps to be available on their tablets, Macs, and even phones, not just on Windows PCs. The two Microsoft platforms that enable cross-platform apps are Xamarin (Maui) and Blazor.

Xamarin enables you to build native apps that deploy to Windows, iOS, Android, and other client operating systems. The upside is that these are native apps with full native capabilities. The downside is that they are native apps that must be deployed to client devices, and deployment can be very complex.

Blazor WebAssembly enables you to build browser-native apps that deploy to client devices via the browser, and _run on the client device_ just like your Windows Forms and WPF apps do today. The upside is that deployment is easy, and the apps run in any modern browser. The downside is that the apps are restricted by the browser sandbox and so can't tap into all native capabilities of each client device.

Deciding between these four options requires some research, evaluation, and thought at both the business and technical levels.

## Higher-Value App Scenarios

Finally, there are some scenarios where I think there's high value in considering .NET 5, or at least .NET Core 3.1. These include:

* Apps that need cross-platform deployment
* Server-side web apps running in containers
* Server-side web APIs running in containers
* Small devices and IoT
* Blazor apps

### Cross-Platform Deployment

If you want to run your code on Linux or other non-Windows operating systems you clearly need to modernize from .NET Framework to .NET Core 3.1 or .NET 5. Modern .NET supports Linux as a first-class deployment target, and a large number of projects [Magenic](https://magenic.com) is doing for our clients do target Linux servers.

> 💭 My personal view is that the future of server-side computing is Linux-based containers, and this informs all server-side computing strategies for me.

### Containers

Containers are not only typically cross-platform, but they should be treated as being resource constrained. In particular, the less memory and CPU used by a container, the more value you can get from your overall container deployment strategy.

This means that the memory and performance enhancements in .NET 5 over .NET Core 3.1 might be a compelling reason for you to consider upgrading.

At the very least, using .NET Core 3.1 to build your container-based code means you can take full advantage of the various cloud-native support features built into modern .NET around configuration.

### Small Devices and IoT

Similar to containers, if you are deploying your app to IoT or other smaller devices with limited memory and resources, performance, memory consumption, and the ability to do things like AOT (ahead of time compilation) and trimming become very important.

A lot of these capabilities have been introduced with .NET 5, and if you deploy to devices with limited resources I strongly recommend you research and evaluate whether these features will help you.

### Blazor

I'd argue that if you are already targeting Blazor, or are planning to target it now, that you have a decent acceptance of risk. This is because Blazor is still quite new, and yet you are probably as excited about it as I am!

Blazor in .NET 5 is substantially improved, especially in WebAssembly, compared to .NET Core 3.1, and I suspect anyone building Blazor apps will generally upgrade to .NET 5.

As always, this is a value judgement, and you should evaluate the risks and benefits as I discussed at the start of this post.

## Summary

You can probably tell that I am very excited about .NET 5, and particularly how well it supports Linux containers and Blazor.

I also think this is an important release for anyone maintaining Windows Forms and WPF apps, because upgrading from .NET Framework to .NET 5 is a realistic possibility, and a way to stop accruing tech debt on apps that are very often critical to your organization.

My commercial: if you need help assessing the business and technical considerations around app modernization, including possible cloud migrations, please consider contacting [Magenic](https://magenic.com), as we have the people, process, and experience to help you modernize and cloud-enable your software fast.