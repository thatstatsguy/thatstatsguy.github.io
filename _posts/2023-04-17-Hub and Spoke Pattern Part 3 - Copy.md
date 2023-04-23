---
layout: post
title:  Hub and Spoke Pattern in Blazor - Part 3
date:   2023-04-16
description: A MediatR approach
tags: Blazor C#
categories: Patterns
---
Welcome back to the final installment of the Hub and Spoke pattern series. We're building an exchange rate application which receives real time notifications via a rest API. In the previous article we set up our our service which directs notifications to all Exchange Rate components in our application. In this article we'll look at pre-existing libraries that could make our lives a little simpler.

## Will MediatR work?
MediatR is a well known dotnet library for dealing with notifications and requests. The documentation for the library doesn't fill me with hope given the following statement:

> or with an assembly:
>
> services.AddMediatR(cfg => cfg.RegisterServicesFromAssembly(typeof(Startup).Assembly));
>
> This registers:
>
> - `IMediator` as transient
>
> ...
> - INotificationHandler<> concrete implementations as transient

My understanding of this is that MediatR will try and tie itself to a single instance of a component (kinda like a service) which obviously won't work in a Blazor environment with multiple instances of the same component.

## Testing it out
All iterations of this code can be found in the git history on my [repo](https://github.com/thatstatsguy/DesignPatterns/tree/main/Hub%20and%20Spoke/Part3/CurrencyDisplay). I added the MediatR nuget package to my project and added the following to my `Program.cs` file.

```
builder.Services
    .AddMediatR(cfg => cfg.RegisterServicesFromAssembly(typeof(Program).Assembly))
```

In my `ExchangeRate.razor` file I've tweaked the code as follows:

```
@using CurrencyDisplay.DTO
@using System.Threading
@using CurrencyDisplay.Interfaces
@using CurrencyDisplay.Services
@using MediatR
@implements INotificationHandler<PriceUpdate>

<div class="card m-3">
  <div class="card-body">
      <p>From: @FromCurrency</p>
      <p>To: @ToCurrency</p>
      <p>ExchangeRate: @_exchangeRate</p>
  </div>
</div>

@code {
    
    [Parameter, EditorRequired]
    public string FromCurrency { get; set; }
    [Parameter, EditorRequired]
    public string ToCurrency { get; set; }
    
    private double _exchangeRate;
    
    public Task Handle(PriceUpdate priceUpdate, CancellationToken cancellationToken)
    {
        if (FromCurrency != priceUpdate.FromCurrency || ToCurrency != priceUpdate.ToCurrency) return Task.CompletedTask;
        _exchangeRate = priceUpdate.ExchangeRate;
        InvokeAsync(StateHasChanged);
        return Task.CompletedTask;
    }
}
```

Testing the application, my worries are realised. Placing a breakpoint in the `Handle` method on the if statement, it seems like a new component is being created which doesn't correspond to any of the component instances I've created.

{% include figure.html path="assets/img/HubSpokeMediatR.png" class="img-fluid rounded z-depth-1" %}

## MediatR.Courier to the rescue?
I stumbled across the [MediatR.Courier](https://github.com/KuraiAndras/MediatR.Courier) library while trying to resolve [this issue](https://github.com/jbogard/MediatR/issues/564) as a result of trying slightly different implementation mentioned in [this article](https://dotnetcoretutorials.com/the-mediator-pattern-part-3-mediatr-library/).

The documentation on the library on what makes this any different to MediatR's `INotificationHandler` confirms my previous suspicion. 
> Courier is a simple implementation of the second choice: Let some middleman handle notifications from MediatR and then pass those notifications to subscribers

Implementation details require only a slight add on to the existing implementation. After installing the nuget package, the required change to the MediatR service registration is 

```
builder.Services
    .AddMediatR(cfg => cfg.RegisterServicesFromAssembly(typeof(Program).Assembly))
    .AddCourier(typeof(Program).Assembly);
```

In the `ExchangeRate.razor` file, the following code changes are made.

```
@using CurrencyDisplay.DTO
@using System.Threading
@using CurrencyDisplay.Interfaces
@using CurrencyDisplay.Services
@using MediatR.Courier
@inject ICourier Courier
<div class="card m-3">
  <div class="card-body">
      <p>From: @FromCurrency</p>
      <p>To: @ToCurrency</p>
      <p>ExchangeRate: @_exchangeRate</p>
  </div>
</div>

@code {
    
    [Parameter, EditorRequired]
    public string FromCurrency { get; set; }
    [Parameter, EditorRequired]
    public string ToCurrency { get; set; }
    
    private double _exchangeRate;
    
    void HandleNotification(PriceUpdate priceUpdate, CancellationToken cancellationToken)
    {
        if (FromCurrency != priceUpdate.FromCurrency || ToCurrency != priceUpdate.ToCurrency) return;
        _exchangeRate = priceUpdate.ExchangeRate;
        InvokeAsync(StateHasChanged);
    }

    protected override void OnAfterRender(bool firstRender)
    {
        if (firstRender)
        {
            Courier.Subscribe<PriceUpdate>(HandleNotification);
        }
    }
}
```

The code works again (hooray!). However, this doesn't differ a huge amount from the original implementation. This would be a good option for your if you're not phased what this is doing in the background and you don't need any bespoke flexibility other than just basic events.


## Wrapping up
The Courier library goes on to suggest other libraries which kind of do the same thing that the Courier library does. For now it looks like this library and the "home grown" approach is what we'd need to use - that being said, it's not all bad since it got our code working! If you find any other libraries on this topic which may make this simple, be sure to let me know!

Until next time :) 

## Further reading
- [dotnet core tutorials](https://dotnetcoretutorials.com/the-mediator-pattern-part-3-mediatr-library/)