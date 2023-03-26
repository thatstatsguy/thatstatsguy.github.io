---
layout: post
title:  Tuple types in .Net
date:   2023-03-26
description: And how to define them
tags: C# F#
categories: til
---
Here's a quick little recap of tuples in .Net and how to define them.

# Overview
Tuples are a fantastic way to work with data in an application - they're quick to define and really easy to work with (see tuple deconstruction in F#). Before I go into describing how to define tuples, it's worth spending a moment discussing subtleties in what's happening under the hood when you're defining tuples.

Tuples are defined as either value types or reference types. This, like most things, has implications on how your program behaves as well as its performance. If you're an F# dev, I highly recommend reading [this](https://www.bartoszsypytkowski.com/writing-high-performance-f-code/) article for a good discussion on when to use value types vs reference types. That being said, let's jump into how these are defined 

# C#

In C#, tuples can be defined as reference types using the structure defined in `System.Tuple`. An example of how they are defined and used is given below.

```
public class Test
{
    //defining a method which returns a tuple
    Tuple<int, bool> TestMethod()
    {
        return new Tuple<int, bool>(123, false);
    }
    
    //using the tuple
    void TestMethod2()
    {
        var result = TestMethod();

        Console.WriteLine($"First element of tuple {result.Item1}");
        Console.WriteLine($"Second element of tuple {result.Item2}");
    }
}
```

The alternative to reference type tuples, value tuples, are defined (and used) as follows:

```
public class Test
{
    //defining a method which returns a tuple
    (int, bool) TestMethod()
    {
        return (123, false);
    }
    
    //using the tuple
    void TestMethod2()
    {
        //an example of tuple deconstruction
        var (someNumberResult, someBooleanResult) = TestMethod();

        Console.WriteLine($"First element of tuple {someNumberResult}");
        Console.WriteLine($"Second element of tuple {someBooleanResult}");

        //a second example of how they can be defined 
        (double, int) t = (4.5, 3);
    }   
}
```

For more info, consult the [Microsoft](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples) documentation.

# F#

Tuples in F# are WIDELY used. The language makes working with tuples a breeze (see match statements and deconstructing tuples within lambdas).

The tuples that I see most often in F# code are reference type tuples. Keeping with the previous example, here's some an example usage.

```
let testMethod() : int * bool =
    123, false

let testMethod2() =
    let someNumberResult, someBooleanResult = testMethod()

    printfn $"First element of tuple {someNumberResult}"
    printfn $"Second element of tuple {someBooleanResult}"
```

Pretty much the same as C#, but without the fluff of curly braces.

By contrast, value type tuples require a (little) bit more work to define - and by work I really just mean a couple extra characters. Here's the value type tuples, but for F#:

```
let testMethod() : struct(int * bool) =
        struct(123, false)

let testMethod2() =
    
    //deconstructing the tuple requires the extra struct keyword
    let struct(someNumberResult, someBooleanResult) = testMethod()
    
    printfn $"First element of tuple {someNumberResult}"
    printfn $"Second element of tuple {someBooleanResult}"
    
    //deconstructing the struct to a reference type tuple requires an extra method call
    let someNumberResult2, someBooleanResult2 = testMethod().ToTuple()
    
    printfn $"First element of the converted tuple {someNumberResult2}"
    printfn $"Second element of the converted tuple {someBooleanResult2}"
```

It's easy to be annoyed by the extra characters, but I must say I do appreciate the fact the language is being very intentional in letting you know whether things are being passed by value or by reference. For more info, see the microsoft [documentation](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/tuples)

### Wrapping up
This was a real whirlwind tour tuples in the .Net ecosystem. As always, be mindful of the tuple type you use as this has implications on several things, such as speed and memory (we wouldn't want to needlessly clog up the stack unless we really need to!)

Until next time :) 