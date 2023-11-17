---
layout: post
title: Upgrading to .NET 8
date: 2023-11-17T00:00:00.0000000-06:00
categories: []
tags: []
published: true
permalink: 
featuredImageUrl: 
---
At [VS Live](https://vslive.com) in Orlando this week there were a number of people who had questions about upgrading to .NET 8; mostly from .NET Framework 4.5 or similar versions that are no longer supported.

The high level answer is to follow a path like this:

1. Upgrade to .NET Framework 4.8
1. Change all your project files to the new project file format
1. Switch all your non-UI projects (class libraries) to target .NET Standard 2.0
1. Modernize your UI projects to .NET 8
   1. This might be a "big bang" rewrite if you can afford the risk and other consequences
   1. This might be a piecemeal process that goes page by page over a period of months; though this also has consequences and costs
1. Once all the UI projects (apps) are on .NET 8, retarget your .NET Standard 2.0 class libraries to .NET 8
1. Throw a party 🥳🎉🎈

There are some obvious points of complexity in this process.

## Upgrade to .NET Framework 4.8

This may be a straightforward process, or not. 

Some of your dependencies may not support the latest (and last) version of the .NET Framework, and this might force you to rework code or find new libraries to leverage.

There may be API changes to some parts of .NET that affect your code (depending on how old your existing codebase is today).

## Change Project Files to New Format

Changing to the modern project file format is often fairly easy - once you understand the new format and the way NuGet packages and other dependencies are referenced. It is worth taking the time to understand the new format to make the process easier.

Note that UWP projects can't use the new project structure, so those projects will be stuck using the old project style.

## Target ns2.0 in Class Libraries

Once your projects are using the new project file format, it is easy to identify the target framework for each project. It will be something like "net4.8", and for _class libraries_ you can try changing to "ns2.0" to use .NET Standard 2.0 as the target.

Why? Because .NET Standard 2.0 class libraries can be referenced by .NET 4.8 (really by .NET 4.6.2 and higher) projects and also by .NET 8 projects. This means that all your non-UI projects, if they target ns2.0, can be used in your existing codebase and your new codebase as you modernize the UI app code.

There can be complexities with dependencies when switching to ns2.0. Some older packages may have never been updated to work with ns2.0, though they may also not support net4.8 either. In any case, now is the time to find replacements for such obsolete packages.

As a side note, for folks that are currently using [CSLA .NET](https://cslanet.com) for your business layer, modernizing to CSLA 7 or higher in your business class libraries will make the rest of the process much simpler. CSLA 7 and higher provides full abstractions and support for all modern UI app project types, as well as still supporting all the legacy UI technologies from .NET Framework.

## Modernize UI Projects

This is the hardest part of the whole process, because for apps of any size it means deciding whether to rewrite the entire app all at once or to rewrite forms/pages one by one over time.

Neither option is easy, and both have tradeoffs in terms of short and long term costs, risk to your business, and other consequences. A full discussion of _this_ topic is more than I want to write in this blog post.

In any case, you also need to come up with a strategy about what UI/app technology you want to use for the new .NET 8 app. This involves business and technical considerations.

If your current app meets all business needs and makes your users productive and happy, then it is usually cheapest and simplest to migrate to a similar UI technology in modern .NET. One the other hand, if your current app has shortcomings that can be addressed with better UI technologies in modern .NET, then this might be your opportunity to provide business justification for a more complex modernization.

Similarly, if your current UI technology has a direct mapping to a UI technology in .NET 8, it is often best to take the simple upgrade path. Some older UI technologies are no longer available in modern .NET, and so you will be forced to find a different UI technology for the modernized app.

Here is a list of some of the more common and direct mappings.

| Legacy UI Tech | Modern Equivalent |
| --- |
| Windows Forms | Windows Forms |
| WPF | WPF |
| UWP | WinUI3 or WPF |
| Web Forms | none (consider Blazor) |
| ASP.NET MVC | ASP.NET Core MVC or Razor Pages |
| ASP.NET web services | ASP.NET Core web services |
| Xamarin native | none (consider .NET MAUI) |
| Xamarin.Forms | .NET MAUI |

Here is a list of options I would seriously consider in terms of enabling cross-platform UI scenarios that may better meet your current business needs.

| Legacy UI Tech | Modern Equivalent |
| --- |
| Windows Forms | Blazor or .NET MAUI |
| WPF | Blazor, [Avalonia](https://www.avaloniaui.net/), or .NET MAUI |
| UWP | Blazor, [Platform.Uno](https://platform.uno/), or .NET MAUI |
| Web Forms | Blazor |
| ASP.NET MVC | ASP.NET Core Razor Pages or Blazor |
| ASP.NET web services | ASP.NET Core web services |
| Xamarin native | .NET MAUI |
| Xamarin.Forms | .NET MAUI |

For any of the .NET MAUI options, I would also suggest considering the use of a MAUI Blazor hybrid app, where you build the majority of the app using Blazor, wrapped in a MAUI shell that provides access to native platform capabilities on iOS, Android, Windows, and Mac. This is because I personally find the Blazor UI story to be superior to the MAUI/XAML story.

Also, if you like XAML, it is worth checking out Platform.Uno and Avalonia. They do more than is reflected in the table, and you might find them to be very compelling depending on your needs.

## Retarget Class Libraries to .NET 8

Once you have modernized all your UI/app code to .NET 8 there should be no legacy code left in your codebase. Everything will be targeting .NET 8 or .NET Standard 2.0.

At this point you can change all your class libraries that target "ns2.0" to target "net8.0".

This will unlock the use of all the current language features available in modern .NET, and will also let you use newer package dependencies that only target .NET 8.

## Throw a Party

This was almost certain a lot of work, and it is important to take time to pause and reflect on a job well done!

## Need Help?

If you are using CSLA .NET and would like to engage me for advisory services around modernization of your apps, please reach out. My contact information is on my [link tree](https://linktr.ee/rockylhotka).

If you are doing any sort of cloud modernization, perhaps taking this opportunity to not only modernize to .NET 8, but also to leverage Azure and GitHub cloud services, please reach out to [Xebia|Xpirit](https://xpirit.com), as we are experts in cloud modernization for infrastructure, devops, and cloud-native app dev.