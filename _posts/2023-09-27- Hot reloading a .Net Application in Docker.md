---
layout: post
title:  Hot reloading dockerized .Net applications
date:   2023-09-26
description: Supercharging your development
tags: C# docker
categories: C#
---

When developing a .Net application which is containerized using `docker` and `docker compose` it can be a real pain to constantly rebuild your docker image each and every time you make a tweak to your code. In the past I've used Nodemon to "hot reload" functionality as I've played around on Javascript which automatically built in the new code changes each time I hit save. In today's article we'll see how we can achieve the same thing in dotnet. Link to all code is available via the buttons below.

<a class="btn btn-info" href="https://github.com/thatstatsguy/til/tree/main/Docker%20hot%20reload" role="button">Link to Hot Reload Example</a>

## Getting started
To begin with I've created a sample web api application using `dotnet new webapi`. The only adjustments I've made is adding a route to the index of my api using `app.MapGet("/", () => "Hello World");`. As a test I want my application to hot reload each time I change the return text.

## Using dotnet watch in docker
Ignoring the dockerization aspect of this article, dotnet actually has a built in hot reload feature in the form of `dotnet watch`. You could run this in the root of the application and start making changes to files. What we need to do is bake this functionality into docker to supercharge our dev experience.

To begin with, what we need to do is create a dockerfile which can be used to create an image. Using the Microsoft base image for the dotnet 7 sdk, we can achieve this with

```
FROM mcr.microsoft.com/dotnet/sdk:7.0
WORKDIR /app
EXPOSE 80
EXPOSE 443

COPY . /app
RUN dotnet restore

ENTRYPOINT [ "dotnet", "watch", "run", "--urls", "http://*:8080" ]
```

If you've ever worked with docker the first few lines should be fairly straight forward. We are:
- setting the baseline image, 
- setting the working directory, 
- exposing ports for incoming traffic, 
- copying contents from the build folder into the app folder in the container and we're building from
- doing a dotnet restore

The first interesting bit pops up when specifying the entrypoint which specifies we should be using `dotnet watch run` when starting up the container. This means that dotnet is actively looking for file changes in the `app` directory which will trigger the reload. 

Noticed that we've not specified any volumes here? This is where docker compose comes in to play.

## Using docker compose to wrap things up
Mounting a volume to the container enables us to be able to update the app directly from outside the container. We'll set this up using docker compose but you could do this without it as well if you needed to. The docker compose is pretty concise.

```
services:
  hotreloadexample:
    image: hotreloadexample
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - 8080:8080
    volumes:
      - .:/app

```

The docker compose file does all the normal things you expect of it, specifying the build context and docker file as well as mapping ports. The last section on volumes is of interest as we see it maps the current build directory to the app folder in the container. This means that we should be able to update logic as needed and things will update on the fly.

## Testing out our changes
To test this out, we need to boot up the system with `docker compose up`. In the terminal you should see something like `dotnet watch ⌚ Polling file watcher is enabled` which indicates dotnet is actively watching the files in the app directory.

Navigating to `http://localhost:8080/` you'll see the `Hello World` as expected. Opening up your IDE, if you change `app.MapGet("/", () => "Hello World");` to something else like `app.MapGet("/", () => "Testing");` and hit save the hot reload should being automatically. You should see that your application is rebuilding and will start serving the new response assuming that you've not made a mistake.

Navigating to the same url again, you'll notice that the message has indeed changed after the rebuild. Great success!

## Conclusion 
The above code is a quick and easy way to supercharge your development experience in docker. This is especially the case if you rely on using docker compose to boot up several services for your system.

Until next time :) 