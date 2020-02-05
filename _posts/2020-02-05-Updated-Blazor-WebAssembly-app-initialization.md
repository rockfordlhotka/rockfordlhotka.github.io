---
title: Updated Blazor WebAssembly app initialization
weblogName: Rockford Lhotka
postDate: 2020-02-05
---

In the latest preview of client-side Blazor (3.2.0-preview1.20073.1) the project template has been changed to no longer use a `Startup.cs` approach, but rather to put all initialization in the `Main` method of `Program.cs`.

Also, the app builder type has been changed to `WebAssemblyHostBuilder`.

```c#
  public class Program
  {
    public static async Task Main(string[] args)
    {
      var builder = WebAssemblyHostBuilder.CreateDefault(args);
      builder.RootComponents.Add<App>("app");

      await builder.Build().RunAsync();
    }
  }
```

This affects CSLA .NET, because the previous app initialization support was based around the `Startup.cs` model, similar to ASP.NET Core server apps.

The [updated approach](https://github.com/MarimerLLC/csla/issues/1476) is actually quite nice, in that all CSLA configuration is handled by a single call to `UseCsla`:

```c#
  public class Program
  {
    public static async Task Main(string[] args)
    {
      var builder = WebAssemblyHostBuilder.CreateDefault(args);
      builder.RootComponents.Add<App>("app");

      builder.UseCsla();

      await builder.Build().RunAsync();
    }
  }
```

This new `UseCsla` method adds all necessary services to the IoC container, and does all required client-side configuration of CSLA to work within Blazor WebAssembly.

An overload supports additional configuration. For example:

```c#
  builder.UseCsla(
    (config) => config
      .DataPortal()
        .DefaultProxy(typeof(Csla.DataPortalClient.HttpProxy), "https://myserver/api/dataportal"));
```

I rather like the fact that Microsoft has changed the template so it doesn't seem so "server-like", given that Blazor WebAssembly is a smart client app, not a server app. This app initialization scheme is more similar to what you'd see in WPF or UWP, and that's fine with me.
