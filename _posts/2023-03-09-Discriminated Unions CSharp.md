---
layout: post
title:  Discriminated Unions in C#
date:   2023-03-09
description: Functional meets OOP with DUs
tags: til
categories: Coding
---
In my current role, I've spent a good chunk of the last few years working in F#. I've learned to love how the language protects you from yourself and allows self-documenting code. One such feature in F# is `Discriminated Unions` or `DUs` which allow you to wrap several independent "things" under a single banner. I recently learned that C# has the `OneOf` library which enables similar sort of functionality. Today I'll show you some code snippets which enable you to write DUs in C#.

## Setting the stage - an F# example

Suppose we're working with a system processing orders. In this ordering system, orders can be `valid` or `invalid` and your job is to process inputs to determine whether orders are valid or invalid. In F#, with a `DU` we are able to write this code up as

```
type ProcessedOrder = 
    | Valid of Order
    | Invalid of int

```
where `Order` is a record for some piece of data e.g.

```
type Order = 
    {
        Id : int
        ContactNumber : string
    }
```

if you make a function which validates the orders, it may take in an array of orders and produce an array of Processed Orders e.g.
```
let processOrders (inputOrders : Order[]) : ProcessedOrders[] =
    // a simple validation of whether a contact number is specified.
    inputOrders
    |> Array.map(fun order -> 
        match x == "" with
        | true -> Invalid order.Id
        | false -> Valid order
    )

```

The beauty of `DUs` really starts to shine when you use the results of this function as the language forces you to deal with each type of result.

```
let processAndLogInvalidOrders (inputOrders : Order[]) = 
    inputOrders
    |> processOrders
    // choose is the same as a F# map and filter (or a linq select and filter in C#)
    |> Array.choose(fun x ->
        match x with 
        | Valid order -> Some order
        | Invalid orderId -> logInvalidOrderId orderId
    )
```
Notice in the above code how the language does two things for us. Firstly, it's self documenting code i.e. it's clear when I'm working with the valid vs invalid order. Secondly, if I forget to match on one of the types the compiler warns me that this is not safe an may produce problems at run time.

## OneOf in C#

The [OneOf](https://github.com/mcintyre321/OneOf) library enables you achieve a similar type of implementation. Let's quickly whip up a few classes which describe an `order`, `valid order` and `invalid order`.

```
public class Order
{
    public int Id { get; set; }
    public string ContactNumber { get; set; }

    public Order(int id, string contactNumber)
    {
        Id = id;
        ContactNumber = contactNumber;
    }
}

public class ValidOrder
{
    public Order Order { get; }
    public ValidOrder(Order order)
    {
        Order = order;
    }
}

public class InvalidOrder
{
    public int Id { get; }
    public InvalidOrder(Order order)
    {
        Id = order.Id;
    }
}
```

Before we jump into any methods, we need to define a new class which describes our processed orders.

```
[GenerateOneOf]
public partial class ProcessedOrder : OneOfBase<ValidOrder, InvalidOrder>
{}
```
Disclaimer, I realise that there's a lot going on here, but this is the most compact format to show it in - see the attached video at the end of the article. 

What `OneOf` does is enable you to specify several classes belonging to a common group of "things". In this case it's the `ProcessedOrder` class. The `OneOf` and `OneOf.SourceGenerator`  extension library enables this super compact format.

We'll define two methods, one to process the orders and one to do something with those results.

```
ProcessedOrder[] ProcessOrders(Order[] orders)
{
    var processedOrders = new List<ProcessedOrder>();
    foreach (var order in orders)
    {
        if (string.IsNullOrEmpty(order.ContactNumber))
        {
            processedOrders.Add(new InvalidOrder(order));
        }
        else
        {
            processedOrders.Add(new ValidOrder(order));
        }
    }

    return processedOrders.ToArray();
}

void LogOrders(ProcessedOrder[] processedOrders)
{
    foreach (var order in processedOrders)
    {
        order.Switch(
            validOrder =>
                Console.WriteLine(
                    $"Invalid Order, Id : {validOrder.Order.Id}, Contact : {validOrder.Order.ContactNumber}"),
            inValidOrder => Console.WriteLine($"Valid Order, Id : {inValidOrder.Id}"));

    }
}
```

In the first method, we're doing the same check that we did in F# - check the contact number and add it a list if the string is empty. Notice how we're seemingly adding objects of different types to the same list - I expect alarm bells going off in your head. Isn't C# meant to be strongly typed? How are we mixing types? 

What the `DU` style functionality of `OneOf` enables us is to wrap the objects into a "group". Where this starts to get really cool is where you want to produce different behaviour based on the processed type. In `LogOrder` we change up the behaviour of the logging based on type of object we're dealing with using the `switch` command. What might not be apparent is that we're actually forced to supply an implementation for all cases of the OneOf class.

## Conclusion
This has really been a whirlwind tour of what `OneOf` can offer. I didn't even get a chance to mention was how you could use this to effectively get rid of nullables in your code and represent them as an option in the `OneOf` class - hooray for no more null pointer exceptions! There's a really great video on this which I can strongly recommend. https://www.youtube.com/watch?v=7z-xjijYfcI

Until next time :) 
