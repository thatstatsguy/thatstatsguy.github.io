---
layout: post
title:  ASP .Net Controller Classes and F# Web APIs
date:   2023-03-18
description: Do as I say, don't do as I did
tags: F#
categories: F#
---
This article is going to be a quickfire guide to starting off with web API's in F#. I'm writing up the article I wish I had when starting off building web APIs - no hate on the other tutorials out there, they just didn't answer all my questions. 

Starting off, bootstrapping up a webapi example project in F# is relatively straightforward using the .net sdk.

```
dotnet new webapi -lang F# -o ContollerTests
```

Navigating into the controller folder within the newly created project we're greeted with the "FSharpified" version of the weather forecast service test project which is present in many of the sdk examples.

```
namespace ControllerTests.Controllers

open System
open Microsoft.AspNetCore.Mvc
open Microsoft.Extensions.Logging
open ControllerTests

[<ApiController>]
[<Route("[controller]")>]
type WeatherForecastController (logger : ILogger<WeatherForecastController>) =
    inherit ControllerBase()

    let summaries =
        [|
            "Freezing"
            "Bracing"
            "Chilly"
            "Cool"
            "Mild"
            "Warm"
            "Balmy"
            "Hot"
            "Sweltering"
            "Scorching"
        |]

    [<HttpGet>]
    member _.Get() =
        let rng = System.Random()
        [|
            for index in 0..4 ->
                { Date = DateTime.Now.AddDays(float index)
                  TemperatureC = rng.Next(-20,55)
                  Summary = summaries.[rng.Next(summaries.Length)] }
        |]

```

Booting up the application and navigating to `https://localhost:44309/weatherforecast` in the browser gives us some sample responses from the API. So far, so good.


### Customizing responses from the API
Having worked in exactly the same api in C#, I expected everything that worked in C# to "just work" out the box with F#. As an example, if we wanted to return an 200 response, using `OK` worked fine.

```
//tweaked from the original example provided in the webapi example
//which returns IEnumerable<WeatherForecast>
[HttpGet(Name = "GetWeatherForecast")]
public IActionResult Get()
{
    var returnResult =
        Enumerable.Range(1, 5).Select(index => new WeatherForecast
        {
            Date = DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            TemperatureC = Random.Shared.Next(-20, 55),
            Summary = Summaries[Random.Shared.Next(Summaries.Length)]
        })
        .ToArray();
    return Ok(returnResult);
}
```

Trying the same thing in F# results in an error because the convenience method `Ok` isn't accessible unless you use the `base` keyword i.e. 

```
[<HttpGet>]
member _.Get() : IActionResult =
    let rng = System.Random()
    let result = 
        [|
            for index in 0..4 ->
                { Date = DateTime.Now.AddDays(float index)
                    TemperatureC = rng.Next(-20,55)
                    Summary = summaries.[rng.Next(summaries.Length)] }
        |]
    base.Ok(result)
```

A minor gripe I have with F# is that I always find myself confused when to use the different keywords. In certain scenarios e.g. the `Get` command, we're using an `_` to refer to the current type (confusingly, one can add as many underscores as you like and this still compiles). In this scenario, we need to use the `base` keyword. I've even seen scenarios in which the `this` keyword is used (I will find an example and post it here later). For more info on this see [Source 1](https://stackoverflow.com/questions/15091468/f-inherit-from-c-sharp-class-access-protected-fields) and [Source 2](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/keyword-reference).

### Actions within the controller

API controllers allow us to register additional actions under each controller e.g. if I had a second controller `SpecialWeatherForecastController`, I may want a route that I can access at `https://localhost:7260/specialweatherforecast/givemeaforecast`. One does this by registering the `[action]` tag above a method (or in the controller definition as you'll soon see) along with inheriting from `Controller` instead of `ControllerBase`. If you were reading the article description and wondering what I meant, please reread the previous sentence - I spent 30 minutes staring at my screen thinking I was going insane because my actions weren't registering. If you use `ControllerBase` your actions **will not register**. Creating the new controller `SpecialWeatherForecastController` with a dummy action `GiveMeAForecast` is defined below.

```
namespace ControllerTests.Controllers

open System
open Microsoft.AspNetCore.Mvc
open ControllerTests

[<ApiController>]
[<Route("[controller]/[action]")>]
type SpecialWeatherForecastController() =
    inherit Controller()
    
    [<HttpGet>]
    member _.GiveMeAForecast() =
        [|
            {
                Date = DateTime.Today
                TemperatureC = 30
                Summary = "Insert Summary here" }
        |]
```
Notice how in the route definition we use `[<Route("[controller]/[action]")>]` to indicate that we're going to register actions under the current controller. Booting up the api and navigating to `https://localhost:7260/specialweatherforecast/givemeaforecast` should now give you the output specified above.

### Testing the API
Testing the Api is fairly easy - for easy of use I've added a tests folder to the solution. In it, I'll be testing the new action we just created. A sample test is below.

```
namespace ControllerTests

open System
open ControllerTests.Controllers
open Microsoft.AspNetCore.Mvc
open Xunit

module SpecialWeatherForecastControllerShould =
    
    [<Fact>]
    let ``Return dummy results specified in code``() =
        
        let controller = new SpecialWeatherForecastController()
        let response = controller.GiveMeAForecast() :?> OkObjectResult
        
        let expectedResult: WeatherForecast[] =
            [|
                {
                    Date = DateTime.Today
                    TemperatureC = 30
                    Summary = "Insert Summary here" }
            |]
        
        Assert.Equal(200, response.StatusCode.Value)
        
        Assert.Equal<WeatherForecast[]>(expectedResult, (response.Value :?> WeatherForecast[]))
```
Really nothing special to see here, create a new instance of the type, fire up the method you want to test and check the results.

### Wrapping up
Hopefully this will be valuable information for anyone who's doing the some work in F# ASP.NET web APIs. Knowing myself, I'll be using this as a personal cheat sheet at some point in the future. The code for the examples above can be found here. 

Until next time :) 