---
layout: post
title: Blazor 8 Render Mode Detection
date: 2024-03-30T18:04:38.6275143-05:00
categories: []
tags: []
published: true
permalink: 
image: 
---
In .NET 8 the Blazor UI framework introduced some new render modes for components and pages. The available render modes are now:

| Render Mode | Description |
| --- |
| Server-static | Blazor pages are rendered into HTML/CSS on the server, and that text is then sent to the browser |
| Server-static (streaming) | Like _server-static_, except that the server sends content to the browser, and then will continue to send more content over time until complete |
| Server-interactive | This is like Blazor 6/7 Blazor Server projects, where a SignalR connection is established between browser and server, and the user gets a fully interactive experience; by default pages will render first as Server-static, then switch to Server-interactive |
| WebAssembly-interactive | This is like Blazor 6/7 Blazor WebAssembly projects, where your Blazor code runs entirely in the browser on the client using WebAssembly (wasm); by default pages will render first as Server-static, then switch to WebAssembly-interactive |
| InteractiveAuto | An InteractiveAuto page will render first as server-static, and then will switch (on first run) to Server-interactive while your wasm code downloads in the background; subsequent visits to the page will be WebAssembly-interactive |

> ℹ️ People may use other terms for these modes, and that's fine. I'm spelling out the terminology I'm using in this blog post to minimize confusion.

Sometimes you might want to know, at runtime, the render mode being used currently for a given component. This is particularly useful for InteractiveAuto pages, because they will often be rendered three different ways over time. It can also be useful when using Server-interactive and WebAssembly-interactive pages, so you know when and if they are pre-rendered as Server-static before switching to an interactive mode.

> ℹ️ The Blazor team has indicated that in .NET 9 they will provide a way to detect render modes. This blog post demonstrates a technique you can use until that is available.

The code for this blog post is on GitHub as [RenderModes](https://github.com/JasonBock/BlazorTopToBottom/tree/main/src/Rocky/RenderModes). It is part of a workshop we have been presenting at [VS Live conferences](https://vslive.com).

## Component Lifecycle

One simple way to know if a page is interactive or not, is to pay attention to the Blazor component. 

All pages initialize, and you can override `OnInitialized` or `OnInitializedAsync` to be notified that this has happened.

Only interactive pages invoke the `OnAfterRender` or `OnAfterRenderAsync` methods.

This means that if your page is initialized by never invokes `OnAfterRender`, it is server-static rendered.

The trick with doing any work in `OnAfterRender` is that it is invoked _after the component has rendered_, and so any changes you make to the output won't be displayed unless you invoke `StateHasChanged`. WARNING ⚠️: When you invoke `StateHasChanged`, this will cause `OnAfterRender` to run again!

## Code-Based Solution

A more sophisticated solution is to write some code so it is possible to detect, in detail, the render mode currently being used.

This means writing a bit of code to detect whether there's an active SignalR connection, to detect the `StreamRendering` attribute on a page, and to determine if the page is running in a browser.

I'll discuss each of these requirements, and then show how they all work together.

### Detecting An Active SignalR Connection

SignalR connections are called _circuits_, and it is necessary to know whether the current Blazor component has an active circuit. To do this requires the use of a little dependency injection (DI).

#### ActiveCircuitState Class

First, in the `RenderModes.Client` project look at the `ActiveCircuitState` class:

```c#
    public class ActiveCircuitState
    {
        public bool CircuitExists { get; set; }
    }
```

This type needs to be available to server-side and WebAssembly code, and so it is declared in the client project. The client project is referenced by the server project, so the code is available on the server, and also the client project is a DLL that may be deployed to the browser so it can run in the WebAssembly runtime on the client.

All this class does is maintain a value indicating whether a SignalR circuit exists. On the WebAssembly client the value defaults to `false`, and that is always correct. On the server the value will be set based on whether a circuit exists.

#### ActiveCircuitHandler Class

The `ActiveCircuitHandler` class in the `RenderModes` server project detects whether a circuit exists and sets the value in the `ActiveCircuitState` object as appropriate. To do this, it uses DI to have the `ActiveCircuitState` injected, and it is a subclass of `CircuitHandler`, the ASP.NET type for handling SignalR circuits:

```c#
    public class ActiveCircuitHandler(ActiveCircuitState state) : CircuitHandler
    {
        public override Task OnCircuitOpenedAsync(Circuit circuit, CancellationToken cancellationToken)
        {
            state.CircuitExists = true;
            return base.OnCircuitOpenedAsync(circuit, cancellationToken);
        }

        public override Task OnCircuitClosedAsync(Circuit circuit, CancellationToken cancellationToken)
        {
            state.CircuitExists = false;
            return base.OnCircuitClosedAsync(circuit, cancellationToken);
        }
    }
```

Because this class is a subclass of `CircuitHandler` it can override the `OnCircuitOpenedAsync` and `OnCircuitClosedAsync` methods. These methods are invoked when a circuit is opened or closed, making it possible to set the `CircuitExists` property of the injected `ActiveCircuitState` object.

#### Registering the DI Services

Now that you understand the two services necessary to detect an active SignalR circuit, it is necessary to register them with DI as the server and WebAssembly client start up.

In the server project's `Program.cs` file the two services are registered:

```c#
builder.Services.AddScoped<ActiveCircuitState>();
builder.Services.AddScoped(typeof(CircuitHandler), typeof(ActiveCircuitHandler));
```

The services are registered as _scoped_ because Server-interactive Blazor components _for each specific user_ run inside their own DI scope. Having the services scoped means that they are providing information on a per-user basis.

In the client project's `Program.cs` file only the `ActiveCircuitState` service is registered. Because this `Program.cs` code will only be invoked on a WebAssembly client, there will never be a SignalR circuit, and so there's no need for a handler to detect a circuit.

```c#
builder.Services.AddScoped<RenderModes.ActiveCircuitState>();
```

At this point you should understand how these two services are used on the server and client to reliably detect whether the current Blazor environment has an active SignalR connection.

### Detecting the StreamRendering Attribute on a Page

A Server-static page can be marked with the `StreamRendering` attribute to tell Blazor to stream the HTML/CSS output to the browser. This attribute can be detected using reflection against a reference to the Blazor page object:

```c#
page.GetType().GetCustomAttributes(
  typeof(StreamRenderingAttribute), true).Length > 0
```

This line of code uses `GetType` to get the page's `Type` object, and then uses the reflection `GetCustomAttributes` method to get an array containing any instances of the `StreamRendering` attribute on the page. If any such instances exist, then the page has been marked for streaming.

### Detecting Running in a Browser

Detecting whether your code is running in a browser can be done by using a built-in .NET feature. For example:

```c#
var isBrowser = OperatingSystem.IsBrowser();
```

At this point you should understand how to detect whether a SignalR connection exists, whether the page has the `StreamRendering` attribute, and whether the code is currently running in the browser.

Using these three concepts it is possible to write a class that detects the current rendering mode.

### Putting it All Together

In the `RenderMode.Client` project you will find the `RenderModeProvider` class:

```c#
    public class RenderModeProvider(ActiveCircuitState activeCircuitState)
    {
        public string GetRenderMode(ComponentBase page)
        {
            string result;
            var isBrowser = OperatingSystem.IsBrowser();
            if (isBrowser)
                result = "wasm-interactive";
            else if (activeCircuitState.CircuitExists)
                result = "server-interactive";
            else if (page.GetType().GetCustomAttributes(typeof(StreamRenderingAttribute), true).Length > 0)
                result = "server-static (streaming)";
            else
                result = "server-static";
            return result;
        }
    }
```

This class relies on DI to gain access to the `ActiveCircuitState` for the current user, and implements a `GetRenderMode` method to return the current render mode.

> ℹ In a "real" implementation this should return an enum value rather than a string value. For example, see the implementation in the [CSLA .NET ProjectTracker sample app](https://github.com/MarimerLLC/csla/tree/main/Samples/ProjectTracker/ProjectTracker.Blazor).

This method implements a fairly simple workflow.

If the code is running in the browser then it is obviously WebAssembly-interactive.

If the code has an active SignalR connection, then it is Server-interactive.

Otherwise the code is Server-static, and the only question is whether the `StreamRendering` attribute exists on the page.

This service needs to be registered for DI on the server and client so it can be injected into any Blazor component or page where you want to know the render mode. In both `Program.cs` files you will find this line of code:

```c#
builder.Services.AddTransient<RenderModes.Client.RenderModeProvider>();
```

The service is marked as transient, so each time the service is requested a new instance is created.

### Using the RenderModeProvider Service

To use the `RenderModeProvider` service in a Blazor page or component, the service must be injected into the page. In the example solution, the Home, Weather, and Counter pages all inject the service:

```c#
@rendermode InteractiveAuto
```

They then display the value as part of the HTML output:

```html
<div class="alert-info">@renderMode</div>
```

Finally, in the `@code` for the component, they set the `renderMode` field with the correct value as the component is initialized:

```c#
    private string renderMode = "unknown";

    protected override void OnInitialized()
    {
        renderMode = renderModeProvider.GetRenderMode(this);
    }
```

At this point you should understand how the `RenderModeProvider` class works and how you can use it in your Blazor pages or components to detect the current render mode.

### Exploring the Render Modes

I recommend you download and run the code from GitHub. Watch carefully as you navigate to the Counter page, leave, and then return.

That page will be Server-static, then Server-interactive, then Server-static, then WebAssembly-interactive. This is because the page is set to the InteractiveAuto render mode, so it leverages all the render modes as appropriate.

The new render modes in Blazor 8 are very powerful and can be extremely useful when building apps. It is useful, and often critical, to understand the current render mode of a component or page so you can properly get and display data for the user.