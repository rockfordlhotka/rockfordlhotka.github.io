---
title: Accessing User Identity on a Blazor Wasm Client
abstract: ''
keywords: ''
categories: ''
weblogName: Rockford Lhotka
postId: 
postDate: 2024-10-13T13:50:42.0008829-05:00
postStatus: publish
dontInferFeaturedImage: false
dontStripH1Header: false
---
On the server, Blazor authentication is fairly straightforward because it uses the underlying ASP.NET Core authentication mechanism.

I'll quickly review server authentication before getting to the WebAssembly part so you have an end-to-end understanding.

I should note that this post is all about a Blazor 8 app that uses per-component rendering, so there is an ASP.NET Core server hosting Blazor server pages, and there may also be pages using `InterativeAuto` or `InteractiveWebAssembly` that run in WebAssembly on the client device.

## Blazor Server Authentication

Blazor Server components are running in an ASP.NET Core hosted web server environment. This means that they can have access to all that ASP.NET Core has to offer.

For example, a server-static rendered Blazor server page can use HttpContext, and therefore can use the standard ASP.NET Core `SignInAsync` and `SignOutAsync` methods like you'd use in MVC or Razor Pages.

### Blazor Login Page

Here's the razor markup for a simple `Login.razor` page from a Blazor 8 server project with per-component rendering:

```html
@page "/login"

@using BlazorHolWasmAuthentication.Services
@using Microsoft.AspNetCore.Authentication
@using Microsoft.AspNetCore.Authentication.Cookies
@using System.Security.Claims

@inject UserValidation UserValidation
@inject IHttpContextAccessor httpContextAccessor
@inject NavigationManager NavigationManager

<PageTitle>Login</PageTitle>

<h1>Login</h1>

<div>
  <EditForm Model="userInfo" OnSubmit="LoginUser" FormName="loginform">
      <div>
          <label>Username</label>
          <InputText @bind-Value="userInfo.Username" />
      </div>
      <div>
          <label>Password</label>
          <InputText type="password" @bind-Value="userInfo.Password" />
      </div>
      <button>Login</button>
  </EditForm>
</div>

<div style="background-color:lightgray">
  <p>User identities:</p>
  <p>admin, admin</p>
  <p>user, user</p>
</div>

<div><p class="alert-danger">@Message</p></div>
```

This form uses the server-static form of the `EditForm` component, which does a standard postback to the server. Blazor uses the `FormName` and `OnSubmit` attributes to route the postback to a `LoginUser` method in the code block:

```csharp
@code {

    [SupplyParameterFromForm]
    public UserInfo userInfo { get; set; } = new();

    public string Message { get; set; } = "";

    private async Task LoginUser()
    {
        Message = "";
        ClaimsPrincipal principal;
        if (UserValidation.ValidateUser(userInfo.Username, userInfo.Password))
        {
            // create authenticated principal
            var identity = new ClaimsIdentity("custom");
            var claims = new List<Claim>();
            claims.Add(new Claim(ClaimTypes.Name, userInfo.Username));
            var roles = UserValidation.GetRoles(userInfo.Username);
            foreach (var item in roles)
                claims.Add(new Claim(ClaimTypes.Role, item));
            identity.AddClaims(claims);
            principal = new ClaimsPrincipal(identity);

            var httpContext = httpContextAccessor.HttpContext;
            if (httpContext is null)
            {
                Message = "HttpContext is null";
                return;
            }
            AuthenticationProperties authProperties = new AuthenticationProperties();
            await httpContext.SignInAsync(
                CookieAuthenticationDefaults.AuthenticationScheme,
                principal,
                authProperties);

            NavigationManager.NavigateTo("/");
        }
        else
        {
            Message = "Invalid credentials";
        }
    }


    public class UserInfo
    {
        public string Username { get; set; } = string.Empty;
        public string Password { get; set; } = string.Empty;
    }
}
```

The username and password are validated by a `UserValidation` service. That service returns whether the credentials were valid, and if they were valid, it returns the user's claims.

The code then uses that list of claims to create a `ClaimsIdentity` and `ClaimsPrincpal`. That pair of objects represents the user's identity in .NET.

The `SignInAsync` method is then called on the `HttpContext` object to create a cookie for the user's identity (or whatever storage option was configured in `Program.cs`).

From this point forward, ASP.NET Core code (such as a web API endpoint) and Blazor server components (via the Blazor `AuthenticationStateProvider` and `CascadingAuthenticationState`) all have consistent access to the current user identity.

### Blazor Logout Page

The `Logout.razor` page is simpler still, since it doesn't require any input from the user:

```html
﻿
@page "/logout"

@using Microsoft.AspNetCore.Authentication
@using Microsoft.AspNetCore.Authentication.Cookies

@inject IHttpContextAccessor httpContextAccessor
@inject NavigationManager NavigationManager

<h3>Logout</h3>

@code {
    protected override async Task OnInitializedAsync()
    {
        var httpContext = httpContextAccessor.HttpContext;
        if (httpContext != null)
        {
            var principal = httpContext.User;
            if (principal.Identity is not null && principal.Identity.IsAuthenticated)
            {
                await httpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
            }
        }
        NavigationManager.NavigateTo("/");
    }
}
```

The important part of this code is the call to `SignOutAsync`, which removes the ASP.NET Core user token, thus ensuring the current user has been "logged out" from all ASP.NET Core and Blazor server app elements.

### Configuring the Server

For the `Login.razor` and `Logout.razor` pages to work, they must be server-static (which is the default for per-component rendering), and `Program.cs` must contain some important configuration.

First, some services must be registered:

```csharp
builder.Services.AddHttpContextAccessor();

builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
  .AddCookie(); 
builder.Services.AddCascadingAuthenticationState();

builder.Services.AddTransient<UserValidation>();
```

The `AddHttpContextAccessor` registration makes it possible to inject an `IHttpContextAccessor` service so your code can access the `HttpContext` instance.

> ⚠️ Generally speaking, you should only access `HttpContext` from within a server-static rendered page.

The `AddAuthentication` method registers and configures ASP.NET Core authentication. In this case to store the user token in a cookie.

The `AddCascadingAuthenticationState` method enables Blazor server components to make use of cascading authentication state.

Finally, the `UserValidation` service is registered. This service is implemented by you to verify the user credentials, and to return the user's claims if the credentials are valid.

Some further configuration is required after the services have been registered:

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

### Enabling Cascading Authentication State

The `Routes.razor` component is where the user authentication state is made available to all Blazor components on the server:

```html
<CascadingAuthenticationState>
    <Router AppAssembly="typeof(Program).Assembly" AdditionalAssemblies="new[] { typeof(Client._Imports).Assembly }">
        <Found Context="routeData">
            <AuthorizeRouteView RouteData="routeData" DefaultLayout="typeof(Layout.MainLayout)" />
            <FocusOnNavigate RouteData="routeData" Selector="h1" />
        </Found>
    </Router>
</CascadingAuthenticationState>
```

Notice the addition of the `CascadingAuthenticationState` element, which cascades an `AuthenticationState` instance to all Blazor server components.

Also notice the use of `AuthorizeRouteView`, which enables the use of the authorization attribute in Blazor pages, so only an authorized user can access those pages.

### Adding the Login/Logout Links

The final step to making authentication work on the server is to enhance the `MainLayout.razor` component to add links for the login and logout pages:

```html
@using Microsoft.AspNetCore.Components.Authorization
@inherits LayoutComponentBase

<div class="page">
    <div class="sidebar">
        <NavMenu />
    </div>

    <main>
        <div class="top-row px-4">
            <AuthorizeView>
                <Authorized>
                    Hello, @context!.User!.Identity!.Name
                    <a href="logout">Logout</a>
                </Authorized>
                <NotAuthorized>
                    <a href="login">Login</a>
                </NotAuthorized>
            </AuthorizeView>
            </div>

        <article class="content px-4">
            @Body
        </article>
    </main>
</div>

<div id="blazor-error-ui">
    An unhandled error has occurred.
    <a href="" class="reload">Reload</a>
    <a class="dismiss">🗙</a>
</div>
```

The `AuthorizeView` component is used, with the `Authorized` block providing content for a logged in user, and the `NotAuthorized` block providing content for an anonymous user. In both cases, the user is directed to the appropriate page to login or logout.

At this point, all _server-side_ Blazor components can use authorization, because they have access to the user identity via the cascading `AuthenticationState` object.

This doesn't automatically extend to pages or components running in WebAssembly on the browser. That takes some extra work.

## Blazor WebAssembly User Identity

There is nothing built in to Blazor that automatically makes the user identity available to pages or components running in WebAssembly on the client device.

You should also be aware that there are possible security implications to making the user identity available on the client device. This is because any client device can be hacked, and so a bad actor could gain access to any `ClaimsIdentity` object that exists on the client device. As a result, a bad actor could get a list of the user's claims, if those claims are on the client device.

In my experience, if developers are using client-side technologies such as WebAssembly, Angular, React, WPF, etc. they've already reconciled the security implications of running code on a client device, and so it is probably not an issue to have the user's roles or other claims on the client. I will, however, call out where you can filter the user's claims to prevent a sensitive claim from flowing to a client device.

The basic process of making the user identity available on a WebAssembly client is to copy the user's claims from the server, and to use that claims data to create a copy of the `ClaimsIdentity` and `ClaimsPrincipal` on the WebAssembly client.

### A Web API for ClaimsPrincipal

The first step is to create a web API endpoint on the ASP.NET Core (and Blazor) server that exposes a copy of the user's claims so they can be retrieved by the WebAssembly client.

For example, here is a controller that provides this functionality:

```csharp
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;

namespace BlazorHolWasmAuthentication.Controllers;

[ApiController]
[Route("[controller]")]
public class AuthController(IHttpContextAccessor httpContextAccessor)
{
    [HttpGet]
    public User GetUser()
    {
        ClaimsPrincipal principal = httpContextAccessor!.HttpContext!.User;
        if (principal != null && principal.Identity != null && principal.Identity.IsAuthenticated)
        {
            // Return a user object with the username and claims
            var claims = principal.Claims.Select(c => new Claim { Type = c.Type, Value = c.Value }).ToList();
            return new User
            {
                Username = principal.Identity!.Name,
                Claims = claims
            };
        }
        else
        {
            // Return an empty user object
            return new User();
        }
    }
}

public class Credentials
{
    public string Username { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
}

public class User
{
    public string Username { get; set; } = string.Empty;
    public List<Claim> Claims { get; set; } = [];
}

public class Claim
{
    public string Type { get; set; } = string.Empty;
    public string Value { get; set; } = string.Empty;
}
```

This code uses an `IHttpContextAccessor` to access `HttpContext` to get the current `ClaimsPrincipal` from ASP.NET Core.

It then copies the data from the `ClaimsIdentity` into simple types that can be serialized into JSON for return to the caller.

Notice how the code doesn't have to do any work to determine the identity of the current user. This is because ASP.NET Core has already authenticated the user, and the user identity token cookie has been unpacked by ASP.NET Core before the controller is invoked.

The line of code where you could filter sensitive user claims is this:

```csharp
            var claims = principal.Claims.Select(c => new Claim { Type = c.Type, Value = c.Value }).ToList();
```

This line copies _all_ claims for serialization to the client. You could filter out claims considered sensitive so they don't flow to the WebAssembly client. Keep in mind that any code that relies on such claims won't work in WebAssembly pages or components.

In the server `Program.cs` it is necessary to register and map controllers.

```csharp
builder.Services.AddControllers(); 
```

and

```csharp
app.MapControllers();
```

At this point the web API endpoint exists for use by the Blazor WebAssembly client.

### Getting the User Identity in WebAssembly

Blazor always maintains the current user identity as a `ClaimsPrincpal` in an `AuthenticationState` object. Behind the scenes, there is an `AuthenticationStateProvider` service that provides access to the `AuthenticationState` object.

On the Blazor server we generally don't need to worry about the `AuthenticationStateProvider` because a default one is provided for our use.

On the Blazor WebAssembly client however, we must implement a custom `AuthenticationStateProvider`. For example:

```csharp
using Microsoft.AspNetCore.Components.Authorization;
using System.Net.Http.Json;
using System.Security.Claims;

namespace BlazorHolWasmAuthentication.Client;

public class CustomAuthenticationStateProvider(HttpClient HttpClient) : AuthenticationStateProvider
{
    private AuthenticationState AuthenticationState { get; set; } =
        new AuthenticationState(new ClaimsPrincipal());
    private DateTimeOffset? CacheExpire;

    public override async Task<AuthenticationState> GetAuthenticationStateAsync()
    {
        if (!CacheExpire.HasValue || DateTimeOffset.Now > CacheExpire)
        {
            var previousUser = AuthenticationState.User;
            var user = await HttpClient.GetFromJsonAsync<User>("auth");
            if (user != null && !string.IsNullOrEmpty(user.Username))
            {
                var claims = new List<System.Security.Claims.Claim>();
                foreach (var claim in user.Claims)
                {
                    claims.Add(new System.Security.Claims.Claim(claim.Type, claim.Value));
                }
                var identity = new ClaimsIdentity(claims, "auth_api");
                var principal = new ClaimsPrincipal(identity);
                AuthenticationState = new AuthenticationState(principal);
            }
            else
            {
                AuthenticationState = new AuthenticationState(new ClaimsPrincipal());
            }
            if (!ComparePrincipals(previousUser, AuthenticationState.User))
            {
                NotifyAuthenticationStateChanged(Task.FromResult(AuthenticationState));
            }
            CacheExpire = DateTimeOffset.Now + TimeSpan.FromSeconds(30);
        }
        return AuthenticationState;
    }

    private static bool ComparePrincipals(ClaimsPrincipal principal1, ClaimsPrincipal principal2)
    {
        if (principal1.Identity == null || principal2.Identity == null)
            return false;
        if (principal1.Identity.Name != principal2.Identity.Name)
            return false;
        if (principal1.Claims.Count() != principal2.Claims.Count())
            return false;
        foreach (var claim in principal1.Claims)
        {
            if (!principal2.HasClaim(claim.Type, claim.Value))
                return false;
        }
        return true;
    }

    private class User
    {
        public string Username { get; set; } = string.Empty;
        public List<Claim> Claims { get; set; } = [];
    }

    private class Claim
    {
        public string Type { get; set; } = string.Empty;
        public string Value { get; set; } = string.Empty;
    }
}
```

This is a subclass of `AuthenticationStateProvider`, and it provides an implementation of the `GetAuthenticationStateAsync` method. This method invokes the server-side web API controller to get the user's claims, and then uses them to create a `ClaimsIdentity` and `ClaimsPrincipal` for the current user.

This value is then returned within an `AuthenticationState` object for use by Blazor and any other code that requires the user identity on the client device.

One key detail in this code is that the `NotifyAuthenticationStateChanged` method is only called in the case that the user identity has changed. The `ComparePrincipals` method compares the existing principal with the one just retrieved from the web API to see if there's been a change.

It is quite common for Blazor and other code to request the `AuthenticationState` very frequently, and that can result in a lot of calls to the web API. Even a cache that lasts a few seconds will reduce the volume of repetitive calls significantly. This code uses a 30 second cache.

### Configuring the WebAssembly Client

To make Blazor use our custom provider, and to enable authentication on the client, it is necessary to add some code to `Program.cs` _in the client project_:

```csharp
using BlazorHolWasmAuthentication.Client;
using Marimer.Blazor.RenderMode.WebAssembly;
using Microsoft.AspNetCore.Components.Authorization;
using Microsoft.AspNetCore.Components.WebAssembly.Hosting;

var builder = WebAssemblyHostBuilder.CreateDefault(args);

builder.Services.AddScoped(sp => new HttpClient { BaseAddress = new Uri(builder.HostEnvironment.BaseAddress) });

builder.Services.AddAuthorizationCore();

builder.Services.AddScoped<AuthenticationStateProvider, CustomAuthenticationStateProvider>();

builder.Services.AddCascadingAuthenticationState();

await builder.Build().RunAsync();
```

The `CustomAuthenticationStateProvider` requires an `HttpClient` service, and relies on the `AddAuthorizationCore` and `AddCascadingAuthenticationState` to properly function.

## Summary

The preexisting integration between ASP.NET Core and Blazor on the server make server-side user authentication fairly straightforward.

Extending the authenticated user identity to WebAssembly hosted pages and components requires a little extra work: creating a controller on the server and custom `AuthenticationStateProvider` on the client.
