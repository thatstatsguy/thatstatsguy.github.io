---
layout: post
title:  Integrating Fluent Validation into Blazor
date:   2023-04-31
description: Fluent Validation > Data Annotations
tags: Validation til
categories: Blazor
---
Validating user inputs is an important part of a typical application. As the saying goes, garbage in, garbage out so it's important to ensure that we're checking what users are supplying to us. There are two popular methods to do validation in ASP.net Core/Blazor (that I'm aware of) - Data Annotations and Fluent Validation. Today we're going to compare the two for the purpose of using in Blazor.

## The end goal
Today's example is simulating two user inputs for a data capturing service. The first user input is used for capturing Employee information and the second is for capturing Manager information. The conditions for valid inputs are as follows:
- The names of employees and managers can't be empty or longer than 50 characters and 
- The age of managers can't be zero or more than 100

Our example will showcase the validation capabilities of Data Annotations and Fluent Validation. Employee data will be validated by Data Annotations and Manager data by Fluent Validation.

## Validation with Data Annotations
All code from today's article can be found [here](https://github.com/thatstatsguy/til/tree/main/FluentValidationTests) if you'd like to jump ahead. I've spun up a new Blazor server with `dotnet new blazorserver` and cleaned out all the unimportant pages. First we need to add in the classes for our Employees and Managers.

```
public class Employee
{
    public string? Name { get; set; }

    public string? Surname { get; set; }
}
```

```
public class Manager
{
    public string Name { get; set; } = "";
    public string Surname { get; set; } = "";   
    public int Age { get; set; }
}
```

Adding in the validations with Data Annotations is fairly easy to do and can be seen below.

```
public class Employee
{
    [Required(ErrorMessage = "Name is required. It cannot be empty")]
    [MaxLength(50, ErrorMessage = "The maximum name length can only be 50 characters long")]
    public string? Name { get; set; }

    [Required(ErrorMessage = "Surname is required. It cannot be empty")]
    [MaxLength(50, ErrorMessage = "The maximum surname length can only be 50 characters long")]
    public string? Surname { get; set; }
}
```

To implement data validation, we'll make use of the `EditForm` which is baked into Blazor. I'm adding the input form on the index page as follows.

```
@page "/"

//Code omitted for brevity

<div class="mb-3">
    
    @if (_validEmployeeSubmitted)
    {
        <p>Valid Employee @_employee.Name @_employee.Surname Submitted</p>
    }
    else
    {
        <EditForm Model="_employee" OnValidSubmit="ValidEmployeeSubmission">
            <div class="form-group mt-1">
                <label for="employeeName">Please provide a employee name</label>
                <InputTextArea id="employeeName" rows="1" class="form-control h-50" @bind-Value="_employee.Name"/>
            </div>
            <div class="form-group mt-1">
                <label for="employeeSurname">Please provide a employee name</label>
                <InputTextArea id="employeeSurname" rows="1" class="form-control h-50" @bind-Value="_employee.Surname"/>
            </div>
            <button type="submit" class="btn btn-primary my-3">Submit Employee</button>
            <DataAnnotationsValidator/>
            <ValidationSummary/>
            
        </EditForm>
    }
</div>

@code
{
    private bool _validEmployeeSubmitted = false;
    
    private Employee _employee = new();
    
    private void ValidEmployeeSubmission()
    {
        _validEmployeeSubmitted = true;
        StateHasChanged();
    }
}
```

The components `<DataAnnotationsValidator/>` and `<ValidationSummary/>` are used to trigger the validations specified in the class definition. The line `OnValidSubmit = "ValidEmployeeSubmission"` ensures the data is submitted **only** when all validations have passed. Booting up the application, you should see the following if either name isn't specified.

{% include figure.html path="assets/img/Employee.png" class="img-fluid rounded z-depth-1" %}

So far so good, Data Annotations are quite simple to implement, but I must say they do make the definition of the class very messy. Additionally, the moment you have any complicated requirements, Data Annotations may fall short of requirements. Let's have a look at using Fluent Validation.

## Data Validation with Fluent Validation

The [FluentValidation](https://docs.fluentvalidation.net/en/latest/) library is a well known for implementing validations in .Net. It relies on defining a secondary class to specify fluent validation rules which can be used in several ways. Implementing this for our manager class, the validation rules can be seen below:

```
public class ManagerValidator : AbstractValidator<Manager>
{
    public ManagerValidator()
    {
        RuleFor(p => p.Name)
            .NotEmpty().WithMessage("Manager name needs to be specified")
            .MaximumLength(50).WithMessage("Manager name cannot be longer than 50 characters");

        RuleFor(p => p.Surname)
            .NotEmpty().WithMessage("Manager surname needs to be specified")
            .MaximumLength(50).WithMessage("Last name cannot be longer than 50 characters");
        
        RuleFor(p => p.Age)
            .NotNull()
            .GreaterThan(0)
            .Must(IsAgeValid).WithMessage("Age is too high!");
    }
    
    /// <summary>
    /// Custom logic validation
    /// </summary>
    /// <param name="age"></param>
    /// <returns></returns>
    private static bool IsAgeValid(int age)
    {
        return age < 100;
    }
}
```

As seen the code above, the class inherits from an `AbstractValidator` class and custom rules are provided in the constructor. The rules that are specified here are very readable and custom messages can easily be specified for each case that fails. Just for fun, I've shown how you can set up a custom method for checking Age Validation using a static method. This is not needed as you could use `.IsLessThan` instead.

We now need to implement the fluent validation for our Blazor inputs. The library [doesn't support this feature out of the box](https://docs.fluentvalidation.net/en/latest/blazor.html), however there is a library which it recommends to achieve this called [Blazored.FluentValidation](https://github.com/Blazored/FluentValidation). I've gone ahead and installed the latest package for this. 

Navigating back to our index page, the following code is set up:

```
@page "/"
@using FluentValidationTests.Classes
@using Blazored.FluentValidation

// Code omitted for brevity
    
<div class="my-1">
    @if (_validManagerSubmitted)
    {
        <p>Valid Manager @_manager.Name @_manager.Surname Submitted</p>
    }
    else
    {
        <EditForm Model="_manager" OnValidSubmit="ValidManagerSubmission">
            <div class="form-group mt-1">
                <label for="ManagerName">Please provide a manager name</label>
                <InputTextArea id="ManagerName" rows="1" class="form-control h-50" @bind-Value="_manager.Name"/>
            </div>
            <div class="form-group mt-1">
                <label for="ManagerSurname">Please provide a manager surname</label>
                <InputTextArea id="ManagerSurname" rows="1" class="form-control h-50" @bind-Value="_manager.Surname"/>
            </div>
            <div class="form-group mt-1">
                <label for="ManagerAge">Please provide age</label>
                <InputNumber id="ManagerAge" class="form-control h-50" @bind-Value="_manager.Age"/>
            </div>
            <button type="submit" class="btn btn-primary my-3">Submit Manager</button>
            <FluentValidationValidator/>
            <ValidationSummary/>
            
        </EditForm>
    }
</div>

@code
{
    // Code omitted for brevity
    private bool _validManagerSubmitted = false;
    
    private Manager _manager = new();
    
    private void ValidManagerSubmission()
    {
        _validManagerSubmitted = true;
        StateHasChanged();
    }
}
```
As seen above, the code really doesn't differ much to the first implementation except for the fact we're now using a new library and specifying a different component to use with `<FluentValidationValidator/>` replacing `<DataAnnotationsValidator/>`. Booting up the application again you'll see that the same style of validations are available as when we were taking inputs for employees.

## Final Thoughts
I really enjoyed using data validations while writing this article. To me, it makes sense that separation of concerns/single responsibility is at play with the class used to define the data contract and the validator class is used to enforce validation. Additionally, I've really only scratched the surface of what you can do with `FluentValidation`. Do yourself a favour and take a look at the examples mentioned in the links below. As always, the code for this article can be found on my [github repo](https://github.com/thatstatsguy/til/tree/main/FluentValidationTests).

Until next time :) 

## Further Reading
- [Handling empty strings with Data Annotations](https://stackoverflow.com/questions/23939738/how-can-i-use-data-annotations-attribute-classes-to-fail-empty-strings-in-forms)
- [MaxLength and MinLength](https://stackoverflow.com/questions/18276853/string-minlength-and-maxlength-validation-dont-work-asp-net-mvc)
- [Using Data Annotations in Blazor](https://learn.microsoft.com/en-us/aspnet/core/blazor/forms-and-input-components?view=aspnetcore-7.0)
- [Fluent Validation for Blazor](https://docs.fluentvalidation.net/en/latest/blazor.html)
- [Blazored Fluent Validation](https://github.com/Blazored/FluentValidation)
- [Fluent Validation Example with Async](https://github.com/Blazored/FluentValidation/tree/main/samples)