---
layout: post
title:  Updating Values in Blazor Components using Lifecycle Methods
date:   2023-05-07
description: 
tags: Lifecyle til Components
categories: Blazor
---
When starting out with Blazor, you really just want to dip your toes into water and not be buried with theory on [Blazor Component Lifecycles](https://learn.microsoft.com/en-us/aspnet/core/blazor/components/lifecycle?view=aspnetcore-7.0). Typically you'll start implementing some kind of spike application to test things out, but as that starter application turns into something bigger, you may find yourself scratching your head as to why certain [child components don't see to be updating](https://stackoverflow.com/questions/60096040/when-should-i-call-statehaschanged-and-when-blazor-automatically-intercepts-that). Now to be clear, I believe understanding the component lifecycle in Blazor is a must, but if you're starting out it can be a bit much to wrap your head around to do a basic application and I find that it's easy to fall into bad non-sustainable programming habits. Today's article is discussing all the ways you might consider updating values in Blazor along with ways you really shouldn't be using unless you have to!

## The example
We're building out an example application using the `dotnet new blazorserver` Blazor template. The idea is to place a child, grandchild and great grandchild component on the index page and see what happens when we want to pass updated values to the child components to process. In our example, we'll be passing a "Hello Word" message to our descendant components. All code from today's article can be found [here](https://github.com/thatstatsguy/til/tree/main/ChildComponentRefreshing).

## Baseline

As a baseline, I've gone ahead and created the `BaseChild`, `BaseGrandChild` and `BaseGreatGrandChild` components. An example of the `BaseChild` is below.

```
<h6>@_calculatedMessage</h6>
<BaselineGrandChild Message="@Message"/>
@code {
    
    [Parameter]
    public string Message { get; set; } = string.Empty;
    
    private string _calculatedMessage = "Child Component Value: ???";

    protected override void OnInitialized()
    {
        _calculatedMessage = $"Child Component Value: {Message}";
    }
}
```

The code between the components is almost identical. The only significant difference is whether our calculated field is child, grandchild or great grandchild. Notice how in the child component we've set up a `Grandchild` component which uses the `Message` parameter from the `Child` component. Using the linked code, navigate to the `index.razor` file. Note the `ChangeValues` method which is the onclick event for the change values button. In the method, we're explicitly changing the input parameter into the component as shown below.

```
<BaselineChild Message="@_baselineComponentMessage"></BaselineChild>
```

Booting up the example application in the linked code, you'll be able to click the `ChangeValues`, but nothing is going to happen. This is a great example of what I used to run into when starting out in Blazor. I assumed that Blazor would just automagically pick up the value and do something similar to a `StateHasChanged` method call to refresh all "downstream" components. 

## Manual Refreshing
Without knowing the component lifecycle, you may find yourself implementing your own refresh methods across all the components to handle this scenario where you want to update values in downstream components. The refresh method for a new component `RefreshChild` may look something like this.

```
public void Refresh(string message)
{
    _calculatedMessage = $"Child Component Value: {message}";
    //trigger refreshing on downstream components
    _refreshGrandChild.Refresh(message);
    StateHasChanged();
}
```

Note that you'll need to do three things to "get this to work":
1. You'll need to create your own refresh methods on **each** downstream component
2. The parent of the child component will have to go and manually trigger the refresh component i.e. the child component needs to manually trigger the refresh on the grandchild and so on.
3. Each component needs to call `StateHasChanged` in order to get the changes propagated to the DOM by Blazor.

This is "OK" for a small scale application where the complexity isn't an issue, but maintaining this sort of refreshing methodology in a larger system is a nightmare to maintain and is a bit of a code smell. It will "work" but it comes at a very high cost. Let's have a look at how we can use the component lifecycle to assist us.

## Using OnParameterSet
The Blazor component lifecycle has several "baked in" methods, one of which is the `OnParameterSet` method. This is automatically fired whenever a component parameter is updated. An additional benefit of this method is the `StateHasChanged` is implicitly called for you! A sample of such code is given below.

```
<h6>@_calculatedMessage</h6>
<OnParameterSetGrandChild Message="@Message"></OnParameterSetGrandChild>
@code {
    private string _calculatedMessage = "Child Component Value: ???";
    
    [Parameter]
    public string Message { get; set; } = string.Empty;

    protected override void OnParametersSet()
    {
        _calculatedMessage = $"Child Component Value: {Message}";
    }
}
```
Notice how simple our code has become. We set up the `OnParameterSet` method logic to deal with a new parameter value and that's it! Since the value of the the `OnParameterSetGrandChild` Message parameter is bound to the Message parameter of the child component, updating the value in the child component will automatically trigger `OnParameterSet` in `OnParameterSetGrandChild`. 

## Using Cascading Parameters
We can take this one step further, let's assume for a moment we need to update a value on the index page and it should automatically update the display in the `GreatGrandChild` component. Using the previous method, we'd need to propagate the `Message` parameter by defining the parameter in each of the components and manually passing it through. With `Cascading Parameters` we can bypass that and have the parameter be automatically available to all downstream components to use. The code for this in the `GreatGrandChild` component looks as follows:

```
<h6>@_calculatedMessage</h6>

@code {
    private string _calculatedMessage = "Great Grandchild Component Value: ???";

    [CascadingParameter]
    public string Message { get; set; } = string.Empty;


    protected override void OnParametersSet()
    {
        _calculatedMessage = $"Great Grandchild Component Value: {Message}";
    }
}
```

Not a huge difference to before, but now we're working with a `[CascadingParameter]` attribute instead of `[Parameter]`. In the index file, we define the cascading parameter "scope" as follows:

```
<CascadingValue Value="@_cascadingComponentMessage">
    <CascadeChild/>
</CascadingValue>
```
Since we've defined the Cascading parameters in this way, all "downstream" components defined in `CascadeChild` will be able to use this parameter.

## Wrapping Up
To summarise the above, it can really pay dividends to read up on the lifecycle components within Blazor, it can lead to better, cleaner and more efficient code. Give it a try with your next Blazor application!

Until next time :)