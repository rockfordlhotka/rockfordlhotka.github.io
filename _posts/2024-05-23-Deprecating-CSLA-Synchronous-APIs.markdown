---
layout: post
title: Deprecating CSLA Synchronous APIs
date: 2024-05-23T00:00:00.0000000-05:00
categories: []
tags: []
published: true
permalink: 
image: 
---
Over the years (decades actually), [CSLA](https://cslanet.com) has gone through many, many major changes.

We're starting to discuss the possibility of dropping support for synchronous APIs in many cases. For example, do we still need sync data portal methods? Or sync business rules?

[Please fill out our survey](https://forms.office.com/r/0J5JPXPYc4)

Looking back over time, changes have included:

* Moving from VB/COM to .NET
* Embracing generic types
* Building the framework code in C# instead of VB.NET
* Opening the data portal for asmx, WCF, HTTP, gRPC, and RabbitMQ transports (and eliminating Remoting, asmx, and WCF)
* Implementing our own serialization formatter
* Supporting async APIs alongside sync APIs
* Requiring the use of dependency injection

There have been many other important changes too, but these highlight the slow evolution of CSLA to keep up with modern development practices and .NET from 1997 to today.

Deprecating sync APIs would have a major impact on a lot of existing code. Not so much for modern Blazor, Razor Pages, or MVC apps, but certainly for most Windows Forms, WPF, Web Forms and some MAUI apps.

This would mean the data portal would require the use of async, so no more `Fetch`, only `FetchAsync`. 

A cascading impact is that any business rules that use the data portal would have to become async rules.

> As an aside, we're also talking about a scheme by which async rules could be marked to run in priority order rather than concurrently - to help preserve existing sync priority rule order behaviors.

Knowing that this change would have a massive impact on existing codebases built on CSLA, it is important to gather feedback from you so we can determine whether this is a workable idea or not. And if it is workable, what kind of timeframe might be realistic.

[Please fill out our survey](https://forms.office.com/r/0J5JPXPYc4)