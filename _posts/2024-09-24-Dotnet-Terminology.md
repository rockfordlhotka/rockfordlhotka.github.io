---
title: .NET Terminology
abstract: ''
keywords: ''
categories: ''
weblogName: Rockford Lhotka
postId: 
postDate: 2024-09-24T13:50:42.0008829-05:00
postStatus: publish
dontInferFeaturedImage: false
dontStripH1Header: false
---
I was recently part of a conversation thread online, which reinforced the naming confusion that exists around the .NET (dotnet) ecosystem. I thought I'd summarize my responses to that thread, as it surely can be confusing to a newcomer, or even someone who blinked and missed a bit of time, as things change fast.

## .NET Framework

There is the Microsoft .NET Framework, which is tied to Windows and has been around since 2002 (give or take). It is now considered "mature" and is at version 4.8. We all expect that's the last version, as it is in maintenance mode.

I consider .NET Framework (netfx) to be legacy.

## Modern .NET

There is modern .NET (dotnet), which is cross-platform and isn't generally tied to any specific operating system.

I suppose the term ".NET" encompasses both, but most of us that write and speak in this space tend to use ".NET Framework" for legacy, and ".NET" for modern .NET.

The .NET Framework and modern .NET both have a bunch of sub-components that have their own names too. Subsystems for talking to databases, creating various types of user experience, and much more. Some are tied to Windows, others are cross platform. Some are legacy, others are modern.

It is important to remember that modern .NET is cross-platform and you can develop and deploy to Linux, Mac, Android, iOS, Windows, and other operating systems. It also supports various CPU architectures, and isn't tied to x64.

## Modern Terminology

The following table tries to capture most of the major terminology around .NET today.

| Tech | Status | Tied to Windows | Purpose |
| --- | --- | --- | --- |
| .NET (dotnet) 5+ | modern | No | Platform |
| ASP.NET Core | modern | No | Web Framework |
| Blazor | modern | No | Web SPA framework |
| ASP.NET Core MVC | modern | No | Web UI framework |
| ASP.NET Core Razor Pages | modern | No | Web UI framework |
| .NET MAUI | modern | No | Mobile/Desktop UI framework |
| MAUI Blazor Hybrid | modern | no | Mobile/Desktop UI framework |
| ADO.NET | modern | No | Data access framework |
| Entity Framework | modern | No | Data access framework |
| WPF | modern | Yes | Windows UI Framework |
| Windows Forms | modern | Yes | Windows UI Framework |

## Legacy Terminology

And here is the legacy terminology.

| Tech | Status | Tied to Windows | Purpose |
| --- | --- | --- | --- |
| .NET Framework (netfx) 4.8 | legacy | Yes | Platform |
| ASP.NET | legacy | Yes | Web Framework |
| ASP.NET Web Forms | legacy | Yes | Web UI Framework |
| ASP.NET MVC | legacy | Yes | Web UI Framework |
| Xamarin | legacy (deprecated) | No | Mobile UI Framework |
| ADO.NET | legacy | Yes | Data access framework |
| Entity Framework | legacy | Yes | Data access framework |
| UWP | legacy | Yes | Windows UI Framework |
| WPF | legacy | Yes | Windows UI Framework |
| Windows Forms | legacy | Yes | Windows UI Framework |

## Messy History

Did I leave out some history? Sure, there's the whole ".NET Core" thing, and the .NET Core 1.0-3.1 timespan, and .NET Standard (2 versions).

Are those relevant in the world right now, today? Hopefully not really! They are cool bits of history, but just add confusion to anyone trying to approach modern .NET today.

## What I Typically Use

What do _I personally_ tend to use these days?

I mostly:

* Develop modern dotnet on Windows using mostly Visual Studio, but also VS Code and Rider 
* Build my user experiences using Blazor and/or MAUI Blazor Hybrid
* Build my web API services using ASP.NET Core
* Use ADO.NET (often with the open source [Dapper](https://github.com/DapperLib/Dapper)) for data access
* Use the open source [CSLA .NET](https://cslanet.com) for maintainable business logic
* Test on Linux using Ubuntu on WSL
* Deploy to Linux containers on the server (Azure, Kubernetes, etc.)

## Other .NET UI Frameworks

Finally, I would be remiss if I didn't mention some other fantastic cross-platform UI frameworks based on modern .NET:

* [Uno Platform](https://platform.uno)
* [Avalonia](https://avaloniaui.net/)
* [OpenSilver](https://opensilver.net/)
