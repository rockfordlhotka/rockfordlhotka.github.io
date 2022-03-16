---
layout: post
title: ASP.NET Core and Blazor Identity and State
date: 2022-03-16T00:00:00.0000000-05:00
categories: []
tags: []
published: true
permalink: 
featuredImageUrl: 
---
Over the past several weeks I've been wrestling with the way in which ASP.NET Core (aspnetcore) and server-side Blazor manage things like context, identity, and state. This post won't cover _everything_, but I want to at least document what I've learned (with the help of some great people from the #cslanet community).

aspnetcore supports a number of different app and service types, and allows these to run in the same web site on the server. This is pretty nice, but is the root of the complexity I'm discussing in this post. The reason, is that not all these app and service types handle things like context, identity, and state the same way, and they can conflict with each other when running in the same web site (think: Visual Studio aspnetcore project).

For example, you might create a project to host:

1. MVC pages and/or Razor Pages
1. Server-side Blazor
1. Hosted services

Each of these has different rules about when and if `HttpContext` is valid. Notice I use the word _valid_, because in some cases you won't have access to `HttpContext`, other times you will have access to a _valid_ `HttpContext`, and other times you'll have access to an _invalid_ `HttpContext` instance.

## MVC and Razor Pages

When building pages, controllers, and other assets for MVC or Razor Pages, you will have access to a valid `HttpContext`. You can rely on this object to provide you things like the current user identity, and you can maintain state in `HttpContext` if desired. This `HttpContext` instance exists for the duration of each request from the client, and the instance is consistently available to you for the duration of the request.

This is the oldest model, and so a lot of us have written substantial codebases over the years that rely on `HttpContext` to access context, identity, and per-request state.

## Server-side Blazor

When building a server-side Blazor (SSB) app, you _must not_ use `HttpContext`. You can actually get an instance of `HttpContext`, but you must consider it _invalid_ and off limits.

### Determining if HttpContext is Invalid

If you are a UI author all you need to know is that you can't interact with `HttpContext`. Sure, you can inject `IHttpContextAccessor` and get an `HttpContext` instance - but it is invalid!

If you are a library author, where you are building code that may be used by MVC, Razor Pages, SSB, and other aspnetcore app types, then things get complex. You can easily get (or not get) an instance of `HttpContext` by injecting `IHttpContextAccessor`, but there's no obvious way to know whether that `HttpContext` is actually valid. Arg!

> I think this is a bug in aspnetcore. If the instance is invalid, then they shouldn't be giving us that instance in the first place.

The way to know the `HttpContext` is invalid is to create a scoped DI service so you can tell whether there's an active SignalR _circuit_ in use. Only SSB apps have a circuit, so if you get an `HttpContext` and also find that there's a circuit, then you know the `HttpContext` is invalid.

Here's the CSLA .NET [service to find whether there's a circuit](https://github.com/MarimerLLC/csla/blob/main/Source/Csla.AspNetCore/Blazor/ActiveCircuitHandler.cs) (storing the value in another scoped service named [ActiveCircuitState](https://github.com/MarimerLLC/csla/blob/main/Source/Csla.AspNetCore/Blazor/ActiveCircuitState.cs)):

```c#
using System.Threading;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.Server.Circuits;

namespace Csla.AspNetCore.Blazor
{
  /// <summary>
  /// Circuit handler indicating if code in server-side Blazor.
  /// </summary>
  public class ActiveCircuitHandler : CircuitHandler
  {
    /// <summary>
    /// Creates an instance of the type.
    /// </summary>
    /// <param name="activeCircuitState"></param>
    public ActiveCircuitHandler(ActiveCircuitState activeCircuitState)
    {
      ActiveCircuitState = activeCircuitState;
    }

    private ActiveCircuitState ActiveCircuitState { get; set; }

    /// <summary>
    /// Handler for the OnCircuitOpenedAsync call
    /// </summary>
    /// <param name="circuit">The circuit in which we are running</param>
    /// <param name="cancellationToken">The cancellation token provided by the runtime</param>
    public override Task OnCircuitOpenedAsync(Circuit circuit, CancellationToken cancellationToken)
    {
      ActiveCircuitState.CircuitExists = true;
      return base.OnCircuitOpenedAsync(circuit, cancellationToken);
    }
  }
}


namespace Csla.AspNetCore.Blazor
{
  /// <summary>
  /// Provides access to server-side Blazor
  /// circuit information required by CSLA .NET.
  /// </summary>
  public class ActiveCircuitState
  {
    /// <summary>
    /// Gets a value indicating whether the current scope
    /// has a circuit (is running in server-side Blazor).
    /// </summary>
    public bool CircuitExists { get; set; } = false;
  }
}
```

During app startup these are registered as scoped services:

```c#
services.AddScoped<ActiveCircuitHandler>();
services.AddScoped<ActiveCircuitState>();
```

Then, in your library code, you can do something like this (relying on the constructor parameters being injected):

```c#
public class MyLibraryComponent
{
    public MyLibraryComponent(IHttpContextAccessor httpContextAccessor, ActiveCircuitState activeCircuitState)
    {
        var httpContext = httpContextAccessor.HttpContext;
        var httpContextValid = httpContext != null && !activeCircuitState.CircuitExists;
        // now you know whether you have a valid HttpContext
    }
}
```

This allows you to know whether you have an `HttpContext` at all, and whether it is valid. From here, your library code can decide to get state and identity information from the `HttpContext` instance, or manage it in some other way.

### App-Level State

Server-side Blazor relies on dependency injection (DI) scope to maintain context and state for the app. Any state that you want to maintain for the overall SSB app needs to be in a scoped DI service that you create. This is pretty easy. Define a DTO type:

```c#
public class MyStateService
{
    public string SomeState { get; set; }
}
```

And then during app startup, declare this as a scoped service:

```c#
services.AddScoped<MyStateService>();
```

Now any page or service in your SSB app can inject `MyStateService` to access that app-level state.

> This is a very nice model, and I understand why they chose it. Unfortunately, it is _different_ from everything else in aspnetcore or any other past UI framework from Microsoft, and so it can take a bit to wrap your mind (and any existing code) around this new model.

### User Identity

SSB uses some aspnetcore types to manage the current user identity. These types come from aspnetcore, but many of us never really used them, because we just relied on `HttpContext`. Because `HttpContext` isn't valid in SSB apps, we now _must_ interact directly with the aspnetcore identity model.

Specifically, use the `AuthenticationStateProvider` service to be notified when the current user identity changes. You shouldn't need to create this yourself, as most identity frameworks provide this service for you.

> If you are implementing your own custom authentication, you'll probably need to create a subclass of `AuthenticationStateProvider`.

This service has a `GetAuthenticationStateAsync` method you can use to access an `AuthenticationState` instance. This type has a `User` property that returns the `ClaimsPrincipal` for the current user.

To get the current user, inject `AuthenticationStateProvider`, call its `GetAuthenticationStateAsync` method, and access the `User` property of the result.

There is a lot more to the authentication API provided by aspnetcore; more than I am covering in this post, but at least this should provide you with the basic rules: don't use `HttpContext`, and instead rely on the `AuthenticationStateProvider` for your authentication model.

## Hosted Service

aspnetcore allows you to create a hosted service by implementing the `IHostedService` interface. A hosted service does not have access to `HttpContext` at all, so you don't need to worry about it being valid or not.

A hosted service runs in its own DI scope, and so you can use scoped services (like SSB) to maintain any context you require.

A hosted service typically runs based on a timer or some other trigger, and so it doesn't really have a user.

## Conclusion

I and a couple other members of the #cslanet community have been fighting with this for quite some time now, trying to get the CSLA .NET framework to support these scenarios all in the same web site, without breaking non-web scenarios like WPF, MAUI, etc.

It has been quite a process, and a lot of learning. I don't know if this post will make things more clear or not, but at least it is an attempt to capture some of the high level learnings.

## References

* https://docs.microsoft.com/en-us/aspnet/core/fundamentals/http-context?view=aspnetcore-6.0#blazor-and-shared-state
* https://github.com/MarimerLLC/csla/discussions/2687
* https://github.com/MarimerLLC/csla/issues/2798
* https://stackoverflow.com/questions/60264657/get-current-user-in-a-blazor-component
* https://docs.microsoft.com/en-us/aspnet/core/blazor/security/server/threat-mitigation?view=aspnetcore-6.0#blazor-and-shared-state
* https://gist.github.com/SteveSandersonMS/ba16f6bb6934842d78c89ab5314f4b56
* https://github.com/dotnet/aspnetcore/issues/28684
* https://gist.github.com/SteveSandersonMS/175a08dcdccb384a52ba760122cd2eda#implementing-a-custom-authenticationstateprovider