---
layout: post
title: Unit Testing CSLA Rules With Rocks
postDate: 2025-10-02T13:39:53.3750407-05:00
categories: []
tags: []
published: true
permalink: 
featuredImageUrl: 
---
One of the most powerful features of [CSLA .NET](https://cslanet.com) is its business rules engine. It allows you to encapsulate validation, authorization, and other business logic in a way that is easy to manage and maintain.

In CSLA, a rule is a class that implements `IBusinessRule`, `IBusinessRuleAsync`, `IAuthorizationRule`, or `IAuthorizationRuleAsync`. These interfaces define the contract for a rule, including methods for executing the rule and properties for defining the rule's behavior. Normally a rule inherits from an existing base class that implements one of these interfaces.

When you create a rule, you typically associate it with a specific property or set of properties on a business object. The rule is then executed automatically by the CSLA framework whenever the associated property or properties change.

The advantage of a CSLA rule being a class, is that you can unit test it in isolation. This is where the [Rocks](https://github.com/jasonbock/rocks) mocking framework comes in.

Rocks allows you to create mock objects for your unit tests, making it easier to isolate the behavior of the rule you are testing. You can create a mock business object and set up expectations for how the rule should interact with that object. This allows you to test the rule's behavior without having to worry about the complexities of the entire business object.

In summary, the combination of CSLA's business rules engine and the Rocks mocking framework provides a powerful way to create and test business rules in isolation, ensuring that your business logic is both robust and maintainable.

All code for this article can be found in this [GitHub repository](https://github.com/MarimerLLC/CslaHol) in Lab 02.

## Creating a Business Rule

As an example, consider a business rule that sets an `IsActive` property based on the value of a `LastOrderDate` property. If the `LastOrderDate` is within the last year, then `IsActive` should be true; otherwise, it should be false.

```csharp
﻿using Csla.Core;
using Csla.Rules;

namespace BusinessLibrary.Rules;

public class LastOrderDateRule : BusinessRule
{
  public LastOrderDateRule(IPropertyInfo lastOrderDateProperty, IPropertyInfo isActiveProperty) : base(lastOrderDateProperty)
  {
    InputProperties.Add(lastOrderDateProperty);
    AffectedProperties.Add(isActiveProperty);
  }

  protected override void Execute(IRuleContext context)
  {
    var lastOrderDate = (DateTime)context.InputPropertyValues[PrimaryProperty];
    var isActive = lastOrderDate > DateTime.Now.AddYears(-1);
    context.AddOutValue(AffectedProperties[1], isActive);
  }
}
```

This rule inherits from `BusinessRule`, which is a base class provided by CSLA that implements the `IBusinessRule` interface. The constructor takes two `IPropertyInfo` parameters: one for the `LastOrderDate` property and one for the `IsActive` property. The `InputProperties` collection is used to specify which properties the rule depends on, and the `AffectedProperties` collection is used to specify which properties the rule affects.

The `Execute` method is where the rule's logic is implemented. It retrieves the value of the `LastOrderDate` property from the `InputPropertyValues` dictionary, checks if it is within the last year, and then sets the value of the `IsActive` property using the `AddOutValue` method.

## Unit Testing the Business Rule

Now that we have our business rule, we can create a unit test for it using the Rocks mocking framework.

First, we need to bring in a few namespaces:

```csharp
﻿using BusinessLibrary.Rules;
using Csla;
using Csla.Configuration;
using Csla.Core;
using Csla.Rules;
using Microsoft.Extensions.DependencyInjection;
using Rocks;
using System.Security.Claims;
```

Next, we can use Rocks attributes to define the mock types we need for our test:

```csharp
[assembly: Rock(typeof(IPropertyInfo), BuildType.Create | BuildType.Make)]
[assembly: Rock(typeof(IRuleContext), BuildType.Create | BuildType.Make)]
```

These lines of code only need to be included once in your test project, because they are assembly-level attributes. They tell Rocks to create mock implementations of the `IPropertyInfo` and `IRuleContext` interfaces, which we will use in our unit test.

Now we can create our unit test method to test the `LastOrderDateRule`.

To do this, we need to arrange the necessary mock objects and set up their expectations. Then we can execute the rule and verify that it behaves as expected.

The rule has a constructor that takes two `IPropertyInfo` parameters, so we need to create mock implementations of that interface. We also need to create a mock implementation of the `IRuleContext` interface, which is used to pass information to the rule when it is executed.

```csharp
    [TestMethod]
    public void LastOrderDateRule_SetsIsActiveBasedOnLastOrderDate()
    {
      // Arrange
      var inputProperties = new Dictionary<IPropertyInfo, object>();

      using var context = new RockContext();

      var lastOrderPropertyExpectations = context.Create<IPropertyInfoCreateExpectations>();
      lastOrderPropertyExpectations.Properties.Getters.Name()
          .ReturnValue("name")
          .ExpectedCallCount(2);
      var lastOrderProperty = lastOrderPropertyExpectations.Instance();
      var isActiveProperty = new IPropertyInfoMakeExpectations().Instance();

      var ruleContextExpectations = context.Create<IRuleContextCreateExpectations>();
      ruleContextExpectations.Properties.Getters.InputPropertyValues().ReturnValue(inputProperties);
      ruleContextExpectations.Methods.AddOutValue(Arg.Is(isActiveProperty), true);

      inputProperties.Add(lastOrderProperty, new DateTime(2025, 9, 24, 18, 3, 40));

      // Act
      var rule = new LastOrderDateRule(lastOrderProperty, isActiveProperty);
      (rule as IBusinessRule).Execute(ruleContextExpectations.Instance());

      // Assert is automatically done by Rocks when disposing the context
    }
```

Notice how the Rocks mock objects have expectations set up for their properties and methods. This allows us to verify that the rule interacts with the context as expected. This is a little different from more explicit `Assert` statements, but it is a powerful way to ensure that the rule behaves correctly.

For example, notice how the `Name` property of the `lastOrderProperty` mock is expected to be called twice. If the rule does not call this property the expected number of times, the test will fail when the `context` is disposed at the end of the `using` block:

```csharp
        lastOrderPropertyExpectations.Properties.Getters.Name()
            .ReturnValue("name")
            .ExpectedCallCount(2);
```

This is a powerful feature of Rocks that allows you to verify the behavior of your code without having to write explicit assertions.

The test creates an instance of the `LastOrderDateRule` and calls its `Execute` method, passing in the mock `IRuleContext`. The rule should set the `IsActive` property to true because the `LastOrderDate` is within the last year.

When the test completes, Rocks will automatically verify that all expectations were met. If any expectations were not met, the test will fail.

This is a simple example, but it demonstrates how you can use Rocks to unit test CSLA business rules in isolation. By creating mock objects for the dependencies of the rule, you can focus on testing the rule's behavior without having to worry about the complexities of the entire business object.

## Conclusion

CSLA's business rules engine is a powerful feature that allows you to encapsulate business logic in a way that is easy to manage and maintain. By using the Rocks mocking framework, you can create unit tests for your business rules that isolate their behavior and ensure that they work as expected. This combination of CSLA and Rocks provides a robust and maintainable way to implement and test business logic in your applications.
