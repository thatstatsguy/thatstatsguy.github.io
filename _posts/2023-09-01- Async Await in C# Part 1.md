---
layout: post
title:  Async Await in C# (Part 1)
date:   2023-06-26
description: 
tags: C# til async
categories: C#
---

I recently worked through two resources for asynchronous programming in C# that I thought worth writing about here. For anyone who's ever started out with asynchronous programming, it can sometimes feel like you just keep throwing the Task, async await keywords into your code like some kind of magic spice until things start working. No judgement - we've all been there.

In this two part series I'll try my best to show how you can avoid causing needless drama for yourself with asynchronous programming. Code for this can be found at the link below.

<a class="btn btn-dark" href="https://github.com/thatstatsguy/til/tree/main/AsyncAwaitWalkThrough" role="button">Link to Code (Sample project 1)</a>

<a class="btn btn-dark" href="https://github.com/thatstatsguy/til/tree/main/Task%20Parallel%20Library/TPL" role="button">Link to Code (Sample project 2)</a>

## Some notes before we start
The content for this series was derived, in part, from the following [youtube](https://www.youtube.com/watch?v=n6kiJKr4_oA) video and [pluralsight](https://app.pluralsight.com/library/courses/getting-started-with-asynchronous-programming-dotnet/learning-check) course. The ideas presented are similar, but I've condensed these down into a smaller application which is hopefully easier to follow.

## Asynchronous programming in C# - the problem

Oftentimes, we find ourselves developing applications with front ends that are expected to remain responsive even when there's heavy computation happening in the background. Asynchronous programming is a method by which we go from blocking the main thread to non blocking by avoiding heavy lifting on the main thread.

Why is blocking the main thread of an application a bad idea you might ask? When you block an application's main thread you may run into:
- an unresponsive UI
- the application no longer being able to process other requests

## Task in the .Net ecosystem
Tasks enable a lot of the asynchronous programming we'll discuss in this series. Before going any further, it's worth noting that asynchronous programming is not the same as parallel programming.

Writing code for the parallel library entails writing code which the computer will compute as fast as fast as possible. There's no concern for whether this activity will block a thread.

Async programming in C# also enables the user to compute things in parallel, but the devil is in the details as to how all of this goes down behind the scenes.
- Async programming allows you can access a result as things are done
- work is distributed to a different thread. i.e. don't block current thread
- In the event no threads are available, .net puts these Tasks on a queue and takes care of it for you and computes it when it's ready.

It's also worth noting that using the asynchronous programming model enables threads to be reused while they are waiting for results. This is known as returning the thread to the thread pool.

## Why not just use Task.Run() and Task.Result?

Relying on `Task.Run` and `Task.Result` can cause many headaches. For one, if you use `Task.Run` you're only able to process as many requests as your PC has threads and you'll run out of threads quickly.

Another problem with this approach is how to know when the Task is finished? Using `Task.Result` is a gamble you take as you have to be very certain your task is finished otherwise you risk some fun exceptions popping out to greet you.

# Async await
Before `async` and `await` keywords were a thing, one would write tasks with continuations i.e. define task A, use the `.ContinueWith` method to "chain" together various tasks. An example of this can be seen in example 4 in the first project sample.

Without `async` and `await` you need to constantly be thinking about:
- How to deal with exceptions if and when they occur
- code readability . Writing code with continuations just feels more messy compared to `async` and `await` which we'll look at.
- You need to be very aware of the thread you're working on. You'll be painfully aware of this when you try rendering UI changes from the non UI thread and receive an exception complaining about trying to execute code on the dispatcher (i.e. UI) thread.

The keywords hide a lot of complexity  as the `await` keyword does all the continuations for you. The benefit of this "magic" is improved code readability. The `await` keyword also returns back to the original context when done i.e. if we're creating a task from UI thread we'll return back to ui thread.

Another benefit of the `async` `await` keywords is that they validates success/failure of task i.e. you'll see the exceptions and be forced to deal with them instead of them being "swallowed" which we'll see shortly.

## Conclusion 
In the next article we'll take a look at practical example of how you can use asynchronous programming (both the right way and the wrong way)