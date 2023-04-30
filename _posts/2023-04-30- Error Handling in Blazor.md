---
layout: post
title:  Error handling in Blazor
date:   2023-04-30
description: What to do when components go rouge.
tags: Error-Handling til
categories: Blazor
---
I ran into an interesting scenario the other day whilst using a 3rd party library to plot some grids. I accidentally discovered an edge case which raised an exception in the library in specific scenarios. No problem, normally if the issue is intermittent/unknown one could just wrap up the problematic code in try catch until you've figured out the issue. I then discovered that Blazor doesn't have a like-for-like `try catch` block for handling errors while rendering components which led me to today's discussion on error handling in Blazor.

## The Error Boundary Component

[Microsoft](https://learn.microsoft.com/en-us/aspnet/core/blazor/fundamentals/handle-errors?view=aspnetcore-7.0) has done us a solid and actually provided something for us, the `ErrorBoundary` component. Let's spin up a new Blazor server with `dotnet new blazorserver` to see this in action. First, we'll raise an exception on the counter page i.e.

```
@code {
    private int currentCount = 0;

    private void IncrementCount()
    {
        throw new Exception("The system is offline");
    }
}
```
At this point, hitting the `Click Me` button would create the dreaded error UI ribbon.

{% include figure.html path="assets/img/ErrorRibbon.png" class="img-fluid rounded z-depth-1" %}


The `ErrorBoundary` component can be placed somewhere central like the main layout to ensure errors are handled. The code below is an example of how you might handle a scenario like this.

```
<article class="content px-4">
    <ErrorBoundary>
        <ChildContent>
                @Body
            </ChildContent>
            <ErrorContent>
                <p class="errorUI">Well...this is awkward</p>
            </ErrorContent>    
    </ErrorBoundary>
</article>
```

Clicking on `Click Me` results in the following message to be displayed without the error ribbon at the bottom of the screen.

{% include figure.html path="assets/img/Awkward.png" class="img-fluid rounded z-depth-1" %}

## Some notes
Important to note in the above example is that the exception is still considered unhandled. Don't consider this a free pass to never deal with exceptions! Additionally, Microsoft suggests that narrow scoping of the error boundaries is advisable. As an example, let's add in a component called `MyComponent` which is added to the DOM when the `Click Me to Add MyComponent` is clicked. The component will raise an exception on initialization and we'll catch this with an Error boundary. The code for this is shown below.

**Adding MyComponent**
```
<h3>Hello World</h3>

@code {
    protected override void OnInitialized()
    {
        throw new Exception("Test exception");
    }
}
```

**Adding new Button to Counter Page**

```
@page "/counter"

<PageTitle>Counter</PageTitle>

<h1>Counter</h1>

<p role="status">Current count: @currentCount</p>

<div>
    <button class="btn btn-primary" @onclick="IncrementCount">Click me</button>    
</div>
<div class="mt-3">
    @if (_loaded)
    {
        
        <ErrorBoundary>
            <ChildContent>
                <MyComponent/>
            </ChildContent>
            <ErrorContent>
                <p>MyComponent failed to load</p>
            </ErrorContent>    
        </ErrorBoundary>
    }

    <button class="btn btn-primary" @onclick="AddMyComponent">Click me to Add MyComponent</button>
    
</div>

@code {
    private int currentCount = 0;

    private bool _loaded = false;
    private void IncrementCount()
    {
        throw new Exception("The system is offline");
    }

    private void AddMyComponent()
    {
        _loaded = true;
        StateHasChanged();
    }

}
```

Testing this out you should now see the error boundary being hit when you click on the new button.

## Wrapping up
And that's it! Feel free to take a look at the [documentation](https://learn.microsoft.com/en-us/aspnet/core/blazor/fundamentals/handle-errors?view=aspnetcore-7.0) has written up on additional cases for error handling. All code from today's article can be found on my [Github repo](https://github.com/thatstatsguy/til/tree/main/Blazor%20Error%20Handling).

Until next time :) 