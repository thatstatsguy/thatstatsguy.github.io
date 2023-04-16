---
layout: post
title:  Hub and Spoke Pattern in Blazor - Part 1
date:   2023-04-10
description: Adding a RESTful API to a blazor server application
tags: Blazor C#
categories: Patterns
---
Interactivity within web apps and how components react to external events has always felt like a bit of black box magic to me. After watching a video on [blazor architecture patterns](https://www.youtube.com/watch?v=noGh9oA86X0) I thought it would be a bit of fun to implement the hub and spoke pattern for Blazor.

## The use case
The example I'll be using to explore the concept is a exchange rate application. A user will be shown the exchange rate for n different currencies. The server exposes a rest api which can receive updates from some external third party. Upon receiving the request, all users should have their screens updated. 

## Hub and Spoke Pattern
The Hub and Spoke pattern (as defined [here](https://www.oreilly.com/library/view/architectural-patterns/9781787287495/812e1f41-5809-4658-9640-21762f60438c.xhtml)) is a pattern which allows several applications (or parts thereof) to be connected to a central hub. Any incoming data is received by the hub, manipulated for the target system if necessary and passed on to the target system.

In our application, we've got one or more clients connected to the blazor server. Our exchange rate's job is to "listen" to external price update events coming in an update the visuals accordingly. The process of listening for external price update events is where we'll implement the hub and spoke pattern. First, we need to set up a web api which can receive price update requests.

## Setting up a RESTful API Service
Before reacting to incoming events we first need something listening for incoming requests. All code for this article can be found [here](https://github.com/thatstatsguy/DesignPatterns/tree/main/Hub%20and%20Spoke/Part1/CurrencyDisplay).

Starting off, let's bootstrap a blazor server application with `dotnet new blazorserver -o CurrencyDisplay`.

We'll be using an MVC controller to create our REST api endpount. To do this, I've created a folder called `Controllers` which will contain our price update controller. Inside the folder, create a `PriceController.cs` file and put the following code in it.

```
using CurrencyDisplay.DTO;
using Microsoft.AspNetCore.Mvc;

namespace CurrencyDisplay.Controllers;

[ApiController]
[Route("[controller]/[action]")]
public class PriceController : Controller
{
    [HttpPost]
    public IActionResult Update([FromBody] PriceUpdate priceUpdate)
    {
        return Ok("TODO");
    }    
}
```
The code at this point won't compile because we've not defined the DTO objects referenced in the code above. As with the controller, create a `DTO` directory with a file called `PriceUpdate.cs`. Copy the following code into it.

```
namespace CurrencyDisplay.DTO;

/// <summary>
/// Represents a price update for a given currency conversion e.g. ZAR -> USD 
/// </summary>
/// <param name="FromCurrency">The currency being exchanged e.g. ZAR</param>
/// <param name="ToCurrency">The currency being converted to e.g. USD</param>
/// <param name="CurrencyConversionRate">The amount of the target currency received for each "unit" of the original currency.</param>
public record PriceUpdate(string FromCurrency, string ToCurrency, double CurrencyConversionRate);
```

Great, we've created a controller which is going to act as our endpoint - we'll access it with a `POST` request to `/Price/Update` with our price update details sent in JSON format. 

At the moment this code won't do anything because our application doesn't know to use this controller as an endpoint. To register the endpoint we can add `app.MapControllers();` to the `Program.cs` file.

## Testing our new API
Testing our application can be done with something like Postman. Our request body needs to conform to the DTO object specifications e.g. 

```
{
  "fromCurrency": "ZAR",
  "toCurrency": "USD",
  "currencyConversionRate": 0.08
}
```

Making a request to the endpoint should return `TODO` as a result.

If you're not in the mood to use Postman, you can always use Swagger to achieve this - I've gone ahead and put in the required code in the `Program.cs` file along with the required imports - have a look at the code in my repo if you want to do this. You can access the swagger UI at `https://localhost:7177/swagger/index.html`

## Wrapping up
We've added in a RESTful API. In the next article, we'll set up our exchange rate display and make it react to external events. See you at the [next article](https://thatstatsguy.github.io/blog/2023/Hub-and-Spoke-Pattern-Part-2/) :)