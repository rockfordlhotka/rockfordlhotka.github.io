---
layout: post
title: "ASP.NET Core and CSLA 5"
date: 2019-09-12
---

I've recently blogged about how CSLA 5 [supports Blazor](https://blog.lhotka.net/2019/09/02/BlazorSupportInCslaV5) and how it [supports Uno](https://blog.lhotka.net/2019/09/04/Uno-Platform-And-WebAssembly-With-Csla-v5). Clearly I am bullish on WebAssembly.

However, there remains a lot of value in server-side web development, and CSLA 5 also supports ASP.NET Core MVC, so I want to blog about that as well.

This post describes code in the [Samples/MvcExample](https://github.com/MarimerLLC/csla/tree/master/Samples/MvcExample) solution in the CSLA repo.

## Configuration

As is typical with ASP.NET, the first step to getting anything working is to configure the app in the `Startup.cs` class.

### ConfigureServices

In the `ConfigureServices` method three CSLA-related things are configured:

```c#
      services.AddMvc(options =>
      {
        options.ModelBinderProviders.Insert(0, new Csla.Web.Mvc.CslaModelBinderProvider());
      }).SetCompatibilityVersion(CompatibilityVersion.Version_2_2);

      services.AddCsla();
      services.AddTransient(typeof(DataAccess.IPersonDal), typeof(DataAccess.PersonDal));
```

The `AddMvc` call is enhanced to specify the use of the CSLA `CslaModelBinderProvider`. I'll discuss this later, but for now it is enough to understand that this is a model binder that enables data binding between MVC views and CSLA domain objects.

The `AddCsla` method does basic configuration of CSLA for use in ASP.NET Core.

The `AddTransient` method has nothing really to do with CSLA: it exists to support dependency injection of my data access layer upon request. 

However it does have a _little_ to do with CSLA, since one of the new features of CSLA 5 is support for dependency injection within your data portal methods. For example, here's the method in the `PersonEdit` business domain class that invokes the DAL to retrieve data:

```c#
    [Fetch]
    private void Fetch(int id, [Inject]DataAccess.IPersonDal dal)
    {
      var data = dal.Get(id);
      using (BypassPropertyChecks)
        Csla.Data.DataMapper.Map(data, this);
      BusinessRules.CheckRules();
    }
```

Notice how the `dal` parameter is injected into the method, based on the `IPersonDal` type being configured in `Startup.cs`.

### Configure

In the `Configure` method of `Startup.cs` there's one line of code for CSLA:

```c#
      app.UseCsla();
```

This configures CSLA for use in ASP.NET Core.

With the configuration complete it is easy to leverage the CSLA support for ASP.NET Core MVC.

## Creating Controllers

The `BusinessLibrary` project contains all the business domain types, and the `DataAccess` project implements the data access layer. Those are all standard CSLA implementations of types, so I won't focus on them in this post, other than to note that the `PersonEdit` class includes a number of business rules, and we'll want to show the results of those rules to the end user.

The web project has a `PersonController` and associated set of CRUD views, created using the standard Visual Studio tooling for creating controllers and views.

![]({{ site.url }}/assets/2019-09-12-AspCore-Csla5/web-project.png)

The majority of the code and Razor markup in those files was created by Visual Studio, not by hand. However, there are some hand-crafted parts.

### Controller Base Type

The `PersonController` class inherits from a base class provided by CSLA:

```c#
  public class PersonController : Csla.Web.Mvc.Controller
```

This base class adds some helper methods designed to simplify interaction with CSLA domain objects. 

### Model Binding

Remember in `Startup.cs` we configured a CSLA model binder. That model binder enables business domain types to be data bound to the views on postback events.

You can see the controller helper methods and model binding in use in the postback `Edit` method in the controller:

```c#
    // POST: Person/Edit/5
    [HttpPost]
    [ValidateAntiForgeryToken]
    public async Task<ActionResult> Edit(int id, PersonEdit person)
    {
      try
      {
        LoadProperty(person, PersonEdit.IdProperty, id);
        if (await SaveObjectAsync<PersonEdit>(person, true))
          return RedirectToAction(nameof(Index));
        else
          return View(person);
      }
      catch
      {
        return View(person);
      }
    }
```

Notice that a `PersonEdit` type is a parameter of the method. This is possible because the CSLA model binder was used to map the postback data from the browser into a domain object. A special model binder is required because CSLA domain objects implement business rules, authorization rules, and other behaviors that make simple binding impossible.

### LoadProperty Helper Method

The `LoadProperty` method allows you to put a value into a property, even if that is normally a readonly property. For example, the `Id` property is normally readonly because it is a primary key from the database. In the stateless web world however, it is necessary to "reload" that property with the value from the browser on postback.

The alternative would be to reload the domain object from the database. And sometimes that might be the right answer, but it does require a database call to get the data - and we (at least in theory) already have all the data in the postback from the browser.

So in this case the data from the browser is used to load the `PersonEdit` object via model binding, and to provide the `Id` property value to the object via `LoadProperty`.

### SaveObjectAsync Helper Method

The `SaveObjectAsync` method is part of the controller base class. It abstracts the process of saving a domain object, and displaying any validation rule results, or exception results, to the end user by integrating with the standard ASP.NET MVC validation display infrastructure.

ASP.NET MVC includes functionality to project client-side JavaScript code based on `DataAnnotations` attributes, and to display errors generated server-side on postback. The `SaveObjectAsync` method leverages that pre-built functionality, along with the powerful CSLA rules engine, to display rule results way beyond the limited capabilities of `DataAnnotations` attributes (though they are supported as well).

## Creating Views

The various views in the `Views/Person` folder are all generated by Visual Studio. The CSLA domain types expose bindable properties, and hide metastate properties such as `IsValid`, so the Visual Studio view generation tooling works as expected.

### Standard Validation

Notice, in the `Edit` view, how the generated markup includes an `asp-validation-for` element:

```html
        <label asp-for="Name" class="control-label"></label>
        <input asp-for="Name" class="form-control" />
        <span asp-validation-for="Name" class="text-danger"></span>
```

Thanks to the `SaveAsyncObject` helper method, _all_ validation rules that are associated with the `Name` property will be shown to the user via this built-in MVC mechanism.

That includes `DataAnnotation` attribute rules, plus any other rules created using the CSLA rules engine. For example, the `PersonEdit` class includes a rule that the `Name` property can't contain a "Z".

![]({{ site.url }}/assets/2019-09-12-AspCore-Csla5/no-z.png)

This is powerful, because with no extra code in the controller or view, all the power of the CSLA rules engine comes into play when building an MVC web site.

### CSLA Html Extensions

A CSLA business domain object exposes a rich set of metastate about the object and each property, metastate that goes way beyond the basic validation supported by MVC. To help you leverage that metastate while creating a view, CSLA provides extensions to the standard `Html` object in Razor.

In the `Edit` view the complete markup for the `Name` property looks like this:

```html
      <div class="form-group">
        <label asp-for="Name" class="control-label"></label>
        <input asp-for="Name" class="form-control" />
        <span asp-validation-for="Name" class="text-danger"></span>
        <span class="text-danger">@Html.ErrorFor(model => model.Name)</span>
        <span class="text-warning">@Html.WarningFor(model => model.Name)</span>
        <span class="text-info">@Html.InformationFor(model => model.Name)</span>
      </div>
```


`WarningFor` displays warning messages, and `InformationFor` displays information messages. These are features of CSLA, designed to provide a richer experience to the user.

The resulting user experience, with a number of these triggered, looks like this:

![]({{ site.url }}/assets/2019-09-12-AspCore-Csla5/all-validation.png)

In practice you use _either_ `asp-validation-for` or `Html.ErrorFor`. The difference is that `asp-validation-for` displays one (of possibly many) validation errors, while `ErrorFor` displays all validation errors for the property. In this example you see the error text twice because I'm demonstrating both options.

As you can see, it is possible to create a much richer user experience by tapping into the `Html` extensions and leveraging the CSLA rules engine.

## Conclusion

The functionality described in this post isn't really new. CSLA has supported ASP.NET MVC for many years. However, CSLA 5 brings this support forward to ASP.NET Core 3 for use in modern server-side web development.
