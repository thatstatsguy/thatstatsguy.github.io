---
layout: post
title:  Async Await in C# (Part 2)
date:   2023-09-25
description: 
tags: C# til async
categories: C#
---

If you haven't already done so, give the [first article](https://thatstatsguy.github.io/blog/2023/Async-Await-in-C-Part-1/) in the series some love. Part 1 covers the various reasons we want to do async programming in the first place, things to look out for. In this article, we'll take a look at ways you can cause havoc by using the async programming model incorrectly and a few other interesting observations. Link to all code is available via the buttons below.

<a class="btn btn-info" href="https://github.com/thatstatsguy/til/tree/main/AsyncAwaitWalkThrough" role="button">Link to Code (Sample project 1)</a>

<a class="btn btn-info" href="https://github.com/thatstatsguy/til/tree/main/Task%20Parallel%20Library/TPL" role="button">Link to Code (Sample project 2)</a>

## Lets get ready to break some programs
Instead of telling you what not to do, let's see what happens when you use the async programming model incorrectly. First off, download the sample blazor project and boot it up with 
```
dotnet restore
dotnet run
```
The application is the template blazor server project with several buttons on it showcasing various aspects of the async programming model. It aint a looker but it works.

{% include figure.html path="assets/img/asyncwalkthrough.png" class="img-fluid rounded z-depth-1" %}

## How to freeze your application
First up is what happens when you do something that clogs up the main application or UI thread. For this example suppose you want to display a custom message on the screen after two seconds. This will will be triggered by clicking `Example 1`.

While this is happening you also want to be able to set this message to something random by clicking the `Set to a random message` button. Running the application, if you click on `Example 1` and then quickly click `Set to a random message` you'll notice the application seems to do nothing at all.

Digging into the code you'll notice a `DoSomething` method that's triggered.
```
private void DoSomething()
{
    //simulates a heavy computation
    Thread.Sleep(5000);
    _message = "Look what you did!";
    StateHasChanged();
    
}
```
Here we're sleeping the main thread by calling `Thread.Sleep`. By sleeping the main thread it doesn't matter how many times you click the other button it won't do anything until the 5 seconds has elapsed.

## How to unfreeze the application
Notice in the previous code we've not touched anything related to tasks or the async programming model. What we want to do is get this work happening on a different thread. To to this, we could change up our code as follows.

Firstly, we need to tweak our `DoSomething` code as follows.
```
private void DoSomething()
{
    //simulates a heavy computation
    Thread.Sleep(5000);
    _message = "Look what you did!";
    InvokeAsync(StateHasChanged);
    
}
```
and the calling code will change as follows.
```
private async Task Example1()
{
    await Task.Run(DoSomething);
}
```
Don't worry too much about the `InvokeAsync` bit for now, we'll cover that next.

## How to make your application swallow exceptions

Using Tasks incorrectly can result in the application not behaving as expected as well as exceptions which are swallowed. A good example of this is shown in Example 2 where I've tried to change something on the main thread from a non-ui thread. The `DoSomething` method is wrapped in a `Task.Run` i.e.

```
private void Example2()
{
    Task.Run(DoSomething);
}
```
and that's it. Running the application in debug you'll notice that an exception is thrown `System.InvalidOperationException: The current thread is not associated with the Dispatcher.` but the application doesn't crash and it also doesn't update the message on the UI.

The reason for this that `Task` (at least to my understanding) works more like a class where all exceptions are captured as internal variables. Without using await, these exceptions aren't checked for in the calling code so you won't see these exceptions unless you explicitly check for them - possible to do, but very tedious. 

Back to the code, you'll notice this is exactly the same scenario we had previously. The exception is telling us that we're trying to manipulate the main thread from a thread that's not allowed to. Luckily, Blazor has baked in methods which enable you to do this from other threads so the `InvokeAsync method is used`

## How to make your application crash with literally no warning

You'll notice that the async programming model is built with `async` `await` AND `Task`. There's a good reason for this and the moment you start using things on their own things can and will go wrong.

Take for example `Example 8` using the async keyword without the Task.

```
private async void DoSomething8()
{
    var t = Task.Run(() => Thread.Sleep(2000));
    var c = t.ContinueWith(_ =>
    {
        return "Test!";
    });
    var result = await c;
    _message = result;
    throw new Exception("Things go boom");
}
```

The calling code looks pretty normal i.e.

```
private void Example8()
{
    try
    {
        DoSomething8();
    }
    catch (Exception e)
    {
        Console.WriteLine(e);   
    }
    
}
```
At a quick glance you'd be forgiven for thinking that the exception is going to be caught in the `try catch` block. Unfortunately, since the `await` keyword isn't used, the application has no way of knowing it should hang around and check for an exception. The code then "carries on" and two seconds later the application will crash with no warning whatsoever. 

In general it's advised to avoid `async void` wherever possible because the `async` mechanism is designed to avoid funnies like this. As noted in the youtube video and from personal experience this isn't always possible since button on click handlers are sometimes expected to have the `async void` return type.

## Conclusion 
In a classic "Do as I say, don't do as I do" we've shown three different ways you might want to be careful when using the async programming model. In the next article we'll take a look at how you can improve the speed of your application by better understanding the cost of switching between threads.