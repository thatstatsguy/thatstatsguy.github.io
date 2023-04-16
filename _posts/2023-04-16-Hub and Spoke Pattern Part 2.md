---
layout: post
title:  Hub and Spoke Pattern in Blazor - Part 2
date:   2023-04-16
description: A "roll your own" implementation
tags: Blazor C#
categories: Patterns
---
Hi and welcome back to this series of articles where I'm exploring the Hub and Spoke pattern. In this series of articles, I'm exploring the hub and spoke architecture by building an exchange rate application which receives real time notifications via a rest API. In the previous article we set up the API that would receive requests. It's now time to set up something that can receive real time notifications and send it on to all components listening for update events.

## The event handler
In this example I'm taking the example provided [here](https://github.com/jeffreypalermo/blazormvc) and stripping it down to it's most basic parts. The idea here is to register a service in which "interested" components can register their interest for a given event. In our example, the end goal is to set up exchange rate Razor components which will register themselves in order to listen for events.

First off we're defining an interface which the service will use. The interface will define functionality that components can use to register themselves and other components (like the API controller) can use to fire off notification events.

```
public interface IUiBus
{
    void Register(IListener listener);
    void UnRegister(IListener listener);
    IListener<T>[] GetListeners<T>() where T : IUiBusEvent;
    void Notify<T>(T notification) where T : IUiBusEvent;
    void UnRegisterAll();
}
```
where `IUiBusEvent` is defined as

```
public interface IUiBusEvent
{
    
}
```

This is a peculiar implementation detail from the original implementation which I quite like. Essentially, it forces the developer to mark the notification "payload" type as some information related to a UI Event. In a sense this is a clever way of self documentation as well as preventing random parts of the application sending through pointless notifications that would otherwise never be used.

Similar to the `IUiBus` interface, we need to define an interface that all interested parties will implement in order to handle events.

```
public interface IListener
{

}

public interface IListener<T> : IListener where T : IUiBusEvent
{
    void Handle(T theEvent);
}
```

A class called `VisualsUpdateBus` is defined which implements the interface and defines the logic used to register interested parties as well as receive notification events.

```
public class VisualsUpdateBus : IUiBus
{
    private readonly ISet<IListener> _listeners = new HashSet<IListener>();
    private readonly ILogger<VisualsUpdateBus> _logger;

    public VisualsUpdateBus(ILogger<VisualsUpdateBus> logger)
    {
        _logger = logger;
    }

    public void Register(IListener listener)
    {
        if (listener == null) return;
        _logger.LogDebug("Listener registering: {0}", GetObjectIdentifier(listener));

        _listeners.Add(listener);
    }

    //code redacted here for brevity - see github repo for full example

    public IListener<T>[] GetListeners<T>() where T : IUiBusEvent
    {
        IEnumerable<IListener> listeners = _listeners.Where(c => c is IListener<T>);
        IEnumerable<IListener<T>> enumerable = listeners.Select(listener => (IListener<T>)listener);
        return enumerable.ToArray();
    }

    public void Notify<T>(T notification) where T : IUiBusEvent
    {
        IListener<T>[] listeners = GetListeners<T>();
        string message = string.Format("Notifying {0} of {1} listeners with {2} on bus {3}",
            listeners.Length, _listeners.Count, notification.GetType().Name, GetHashCode());
        
        _logger.LogInformation(message);
        foreach (IListener<T> listener in listeners)
        {
            _logger.LogInformation("Notifying: {0} with {1}", 
                GetObjectIdentifier(listener), notification.GetType().Name);
            listener.Handle(notification);
        }
    }
    //code redacted here for brevity - see github repo for full example
}
```

As you'll see above, the register event takes an input object instance which implements the IListener interface and adds it into the `_listeners` hash set. When the notify method is received, all object instances implementing the given type have the `Handle` method called to notify them of the event.

Registering our new service can be done using `builder.Services.AddSingleton<IUiBus, VisualsUpdateBus>();` in the `Program.cs` file.

## Implementing an Exchange Rate Component
The price update razor component is nothing particularly exciting, it takes as input the "To" and "From" Currencies and has a private field for the exchange rate which can be updated.

```
@using CurrencyDisplay.DTO
@using System.Threading
@using CurrencyDisplay.Interfaces
@using CurrencyDisplay.Services
@implements CurrencyDisplay.Interfaces.IListener<PriceUpdate>
@inject IUiBus Bus
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
    
    public void Handle(PriceUpdate priceUpdate)
    {
        if (FromCurrency != priceUpdate.FromCurrency || ToCurrency != priceUpdate.ToCurrency) return;
        _exchangeRate = priceUpdate.ExchangeRate;
        InvokeAsync(StateHasChanged);
    }
    
    protected override Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            Bus.Register(this);    
        }
        
        return base.OnAfterRenderAsync(firstRender);   
    }
}
```
Notice that the given razor component implements the `IListener<PriceUpdate>` interface along with the associated `Handle` method. When notification events are fired off, the `Handle` method is called. If the "To" and "From" currencies match up, the component will update the exchange rate.

Additionally, also note that the `IUiBus` is implemented so that the component can register itself to receive notifications from the service. This is done in the `OnAfterRenderAsync` method. Make sure not to do this in `OnInitialised` as the method may be called more than once during the component lifecycle leading to your component being registered twice by accident.

## Firing Notification Events from the Controller
Since our UI eventing service is registered via the IOC container, it can be injected into the controller via dependency injection. Below is a snippet of the updated Controller code which fires off a notification event when the update action is called.

```
[ApiController]
[Route("[controller]/[action]")]
public class PriceController : Controller
{
    private readonly IUiBus _eventBus;
    public PriceController(IUiBus eventBus)
    {
        _eventBus = eventBus;
    }
    [HttpPost]
    public IActionResult Update([FromBody] PriceUpdate priceUpdate)
    {
        _eventBus.Notify(priceUpdate);
        return Ok("Price Update Processed");
    }    
}
```
## Creating Exchange Rate Components

To complete the example, let's create three components on the index page.

```
@page "/"
<PageTitle>Index</PageTitle>

<ExchangeRate FromCurrency="USD" ToCurrency="ZAR"/>
<ExchangeRate FromCurrency="USD" ToCurrency="EUR"/>
<ExchangeRate FromCurrency="EUR" ToCurrency="ZAR"/>
```

## Testing out the application
Booting up the application, you should now see three components displayed.

{% include figure.html path="assets/img/HubSpoke.png" class="img-fluid rounded z-depth-1" %}

I'm using postman to check this by sending a post request to `https://localhost:7177/price/update` along with the following body

```
{
  "fromCurrency": "EUR",
  "toCurrency": "ZAR",
  "ExchangeRate": 1.427
}
```
Doing this you should now see that the application has been refreshed **only** on the component with the matching currencies.

{% include figure.html path="assets/img/HubSpokeUpdated.png" class="img-fluid rounded z-depth-1" %}

## Comments
All code for this article is available on my [github repo](https://github.com/thatstatsguy/DesignPatterns/tree/main/Hub%20and%20Spoke/Part2/CurrencyDisplay). Based on the implementation provided, I have three comments:
1. I've said this before, but I do find it interesting that  [this](https://github.com/jeffreypalermo/blazormvc) implementation stops random events being fired off by forcing the notification payload to implement a given interface. That being said, it can be confusing to a new user on a repo when they see a record implenting an empty interface.
2. It's great to have such fine grained control over whether or not a component registers itself by forcing it to manually call  the `Register` method on the `VisualUiBus` service. However, I can see someone may want an implementation detail where everything is automatically registered without calling the `Register` event. 
    - I can't remember where I've seen this, but this can be done by inheriting from a base class which takes care of the registration and all you need to do is override the Handle method for the custom implementation details. 
    - Other options may exist other than the one I've found.
3. I'm definitely not the first person who's had to deal with real time notifications and eventing via a pattern like this - something like the MediatR library likely has a spin off for this in Blazor.

## Wrapping up
We now have a working implementation for an application that receives real time notifications. In the final article of the series, we'll explore how we can use already existing libraries to achieve the same implementation. 

Until next time :) 