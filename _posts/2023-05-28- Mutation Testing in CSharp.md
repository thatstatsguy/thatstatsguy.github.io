---
layout: post
title:  Mutation Testing in C#
date:   2023-05-07
description: 
tags: C# til
categories: Testing
---

I recently came across the concept of mutation testing in this [webinar](https://www.youtube.com/watch?v=9BoKyeZapLs) hosted by Jetbrains. The concept of mutation testing is simple 

> How do I assess the quality of my test suite?

Your initial knee jerk reaction (like mine) was immediately to think of code coverage - heads up we were both wrong. The thing about **code coverage** is that it's a tool to view **what lines of code** have been executed by your tests. Mutation tests do way more than than this. Among other things, mutation tests (try) help you address the following questions:
- Do I have any pointless/redundant tests in my codebase?
- Does any of my code have redundancies?
- Am I missing tests that I should be writing?

The idea with mutation testing is to see what happens when bugs are introduced into your code. By introducing these bugs, we're able to do a check if our tests are actually testing the right thing.

When I say bugs, I'm referring to common things we as devs may make while doing development work i.e. making something `<` instead of `<=`, leaving code out that should be there etc. In mutation testing, each time we tweak code using this method we create a **mutant**. If our tests are any good, we expect these changes to be caught and thereby "killing" the mutant. Likewise, if this test didn't cause our test to fail, we have mutant on our hands which has survived and needs to be investigated.

## Some Project Setup with Stryker
Withing the .NET ecosystem, Stryker is one library we can use to perform mutation testing. I've set up a [test project](https://github.com/thatstatsguy/til/tree/main/MutationTesting) to test out mutation testing. To run this, download the files and hit `dotnet tool restore`. Alternatively go have a look at the [Stryker website](https://stryker-mutator.io/docs/stryker-net/getting-started/). The examples used here were heavily influenced by the above Jetbrains webinar and [this](https://www.youtube.com/watch?v=LoFJajoJQ2g) GOTO conference presentation.

The test project is nothing special, we have a division method along with fizzbuzz and foobar methods.
```
public class Calculator
{
    public static float Divide(float number1, float number2)
    {
        return number1 * number2;
    }
}

public class Fizz
{
    public static void Buzz(int number1)
    {
        if (number1 <= 10)
        {
            throw new Exception("Test");
        }

        if (number1 >= 10)
        {
            Foo();
        }
    }

    private static void Foo()
    {
        
    }
}

public class Foo
{
    int min = 999;

    public int Bar(int number)
    {
        if (number < min)
        {
            min = number;
        }

        return min;
    }
}
```

To me at least, these methods are simple enough to be self explanatory. One is a division example where we're accidentally multiplying numbers instead of dividing. The next piece of code is a variable test to check if a number meets a criteria in order to perform some custom logic. The last bit of code is another variation on a criteria check. The tests for these are equally unimaginative: 

```
public class CalculatorShould
{
    
    [Fact]
    public void DivideTwoNumbers2()
    {
        var actual = Calculator.Divide(1, 1);
        Assert.Equal(1, actual);   
    }
}

public class FizzShould
{
    [Fact]
    public void BuzzTest()
    {
        var exception = Assert.Throws<Exception>(() => Fizz.Buzz(5));
        
        Assert.Equal("Test", exception.Message);   
    }
}

public class FooShould
{
    [Fact]
    public void BarTest()
    {
        var c = new Foo();
        var actual = c.Bar(1);
        Assert.Equal(1, actual);   
    }
}
```

## Letting the mutants loose
Running the mutation tests is as simple as

```
dotnet tool restore
dotnet stryker
```
Assuming your code builds fine, stryker will now run and produced some outputs for you. Looking at the results of the html file, you will notice that we've got mutants surviving in all of our cases.

### Scenario 1 - Arithmetic Mutation Survived
In our calculator example, our unit test would have passed with flying colours, but our code was fundamentally flawed. The mutation tests point this out by highlighting that swapping out a multiplication for a division produced an identical result (this is known as an arithmetic mutant surviving). This highlights two potential problems:
- Our code was incorrect or
- The tests that were written suck and didn't adequately test our code

In this scenario, both of these turn out to be true. Changing the code and tests to make this more robust is an easy thing to do.

### Scenario 2 - Equivalent Mutants
From the report we can see that `if (number1 >= 10)` is being flagged as something called an **equivalent mutant**. What this means is that the operator used could be changed without causing any tests to fail. What might not be obvious as first is that the mutant has highlighted redundant code. Inspecting the code a bit further, we already do a check on `number1` earlier in the code. This means that the code here is in fact redundant and can be removed safely.

### Scenario 3 - Equivalent Mutants (Again)

In the final bit of code, the following is being flagged as an equivalent mutant.

```
if (number < min)
{
    min = number;
}
```
The reason for this is the `<` operator can be replaced with `<=` without tests failing. The mutation testing here gives us an opportunity to re-express some of our code to better show our intent. In this scenario, we could just do 

```
min = Math.Min(min, number);
```

## Wrapping Up
To summarize what's been mentioned above, mutation tests are a really great way to check the code you write. There are scenarios where in which it's fine to let mutants survive, but this is a good opportunity to do a check in with your code before you accidentally release a bug out into the wild!

Until next time :)

## Further Resources
- [Github Repo for mutation testing webinar](https://github.com/Flash0ver/F0-Talks-MutationTesting)
- [Making Mutants Work for You](https://www.youtube.com/watch?v=LoFJajoJQ2g)