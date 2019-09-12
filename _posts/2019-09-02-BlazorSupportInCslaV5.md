---
layout: post
title: Blazor support in CSLA v5
date: 2019-09-02
featured-image: https://raw.github.com/MarimerLLC/csla/master/Support/Logos/csla%20win8_mid.png
---

I'm excited about two things in our industry right now: containers (specifically Kubernetes) and WebAssembly (specifically [Blazor](https://blazor.net) and [Uno](https://platform.uno)).

CSLA .NET version 4.11 got a lot of new and exciting features to support container and Kubernetes scenarios, and there are some more coming in CSLA version 5 as well.

But over the past few days I've been building a new `Csla.Blazor` package to provide some basic UI support when using CSLA 5 with Blazor. Specifically client-side Blazor, which is the really exciting part, though this should all work fine on server-side Blazor as well.

![wasm]({{ site.url }}/assets/Blazor-support-in-CSLA-v5/wasm-logo.png)
![CSLA .NET](https://raw.github.com/MarimerLLC/csla/master/Support/Logos/csla%20win8_mid.png)

## Application Context Manager

Part of this is a context manager that helps simplify configuration and context management within the Blazor client environment. Specifically:

1. The `HttpProxy` is set to use text-based serialization, because Blazor (wasm) doesn't currently support passing binary data via HttpClient
1. The `User` property is maintained in a `static`, just like in all other smart client scenarios

## Configuring a Blazor Client App

Blazor relies on the standard `Startup` class like ASP.NET Core for configuring a client app. CSLA supports this model via an `AddCsla` method and the fluent `CslaConfiguration` system. As a result, basic configuration looks like this:

```c#
using Csla;
using Csla.Configuration;
using Microsoft.AspNetCore.Components.Builder;
using Microsoft.Extensions.DependencyInjection;

namespace BlazorExample.Client
{
  public class Startup
  {
    public void ConfigureServices(IServiceCollection services)
    {
      services.AddCsla();
      services.AddTransient(typeof(IDataPortal<>), typeof(DataPortal<>));
      services.AddTransient(typeof(Csla.Blazor.ViewModel<>), typeof(Csla.Blazor.ViewModel<>));
    }

    public void Configure(IComponentsApplicationBuilder app)
    {
      app.AddComponent<App>("app");

      CslaConfiguration.Configure().
        ContextManager(typeof(Csla.Blazor.ApplicationContextManager)).
        DataPortal().
          DefaultProxy(typeof(Csla.DataPortalClient.HttpProxy), "/api/DataPortal");
    }
  }
}
```

Blazor defaults to providing `HttpClient` as a service, and this code adds mappings for `IDataPortal<T>` and `ViewModel<T>`. 

Notice that it also configures the app to use the `ApplicationContextManager` designed to support Blazor.

The `HttpProxy` data portal channel will gain access to the environment's `HttpClient` via dependency injection.

## Data Portal Server Needs to Use Text

As noted above, the `HttpClient` implementation currently used by .NET in wasm can't transfer binary data. As a result both client and server need to be configured to use text-based data transfer (basically Base64 encoded binary data). This is automatic on the Blazor client, but the data portal server controller needs to use text as well. Here's the controller code from the server's endpoint:

```c#
  [Route("api/[controller]")]
  [ApiController]
  public class DataPortalController : Csla.Server.Hosts.HttpPortalController
  {
    public DataPortalController()
    {
      UseTextSerialization = true;
    }
  }
```

If your data portal needs to support Blazor and non-wasm clients, you'll need two controllers, one for wasm clients and one for everything else.

## ViewModel Type

The new `Csla.Blazor.ViewModel` type provides basic support for creating a Razor Page that binds to a business domain object via the viewmodel. 

As with all the previous XAML-based viewmodel types, this one exposes the domain object via a Model property, because the CSLA-based domain object already fully supports data binding. It would be a waste of code (to write, debug, test, and maintain) to duplicate all the properties from a CSLA-based domain class in a viewmodel.

Also, like the previous XAML-based viewmodel types, this one supports some basic verbs/operations that are likely to be triggered by the UI. Specifically the create/fetch and save operations.

Finally, the viewmodel type exposes a set of metastate methods designed to allow the Razor Page to easily understand and bind to the state of the business object. For example, is the object currently saveable? What are the information/warning/error validation messages for a given property? Is a property currently running any async business rules?

You can use all these metastate values to create a rich UI, much like in XAML, with no code. The Blazor data binding model, combined with the new `ViewModel` typically provide everything necessary.

To use this type in a page, make sure to add it to the services in `Startup` as shown earlier. Then inject it into the page:

```razor
@inject Csla.Blazor.ViewModel<PersonEdit> vm
```

And in the `@code` block call the `RefreshAsync` method:

```c#
@code {
  protected override async Task OnInitializedAsync()
  {
    await vm.RefreshAsync();
  }
}
```

The `RefreshAsync` method has various default behaviors around how it knows to fetch or create an instance of the business domain type:

1. If the domain type is read-only it always does a fetch
1. If the domain type is editable and no criteria parameters are provided it does a create
1. If the domain type is editable and criteria is provided it does a fetch

You can override this behavior, but these defaults work well in many cases.

## BlazorExample Sample

You can look at the [Samples/BlazorExample](https://github.com/MarimerLLC/csla/tree/master/Samples/BlazorExample) sample to see how this comes together. That sample is the basic Blazor start template, plus the ability to add/edit person objects, and get a list of people in the "database" (a mock in-memory data store).
