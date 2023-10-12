---
layout: post
title: Blazor 8 State Management
date: 2023-10-12T00:00:00.0000000-05:00
categories: []
tags: []
published: true
permalink: 
featuredImageUrl: 
---
A few months ago I filed an issue for discussion in GitHub regarding managing per-user state in Blazor in .NET 8.

https://github.com/dotnet/aspnetcore/issues/47796

I suspect this state management issue is going to be a major hurdle for Blazor developers who want to create rich Blazor apps and also use the new .NET 8 capabilities. This blog post won't provide a _solution_ to the problem, but now that RC2 is available I thought it was a good time to at least document the "normal" behaviors you will have to deal with.

## .NET 6 and 7 Behavior

First, it is important to understand that in .NET 6 and 7 there were really two models for Blazor: server and wasm (WebAssembly). In both cases, a dependency injection (DI) scope was created by Blazor that would exist for the lifetime of our app.

In Blazor server, this DI scope is typically one of many that exist within aspnetcore on the web server. Each scope is per-user, and so any scope-level DI services are therefore also per-user. Singleton services are shared by all users on that web server.

In Blazor wasm, this DI scope is the only scope running in the browser tab for the app. The app is isolated within the tab and so the root DI provider is the only provider. As a result, any scoped services are per-user, as are any singleton services.

In neither case could you use HttpContext. In .NET 6 or 7, Blazor and HttpContext did not interoperate.

Prior to .NET 8, is that you could always count on using scoped services to maintain things like the current user identity or any other per-user scope that should exist for the lifetime of the app.

## .NET 8 Behavior

.NET opens up some exciting new rendering options for Blazor, which have the potential to change the way per-user state is managed.

Chris Sainty has a good blog post about the [Blazor 8 rendering models](https://chrissainty.com/blazor-in-dotnet-8-full-stack-web-ui/).

> It is important to understand that the Blazor server and wasm deployment and rendering options _still exist_ in .NET 8, and so you can choose to continue to use the same behaviors we've had in .NET 6 and 7.

If you choose to use the new, more dynamic, rendering options available in Blazor 8, you will get this new state behavior.

Specifically, what happens is that there is no longer any concept of a DI scope that exists over the lifetime of the Blazor app. Instead, there are multiple models:

| Model | Definition | Consequence | Long-lived scope? | HttpContext? |
| --- | --- | --- | --- | --- |
| Server-rendered pages | A Blazor page that is rendered on the server, sending content to the browser without establishing any SignalR or wasm connection to the browser | A DI scope exists for the lifetime of the single page render process, and this scope is gone as soon as the page rendering is complete. HttpContext is available to your code | No | Yes |
| Streamed pages | A Blazor page that is rendered on the server, sending content to the browser as a stream without establishing any SignalR or wasm connection to the browser | A DI scope exists for the lifetime of the single page render process, and this scope is gone as soon as the page rendering is complete. HttpContext is available to your code | No | Yes |
| Blazor server page | A Blazor server page that establishes a SignalR connection to the browser | A DI scope exists on the server that corresponds to the SignalR connection, and this scope goes away as soon as the connection is broken. If, at any point, no Blazor server pages/components are running, the SignalR connection and DI scope are gone. HttpContext is not available to your code | Maybe | No |
| Blazor wasm page | A Blazor wasm page that runs in the browser | A DI scope exists on the client for the lifetime of the single page. This scope is gone when the user nagivates to any page that is _not also_ a Blazor wasm page. HttpContext is not available to your code | Maybe | No |
| Blazor server island | A Blazor server component within another page that establishes a SignalR connection to the browser | A DI scope exists on the server that corresponds to the SignalR connection, and this scope goes away as soon as the connection is broken. If, at any point, no Blazor server pages/components are running, the SignalR connection and DI scope are gone. HttpContext is not available to your code | No | No |
| Blazor wasm island | A Blazor wasm component that runs in the browser within another page | A DI scope exists on the client for the lifetime of the wasm island. This scope is gone when the Blazor wasm island is no longer active in the browser. HttpContext is not available to your code | No | No |

As you can see, there is no longer any consistent way to store or access per-user state over the lifetime of a Blazor app when using the new rendering models in .NET 8. I want to remind you that you _can still use_ the Blazor server and Blazor wasm models like in .NET 6 or 7, where there is still a consistent state model based on DI scope.

Keep in mind that per-user state includes the current user identity. The "official" ways to access the current user identity are now (in Blazor 8) split between HttpContext and the scoped AuthenticationStateProvider service. Because neither technique are globally available, there doesn't appear to be any standard out-of-the-box way to consistently access the current user identity across all Blazor pages, components, or code.

## Options for Per-User State

There are no official options for consistently maintaining per-user state when using the new render modes in Blazor 8.

The simplest thing to do, if you are building a Blazor app, is to continue to use the Blazor server or Blazor wasm rendering/deployment modes from .NET 6 and 7.

I _suspect_ that maybe there's a way to use something like a REDIS cache to offload all per-user state to some other server process, and to then have all scoped state-related services delegate to that cache to retrieve and update any state data. Obviously this incurs complexity and latency, and may be quite complex for any Blazor wasm pages/islands.

## Conclusion

I wanted to get this information online because I am sure most Blazor developers will run into the per-user state issue almost immediately upon trying the new Blazor 8 rendering models.

I also expect that solutions will be developed to this problem. Nopefully Microsoft will implement some official solution in a future version of .NET, and in the meantime I look forward to open-source or even commercial packages that provide ways to maintain per-user state and user identity across Blazor server rendered, Blazor server, and Blazor wasm pages and components.

Certainly this is something I'm researching as part of [CSLA .NET](https://cslanet.com), though I don't have a solution as yet.
