---
layout: post
title: Uno Platform and WebAssembly with CSLA
date: 2019-09-04
---

I recently blogged about the new support coming in [CSLA 5 for Blazor](https::/blog.lhotka.net/2019-09-02-BlazorSupportInCSLAV5). Shipping in .NET Core 3, [Blazor](https://blazor.net) is an HTML-based UI framework for WebAssembly.

There's another very cool UI framework for WebAssembly that sits on top of .NET: [Uno Platform](https://platform.uno).

![Uno Platform]({{ site.url }}/assets/Uno-Platform-And-WebAssembly-With-Csla-v5/uno-logo.png)
![CSLA](https://raw.github.com/MarimerLLC/csla/master/Support/Logos/csla%20win8_mid.png)

This UI framework relies on XAML (specifically the UWP dialect) to not only reach WebAssembly, but also Android and iOS, as well as Windows 10 of course. In short, the Uno approach allows you to write one codebase that can run on:

1. Any modern browser via WebAssembly
1. Android devices
1. iOS devices
1. Windows 10

Uno is similar to Xamarin Forms, in that it leverages the mono runtime to run code on iOS, Android, and WebAssembly. The primary differences are that Xamarin Forms has its own dialect of XAML (vs the UWP dialect), and doesn't target WebAssembly.

## Solution Structure

When you create an Uno solution you get a number of projects:

1. A project with the implementation shared across all platforms
1. An iOS project
1. An Android project
1. A UWP (Windows 10) project
1. A wasm (WebAssembly) project

You can see a working example of this in the [CSLA UnoExample sample app](https://github.com/MarimerLLC/csla/tree/master/Samples/UnoExample).

Following typical CSLA best practices, you'll also add a .NET Standard 2.0 Class Library project for your business domain types, and at least one other for your data access layer implementation. You'll also normally have an ASP.NET Core project that acts as your application server, because a typical business app needs to interact with server-side resources like databases.

![]({{ site.url }}/assets/Uno-Platform-And-WebAssembly-With-Csla-v5/solution-layout.png)

It is important to understand that, like Xamarin Forms, the platform-specific projects for iOS, Android, UWP, and wasm have almost no code. They exist to bootstrap the app on each type of platform. This is true of the app server code also, it is just there to provide an endpoint for the apps. All the _real_ code is in the shared project, your business library, and your data access library.

## Business Layer

The purpose of CSLA .NET is to provide a home for business logic. This is done by enabling the creation of business domain classes that encapsulate all business logic, and that's supported through consistent coding structures and a rules engine.

To improve developer productivity CSLA also abstracts many platform differences around data binding to various types of UI and interactions with app servers and data access layers.

> ℹ Much more information about CSLA is available in the free [Using CSLA 2019: CSLA Overview ebook](https://store.lhotka.net/using-csla-2019-csla-net-overview).

As a demo, nothing in the UnoExample is terribly complex, but it does demonstrate some very basic types of rule: informational messaging, warning messaging, and validation errors. It doesn't demonstrate things like calculated values, cross-object rules, etc. There's a lot more info about the rules engine in the [CSLA RuleTutorial sample](https://github.com/MarimerLLC/csla/tree/master/Samples/NET/cs/RuleTutorial).

The `BusinessLayer` project has a couple custom rules: `CheckCase` and `InfoText`, and it relies on the `Required` attribute from `System.ComponentModel.DataAnnotations` for a simple validation rule.

The `PersonEdit` class relies on these rules to enable basic creation and editing of a "person" domain concept:

```c#
  [Serializable]
  public class PersonEdit : BusinessBase<PersonEdit>
  {
    public static readonly PropertyInfo<int> IdProperty = RegisterProperty<int>(nameof(Id));
    public int Id
    {
      get { return GetProperty(IdProperty); }
      set { SetProperty(IdProperty, value); }
    }

    public static readonly PropertyInfo<string> NameProperty = RegisterProperty<string>(nameof(Name));
    [Required]
    public string Name
    {
      get { return GetProperty(NameProperty); }
      set { SetProperty(NameProperty, value); }
    }

    protected override void AddBusinessRules()
    {
      base.AddBusinessRules();
      BusinessRules.AddRule(new InfoText(NameProperty, "Person name (required)"));
      BusinessRules.AddRule(new CheckCase(NameProperty));
    }
    
    // data access abstraction below...
  }
```

The `PersonEdit` class also leverages the CSLA data portal to abstract the concept of an application server (or not) and prescribes how to interact with the data access layer. This code also leverages the new CSLA version 5 support for method-level dependency injection:

```c#
    // properties and business rules above...

    [Create]
    private void Create()
    {
      Id = -1;
      BusinessRules.CheckRules();
    }

    [Fetch]
    private void Fetch(int id, [Inject]DataAccess.IPersonDal dal)
    {
      var data = dal.Get(id);
      using (BypassPropertyChecks)
        Csla.Data.DataMapper.Map(data, this);
      BusinessRules.CheckRules();
    }

    [Insert]
    private void Insert([Inject]DataAccess.IPersonDal dal)
    {
      using (BypassPropertyChecks)
      {
        var data = new DataAccess.PersonEntity
        {
          Name = Name
        };
        var result = dal.Insert(data);
        Id = result.Id;
      }
    }

    [Update]
    private void Update([Inject]DataAccess.IPersonDal dal)
    {
      using (BypassPropertyChecks)
      {
        var data = new DataAccess.PersonEntity
        {
          Id = Id,
          Name = Name
        };
        dal.Update(data);
      }
    }
```

As is normal with CSLA, any interaction with the database is managed by the data access layer, and management of private fields or data within the domain object is managed in these data portal methods, for a clean separation of concerns.

## Data Access Layer

I am not going to dive into the data access layer (DAL) in any depth. The implementation in the sample is a pure in-memory model relying on LINQ and a `static` collection as a stand-in for a database `Person` table.

This approach is valuable for two scenarios:

1. A demo or example where I want the code to "just work" without forcing you to create a database
1. Integration testing scenarios where automated integration or user acceptance testing should be fast, and should be performed on a known set of data - without having to reinitialize a real database each time

This sample code isn't _quite_ at the level to accomplish the second goal, but it is easily achieved, especially given that the DAL is injected into the code via DI.

## App Server

The `AppServer` project is an ASP.NET Core project using the empty API template. So it just contains a `Controllers` folder and some configuration. It also _references_ the business and data access layers so they are available on the hosting server (maybe IIS, maybe in a Docker container in Kubernetes - thanks to .NET Core there's a lot of flexibility here).

### Configuration

In the `Startup` class CSLA and the DAL are added to the available services in the `ConfigureServices` method:

```c#
      services.AddCsla();
      services.AddTransient(typeof(DataAccess.IPersonDal), typeof(DataAccess.PersonDal));
```

And the `Configure` method configures CSLA:

```c#
      app.UseCsla();
```

It is also important to note that the app server is configured for CORS, because the wasm client runs in a browser, and isn't deployed from the app server. Without CORS configuration the app server would reject HTTP requests from the wasm client app.

### Data Portal Controllers

The reason for the app server is to expose endpoints for use by the client apps on iOS, Android, Windows, and wasm. CSLA has a component called the data portal that abstracts the entire idea of an app server and the network transport used to interact with any app server. As a result, an app server exposes "data portal endpoints" using components supplied by CSLA.

In the `Controllers` folder are `DataPortalController` and `DataPortalTextController` classes. The `DataPortalController` looks like this:

```c#
  [Route("api/[controller]")]
  [ApiController]
  public class DataPortalController : Csla.Server.Hosts.HttpPortalController
  {
    [HttpGet]
    public string Get()
    {
      return "Running";
    }
  }
```

The `Get` method is entirely optional. I tend to implement one because it makes it easier to troubleshoot my web server. But the _real_ behavior of a data portal endpoint is in its ability to handle a `POST` request, and that's already provided via the `HttpPortalController` base class.

The only difference in the `DataPortalTextController` is one line in the constructor:

```c#
    public DataPortalTextController()
    {
      UseTextSerialization = true;
    }
```

This is due to a current limitation of .NET running in WebAssembly in the browser: the `HttpClient` can't transfer binary data, only text data.

Normally the CSLA data portal transfers data as binary, often compressed. The whole point is to minimize data over the network and maximize performance. However, in the case of a wasm client app that binary data needs to be Base64 encoded into text for transfer over the network.

The result are two data portal endpoints, one binary, the other text. Otherwise they do the same thing.

## Uno UI Apps

The remaining projects in the solution rely on Uno to implement a common XAML-based UI with apps for iOS, Android, Windows, and wasm. Each of these apps is called a "head", and they all rely on a common implementation from the `UnoExample.Shared` project.

### Platform-Specific UI Projects

Each head project is a bootstrap for the client app, providing platform or operating system specific startup code and then handing off to the shared code for all the "real work". I'm not going to explore the various head apps, because they are template code - nothing I wrote.

> ⚠ The current Uno templates start with an `Assets` folder in the shared project. That'll cause compiler warnings from Android. Move that `Assets` folder from the shared project to the UWP project to solve this problem.

> ⚠ Do not update the console logging dependency in NuGet. It starts at version 1.1.1, and if you upgrade that'll cause a runtime issue with threading in the wasm head.

> ⚠ You may need to update the current target OS versions for Android and UWP, as the Uno template targets older versions of both.

### Shared Project

The way Uno works is that you implement nearly all of your application's UI logic in a shared project, and that project is compiled and deployed via each platform-specific head project. In other words, the code in the shared project is compiled into the iOS project, and into the Android project, and into the Windows project, and into the wasm project.

The result, is that as long as you don't do anything platform-specific, all your UI client code ends up in this one shared project.

### Configuration

When each platform-specific head app starts up it hands off control to the shared code as soon as any platform-specific details are handled. The entry point to the shared project is via `App.xaml` and any code behind in `App.Xaml.cs`.

CSLA is configured in the constructor of the `App` class. In the `UnoExample` code there are two sets of configuration, only one of which should be uncommented at a time.

#### Use Text-based Data Transfer for WebAssembly

As I mentioned when discussing the app server, the `HttpClient` object in .NET on WebAssembly can't currently transfer binary data. This means that CSLA needs to be configured to Base64 encode the data sent to the app server. In the constructor of the `App` class there is this code:

```c#
#if __WASM__
      Csla.DataPortalClient.HttpProxy.UseTextSerialization = true;
#endif
```

This uses a _compiler directive_ that is predefined by the wasm UI project to conditionally compile a line of code. In other words, this line of code is only compiled into the wasm project, and it is totally ignored for all the other project types.

#### Run "App Server" In-Process

After that, the constructor configures CSLA to run the "app server" code in-proc on the client. This is useful for demo purposes as it means each app will "just run" without you having to set up a real app server:

```c#
      CslaConfiguration.Configure().
        ContextManager(new Csla.Xaml.ApplicationContextManager());
      var services = new ServiceCollection();
      services.AddCsla();
      services.AddTransient(typeof(DataAccess.IPersonDal), typeof(DataAccess.PersonDal));
```

CSLA is configured to use an `ApplicationContextManager` designed for Uno. This kind of gets into the internals of CSLA, but CSLA relies on different context managers for different runtime environments, because in some cases context is managed by `HttpContext`, others on a per-thread basis, and still others via `static` fields.

The use of `ServiceCollection` configures the .NET Core dependency injection subsystem so CSLA can inject the correct implementation of the data access layer upon request.

> ℹ In a real app of this sort the data access layer would _not_ be deployed to, or available on, the client apps. It would only be deployed to the app server. But this is a demo, and it is far more convenient for you to be able to just run the various head apps without first having to set up an app server.

#### Invoke a Remote App Server

For the UWP and wasm heads you can easily comment out the "local app server" configuration and uncomment the configuration for the actual app server:

```c#
      string appserverUrl = "http://localhost:60223/api/dataportal";
      if (Csla.DataPortalClient.HttpProxy.UseTextSerialization)
        appserverUrl += "Text";
      CslaConfiguration.Configure().
        ContextManager(new Csla.Xaml.ApplicationContextManager()).
        DataPortal().
          DefaultProxy(typeof(Csla.DataPortalClient.HttpProxy), appserverUrl);
```

This code sets up a URL for the app server endpoint, appending "Text" to the controller name if text-based encoding should be used.

It then configures the application context manager, and also tells the CSLA data portal to use the `HttpProxy` type to communicate with the app server, along with the endpoint URL.

> ⚠ This only works with the Windows and wasm client apps. It won't work with the iOS or Android client apps because they won't have access to your `localhost`. If you want to use an app server with the Android or iOS apps you'll need to deploy the `AppServer` project to a real app server.

Notice that there is no configuration of the data access layer in this scenario. That's because the data access layer is only invoked on the app server, not on the client. This is a more "normal" scenario, and in this case the client-side head projects would _not_ reference the `DataAccess` project at all, so that code wouldn't be deployed to the client devices.

### UI Implementation

I'm not going to walk through the UI implementation in great detail. If you know XAML it is pretty straightforward, and if you don't know XAML there are tons of resources out there about how UWP apps work.

The really cool thing though, is how Uno manages to provide a largely consistent experience for UWP-style XAML across four different platforms. In particular, I found building this example quite enjoyable because I could use standard debugging against the UWP head, and then just run the code in the wasm head. _Usually_ that means the wasm UI "just works", and that's wonderful!

> ℹ In the cases where I had difficulties, the Uno team is active on the [Uno Gitter channel](https://gitter.im/uno-platform/Lobby) (like a public Teams or Slack), and between the team and the community I got over any hurdles very rapidly.

#### Loading List of People

The `MainPage` displays a list of people in the database, and has a button to add a new person. It also has a couple text output lines to show status and errors. I don't claim that this is a nice or pretty user interface, but it demonstrates important concepts 😊

The important bit is in the `RefreshData` method, where the CSLA data portal is used to retrieve the list of people:

```c#
    private async Task RefreshData()
    {
      this.InfoText.Text = "Loading ...";
      try
      {
        DataContext = await DataPortal.FetchAsync<PersonList>();
        this.InfoText.Text = "Loaded";
      }
      catch (Exception ex)
      {
        OutputText.Text = ex.ToString();
      }
    }
```

This demonstrates a key feature of CSLA: location transparency. The data portal call will work regardless of whether the data portal was configured to run the app server code in-process on the client, or remotely on a server. Even better, though this example app uses HTTP as a transport, you _could_ configure the data portal to use gRPC or RabbitMQ or other network transports, and the code here in the UI wouldn't be affected at all!

> ℹ When editing XAML or codebehind a page you might find that you get all sorts of Intellisense errors. This can be resolved by making sure the Project dropdown in the upper-left of the code editor is set to `UnoExample.UWP`. It'll often default to some other project, and that causes the errors.

![]({{ site.url }}/assets/Uno-Platform-And-WebAssembly-With-Csla-v5/code-editor-errors.png)

In this example notice that the Project dropdown is set to `UnoExample.Droid`, and that is why the code editor is all confused.

#### Editing a PersonEdit Object

The `EditPerson` page is a typical forms-over-data scenario. As the page loads it is data bound to a new or existing domain object:

```c#
    public int PersonId { get; set; } = -1;

    protected override async void OnNavigatedTo(NavigationEventArgs e)
    {
      if (e.Parameter != null)
        PersonId = (int)e.Parameter;

      PersonEdit person;
      this.InfoText.Text = "Loading ...";
      if (PersonId > -1)
        person = await DataPortal.FetchAsync<PersonEdit>(PersonId);
      else
        person = await DataPortal.CreateAsync<PersonEdit>();
      DataContext = person;
      this.InfoText.Text = "Loaded";
    }
```

The user is then able to edit the `Name` property, which has rules associated with it from the business library. Details about those rules (and other metastate about the domain object and individual properties) can be displayed to the user via data binding. The `Csla.Xaml.PropertyInfo` type provides data binding with access to all sorts of metastate about a specific property, and that is used in the XAML:

```xaml
    <TextBox Text="{Binding Name, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" />
    <csla:PropertyInfo x:Name="NameInfo" Property="{Binding Name, Mode=TwoWay}" />
    <TextBlock Text="{Binding ElementName=NameInfo, Path=Value}" />
    <TextBlock Text="{Binding ElementName=NameInfo, Path=IsValid}" />
    <TextBlock Text="{Binding ElementName=NameInfo, Path=InformationText}" Foreground="Blue" />
    <TextBlock Text="{Binding ElementName=NameInfo, Path=WarningText}" Foreground="DarkOrange" />
    <TextBlock Text="{Binding ElementName=NameInfo, Path=ErrorText}" Foreground="Red" />
```

Again, I don't claim to be a UX designer, but this does demonstrate some of the capabilities available to a UI developer given the rich metastate provided by CSLA. The result looks like this:

![]({{ site.url }}/assets/Uno-Platform-And-WebAssembly-With-Csla-v5/sample-ui.png)

Once the user has edited the values on the page, they can click the button to save the person. That also relies on CSLA to provide location transparent code:

```c#
    private async void SavePerson(object sender, RoutedEventArgs e)
    {
      try
      {
        var person = (PersonEdit)DataContext;
        await person.SaveAsync();
        var rootFrame = Window.Current.Content as Frame;
        rootFrame.Navigate(typeof(MainPage));
      }
      catch (Exception ex)
      {
        OutputText.Text = ex.ToString();
      }
    }
```

The `SaveAsync` method uses the data portal to have the DAL insert or update (or delete) the data associated with the domain object. In this case, once the object's data has been saved the user is navigated to the list of people.

## Conclusion

I am very excited about WebAssembly and the ability to run native .NET code in any modern browser. The Uno Platform offers a powerful UI framework for building apps that run native on mobile devices, in Windows, and in any modern browser.

CSLA .NET has always been about providing a home for your business logic, allowing you to write your logic once and to then leverage it on any platform or environment supported by .NET. Thanks to .NET running in WebAssembly, this means that you can take your business logic directly into any modern browser on any device or platform.