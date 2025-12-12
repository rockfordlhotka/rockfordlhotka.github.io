---
layout: post
title: Flowing State in Blazor 8
date: 2023-10-27T00:00:00.0000000-05:00
categories: []
tags: []
published: true
permalink: 
image: 
---
In a recent blog post I discussed [Blazor 8 State Management](https://blog.lhotka.net/2023/10/12/Blazor-8-State-Management), laying out a potential pitfall that Blazor app developers will encounter when using the new automatic render mode introduced in .NET 8.

In this blog post I'm going to walk through a draft solution. There are still some edge cases and timing issues to be resolved, and perhaps there's a whole other approach that's much better (and if so please let me know).

The code for this post is here in the [draft resolution release](https://github.com/rockfordlhotka/Blazor8State/tree/draft-resolution).

The code in this tag/branch flows per-user state between server-rendered, server-interactive, and wasm-interactive pages in a Blazor app, including flowing changes to the per-user state in a wasm page back to server pages. The code isn't optimized, or pretty. Nor does it even give a nod to security or anything. It is literally a first draft to prove that this particular solution can work.

## Solution Overview

At a high level the draft solution is this:

1. Maintain a `sessionId` value as a Guid in a cookie so it is available to server-rendered, server-interactive, and wasm-interactive pages
1. Provide a `SessionManager` DI service that any Blazor code can use to request the current per-user session
1. Provide a `Session` type that stores all per-user session state
1. Use an API endpoint so `SessionManager` can pull the current session state from the server to a wasm page/component
1. Use an API endpoint so a wasm page/component can push the current session state to the server when the user leaves the wasm page

The very first page the user loads (`Home` in this project) is responsible for creating the Guid cookie that represents the `sessionId` value.

> ⚠️ It may be that I can avoid the use of a cookie and instead use a root-level `CascadingValue`. That is something I need to research.

Flowing the per-user state between server-rendered and server-interactive pages is relatively straightforward, because all that code is running on the web server and so has access to the same `SessionManager` instance.

The "fun" part is when the user navigates to a wasm-interactive page, because the wasm code running in the browser is entirely separate from anything running on the server. To overcome that challenge, there's a `StateController` API endpoint that allows the wasm code to request the per-user state so it can be used on the client. And when the user navigates away from a wasm page, that API is called to push the state from the browser back to the server (because I assume the state may have changed).

I'll walk through the parts of the solution.

## Per-User Session

To start with, there's the `Session` class that implements a place to store per-user state:

```c#
namespace Blazor8State.Client
{
    /// <summary>
    /// Per-user session data. The object must be 
    /// serializable via JSON.
    /// </summary>
    public class Session : Dictionary<string, string>
    { }
}
```

There will be one logical instance of this type for each user that is using the app. Really each _user session_, since a user might run the app in different browsers or tabs at the same time, and each of those are considered different user sessions by aspnetcore and Blazor.

Instances of this type will flow to/from the wasm client, and so must be serializable. To keep things extremely simple, I'm using `string` types, but really any types that can be serialized and deserialized via JSON should work fine.

> ️ℹ️ If I use this concept in CSLA it'll be more powerful, as I'll almost certainly use `MobileFormatter` so a more complex object graph could be used.

Notice that this type is declared in the `Blazor8State.Client` project, because it is used in the wasm client as well as the web server.

## Session Manager

The `SessionManager` type is responsible for providing access to each user's per-user `Session` instance. 

This type is registered as a singleton DI service in `Program.cs` in both the server and client app in the solution.

```c#
builder.Services.AddSingleton<SessionManager>();
```

At first glance this might seem simple: just use a `Dictionary<sessionId, Session>`. However, it is important to remember that any user's state might be (at the moment) on the web server or on the browser in wasm. That detail should be transparent to any Blazor app code, and so `SessionManager` is what ensures that the current user session is available, as needed, on the wasm client or the web server.

```c#
using System.Net.Http.Json;

namespace Blazor8State.Client
{
    /// <summary>
    /// Dictionary containing per-user session objects, keyed
    /// by sessionId.
    /// </summary>
    public class SessionManager
    { 
        private Dictionary<string, Session> _sessions = new Dictionary<string, Session>();

        public async Task<Session> GetSession(string key)
        {
            var isBrowser = (System.Environment.OSVersion.Platform == PlatformID.Other);
            if (isBrowser && _sessions.Count == 0)
            {
                var client = new System.Net.Http.HttpClient();
                client.BaseAddress = new Uri("https://localhost:7095/");
                var result = await client.GetFromJsonAsync<Session>("state");
                _sessions.Add(key, result);
            }
            return _sessions[key];
        }

        public bool Contains(string key)
        { return _sessions.ContainsKey(key); }

        public void Add(string key, Session session)
        { _sessions[key] = session; }
    }
}
```

Again, at a simple level it really is a dictionary keyed by `sessionId`. However, the `GetSession` method detects whether the code is currently running in the browser (wasm) or on the server. If the code is running in the browser, the `StateController` API is called to retrieve the per-user state from the server so it is now available on the client.

I'll come back to the controller and API later.

## Accessing the State in a Page

In any _server_ Blazor page, this means the per-user state is always available like this:

```c#
@inject IHttpContextAccessor hca
@inject SessionManager sessions

@code {
    private Session session;

    protected override async Task OnInitializedAsync()
    {
        var httpContext = hca.HttpContext;
        if (httpContext is not null && httpContext.Request.Cookies.ContainsKey("sessionId"))
        {
            var sessionId = httpContext.Request.Cookies["sessionId"];
            session = await sessions.GetSession(sessionId);
        }
    }
}
```

The `IHttpContextAccessor` is used to access the current `HttpContext`, which is used to access the `sessionId` cookie. That cookie is then used as the key to find the current per-user state. This allows any code in the page to access the `session` dictionary.

In a _wasm_ Blazor page it isn't possible to access the cookie until later in the page lifecycle, so this is the code:

```c#
@inject SessionManager sessions
@inject IJSRuntime JsRuntime

@code {
    private Session session;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            var sessionId = await JsRuntime.InvokeAsync<string>("ReadCookie.ReadCookie", "sessionId");
            session = await sessions.GetSession(sessionId);
            StateHasChanged();
        }
    }
}
```

In this case the code is running in the browser, and so the cookie must be retrieved using JavaScript interop. This isn't available during the `OnInitializedAsync` method, and so all the work is deferred until after the page has been rendered.

> ⚠️ The result is a visual glitch and a timing issue, because the page will have rendered in the browser _before_ the per-user state is available.

The `JsRuntime` service is used to invoke the `ReadCookie` method to retrieve the `sessionId` cookie value. That value is then used to get the per-user session state. This allows any code in the page to access the `session` dictionary.

## Pulling the State to the Wasm Client

As I mentioned earlier, if all the code is running on the web server, this is all very easy because the `SessionManager` singleton service is available to all server-side code. However, when the user navigates to a wasm-rendered Blazor page, the per-user state needs to be pulled from the server to the browser so it is available to the Blazor code running on the client.

This happens because the `SessionManager` implements a check in the `GetSession` method to determine if the code is currently running in the browser. If it is running in the browser, then an HTTP call is made to pull the state from the server:

```c#
    if (isBrowser && _sessions.Count == 0)
    {
        var client = new System.Net.Http.HttpClient();
        client.BaseAddress = new Uri("https://localhost:7095/");
        var result = await client.GetFromJsonAsync<Session>("state");
        _sessions.Add(key, result);
    }
```

The API endpoint being invoked is a controller in the server project named `StateController`. It implements a `Get` method:

```c#
        [HttpGet(Name = "GetState")]
        public async Task<Session> Get()
        {
            var httpContext = _contextAccessor.HttpContext;
            var sessionId = httpContext.Request.Cookies["sessionId"];
            var session = await _sessionList.GetSession(sessionId);
            return session;
        }
```

This method uses the `HttpContext` on the server to retrieve the `sessionId` cookie (which should prevent cookie spoofing), and uses that value to retrieve the current user's session state. That state is then returned to the caller - serialized via JSON of course.

At this point the wasm Blazor code running in the browser has a `SessionManager` that contains exactly one set of per-user state: the state for the current user. And all Blazor code running in the browser can interact with that `Session` instance as needed.

## Pushing the State Back to the Server

It is possible that wasm-interactive Blazor pages/components might change state in the `Session` object for the user. When the user navigates away from the current wasm page, they would expect those changes to be persisted, even if the next page they use is server-rendered or server-interactive. This means that the per-user state in the browser must be pushed to the server.

The only reliable way to know when a user has navigated away from a Blazor page is for that page to implement `IDisposable`, so when the user leaves the page the `Dispose` method is invoked. In the sample app, the `Counter` page is initially rendered as server-interactive and then transparently switches to wasm-interactive. This page implements `IDisposable` so any client-side state changes are pushed back to the server.

To prove the point, when the button is clicked on the page, new per-user state is generated in the `IncrementCount` method.

```c#
@page "/counter"
@attribute [RenderModeInteractiveAuto]
@inject SessionManager sessions
@inject IJSRuntime JsRuntime
@implements IDisposable

<PageTitle>Counter</PageTitle>

<p>@sessionId</p>

@if (session is not null)
{
    <p>State: @mystate</p>
}

<h1>Counter</h1>

<p role="status">Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code {
    private int currentCount = 0;
    private string sessionId = "<unread>";
    private Session session;
    private string mystate;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            sessionId = await JsRuntime.InvokeAsync<string>("ReadCookie.ReadCookie", "sessionId");
            session = await sessions.GetSession(sessionId);
            mystate = (string)session["mystate"];
            StateHasChanged();
        }
    }

    private void IncrementCount()
    {
        currentCount++;
        mystate = Guid.NewGuid().ToString();
        session["mystate"] = mystate;
    }

    public void Dispose()
    {
        sessions.UpdateSession(sessionId, session);
    }
}
```

As with all the pages in this sample, the `SessionManager` service is injected into the page. This particular page also implements `IDisposable`, and so has a `Dispose` method:

```c#
    public void Dispose()
    {
        sessions.UpdateSession(myCookieValue, session);
    }
```

This method calls an `UpdateSession` method on the `SessionManager` type to request that the client-side session be pushed to the server.

In the `SessionManager` class this method is implemented:

```c#
        public async Task UpdateSession(string key, Session session)
        {
            if (session == null) return;
            var isBrowser = (System.Environment.OSVersion.Platform == PlatformID.Other);
            if (isBrowser)
            {
                var client = new HttpClient();
                client.BaseAddress = new Uri("https://localhost:7095/");
                await client.PutAsJsonAsync<Session>("state", session);
            }
            else
            {
                Replace(session, _sessions[key]);
            }
        }
```

If this code is running on the browser, an HTTP request is made to push the state from the client to the server.

```c#
    await client.PutAsJsonAsync<Session>("state", session);
```

This invokes the `Put` method of the `StateController` on the web server:

```c#
        [HttpPut(Name = "UpdateState")]
        public async Task Put(Session updatedSession)
        {
            var httpContext = _contextAccessor.HttpContext;
            var sessionId = httpContext.Request.Cookies["sessionId"];
            await _sessionList.UpdateSession(sessionId, updatedSession);
        }
```

This method gets the current cookie from `HttpContext`, and then updates the current user state by calling the `UpdateSession` method of the `SessionManager` service.

Notice that the `UpdateSession` method has two code paths: one when running in the browser, and another when running on the server. When running on the server, this method replaces the old session data with the updated data:

```c#
    Replace(session, _sessions[key]);
```

I'm trying to avoid replacing the server _instance_ of the `session` field so I maybe can use some event to notify pages that the values have changed. Thus far this isn't working.

> ⚠️ This is important, because there's a timing issue that happens here. When the user navigates away from the wasm-rendered page and that page's `Dispose` method runs to invoke the API to update the server state, _the server page has already rendered_ with the old state.

## Remaining Issues

As I noted at the start of this post, what I'm showing here is a rough draft of an idea, and it has some timing issues that cause glitches. And it might have some security issues due to the use of the cookie for `sessionId`.

Certainly on the server, the use of a singleton server like `SessionManager` could be an issue, because malicious code running on the server could access other user's state.

Also, when using server-rendered pages it is possible for Blazor apps to run on server farms, and so maintaining per-user state in _server_ memory is problematic. I think that's easily addressed though, by altering `SessionManager` to pull/push per-user state from an external store such as REDIS or a database.

## Conclusion

This code does establish that the _basic_ concept of flowing state between Blazor pages when using automatic rendering modes is possible. It also reveals some issues that need to be resolved before such a solution would be useful in a real app scenario.

Right now, it seems to me that anyone building "real" apps using Blazor should probably stick with server-rendered or wasm-rendered pages throughout their entire app, and avoid the use of automatic render modes except in very limited scenarios.
