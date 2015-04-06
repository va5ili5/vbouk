---
layout: post
title:  "Unit Test Model State in ASP.NET WebApi"
description: "Unit testing the model validations in ASP.NET WebApi"
date:   2015-04-06
author: Boukonis Vasileios
tags:
- WebApi
- CSharp
categories: Unit Test
---


For the past few days I was writing an ASP.NET Api and I wanted to write some unit tests to test if the model passed to the controller action is valid. Seems a trivial task ...but it's not. Let's create a WebApi Controller called Register with Action Register and a model called Customer with some validations

{% highlight html %}
{% raw %}
public class RegisterController : ApiController
{
    public IHttpActionResult Register(Customer customer)
    {
        return Ok();
    }
}

public class Customer
{
    [Required]
    public string FirstName { get; set; }

    [Required]
    public string LastName { get; set; }

    [Required]
    [EmailAddress]
    public string Email { get; set; }

    [Required]
    [RegularExpression(@"^([a-zA-Z0-9 .&'-]+)$", ErrorMessage = "Invalid Password")]
    [DataType(DataType.Password)]
    public string Password { get; set; }

    [Required]
    [Compare("Password", ErrorMessage = "Password and Confirm Password do not match")]
    [DataType(DataType.Password)]
    public string ConfirmPassword { get; set; }
}
{% endraw %}
{% endhighlight %}

now lets write a unit test and pass an invalid model to registrer action. The following test is expected to be succeed but it fails. Even if the model we pass is invalid no validation errors added to the model state dictionary (represented by the ModelState property of the Controller class).

{% highlight html %}
{% raw %}
public void If_FirstName_Is_Empty_The_Error_Should_Be_The_FirstName_Field_Is_Required()
{
    // Arrange
    var customer = new Customer
    {
        FirstName = string.Empty, //invalid. FirstName field is required
        LastName = "Doe",
        Email = "test@mail.com",
        Password = "testpass",
        ConfirmPassword = "testpass",
    };

    //Act
    var controller = new RegisterController();
    var actionResult = controller.Register(customer);
    int count = controller.ModelState.Values.Count; //get the number of errors added to ModelState

    //Assert
    Assert.IsFalse(controller.ModelState.IsValid);
    Assert.AreEqual(actionResult, typeof(BadRequestResult));
    Assert.AreEqual(count, 1);
    
}
{% endraw %}
{% endhighlight %}

But why? This happens because in the WebApi pipeline the ModelState validity is checked before the execution of the controller action. In the test above the `ModeState.IsValid` is executed before the `Register` action and so the model `customer` is valid. To solve this problem I implemented a new class that inherits from `ApiController` and in the class constructor I mocked the `HttpConfiguration` object and I added a validation method. This method just accept a model, validates it and returns any validation errors in the ModelState

{% highlight html %}
{% raw %}
public class ModelStateValidatorController : ApiController
{
    public ModelStateValidatorController()
    {
        Configuration = (new Mock<HttpConfiguration>()).Object;
    }

    public ModelStateDictionary Validation(object model)
    {
        Validate(model);
        return ModelState;
    }
}
{% endraw %}
{% endhighlight %}

I've also implemented an AssertHelper class for the assertions. This class contains the assertion functions

{% highlight html %}
{% raw %}
public static class AssertHelper
{
    public static void assertModelState(ModelStateDictionary modelState, string key, string errorMessage)
    {
        Assert.IsTrue(modelState.ContainsKey(key));
        Assert.IsTrue(modelState[key].Errors.Count == 1);
        Assert.IsTrue(modelState[key].Errors[0].ErrorMessage == errorMessage);
    }

    //overload. use this in case of multiple error per key
    public static void assertModelState(ModelStateDictionary modelState, string key, string[] errorMessage)
    {
        Assert.IsTrue(modelState.ContainsKey(key));
        for (int errorIndex = 0; errorIndex < errorMessage.Length; errorIndex++)
        {
            Assert.IsTrue(modelState[key].Errors.Count == errorMessage.Length);
            Assert.IsTrue(modelState[key].Errors[errorIndex].ErrorMessage == errorMessage[errorIndex]);
        }
    }
}
{% endraw %}
{% endhighlight %}

The overloaded function can be used in cases where multiple validation errors occured for the same property. Here is a use example

{% highlight html %}
{% raw %}
//combination of validation errors
[TestMethod]
public void Multiple_Validation_Errors()
{
    // Arrange
    var customer = new Customer
    {
        FirstName = string.Empty, //validation error
        LastName = string.Empty,  //validation error
        Email = string.Empty,     //two validation errors for the same property
        Password = "testpass",
        ConfirmPassword = "testpass",
    };

    // Act
    var modelState = new ModelStateValidatorController().Validation(customer);

    //testing keep the errors sum
    int countError = modelState.Select(x => x.Value.Errors.Count).Sum();

    //one validation error per key
    AssertHelper.assertModelState(modelState, "FirstName", "The FirstName field is required.");
    AssertHelper.assertModelState(modelState, "LastName", "The LastName field is required.");

    //multiple validation errors for one key. calls the overloaded function
    AssertHelper.assertModelState(modelState, "Email", new[]
    {
        "The Email field is required.",
        "The Email field is not a valid e-mail address."
    });
}
{% endraw %}
{% endhighlight %}

The source code is available at <a href="https://github.com/va5ili5/WebApi.ModelStateUnitTesting" target="_blank">github</a>