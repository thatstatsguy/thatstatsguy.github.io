---
layout: post
title:  Health Checks in C#
date:   2023-05-07
description: 
tags: C# til HealthChecks
categories: C#
---

When it comes to deploying any application, being able to observe the state of your application is critical. One way of building visibility for your application state is Health Checks. Today, we'll be looking at different ways we can use the built in Health Check functionality in Blazor to increase what we know about the application once it's been deployed.

## What is a health check?
A health check is a summary of the state of your application. Examples of things we might want to monitor include:
- connections to external services via HTTP/GRPC
- Database connections
- Internal data structure states e.g. dead-letter queues

Parts of an application can be in one of several states (e.g. Healthy, Degraded, or UnHealthy) and the function of a health check summary is to alert you to the parts of your application that aren't behaving as expected. Summaries can take many forms, it can be as simple as a plain text, JSON or even HTML.

## A note on the code used in this article
All code from today's article can be found [here](https://github.com/thatstatsguy/til/tree/main/HealthChecks).


## Setting up a Blazor server application to support health checks
The easiest way to support health checks in Blazor is as follows:
- In `program.cs`, add in the `using Microsoft.AspNetCore.Diagnostics.HealthChecks;` using statement. After this is done
- Set up an endpoint that we can we use to check the state of our application. We'll be using `_health` in this article.

```
app
    .MapHealthChecks("/_health", new HealthCheckOptions()
    {
        ResponseWriter = UIResponseWriter.WriteHealthCheckUIResponse
        
})
```
- Finally, set up a starting point where we can add our health checks

```
builder.Services.AddHealthChecks()
```

You'll notice that I've specified my own Response Writer using `UIResponseWriter.WriteHealthCheckUIResponse`. This available through the `AspNetCore.HealthChecks.UI.Client` package and "pretty-fies" the output of our health check when calling the end point. Running the application at this point, calling the endpoint `https://localhost:8080/_health` will always return healthy as no health checks have been defined.

## Setting up a health check for a SQL Database
While we're not actively exploring health checks for databases in this article, it's good to discuss what's available out the box so we're not re-inventing the wheel. A good example of this is health checks for SQL servers. One can easily add in health checks for SQL Servers by adding the `AspNetCore.HealthChecks.SqlServer` package and changing the following line of code to `Program.cs`.
```
builder.Services.AddHealthChecks()
    //To add health check for SQL Server
    .AddSqlServer("<AddSqlServerConnectionString>");
```

This health check will run a sample query against the database and return unhealthy if it is unable to connect to it or something else happens while testing out the external service.


## Setting up health checks for Azure function Apps

You will inevitably run into a scenario where the built in libraries no longer help and you want to implement something custom. We can do this by creating a class which implements the `IHealthCheck` interface and adding it to the list of health checks. The following code sets up a health check for an azure function application. The azure function app is available in the code provided.

```
public class AzureFunctionHealthCheck : IHealthCheck
{
    public async Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = new CancellationToken())
    {
        try
        {
            var functionAppUrl = "https://<FunctionAppHostName>.azurewebsites.net/api/TestMe?code=<FunctionKey>";

            var stringContent =
                new StringContent("{'Name':'Test'}", Encoding.UTF8, "application/json");
            using var client = new HttpClient();
            
            using var httpResponseMessage = 
                await 
                    client.PostAsync(functionAppUrl, stringContent);
            
            return HealthCheckResult.Healthy();

        }
        catch (Exception e)
        {
            return HealthCheckResult.Unhealthy(exception: e);
        }
        
    }
}
```

and within `Program.cs`:
```
builder.Services.AddHealthChecks()
    .AddCheck<AzureFunctionHealthChecks>("AzureFunction");
    //To add health check for SQL Server
    //.AddSqlServer("<AddSqlServerConnectionString>");
```

In the above code, the bespoke code will effectively "ping" the azure function app using a dummy `TestMe` function. If for whatever reason the request fails, return Unhealthy. One could easily incorporate network related checks (such as IP forbidden status code checks) to enhance this.

## Setting up Health Checks for GRPC Services
Setting up health checks for gRPC services requires a little bit more effort between the service and the client (but not much!). To implement this, the server needs to map a health check endpoint that can be called. The server side code for this is as follows. We first need to register the grpc health check service

```
services.AddGrpcHealthChecks()
    .AddCheck("Greeter Service", () => HealthCheckResult.Healthy());
```

following this, an endpoint should be mapped

```
endpoints.MapGrpcHealthChecksService();
```

On the client side, we only need to use the `Grpc.HealthCheck` package along with the following as an example for how we'd go about setting up the Grpc service health check.

```
public class GrpcHealthCheck : IHealthCheck
{
    public async Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = new CancellationToken())
    {
        try
        {
            var channel = GrpcChannel.ForAddress("https://localhost:9001");
            var client = new Grpc.Health.V1.Health.HealthClient(channel);

            var response = await client.CheckAsync(new HealthCheckRequest());
            var status = response.Status;
            return status switch
            {
                HealthCheckResponse.Types.ServingStatus.Serving => HealthCheckResult.Healthy(),
                _ => HealthCheckResult.Unhealthy()
            };
        }
        catch (Exception e)
        {
            return HealthCheckResult.Unhealthy(exception: e);
        }
    }
}
```

In `Program.cs`:
```
builder.Services.AddHealthChecks()
    .AddCheck<AzureFunctionHealthChecks>("AzureFunction")
    .AddCheck<GrpcHealthCheck>("GRPC Service");
```

As a test, I've not started up my grpc service. Calling the health check returns the following:
```
{
    "status": "Unhealthy",
    "totalDuration": "00:00:04.7165371",
    "entries": {
        "AzureFunction": {
            "data": {},
            "description": "Invalid URI: The hostname could not be parsed.",
            "duration": "00:00:00.1927888",
            "exception": "Invalid URI: The hostname could not be parsed.",
            "status": "Unhealthy",
            "tags": []
        },
        "GRPC Service": {
            "data": {},
            "description": "Status(StatusCode=\u0022Unavailable\u0022, Detail=\u0022Error connecting to subchannel.\u0022, DebugException=\u0022System.Net.Sockets.SocketException: No connection could be made because the target machine actively refused it.\u0022)",
            "duration": "00:00:04.6920450",
            "exception": "Status(StatusCode=\u0022Unavailable\u0022, Detail=\u0022Error connecting to subchannel.\u0022, DebugException=\u0022System.Net.Sockets.SocketException: No connection could be made because the target machine actively refused it.\u0022)",
            "status": "Unhealthy",
            "tags": []
        }
    }
}
```



## Wrapping Up
And that's a wrap! This brief introduction on health checks for the .net platform will take your application development to the next level as you'll have a good insight into the state of the application and be able to react accordingly when things go south.

Until next time :)

## Further Resources
- [Nick Chapsas Health Checks Video](https://www.youtube.com/watch?v=p2faw9DCSsY&t=556s)
- [Health Checks for GRPC](https://learn.microsoft.com/en-us/aspnet/core/grpc/health-checks?view=aspnetcore-7.0)
- [Scott Hanselman Article on HealthChecks](https://www.hanselman.com/blog/how-to-set-up-aspnet-core-22-health-checks-with-beatpulses-aspnetcorediagnosticshealthchecks)