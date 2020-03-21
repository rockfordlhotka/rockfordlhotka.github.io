---
title: State of CSLA .NET 2020 Edition
weblogName: Rockford Lhotka
postDate: 2020-03-18
---
![](https://raw.github.com/MarimerLLC/csla/master/Support/Logos/csla%20win8_mid.png)

### tl;dr

I'm very excited about .NET, WebAssembly, Blazor, containers, and Kubernetes right now - and how [CSLA .NET](https://cslanet.com) helps you create and maintain complex business logic in what I believe to be the future enterprise app dev.

This post ended up as sort of a walk through time - through the history of CSLA .NET and the Microsoft app dev world over the past 22+ years. Culminating with how CSLA .NET supports containers, Kubernetes, Blazor, and the future of enterprise app dev in .NET.

### COM-Based Origins

CSLA Classic started in 1996 as a way to build distributed apps that shared business logic in a meaningful way over DCOM/MTS/COM+. It was totally rewritten (multiple times) in the lead-up to the release of .NET in 2002. I spent a lot of time in 2000 and 2001 figuring out how the _core philosophy_ of CSLA would work in this new .NET thing.

### Early .NET

From 2002 to 2008 was really the heyday of CSLA .NET. Many apps were n-tier, built using Windows Forms, Web Forms, WPF, and (for a brief shining time) Silverlight. Smart client apps talking to app servers, with business logic running on client and server. _Exactly_ the scenario CSLA .NET was designed to support.

### The Smart Client Fades

The industry took a detour into server-side web development for a long time. An environment in which CSLA become somewhat less compelling.

> As I'll discuss later however, the current rebirth of the smart client via WebAssembly/Blazor brings _all_ the amazing power and capability of CSLA back into play.

Starting around 2008 it became clear that the only really affordable and viable client experience for enterprise apps was the browser. The iPad broke the hold Windows had over the enterprise user experience, and we swiftly learned it was too expensive to build every app's UI numerous times (for Windows, iOS, Android, etc), leaving the browser as the only viable client app dev target.

A whole lot of "full stack" web software is really just server-side code in ASP.NET MVC or similar: just like the mainframe/minicomputer world from 1990. Which is fine as far as it goes, but personally I find that world pretty boring, given that is where I started my career (with server-side coding).

CSLA remains relevant in this server-side web site and services world. Web sites and services all need business logic, and CSLA provides a home for business logic. That means important parts of CSLA (such as the rules engine and MVC data binding) remain extremely valuable for both full stack and modern web development. This can include the data portal in the case that your enterprise web app uses app servers in addition to web servers.

Speaking of app servers these days always makes me think of containers.

### Containers and Kubernetes

I am convinced that the modern/future deployment scenario for most server-side code is via containers. Probably Linux containers hosted in some sort of container orchestrator like [Kubernetes](https://kubernetes.io/), or others provided by AWS and Azure.

In the 2018-19 timeframe a lot of great features were added to CSLA .NET around Kubernetes and container orchestration support. This all relies on .NET Core so all this works on Windows or Linux servers. As a result, CSLA has some really impressive features (and productivity abstractions) to support app servers running directly on servers or in containers. Personally I get quite excited about some of the features that are designed to lay over the top of something like Kubernetes to enable scaling, fault tolerance, and the ability to allocate specific workloads to specific classes of worker node.

### Smart Client: JavaScript Edition

Back on client devices, over the past few years the smart client world started to come back, with the client being written in JavaScript/TypeScript/Angular/React, forcing everyone to duplicate all their business logic on client and server (or in real life many people write it only on the client and ignore the negative ramifications). As an industry we adopted the one of the most expensive, fragile, and hard to maintain software architectures available.

For a time, I thought node.js was the way out - to at least get us back to where a single set of business logic could be directly run on client and server. And although node made some inroads, it hasn't displaced Java or .NET by a long shot.

As a result, the smart client web world has been stuck in the same hard to maintain and low-productivity architecture pioneered in the 1990's with PowerBuilder client apps talking to C++ app servers.

### Smart Client: WebAssembly Edition

Today we're on the cusp (imo) of a wonderful rebirth of smart client development thanks to [WebAssembly](https://webassembly.org/). The concept of having some type of browser-hosted assembly language is something I've advocated since around 1999.

In the 1999-2001 timeframe, before the dot-bomb, the industry was having really meaningful discussions around whether the browser should be a colorful terminal or a real runtime for code. The dot-bomb sidetracked that discussion; really derailed it for many, many years. The DHTML and then HTML5 evolution, slowly over so very many years, ultimately demonstrated that the browser can be, and probably should be, the singular client-side runtime for most apps. Angular, React, Knockout, Vue, etc. all demonstrate this, as do Slack, Teams, gmail, outlook.com, the Azure Portal, and many other rich smart-client apps that are browser-hosted.

Thanks to Mozilla (and others) we now have the ability to escape the JavaScript hegemony, and run nearly any language natively in the browser, and therefore on any device and OS. Thanks to the open source mono project we have .NET running on wasm in all those browsers. And thanks to the ASP.NET team we have the [Blazor UI framework](https://blazor.net) running on mono (and Uno, Ooui, and other UI frameworks too).

The end result is that all the amazing features of CSLA are now available to "web developers" via Blazor and other UI frameworks based on .NET WebAssembly.

> I put "web developers" in quotes, because it is true that web developers are attracted to Blazor for many reasons. But so are traditional smart client developers from the Windows Forms, WPF, iOS, Android, GTK+, and other worlds. Blazor is a smart client UI framework that happens to run in browsers - so it appeals to web developers and smart client developers equally.

### CSLA Excitement: Embrace Containers and WebAssembly

[CSLA 5 supports Blazor](https://store.lhotka.net/using-csla-5-blazor-and-webassembly), and that support continues to evolve and improve (along with Blazor itself). And it brings all the existing support for container-based server environments, plus the rules engine and all the other happiness provided by CSLA over the years.

You can probably sense my excitement!

For a few years now I've anticipated that the future runtime for most enterprise apps is WebAssembly clients talking to container-based (Linux-hosted) services. CSLA makes that environment truly sing!
