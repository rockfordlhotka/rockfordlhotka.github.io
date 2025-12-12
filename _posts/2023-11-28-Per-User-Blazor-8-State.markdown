---
layout: post
title: Per-User Blazor 8 State
date: 2023-11-28T00:00:00.0000000-06:00
categories: []
tags: []
published: true
permalink: 
image: 
---
About a month ago I created a blog post detailing an issue with the new Blazor 8 automatic rendering model, where there's [no built-in way to have per-user state within a Blazor app](https://blog.lhotka.net/2023/10/12/Blazor-8-State-Management).

This doesn't affect pure Blazor server-interactive or wasm-interactive apps, as long as they don't use any server-rendering (or pre-rendering), but it does affect any Blazor 8 app that uses mixed-mode rendering (auto) between server-rendered, server-interactive, and wasm-interactive.

The "Blazor Web App" template in Visual Studio uses this new "magical" automatic rendering model, and so I suspect _most_ apps going forward will have to deal with the reality that per-user state isn't maintained across the app.

What do I mean by per-user state? Basically what I mean is the sort of thing that in Blazor 6 and 7 would have been maintained in a dependency injection (DI) scoped service. In previous versions of Blazor, it was possible to create a scoped DI service that could maintain any state the app might need between pages on a per-user basis.

With the new automatic/dynamic render mode in Blazor 8, there is no consistent DI scope that will exist over the life of the app. Each server-rendered page gets its own DI scope. If the user moves between server-interactive pages they'll get a consistent DI scope, but as soon as the user moves to a server-rendered or wasm-interactive page that scope is gone. The same is true with wasm-interactive rendered pages.

In Blazor 8 the default is for pages to be server-rendered unless they are marked to be server-interactive or wasm-interactive or auto.

As an additional twist, if a page is marked as wasm-interactive or auto, that page will (by default) be server-rendered (pre-rendered) _and then_ will be reloaded as a wasm-interactive page. This means the page renders twice, with your Blazor UI code running on the server and then on the client.

The following table summarizes what happens when a user navigates from one page to another.

| Start Mode | Target Mode | HttpContext | DI scope |
| --- |
| Server-rendered | Server-rendered | Available | Lost |
| Server-rendered | Server-interactive | Available | Lost |
| Server-rendered | Wasm-interactive | Lost | Lost |
| Server-interactive | Server-rendered | Available | Lost |
| Server-interactive | Server-interactive | Available | Consistent |
| Server-interactive | Wasm-interactive | Lost | Lost |
| Wasm-interactive | Wasm-interactive | n/a | Consistent |
| Wasm-interactive | Server-rendered | n/a | Lost |
| Wasm-interactive | Server-interactive | n/a | Lost |

I have created a sample app called [Blazor8State](http://github.com/rockfordlhotka/Blazor8State) that demonstrates a possible solution to this issue. This blog post is basically documentation for that sample and the solution.

I did blog about [flowing user state in Blazor](https://blog.lhotka.net/2023/10/27/Flowing-State-in-Blazor-8) using a much earlier version of this approach. That POC is what paved the way for a more elegant solution, so I'm taking the time to better document what I've come up with.

> ℹ️ In addition to this being a problem many people will encounter, I needed to solve this issue to enable the `LocalContext` and `ClientContext` concepts in [CSLA .NET](https://cslanet.com).

## Solution Overview

The overall solution approach is to do the following:

1. Maintain a unique per-user session id value
1. Use a dictionary to maintain each user's session state values
1. On the server, maintain a dictionary that contains the session state dictionaries for all users, keyed by the session id value
1. Implement a web API endpoint so wasm-interactive pages can pull and push the current user's session state to and from the server

Most of these behaviors are implemented as DI service implementations, often different between server and wasm client code.

## Creating a User Session Id

The first step in the solution is to create an ID value that is available in the server-rendered, server-interactive, and wasm-interactive render modes. The only solution I have found that covers those three render modes is a cookie.

| Render Mode | Cookie Access |
| --- |
| Server-Rendered | Use `HttpContext` `Request` and `Response` properties |
| Server-Interactive | Use `HttpContext` `Request` property |
| Wasm-Interactive | Use JavaScript interop to access the DOM |

Accessing a cookie means that it is possible to generate and maintain a per-user unique id value that is consistent across all pages in a Blazor app.

### ISessionIdManager Interface

In the `Blazor8State` project, access to the unique per-user session ID value is available through the `ISessionIdManager` interface:

```c#
    public interface ISessionIdManager
    {
        Task<string?> GetSessionId();
    }
```

This interface defines the `GetSessionId` method, which supports async usage, and returns a `string` id value. In the sample app the id is a GUID value, but it can be any value that is unique across all current users.

### Server-side SessionIdManager

In the `Blazor8State` solution there is a `Blazor8State` project, which is the ASP.NET Core server project that hosts the app and runs all server-rendered and server-interactive pages.

In this project is a `SessionIdManager` class that implements the `ISessionIdManager` interface. The `GetSessionId` method is implemented using `HttpContext`, which is available via the standard ASP.NET Core `IHttpContextAccessor` service:

```c#
    public class SessionIdManager(IHttpContextAccessor httpContextAccessor) : ISessionIdManager
    {
        private readonly IHttpContextAccessor HttpContextAccessor = httpContextAccessor;

        public Task<string?> GetSessionId()
        {
            var httpContext = HttpContextAccessor.HttpContext;
            string? result;

            if (httpContext != null)
            {
                if (httpContext.Request.Cookies.ContainsKey("sessionId"))
                {
                    result = httpContext.Request.Cookies["sessionId"];
                }
                else
                {
                    result = Guid.NewGuid().ToString();
                    httpContext.Response.Cookies.Append("sessionId", result);
                }
            }
            else
            {
                throw new InvalidOperationException("No HttpContext available");
            }
            return Task.FromResult(result);
        }
    }
```

If this method doesn't find an existing session id cookie, it creates the cookie with a new GUID value.

In the server-side `Program.cs` this type is set up as the service that provides the session id:

```c#
builder.Services.AddHttpContextAccessor();
builder.Services.AddTransient(typeof(ISessionIdManager), typeof(SessionIdManager));
```

Any server-side code (server-rendered or server-interactive) that needs access to the session id will be provided an instance of _this_ `SessionIdManager` to provide the value from `HttpContenxt`.

Because the only way to access `HttpContext` is via the `IHttpContextAccessor` service, the `AddHttpContextAccessor` method must be invoked in the server-side `Program.cs` as well.

### Client-side SessionIdManager

For pages that are rendered using wasm-interactive the code will run in the browser on the client device. In this case the cookie must be accessed via JavaScript interop.

In the `Blazor8State` solution there is a `Blazor8State.Client` project that is a Razor Component Library (RCL) project. This project contains razor components that can be compiled and run on the server and also on the wasm browser client.

The `SessionIdManager` class in this project is only used when the code is running in wasm on the client. It uses the `JsRuntime` Blazor type to invoke JavaScript in the browser to access the cookie:

```c#
    public class SessionIdManager : ISessionIdManager
    {
        private readonly IJSRuntime JsRuntime;

        public SessionIdManager(IJSRuntime jsRuntime) 
        {
            JsRuntime = jsRuntime;
        }

        public async Task<string?> GetSessionId()
        {
            var result = await JsRuntime.InvokeAsync<string>("ReadCookie.ReadCookie", "sessionId");
            return result;
        }
    }
```

In the client-side `Program.cs` this type is set up as the service that provides the session id:

```c#
builder.Services.AddTransient(typeof(ISessionIdManager), typeof(SessionIdManager));
```

Any client-side code (wasm-interactive) that needs access to the session id will be provided an instance of _this_ `SessionIdManager` to provide the value from the `JsRuntime`.

## Per-User Session Data

In the `Blazor8State` project, per-user state is maintained as a dictionary that has `string` keys and values.

> ℹ️ More complex value types can be used, but they _must be_ serializable via JSON. It is left as an exercise to the reader to broaden the value type beyond string types.

Per-user session state is maintained, in the `Blazor8State` app, using a server-side cache implemented as a DI service.

Per-user state is a dictionary with some extra properties.

```c#
    public class Session : Dictionary<string, string>
    {
        public string SessionId { get; set; } = string.Empty;
        public bool IsCheckedOut { get; set; }
    }
```

The type itself is a subclass of `Dictionary<string, string>` and so contains all per-user data. It also implements properties to provide access to the `SessionId` value (the cookie id), and a `bool` that indicates whether the dictionary is currently in use by a wasm-interactive page - and thus is "checked out" from the server.

I will discuss this idea of checking out the user session state later in the blog post.

### Per-User ISessionManager

The `ISessionManager` interface defines the API through which any page or other code can access the session data for the current user:

```c#
    public interface ISessionManager
    {
        Task<Session> GetSession();
        Task UpdateSession(Session session);
    }
```

This interface provides a `GetSession` method that returns the current user's session dictionary, and an `UpdateSession` method that can be invoked by wasm-interactive pages to update any changed per-user session data.

### Server-side SessionManager

For pages that run entirely on the server, the session data for all users is maintained in a singleton DI service that is set up in `Program.cs`:

```c#
builder.Services.AddSingleton(typeof(ISessionManager), typeof(SessionManager));
```

> ℹ️ The implementation in the `Blazor8State` solution is a simple in-memory cache on the server. A more robust and scalable solution would be to implement the `ISessionManager` interface to maintain all per-user state in something like a REDIS cache or other store that is external to any specific web server.

The implementation of `ISessionManager` on the server is in the `SessionManager` class in the `Blazor8State` project:

```c#
    public class SessionManager : ISessionManager
    { 
        private Dictionary<string, Session> _sessions = 
          new Dictionary<string, Session>();
        private readonly ISessionIdManager _sessionIdManager;

        public SessionManager(ISessionIdManager sessionIdManager)
        {
            _sessionIdManager = sessionIdManager;
        }

        public async Task<Session> GetSession()
        {
            var key = await _sessionIdManager.GetSessionId();
            if (!_sessions.ContainsKey(key))
                _sessions.Add(key, new Session());
            var session = _sessions[key];
            // ensure session isn't checked out by wasm
            while (session.IsCheckedOut)
                await Task.Delay(5);

            return session;
        }

        public async Task UpdateSession(Session session)
        {
            if (session != null)
            {
                var key = await _sessionIdManager.GetSessionId();
                session.SessionId = key;
                Replace(session, _sessions[key]);
                _sessions[key].IsCheckedOut = false;
            }
        }

        /// <summary>
        /// Replace the contents of oldSession with the items
        /// in newSession.
        /// </summary>
        /// <param name="newSession"></param>
        /// <param name="oldSession"></param>
        private void Replace(Session newSession, Session oldSession)
        {
            oldSession.Clear();
            foreach (var key in newSession.Keys)
                oldSession.Add(key, newSession[key]);
        }
    }
```

This class relies on the `ISessionIdManager`, which on the server provides access to the cookie value from the `HttpContext` object. That value is used as the unique per-user key to access the dictionary containing the per-user session values.

The `GetSession` method returns the current user's session state object.

If the current user does not yet have a `Session` object, one is created and added to the dictionary:

```c#
            if (!_sessions.ContainsKey(key))
                _sessions.Add(key, new Session());
```

Perhaps more interesting is the code that prevents access to the per-user `Session` object if it is "checked out" to a wasm-interactive page:

```c#
            while (session.IsCheckedOut)
                await Task.Delay(5);
```

If all the app's pages are server-rendered and server-interactive, the `IsCheckOut` property will always be `false`, and so the value is returned immediately. However, if the user has just navigated back to a server page from a wasm-interactive page, then there will be a delay before the client-side state has been transferred to the server. To ensure that your server-side code has the current per-user state, the server must wait until the `IsCheckedOut` value is `false`. When a wasm-interactive page gets the current per-user state, `IsCheckedOut` is set to `true` to "lock" the state in the server cache.

> ⚠️ This `while` loop should probably have a timeout as part of the implementation. It may be possible for a user to navigate from a wasm-interactive page to a server page and have something go wrong with updating the session state. That would leave the app in an unusable state because `IsCheckedOut` would never be set to `false`, leading to a deadlock.

I will discuss this more as I discuss wasm-interactive pages.

### Client-side WebAssembly SessionManager

When the user navigates to a wasm-interactive page, they are moving from server-side code to client-side code. This means that any per-user state must move from the server to the client device.

As a result, there is a dedicated client-side `SessionManager` type in the `Blazor8State.Client` project:

```c#
    public class SessionManager : ISessionManager
    {
        private Session _session;

        public async Task<Session> GetSession()
        {
            var client = new HttpClient();
            client.BaseAddress = new Uri("https://localhost:7095/");
            var result = await client.GetFromJsonAsync<Session>("state");
            _session = result;
            return _session;
        }

        public async Task UpdateSession(Session session)
        {
            var client = new HttpClient();
            client.BaseAddress = new Uri("https://localhost:7095/");
            await client.PutAsJsonAsync<Session>("state", session);
            _session = session;
        }
    }
```

This class also implements the `ISessionManager` interface. In this case, the `GetSession` method calls a web service to pull the current user state from the server to the client:

```c#
        public async Task<Session> GetSession()
        {
            var client = new HttpClient();
            client.BaseAddress = new Uri("https://localhost:7095/");
            var result = await client.GetFromJsonAsync<Session>("state");
            _session = result;
            return _session;
        }
```

This implies that the server project (`Blazor8State`) has a a web service API controller that supports a get operation. This is in the `Controllers` folder and is named `StateController`. The get operation is implemented like this:

```c#
        [HttpGet(Name = "GetState")]
        public async Task<Blazor8State.Client.Session> Get()
        {
            var session = await _sessionList.GetSession();
            session.IsCheckedOut = true;
            return session;
        }

```

This method returns the current user's `Session` state object. Importantly, it also sets that object's `IsCheckedOut` property to `true`, so server-side code can't access the values until they've been updated from the wasm client back to the server.

The client-side `SessionManager` also implements the `UpdateSession` method, which sends the client-side `Session` object back to the server:

```c#
        public async Task UpdateSession(Session session)
        {
            var client = new HttpClient();
            client.BaseAddress = new Uri("https://localhost:7095/");
            await client.PutAsJsonAsync<Session>("state", session);
            _session = session;
        }
```

Again, this implies that the server controller has a put implementation to update the server-side state:

```c#
        [HttpPut(Name = "UpdateState")]
        public async Task Put(Blazor8State.Client.Session updatedSession)
        {
            await _sessionList.UpdateSession(updatedSession);
        }
```

This invokes the _server-side_ `SessionManager` type's `UpdateSession` method:

```c#
        public async Task UpdateSession(Session session)
        {
            if (session != null)
            {
                var key = await _sessionIdManager.GetSessionId();
                session.SessionId = key;
                Replace(session, _sessions[key]);
                _sessions[key].IsCheckedOut = false;
            }
        }
```

This method ensures that the server-side state is replaced by the potentially changed or new client-side state. Importantly, it also sets the `Session` object's `IsCheckedOut` property to `false` to unlock this user's session data for use by server-side code.

### Detecting Page Navigation

When the user navigates to a wasm-interactive page the current user's state can be retrieved by the client-side `SessionManager` class, which pulls the current state from the server to the client.

It is important to remember that the user state might be altered by the app while the user is interating with a wasm-rendered page. Those updates need to be sent back to the server so the server-rendered and server-interactive pages have that updated state. 

There is a timing issue here, because Blazor has no event or API to tell the page's code that the user is about to navigate to another page. The only way to know that the user has navigated to another page is by having each page implement the `IDisposable` interface. The `Dispose` method is invoked _after the next page has rendered_.

Let me repeat that: the `Dispose` method on the previous page is invoked after the next page has been rendered.

If the per-user state was altered on a wasm-interactive page, the wasm-intertive page's `Dispose` method will invoke the `UpdateSession` method on the client-side `ISessionManager` service, resulting in a put operation to send the user state to the server.

This can only happen if the target server-side page _has rendered_. Here's the issue though: most pages won't want to render _until after they access per-user state_. This is a deadlock situation, where the server-rendered or server-interactive page won't render until it has access to the user state, but the previous (wasm-interactive) page won't update that state until the next page renders!

> ⚠️ IMPORTANT LIMITATION: Server-rendered pages must be streamed. Server-interactive pages must access per-user session state after the page is rendered.

The solution to this deadlock is to make sure that all server-rendered pages are _streamed_, which is not the default. And to make sure that all server-interactive pages don't access per-user state until the `OnAfterRendered` or `OnAfterRenderedAsync` methods are executed.

I will talk about the consequences for all render types.

#### Implementing Wasm-Interactive Pages

Because wasm-interactive pages might change the user state, they need to implement `IDisposable` and use their `Dipose` method to update the server with any changed state. For example, look at the `Counter` page in the `Blazor8State.Client` project.

Implement `IDisposable` as the page is declared:

```c#
@implements IDisposable
```

Then in the `code` block implement the `Dispose` method:

```c#
    public void Dispose()
    {
        var isBrowser = (System.Environment.OSVersion.Platform == PlatformID.Other);
        if (isBrowser)
        {
            sessions.UpdateSession(session);
        }
    }
```

It is important to remember that the `Counter` page _will_ be rendered on the server using the server-rendered mode (pre-rendered) and then rendered on the client using the wasm-interactive mode. This means that the `Dipose` method needs to ensure that it only update the session data if it is running in the browser when rendered by wasm-interactive.

This can be detected by checking the OS version:

```c#
        var isBrowser = (System.Environment.OSVersion.Platform == PlatformID.Other);
        if (isBrowser)
```

> ℹ️ Though it is outside the scope of this blog post, you can look at the `Counter` page to see how the `IsStaticRender` field is used to ensure that the button control can't be clicked by the user until the page has been rendered in interactive mode.

The result is that when the user navigates away from a wasm-interactive page to another page, that the user state on the client is sent to the server, so server-side pages have access to the latest state.

> ⚠️ Again, remember that server-rendered pages must be streaming, and server-interactive pages must access user state after the page has rendered.

#### Implementing Server-Interactive Pages

Server-interactive pages used to be called "Blazor Server" pages. These are pages that use the full Blazor interactive capabilities, including establishing a SignalR connection to the browser.

These pages aren't considered "rendered" until the `OnAfterRendered` and `OnAfterRenderedAsync` methods are invoked.

If the user navigates from a wasm-interactive page to a server page, the client-side page's `Dispose` method won't execute until the target page on the server has rendered. Because the per-user state isn't "unlocked" until the client-side page's `Dispose` method executes, it is _critical_ that the server-side page not try to use user state until after is has rendered.

For a server-interactive page this means accessing the user state in the `OnAfterRenderedAsync` method. you can see an example of this in the `Blazor8State` project's `ServerCounter` page:

```c#
    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        renderLocation = "server";
        sessionId = await sessionIdManager.GetSessionId();
        session = await sessions.GetSession();
        try
        {
            mystate = (string)session["mystate"];
        }
        catch (Exception ex)
        {
            mystate = ex.Message;
        }
        StateHasChanged();
    }
```

The page's state isn't set up in the initialized or parameters set methods, it is set up after rendering.

Because rendering is complete, it is necessary to call `StateHasChanged` in this method to force the page to render the updated state.

#### Implementing Server-Rendered Pages

Server-rendered pages do not implement the full life cycle of a normal Blazor page. Specifically, they do not invoke the after rendered methods: `OnAfterRendered` and `OnAfterRenderedAsync`.

Also, by default, if a server-rendered page waits in its initialized or parameters set methods for a wasm-interative page to transfer user state from the client to the server the page won't ever render.

The solution is to allow the page to "render" so the client-side state is transferred to the server, and then for the page to "finish rendering" once its state is available. This is done by ensuring that server-rendered pages are _streamed_.

The default `Weather` page in the `Blazor8State` project is already marked as streamed:

```c#
@attribute [StreamRendering]
```

The `Home` page is also marked as streamed using this attribute so the user can successfully navigate from the wasm-interactive `Counter` page to the `Home` page.

## Implementing Pages with Per-User State

At this point you should understand the basic concepts of maintaining per-user state in a cache on the server, and how that state is transferred to and from wasm-interactive pages via the `StateController` service.

I will walk through the steps necessary to implement any server-rendered, server-interactive, and wasm-interactive page that uses this state management model.

### Server-Rendered Pages

A server-rendered page can access user session by injecting the `ISessionManager` service:

```c#
@inject ISessionManager sessions
```

The `ISessionManager` service can be used throughout your code in the page as you choose (with exceptions for the home page).

Server-rendered pages _must be streamed_ to avoid deadlocks when navigating from a wasm-interactive page to a server-rendered page:

```c#
@attribute [StreamRendering]
```

Streamed rendering allows the page to "render" and then access user session state to complete rendering.

#### Special Considerations for the Home Page

The first page of the app, typically the home or index page, is special, in that it creates the session id cookie for the app.

The cookie is not created until the page is fully rendered. This means that you can only call the `ISessionManager` service's `GetSession` method _exactly one time_ on this page.

Each call to `GetSession` results in a call (behind the scenes) to the `ISessionIdManager` service's `GetSessionId` method, and each call to that method _on the home page_ will generate a new GUID value that has not yet been written to the cookie.

Again, the first page of the app (typically home or index) _must_ call `GetSession` exactly one time to initialize the session id cookie and the user session object.

### Server-Interactive Pages

A server-interactive page can access user session by injecting the `ISessionManager` service:

```c#
@inject ISessionManager sessions
```

The `ISessionManager` service can not be used until after the page has rendered. This means that the earliest you can access user state is in the `OnAfterRender` and `OnAfterRenderAsync` methods.

### Wasm-Interactive Pages

Wasm-interactive pages are the most complex scenario, because these pages will be prerendered on the server using server-rendering and _then_ rendered again using wasm-interactive rendering.

This means that you will need to implement code in the page that will run on the server, and different code that will run on the wasm client.

A wasm-interactive page can access user session by injecting the `ISessionManager` service:

```c#
@inject ISessionManager sessions
```

The `ISessionManager` service can be used throughout your code in the page, keeping in mind that most wasm-interactive pages render _first_ on the server using the server-render model. This means your page must handle server-rendered and wasm-interactive scenarios.

#### Initialize the Page

Page initialization is different depending on whether your code is executing as part of a server-render or wasm-interactive scenario.

For server-rendered code, you must implement code in the `OnInitializedAsync` method:

```c#
    protected override async Task OnInitializedAsync()
    {
        IsStaticRender = true;
        var isBrowser = (System.Environment.OSVersion.Platform == PlatformID.Other);
        if (!isBrowser)
        {
            renderLocation = "server";
            sessionId = await sessionIdManager.GetSessionId();
            session = await sessions.GetSession();
            try
            {
                mystate = (string)session["mystate"];
            }
            catch (Exception ex)
            {
                mystate = ex.Message;
            }
        }
    }
```

This method mostly executes if the code is not running on the browser (`isBrowser == false`). In this case the `GetSession` method is invoked to get the user state from the server cache.

For wasm-interactive code that runs on the client, you must implement code in the `OnAfterRenderAsync` method:

```c#
    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        IsStaticRender = false;
        var isBrowser = (System.Environment.OSVersion.Platform == PlatformID.Other);
        if (isBrowser)
        {
            if (firstRender)
            {
                renderLocation = "wasm";
                sessionId = await sessionIdManager.GetSessionId();
                StateHasChanged();
                session = await sessions.GetSession();
                try
                {
                    mystate = (string)session["mystate"];
                }
                catch (Exception ex)
                {
                    mystate = ex.Message;
                }
                StateHasChanged();
            }
        }
        else
        {
            if (firstRender)
            {
                StateHasChanged();
            }
        }
    }
```

The important thing to notice is that the `GetSession` method is invoked after the page has rendered, and only when `firstRender` is `true`. The `StateHasChanged` method is invoked to force the page to rerender now that the user session state is available.

The other code in this method deals with disabling and enabling the button control, and that is outside the scope of this blog post.

#### Implement IDisposable

Wasm-interactive pages _must_ implement `IDisposable` to know when the user has navigated to another page:

```c#
@implements IDisposable
```

This means implementing a `Dispose` method:

```c#
    public void Dispose()
    {
        var isBrowser = (System.Environment.OSVersion.Platform == PlatformID.Other);
        if (isBrowser)
        {
            sessions.UpdateSession(session);
        }
    }
```

The `Dispose` method only does work when running on the client, and it calls the `UpdateSession` method to transfer the client-side user session state to the server.

## Conclusion

The `Blazor8State` project is intended to show one possible solution to the issue of maintaining per-user session state between pages in a Blazor app in .NET 8.

Because Blazor 8 doesn't provide consistent access to `HttpContext` or any dependency injection service (singleton or scoped) between server-rendered, server-interactive, and wasm-interactive pages, it is up to you to implement any concept of per-user state.

Maintaining user state in a server cache (in this case on the web server), and providing access via a singleton service on the server and a web API for wasm-interactive appears to be a workable solution.