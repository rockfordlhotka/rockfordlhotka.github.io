---
layout: post
title: CSLA 5 data portal channel using gRPC
date: 2019-08-21T14:37:26.2177100-05:00
---

The new .NET Core 3 release includes support for the gRPC protocol. This is an efficient binary protocol for making network calls, and so is something that CSLA .NET should obviously support.

CSLA already has an extensible channel-based model for network communication via the data portal. Over the years there have been numerous channels, including:

* .NET Remoting (obsolete)
* asmx services (obsolete)
* WCF (of limited value in modern .NET)
* Http

I'm sure there have been others as well. The current recommended channel is via Http (using the `HttpProxy` type), as it best supports performance, routing, and various other features native to HTTP and to the data portal channel implementation.

CSLA .NET version 5.0.0 will include a new gRPC channel. Like all the other channels, this is a drop-in replacement for your existing channel. 

> ⚠ This requires CSLA .NET version 5.0.0-R19082107 or higher

## Client Configuration

On the client it requires a new NuGet package reference and a configuration change.

1. Reference the new `Csla.Channels.Grpc` NuGet package
1. On app startup, configure the data portal as shown here:

```c#
      CslaConfiguration.Configure().
        DataPortal().DefaultProxy(typeof(Csla.Channels.Grpc.GrpcProxy), "https://localhost:5001");
```

This configures the data portal to use the new `GrpcProxy` and provides the URL to the service endpoint. Obviously you need to provide a valid URL.

## Server Configuration

On the server it requires a new NuGet package reference and a bit of code in `Startup.cs` to set up the service endpoint. 

> ⚠ This requires an ASP.NET Core 3.0 project.

1. Reference the new `Csla.Channels.Grpc` NuGet package
1. In the `ConfigureServices` method you must configure gRPC: `services.AddGrpc();`
1. In the `Configure` method add the data portal endpoint:

```c#
      app.UseRouting();

      app.UseEndpoints(endpoints =>
      {
        endpoints.MapGrpcService<Csla.Channels.Grpc.GrpcPortal>();
      });
```

## General Notes

As usual, both client and server need to reference the same business library assembly, which should be a .NET Standard 2.0 library that references the `Csla` NuGet package. This assembly contains implementations of all your business domain classes based on the CSLA .NET base classes.

The gRPC data portal channel uses `MobileFormatter` to serialize and deserialize all object graphs, and so your business classes need to use modern CSLA coding conventions so they work with that serializer.

All the version and routing features added to the [Http data portal channel in CSLA version 4.9.0](http://www.lhotka.net/weblog/CSLANETVersion49NewFeatures.aspx) are also supported in this new gRPC channel, allowing it to take full advantage of container orchestration environments such as Kubernetes.

Also, as with the Http channel, the `GrpcProxy` and `GrpcPortal` types have `virtual` methods you can optionally override to implement compression on the data stream, and (on the client) to support advanced configuration scenarios when creating the underlying `HttpClient` and gRPC client objects.
