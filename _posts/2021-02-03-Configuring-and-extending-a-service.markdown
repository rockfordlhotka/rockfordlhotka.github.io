---
layout: post
title: Configuring and extending a service
date: 2021-02-03T00:19:00.0000000-06:00
categories: []
tags: []
published: true
permalink: 
image: 
---
CSLA has been around for over 23 years now, with the first [CSLA .NET](https://cslanet.com) version released concurrently with .NET Framework 1.0. Over the years it has changed to take advantage of, or allow developers to take advantage of, new and exciting features of .NET, such as generics, async/await, dependency injection, and more.

We are working on CSLA 6, the next major release of #cslanet, and this being a major release, some amount of breaking changes are acceptable. Which is good, because CSLA 6 will move from some basic support for dependency injection (DI), to leveraging it in important ways that enable some very cool scenarios.

One of these scenarios is extensibility. CSLA has relied on a _provider pattern_ model since the first version in .NET. As your app starts up you can choose to provide a whole host of providers to customize or enhance the way CSLA works in many ways. To do this, you provide either types or instances of the providers, and CSLA uses them instead of the default providers that ship in the framework.

That model works fine, but isn't "modern". The modern approach is to use DI, where you register the types or instances to the DI subsystem as the app starts up, and those registered types/instances are provided where needed from the DI container.

The end result is the same: CSLA is configurable because you are able to give it a set of optional alternate implementations to various behaviors. In fact, the actual provider implementations don't need to change much to become "services" (in DI parlance), the biggest change is during app startup and configuration. And, of course, inside CSLA in terms of how CSLA types expect to _find and access_ your provider services.

Although the change is subtle, it has substantial implications in terms of how CSLA itself is tested, and how you can test your code that uses CSLA. I believe these benefits make it worth the effort to switch from the provider model to a DI services model.

OK, that's all actually preamble to the real question: what pattern should we use for providing complex extensibility options to a CSLA type?

As an example, `HttpProxy` (today) has a number of virtual methods, so you can create a subclass, override those methods, and customize how the `HttpClient` or `WebClient` are created, insert headers into requests, and so forth.

_Perhaps_ a new `HttpProxy` should allow you to provide a set of delegates that are invoked instead? So you don't have to create a subclass, but rather you provide a set of delegates?

## Introducing the Example

To simplify this, I have an `ExtensibleExample` type that relies on three points of customization:

```c#
  public class ExtensibleExample
  {
    public void DoSomeWork()
    {
      var someState = new State { SomeState = 0 };
      Console.WriteLine($"Start '{someState}'");
      Point1?.Invoke(someState);
      Console.WriteLine($"After point1 '{someState}'");
      Point2?.Invoke(someState);
      Console.WriteLine($"After point2 '{someState}'");
      Point3?.Invoke(someState);
      Console.WriteLine($"After point3 '{someState}'");
    }

    public delegate void ExtensibilityPoint1(State state);
    public delegate void ExtensibilityPoint2(State state);
    public delegate void ExtensibilityPoint3(State state);
    public ExtensibilityPoint1 Point1;
    public ExtensibilityPoint2 Point2;
    public ExtensibilityPoint3 Point3;
  }
```

One way to use this type is like this:

```c#
      var worker = new ExtensibleExample(null);
      worker.Point1 = (s) => s.SomeState = 1;
      worker.Point2 = (s) => s.SomeState = 2;
      worker.Point3 = (s) => s.SomeState = 3;
      worker.DoSomeWork();
```

This doesn't use DI, and so is kind of awkward, but it provides a starting point to understand how the concept works.

## Introducing Delegate-Based DI

One way to introduce DI would be to require the delegates be injected via a constructor in the `ExtensibleExample` class:

```c#
    public ExtensibleExample(ExtensibilityPoint1 point1, ExtensibilityPoint2 point2, ExtensibilityPoint3 point3)
    {
      Point1 = point1;
      Point2 = point2;
      Point3 = point3;
    }
```

The idea of registering delegates for use by DI comes from Christian Findlay in [this blog post](https://christianfindlay.com/2020/05/15/c-delegates-with-ioc-containers-and-dependency-injection/). 

Then, in the same class, the point fields can be changed:

```c#
    private readonly ExtensibilityPoint1 Point1;
    private readonly ExtensibilityPoint2 Point2;
    private readonly ExtensibilityPoint3 Point3;
```

Next, during app startup, the services need to be registered:

```c#
      var services = new ServiceCollection();
      services.AddSingleton<ExtensibleExample.ExtensibilityPoint1>((s) => s.SomeState = 1);
      services.AddSingleton<ExtensibleExample.ExtensibilityPoint2>((s) => s.SomeState = 2);
      services.AddSingleton<ExtensibleExample.ExtensibilityPoint3>((s) => s.SomeState = 3);
      services.AddTransient<ExtensibleExample>();
      services.AddTransient<MyViewModel>();
      var provider = services.BuildServiceProvider();
```

Now, anywhere in our code, DI can be used to get an instance of the worker, and then the worker can be invoked:

```c#
  public class MyViewModel
  {
    public MyViewModel(ExtensibleExample worker)
    {
      worker.DoSomeWork();
    }
  }
```

This relies on DI creating the `MyViewModel` instance, which requires an `ExtensibleExample` instance, which requires the three delegate instances.

> Notice that, as per Christian Findlay's blog post, I'm using strongly typed delegates, not `Action<>` or `Func<>`. This is because the DI subsystem works on discrete types, so relying on something non-specific like `Action<>` _could work_ but would be very limiting.

So we've consolidated a bunch of code from the point of each call into app startup. Assuming the `ExtensibleExample` type is used multiple times around our codebase this eliminates redundant code.

I'm not real keen on this solution though, because it seems fragile. Maybe I only want to override one extensibility point, leaving the other two to use some default behavior? In fact, how do I even provide default behavior?

Also, suppose I add another extensiblity point to the `ExtensibleExample` type? I'd have to add it to the constructor, and then to the app startup code.

There are a couple possible solutions. One is to have an extension method that provides default behaviors as necessary, and the other is to use an "options" type.

### Extension Method-Based Configuration

If you have done any work with ASP.NET Core or Blazor, you have seen helper methods provided by Microsoft such as `AddMvc`, that add required service registrations to DI so MVC works properly. Here's an extension method that does the same thing for the `ExtensibleExample` type:

```c#
  public static class ExtensibilityExampleExtensions
  {
    public static IServiceCollection AddExtensiblityExample(this IServiceCollection services)
    {
      services.TryAddSingleton<ExtensibleExample.ExtensibilityPoint1>((s) => s.SomeState = 0);
      services.TryAddSingleton<ExtensibleExample.ExtensibilityPoint2>((s) => s.SomeState = 0);
      services.TryAddSingleton<ExtensibleExample.ExtensibilityPoint3>((s) => s.SomeState = 0);
      services.TryAddTransient<ExtensibleExample>();
      return services;
    }
  }
```

I am using `TryAddSingleton` and `TryAddTransient` in the extension method, because this way if I have already registered something for the type the extension method will not override or replace that registration. The regular `AddSingleton` and `AddTransient` methods will replace any existing registration for the type, which is what you want in your app startup code, but not in an extension method that is setting defaults.

With this extension method, the app startup code can be simplified:

```c#
      var services = new ServiceCollection();
      services.AddExtensiblityExample();
      var provider = services.BuildServiceProvider();
```

The extension method registers all the types required for the `ExtensibilityExample` type to be injected anywhere in the codebase. I can then choose to override specific behaviors as I choose:

```c#
      var services = new ServiceCollection();
      services.AddExtensiblityExample();
      services.AddSingleton<ExtensibleExample.ExtensibilityPoint1>((s) => s.SomeState = 42);
      var provider = services.BuildServiceProvider();
```

I like this quite a bit better, because now there are simple, predefined default behaviors, and if I do add an extensibilty point in the future, I can just add it to the extension method and everyone's code keeps working.

### Options-Based Configuration

Another option is to use an "options" type. For example:

```c#
  public class ExtensibleExampleOptions
  {
    public ExtensibleExample.ExtensibilityPoint1 Point1 { get; set; } = (s) => s.SomeState = 0;
    public ExtensibleExample.ExtensibilityPoint2 Point2 { get; set; } = (s) => s.SomeState = 0;
    public ExtensibleExample.ExtensibilityPoint3 Point3 { get; set; } = (s) => s.SomeState = 0;
  }
```

The `ExtensibleExample` class, in this case, needs a constructor that accepts this type:

```c#
    public ExtensibleExample(ExtensibleExampleOptions options)
    {
      Point1 = options.Point1;
      Point2 = options.Point2;
      Point3 = options.Point3;
    }
```

Notice that the `ExtensibleExampleOptions` class contains the default implementations for the services, so they are nicely contained in one location. The `ExtensibleExample` class doesn't really change, other than having a different constructor.

> Note that _now_ I could be using `Action<int>` instead of custom delegate types in the `ExensibilityExample` and `ExtensibleExampleOptions` classes. The only types I'll register with DI are my custom types, and the delegates are "hidden" within my options type.

Now the startup code looks like this:

```c#
      services = new ServiceCollection();
      services.AddSingleton(new ExtensibleExampleOptions
      {
        Point1 = (s) => s.SomeState = 7,
        Point2 = (s) => s.SomeState = 8,
        Point3 = (s) => s.SomeState = 9,
      });
      services.AddTransient<ExtensibleExample>();
      provider = services.BuildServiceProvider();
```

I can choose which, if any, extensibility points to provide in the `ExtensibleExampleOptions` instance.

The downside to using an options type is that I need to create another type. The upside though, is that I no longer need to define multiple specific delegate types, and I can make use of `Action<>` and `Func<>`, and that might simplify my codebase overall. I'd argue, in this case, that I get rid of three custom types (delegates) in favor of one (options) and so save defining and maintaining two custom types.

### Combining Both Solutions

Using an options type doesn't prevent me from also having an extension method. In this case the extension method looks a little different:

```c#
  public static class ExtensibilityExampleExtensions
  {
    public static IServiceCollection AddExtensiblityExample(this IServiceCollection services)
    {
      services.TryAddSingleton<ExtensibleExampleOptions>();
      services.TryAddTransient<ExtensibleExample>();
      return services;
    }
  }
```

Because the default behaviors are encapsulated in the `ExtensibleExampleOptions` class, the extension method becomes very simple, as does the typical app startup code:

```c#
      var services = new ServiceCollection();
      services.AddExtensiblityExample();
      var provider = services.BuildServiceProvider();
```

And if I do want to override some extensiblity point, I override the registered options type:

```c#
      var services = new ServiceCollection();
      services.AddExtensiblityExample();
      services.AddSingleton(new ExtensibleExampleOptions
      {
        Point1 = (s) => s.SomeState = 42
      });
      var provider = services.BuildServiceProvider();
```

In any case, I'm using delegates as extensibility points, and the difference is whether to provide those delegates directly as registered types, or to pass them into my object via an options type.

I'm interested in your thoughts on this: which do you think is best? Or should I just stick with inheritance?
