---
layout: post
title: CSLA 6 First Prerelease
date: 2021-11-29T00:00:00.0000000-06:00
categories: []
tags: []
published: true
permalink: 
featuredImageUrl: 
---
The first prerelease of CSLA .NET version 6.0.0 is now in NuGet.

[What is CSLA .NET?](https://github.com/MarimerLLC/csla/blob/main/docs/What-is-CSLA-.NET.md)

We follow semantic versioning, so CSLA 6 has a lot of breaking changes from CSLA 5. Some due to keeping up with .NET, and others due to requests from the community.

A complete [list of breaking changes](https://github.com/MarimerLLC/csla/issues?q=is%3Aissue+is%3Aclosed+label%3A%22flag%2Fbreaking+change%22+project%3AMarimerLLC%2Fcsla%2F11) in the prerelease can be found in GitHub. I'll highlight some of the ones with the most immediate impact in this post.

## Platform Support

A number of older .NET versions and other technologies have been dropped in CSLA 6. It is easier to list what _is_ supported.

CSLA 6 supports the following "flavors" of .NET:

* .NET Framework 4.6.2
* .NET Framework 4.7.2
* .NET Framework 4.8
* .NET Core 3.1 (LTS)
* .NET Standard 2.0
* .NET 5
* .NET 6 (LTS)

CSLA 6 supports the following deployment targets:

* Windows
* Mac
* Linux
* iOS
* Android

Basically, if .NET 6 runs on your hardware, CSLA 6 should also run on your hardware.

CSLA 6 supports the following UI frameworks:

* Blazor WebAssembly
* Server-side Blazor
* Xamarin.Forms
* Xamarin Native (iOS and Android)
* AspNetCore Razor Pages
* ASP.NET MVC
* ASP.NET Web Forms
* UWP
* WPF
* Windows Forms

As Microsoft completes their work on MAUI, we will implement support for that UI framework as well.

I'm sometimes asked about other UI frameworks, such as Uno.Platform and Avalon. I'm open to people contributing helper projects for those UI frameworks along the line of the comparable helper projects for the UI frameworks listed here. CSLA is open source, and such contributions are what make the community and project strong.

## Dependency Injection Required

CSLA 5 _supported_ dependency injection (DI), but generally didn't require it. CSLA 6 takes a hard dependency on DI (pun intended), and so DI is _required_ to use CSLA 6.

This is true for Console, WinForms, WPF, ASP.NET, aspnetcore, Blazor, and Xamarin/MAUI apps.

For example, here's the DI configuration in `Program.cs` for a .NET 6 Windows Forms app:

```c#
  internal static class Program
  {
    private static IHost Host { get; set; }
    /// <summary>
    ///  The main entry point for the application.
    /// </summary>
    [STAThread]
    static void Main()
    {
      ApplicationConfiguration.Initialize();

      Host = new HostBuilder()
        .ConfigureServices((hostContext, services) => services
          // register window and page types here
          .AddSingleton<MainForm>()
          .AddTransient<Pages.HomePage>()
          .AddTransient<Pages.PersonEditPage>()
          .AddTransient<Pages.PersonListPage>()

          // register other services here
          .AddTransient<DataAccess.IPersonDal, DataAccess.PersonDal>()
          .AddCsla(options => options.AddWindowsForms())
      ).Build();

      var form = Host.Services.GetService<MainForm>();
      Application.Run(form);
    }
  }
```

Note the use of a `HostBuilder` and `ConfigureServices` for configuring the services provided by DI. You don't _technically_ need to use DI to create your forms and user controls, but you _must_ call the `AddCsla` method to add a variety of services that are _required_ by CSLA itself.

Similarly, in the `Program.cs` of an aspnetcore server app:

```c#
builder.Services.AddCsla();
```

In an aspnetcore app there'll be lots of other services configured just to make ASP.NET itself work, and you _must_ include the `AddCsla` method to register the services required by CSLA.

Because these services are registered, you can now rely on them to be injected when you need them. Back to the Windows Forms example, because I'm using DI to create the form and user control objects, I can use DI to get the services needed in each object.

For example, here's the constructor for the user control that edits a `PersonEdit` instance:

```c#
    private IDataPortal<PersonEdit> _portal;

    public PersonEditPage(IDataPortal<PersonEdit> portal)
    {
      _portal = portal;
      InitializeComponent();
    }
```

Notice that the constructor requires a parameter of type `IDataPortal<T>`. This is how you get a data portal instance that allows you to create/fetch/update/delete an instance of type `T`.

In the `Load` event handler for the form, this data portal instance can be used:

```c#
personEdit = await _portal.FetchAsync(personInfo.Id);
```

You can then set the form to use the resulting `personEdit` as its binding source like normal.

The same concept applies to any app, including WPF, Blazor, etc.

## ApplicationContext

A number of important changes have been made to the `Csla.ApplicationContext` type and how it is used.

### ApplicationContext Injection

Prior to CSLA 6, the `ApplicationContext` type was static and so could be used directly from any code. In CSLA 6 the `ApplicationContext` is an instance and is only available from DI.

> This change was necessary to support server-side Blazor, where the user identity and other per-session state is only available via DI.

Fortunately, `ApplicationContext` is one of the services registered by the `AddCsla` method during configuration. This means that you can require this service in your constructors and get access to the current context instance.

The primary downside is that _you can no longer use static factory methods_ to invoke the data portal. This is because it is impossible to inject a service into a static method, and the data portal requires `ApplicationContext`. In fact, the client-side data portal itself is no longer static, and is an instance type so that it can require injection of `ApplicationContext`.

### ApplicationContext.Principal Property

Microsoft has been moving away from supporting the `IPrincipal` type, and instead increasingly requiring the use of `ClaimsPrincipal` instances.

Following their lead, there is now an `ApplicationContext.Principal` property that returns a `ClaimsPrincipal`, and there are increasing scenarios within CSLA where only the `ClaimsPrincipal` type is supported.

The `ApplicationContext.User` property still returns an `IPrincipal` for backward compatibility, but I find that I rarely use anything other than a `ClaimsPrincipal` any longer.

## Data Portal

On to the data portal.

### Client-side DataPortal Type is not Static

In CSLA 6 the client-side `DataPortal` type is now an instance type.

In the past you may have called the data portal like this:

```c#
var obj = await DataPortal.FetchAsync<PersonEdit>(42);
```

The `DataPortal` type is no longer static, and so that calling style is no longer available.

Instead, you must rely on DI to get an instance of the data portal:

```c#
public class PersonEditPage
{
  private IDataPortal<PersonEdit> _portal;
  
  public PersonEditPage(IDataPortal<PersonEdit> portal)
  {
      _portal = portal;
  }
  
  private async void LoadPage()
  {
      this.DataContext = await _portal.FetchAsync(42);
  }
}
```

This is the recommended approach for interacting with data portal instances based on your business layer types. 

There are various alternatives using DI, and perhaps I'll cover them in a different blog post.

### Base DataPortal_XYZ Methods Removed

Starting in CSLA 5 we recommended that you stop using the old `DataPortal_XYZ` method naming scheme, and switch to using the new data portal operation attributes. For example:

```c#
  [Fetch]
  private void Fetch(int id, [Inject] IPersonDal dal)
  {
      var data = dal.Fetch(id);
      // ...
  }
```

In CSLA 6 the base classes in CSLA itself no longer implement any `DataPortal_XYZ` methods. In many cases you probably relied on overriding those methods in your business classes, and at the very least you will need to remove those `override` statements in your code.

You should be aware that, although we do find the old-style `DataPortal_XYZ` methods, that is a fallback after we've looked for (and failed to find) the new data portal operation attributes. In other words, you'll get marginally better performance if you use the attributes, because that's the preferred code path used by CSLA.

### Client-side Configuration

Second, configuring the data portal is now somewhat different, because configuration is managed via DI.

For example, this is the code in a `Program.cs` for a Blazor WebAssembly client app:

```c#
builder.Services.AddCsla(o => o
  .WithBlazorWebAssembly()
  .DataPortal()
    .UseHttpProxy(options => 
      options.DataPortalUrl = "/api/DataPortal"));
```

The `UseHttpProxy` method registers the types necessary for the HttpProxy data portal channel to work, including providing the server endpoint URL.

A similar pattern is used for the GrpcProxy and RabbitMqProxy types.

All client apps will follow this model to configure the data portal client to use a specific proxy type, and provide that type with necessary options like the server endpoint URL.

### Http DataPortal Controller

When setting up an endpoint for the HttpProxy channel, you must create a controller in your aspnetcore app. This controller, like in CSLA 5, is a subclass of `HttpPortalController`, but now it must include a constructor:

```c#
  [Route("api/[controller]")]
  [ApiController]
  public class DataPortalController : Csla.Server.Hosts.HttpPortalController
  {
    public DataPortalController(Csla.ApplicationContext applicationContext)
      : base(applicationContext)
    { }

    [HttpGet]
    public string Get()
    {
      return "Running";
    }
  }
```

The constructor is required because the base type needs to have an instance of `Csla.ApplicationContext` injected to operate correctly.

### LocalProxy and DI Scope

By default, CSLA uses something called the LocalProxy channel, so the "server-side" data portal code actually runs on your client device. This is not new, and this is the behavior you get if you don't specifically call something like the `UseHttpProxy` method during configuration.

The LocalProxy channel will automatically create a DI scope for each request it handles. This means that your "server-side" code can use DI to get scoped services such as database connection, transaction, etc. When the data portal operation completes, the DI scope is disposed, automatically disposing of all services used within that scope.

This is the behavior you would expect, and provides isolation between data portal operations, while enabling the sharing and reuse of connections and transactions _during each operation_.

### No More WCF or Remoting

The data portal _no longer supports WCF_. This is because WCF didn't make the leap from .NET Framework to modern .NET. 

For much the same reason, the Remoting data portal channel has also been removed.

As in CSLA 5, if you need to use a network transport technology that isn't supported, you can create your own data portal channel. The [Grpc channel](https://github.com/MarimerLLC/csla/tree/main/Source/Csla.Channels.Grpc) and [RabbitMq channel](https://github.com/MarimerLLC/csla/tree/main/Source/Csla.Channels.RabbitMq) projects are good examples.

## Data Access

A number of breaking changes have been made to the helper packages CSLA provides for use in building your data access layer.

### Context Manager Types Removed

The `Csla.Data` namespace has provided numerous "helper types" for implementing your data access code. This is still true, but because CSLA now requires DI, some old types are no longer necessary or valuable.

The following have been removed:

* DbContextManager
* ObjectContextManager

And, frankly, any similar context manager types. They are all removed, because you can (and should) use DI to manage database connection and transaction types.

Other types, such as `SafeDataReader` are still available.

### EF 4 and EF 5 Removed

We no longer provide helpers for Entity Framework 4 or Entity Framework 5. There are still helpers for modern versions of Entity Framework.

## Serialization

There are some major changes to serialization in CSLA 6.

### BinaryFormatter and NetDataContractSerializer Not Supported

We have removed support for `BinaryFormatter` and `NetDataContractSerializer` (NDCS). Microsoft has deprecated these types, and during the development of CSLA 6 we encountered scenarios where continuing to support those types became a blocker.

### MobileFormatter Enhancements

The `MobileFormatter` is the default (and only) serializer included in CSLA 6. It has numerous behind-the-scenes enhancements that support various features of CSLA 6 and modern .NET.

### Custom Formatter Support

We did work to enable you to create your own serializer if you desire. Your serializer must exactly match the semantic behaviors of `MobileFormatter`, so it is not possible to just "plug in" something like Protobuf. But you probably could _wrap_ something like Protobuf to create a custom serializer.

This is clearly a super-advanced concept. My preference would be that anyone wanting something better than `MobileFormatter` work with us to enhance `MobileFormatter` itself. However, if you want to undertake the serious effort of building an alternative, please let me know and I'll do what I can to smooth your path.