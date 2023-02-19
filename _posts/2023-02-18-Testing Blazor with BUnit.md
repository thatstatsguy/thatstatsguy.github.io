---
layout: post
title:  Testing Blazor with bUnit
date:   2023-02-19
description: A quick tour of testing Blazor components with bUnit.
tags: til
categories: Testing
---
For anyone who's written production grade code, it's a no brainer that testing your code is of utmost importance. Besides testing for code correctness, testing can be used for ensuring your code is solving it's intended problem (see TDD) as well as preventing new code unintentionally breaking your beloved application.

[bUnit](https://bunit.dev/) is a testing library aimed at helping developers test components in Blazor. In this article we'll be discussing:
    - How to set up bUnit tests for a basic Blazor application 
    - How to reduce boilerplate code using baked in functionality from bUnit.
    - Triggering events to evaluate behaviour.
    - Injecting dependencies and parameters into components

## Setting up a test project in Blazor
I'll be walking through the creation of a new project and adding new tests throughout the article - you can follow along and create the project from scratch or get the final project here. To do this tutorial you'll need the following installed:
    - The latest dotnet sdk
    - An IDE, such as VSCode.

We're going to use the Blazor template provided by Microsoft in this article, in the command line, navigate to the folder you want the project created in and type in `dotnet new Blazorserver -o bunit_examples` to create the test project.

To confirm there were no issues creating the project, navigate to the created folder and run your project.

```
cd bunit_examples
dotnet run
```
You should now see something similar to this displayed.
```
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://localhost:5056
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: C:\Users\thatstatsguy\examples\bunit_examples
```

Navigating to the link displayed in your terminal, you should now see the standard Blazor template application in your browser.

To stop your webserver, use `Ctrl + C` to stop the server. From this point onwards, I'm going to be working in VSCode to build tests for this application. If you use VSCode, simply type in `code .` to open VSCode in the current folder.

## Setting up your first bUnit test
We're going to be writing tests in a new file called `tests.cs`. Go ahead and create this in your IDE. Additionally, we're going to need to install a few packages in order to run our tests. Add the bunit and xunit packages to you project by running the following commands in the terminal.

```
dotnet add package bunit
dotnet add package xunit
dotnet add package xunit.runner.visualstudio
```

We're going to be running tests against the counter Blazor page. Note here that I said **page** and not **component** - while testing this out I realised you're able to tests pages as well as components. At the end the of day a page is just a component with a `@page "/counter"`directive so I suspect under the hood bUnit doesn't differentiate between the two. In your test file, add the following code.

```
namespace bunit_examples;

using Bunit;
using Xunit;
using Pages;

public class CounterShould
{
    [Fact]
    public void ShowCorrectHeading()
    {
        //test code goes here
    }
}

```

You'll see we've added a new class, added some package imports, and defined an empty test. As a simple first test, we're going to run a test against static content e.g. the counter heading at the top of the page. To create our test, we're first going to:
    - create a new test context, 
    - render in the page
    - look for the static context
    - set up a test to assert the content looks a certain way

Adding a few lines of code below will test that our heading on the counter page is indeed an H1 heading with the words Counter.
```
[Fact]
    public void ShowCorrectHeading()
    {
        var ctx = new TestContext();
        var cut = ctx.RenderComponent<Counter>();
        cut.Find("h1").MarkupMatches("<h1>Counter</h1>");
    }
```

Note here that `cut` is short for `Component under test` which is a commonly used term in testing. In the code above, the last line does the assertion that sets the criteria for the test to pass.

Running `dotnet test` in the terminal, you should see that one test was detected and now passes. Hooray - one test down.


## Reducing boilerplate code
bUnit provides a really neat way to reduce boilerplate code in order to get tests running through use of the `TestContext` class. The above test can be rewritten as

```
public class CounterShould : TestContext
{
    [Fact]
    public void ShowCorrectHeading()
    {
        var cut = RenderComponent<Counter>();
        cut.Find("h1").MarkupMatches("<h1>Counter</h1>");
    }
}
```
Notice how the class now inherits the TestContext class and makes no reference to the test context!

## Triggering events and evaluating behaviour
A common test you might want to run is checking what happens when a button is clicked. Let's test and see if the `counter` variable within the `Counter` page is indeed incremented when you click the `Click me` button. The following test does this

```
[Fact]
    public void IncrementCounterWhenClickMeClicked()
    {
        var cut = RenderComponent<Counter>();
        var button = cut.Find("button");
        button.Click();
        cut.Find("p").MarkupMatches("""<p role="status">Current count: 1</p>""");

    }
```

Notice how we're evaluating the state of the component through evaluation of the HTML produced. The reason for this is `currentCount` is a private property and therefore not available to test directly. However, if `currentCount` were a public property, we could test this directly using `cut.Instance.currentCount`.

## Injecting dependencies and component parameters
As your application grows, you may find your application depending on injected services. Luckily, bUnit provides an easy interface to inject dependencies within tests. To illustrate this, let's create a new page called "Special Counter". It's goal is exactly the same as the Counter page, but we'll be adding functionality to display a custom name in the header of the page as well as using an inject service to check if the number is a prime number.

Let's start by creating an interface for our service, add a new folder called `Interfaces` to your project. Within it, create a new file `IMathEngine.cs`. Our service is going to have a single method called `IsAPrime` which takes in a number and returns `true` if the number is a prime and false otherwise. Implementing the interface within the newly created file looks as follows.

```
namespace bunit_examples;

public interface IMathEngine
{
    public bool IsAPrime(int input);
}
```

For our application, we'll need to implement this interface in a class so we can inject the service in the program startup. We'll call this class `ExternalMathEngine.cs`. Implementing the interface on the class looks as follows.

```
namespace bunit_examples;

public class ExternalMathEngine : IMathEngine
{
    /// For the demo, I'm only implementing 
    /// logic for the first button click
    public bool IsAPrime(int input)
    {
        return input switch{
            1 => true,
            _ => false
        };
    }
}
```
Next is adding this service to the program startup which can be done by adding the following line to `Program.cs`

`builder.Services.AddSingleton<IMathEngine, ExternalMathEngine>();`

Next, let's create our new special counter page by creating `SpecialCounter.razor` in the Pages folder.

```
@page "/specialcounter/{Name}"
@inject IMathEngine mathEngine
<PageTitle>Special Counter</PageTitle>

<h1>@Name's Counter</h1>

<p role="status">Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

<p role="status">Is a prime number? @isPrime</p>

@code {
    [Parameter, EditorRequired]
    public string Name {get; set;} = "";
    private int currentCount = 0;

    private bool isPrime {get; set;} = false;

    private void IncrementCount()
    {
        currentCount++;
        isPrime = mathEngine.IsAPrime(currentCount);
    }
}
```

Notice we've added in a parameter `Name` and injected a `IMathEngine` service for us to use. When the button is clicked, the counter will update and the number will be checked for a prime number in the mathEngine service.

I've added the special counter page to the Nav bar to quickly get to this page by adding the following code in `NavMenu.razor`

```
<div class="nav-item px-3">
            <NavLink class="nav-link" href="specialcounter/thatstatsguy">
                <span class="oi oi-plus" aria-hidden="true"></span> Special Counter
            </NavLink>
        </div>
```
Feel free to `dotnet run` at this point to confirm the code works as expected. When you're happy, let's test the code. We want to do two things in this test:
    - inject in a component parameter
    - inject a mocked dependency.

Here's the final test which I've added to `tests.cs`

```
public class SpecialCounterShould : TestContext
{
    [Fact]
    public void UseInjectedParameterAndDependency()
    {
        var mockMathEngine = new Mock<IMathEngine>();
        mockMathEngine
            .Setup(x => x.IsAPrime(It.IsAny<int>()))
            .Returns(false);
        
        Services.AddSingleton<IMathEngine>(mockMathEngine.Object);

        var cut = RenderComponent<SpecialCounter>(parameters => 
            parameters
                .Add(p => p.Name, "TestUser"));
        
        //test that we've succesfully injected a parameter
        cut.Find("h1").MarkupMatches("<h1>TestUser's Counter</h1>");
        
        var button = cut.Find("button");
        button.Click();
        //testing that the counter still works
        cut
            .FindAll("p")
            .First(x=> x.InnerHtml.Contains("count"))
            .MarkupMatches("""<p role="status">Current count: 1</p>""");
        
        //testing that our injected dependency works as expected.
        cut
            .FindAll("p")
            .First(x=> x.InnerHtml.Contains("prime number"))
            .MarkupMatches("""<p role="status">Is a prime number? False""");
    }

}
```

In the first part of the test, we're just setting up the mock for the IMathEngine service. Before rendering the component we register the mocked service in our test context using `Services.AddSingleton`. Notice how easy it is to add parameters when rendering the component!

The final few lines are used to check that the injected parameter is being used correctly and that the injected service produces the correct result for the user.

## Conclusion
As you can see above, bUnit is a great way to quickly write up tests for your components in Blazor. I've really just scratched the surface of what you're able to do with bUnit, but have a look at things like [mocking components](https://bunit.dev/docs/providing-input/substituting-components.html?tabs=moq) and [injecting render fragments](https://bunit.dev/docs/providing-input/passing-parameters-to-components.html?tabs=csharp).

And that's it! Thanks for reading this far - I hope you find this article useful and will keep bUnit in mind next time you're working in Blazor!