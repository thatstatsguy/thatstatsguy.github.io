---
layout: post
title:  CS Concepts (Part 3)
date:   2023-03-11
description: OOP Basics
tags: Concepts
categories: Concepts
---
Hi and welcome back to part 3 of my OOP basics series. Join me as I re-explore OOP concepts and hit the refresh button. The topics we'll be looking at originate (in part) from a list I picked up in this article [article](https://sites.google.com/site/steveyegge2/five-essential-phone-screen-questions).

If you've not already done so, take a look at part [1](https://thatstatsguy.github.io/blog/2023/CS-Concepts-Part-1/) and [2](https://thatstatsguy.github.io/blog/2023/CS-Concepts-Part-2/) of this series where I take a look at some of the basics.

<hr>

### 11.	Interfaces
Interfaces are a collection of methods with a name. Think of interface in terms of a "contract" - anything that uses the interface **must** have supplied an implementation for the methods in the contract. Classes use interfaces by **implementing** the interface and providing implementation details for each of the methods. In C#, an interface can be defined as 

```
public interface ICalculator
{
    public int Add (int Input1, int Input2)
}
```

where the interface name is `ICalculator` and the method in the interface is `Add` which takes in two inputs and returns an `int`. Note the naming convention of indicating an interface with the letter `I` in front of the name. In C#, a class can use the interface as follows

```
public class Calculator : ICalculator
{
    public int Add (int Input1, int Input2)
    {
        return Input1 + Input2;
    }
}
```
Note here that all we've done is given some specific details on what to do with the inputs supplied in the method. Interfaces are fantastic because they enable logic abstraction and fun things like dependency injection. More on this in the next article.

Having seen abstract classes in the previous article, now is a good time to discuss the differences between abstract classes and interfaces.
| |Abstract Class | Interface|
|--- |---|---|
| Can be inherited / Be used as a base class | Yes | No - only implemented
| Can have a constructor | Yes | No
| Methods can have default implementation | Yes - via virtual methods | No

Another note to point out here is that a class implementing an interface can be passed into a method where the input parameter is that method. For example, the class `Calculator` above implemented `ICalculator` - if there is the following method 

```
public int AddThreeNumbers (int Input1, int Input2, int Input3, ICalculator c)
{
    var result1 = c.Add(Input1, Input2);
    return c.Add(result1, Input3);
}
```

then any instance of `Calculator` can be used in place of `c` because the compiler knows this class implements the interface and can be converted into the correct type.

**Further reading**
- [Source 1](https://www.geeksforgeeks.org/difference-between-abstract-class-and-interface-in-java/)

<hr>

### 12. Finalisers vs Destructors vs Disposal

The concept of finalisers and destructors is one that originates from the following problem

> How do I ensure that objects I create don't hang around longer than necessary in memory?

In C#, the garbage collector (GC) is the magic tool which ensures that objects we create are disposed of in memory when no longer needed. Finalisers and destructors are methods on which objects are release from memory.

In general, we have no control over when the GC kicks in and cleans up unused objects in memory. This is referred to as non-deterministic memory clean up because we as programmers have no control when this will kick in.

At least in C#, the finaliser and destructor are one in the same thing as this is the method that the garbage collector calls when cleaning up objects. 

As a side note, since we have no control over when the finaliser is called, there are other methods like the `Dispose` method which allow us to free up objects from memory sooner than when the destructor method is eventually called. We can think of the disposal method as a deterministic disposal of objects vs a destructor which is a non-deterministic disposal of objects in memory.

**Further Reading**
- [Source 1](https://stackoverflow.com/questions/13988334/difference-between-destructor-dispose-and-finalize-method)
- [Source 2](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/finalizers)

<hr>

### 13. Method Overriding vs Overloading

In the [previous article](https://thatstatsguy.github.io/blog/2023/CS-Concepts-Part-2/) we spoke about abstract and virtual methods belonging to a class. Virtual methods enable us to provide a default implementation for a given method which an inheriting class may change if it isn't suited. The replacement of logic is done via a technique called `Method Overriding`. Using the similar example as the one in the previous article, given a class `Knife`

```
public abstract class Knife{
    protected string Name { get; }
    protected float Length { get; }
    public virtual float GetShippingCosts()
    {
        return Length * 2;
    }
}
```

suppose we created a new class called `GyutoKnife` to represent all gyuto knives. Due to the inherent value of this specific knife type, we know that shipping cost factor is in fact 5 times the length of the knife. We may choose to overload the method as follows

```
public class GyutoKnife : Knife
{
    public override float GetShippingCosts()
    {
        return Length * 5;
    }
}
```

#### Method Overriding
Method overloading allows us as coders to defined the same method, but cater for a variety of scenarios. For example, you may have various scenarios where you are calculating shipping costs, method overloading allows you to define and additional method which takes as input and additional cost to be added to the overall shipping costs. An example for a simple class `SantokuKnife` is given below.

```

public class SantokuKnife
{
    public float Length { get; set; }
    public float GetShippingCosts()
    {
        return Length * 5;
    }
    public float GetShippingCosts(float additionalCost)
    {
        return Length * 5 + additionalCost;
    }
}
```

<hr>

### 14. Delegates vs EventCallbacks

You may get to a point in your programming where you ask 

> I'm coding a method which needs some custom functionality to add two numbers, I've already coded this functionality before and I want to use it in several places.

> Can't I pass this functionality into the method?

You've just stumbled onto the idea of delegates, a functional language concept where functions are treated as first class citizens and are passed around as "things". As an example, in the example below we define a delegate called `Add` representing a function which inputs two numbers and returns a number. The `DoAddition` method has no idea how the addition is done, but it has a reference to a function which does the addition for it.

The logic for this custom addition is found in `AddExtraAddition` which adds two numbers and always adds 10 to the final result. The magic happens in `DoSpecialAdditionCalculation` where we call the `DoAddition` method and pass in the method which satisfies the delegate "contract" to take in two numbers and return a number.

```
public delegate double Add(double x, double y);

public class Calculator
{
    
    public double DoAddition(double x, double y, Add additionFunction)
    {
        
        return additionFunction(x, y);
    }

    public double AddExtraAddition(double x, double y)
    {
        // Trivial example of always adding ten extra
        return x + y + 10;
    }

    public double DoSpecialAdditionCalculation(double x, double y)
    {
        return DoAddition(y, y, AddExtraAddition);
    }
}
```

Obviously this is a trivial example of adding numbers, but there are instances where we want to **abstract** away the details of the implementation so that the object being worked with only needs to know about a subset of the details.

It's worth noting that delegates can have methods which are  named and un-named. In the example above, we created a method called `AddExtraAddition`. C# allows us to create an unnamed delegate using lambdas (aka defining functions "on the fly") - in our example it would look as follows:

```

public double DoSpecialAdditionCalculation(double x, double y)
{
    return DoAddition(x, y, (input1, input2) => input1 + input2 + 10 );
}
```

#### Event callbacks

In a similar vein to passing a functionality into function which can be used, **Event callbacks** allow the us to call logic which sits outside the scope of the current code. As you'll see now, delegates come to the rescue again!

```
public delegate void LogResult(double x);

public double DoAdditionWithCallback(double x, double y, LogResult logResult)
{
    var result = x + y;
    logResult(result);
    return result;
}

public double DoAdditionWithLogging(double x, double y)
{
    return DoAdditionWithCallback(x, y, (d => Console.WriteLine($"Result : {d.ToString()}";)) );
}
```

**Further reading**
- [Source 1](https://learn.microsoft.com/en-US/dotnet/csharp/programming-guide/delegates/)
- [Source 2](https://stackoverflow.com/questions/3622160/c-sharp-passing-function-as-argument)
- [Source 3](https://stackoverflow.com/questions/2139812/what-is-a-callback)

With any experience in C#, you may also wonder then what the difference is between an delegate, action and func in C#. For further info, have a look at the sources below.

- [Delegates vs Actions](https://stackoverflow.com/questions/11376657/what-is-the-difference-between-delegate-action-in-c-sharp)
- [Delegates vs Func<>](https://stackoverflow.com/questions/3624731/what-is-func-how-and-when-is-it-used)
<hr>

### 15. Polymorphism

Polymorphism is the greek word for "many-shaped" and is a critical part of OOP. The concept of polymorphism enables us to:
- treat all derived classes objects as objects of their base class
    - this enables them to be put into collections of the base class or 
    - passing in the base class object as a method parameter
    - note that all of this is done at runtime.
- method overrides are enabled via polymorphism. If a virtual method is defined in the base class, the CLR will go and find the override in the class you're working with (if it exists) and execute it.

To borrow from the Microsoft page listed in the sources below, if we define a few basic classes.

```
public class Shape
{
    // A few example members
    public int X { get; private set; }
    public int Y { get; private set; }
    public int Height { get; set; }
    public int Width { get; set; }

    // Virtual method
    public virtual void Draw()
    {
        Console.WriteLine("Performing base class drawing tasks");
    }
}

public class Circle : Shape
{
    public override void Draw()
    {
        // Code to draw a circle...
        Console.WriteLine("Drawing a circle");
        base.Draw();
    }
}
public class Rectangle : Shape
{
    public override void Draw()
    {
        // Code to draw a rectangle...
        Console.WriteLine("Drawing a rectangle");
        base.Draw();
    }
}
public class Triangle : Shape
{
    public override void Draw()
    {
        // Code to draw a triangle...
        Console.WriteLine("Drawing a triangle");
        base.Draw();
    }
}
```

if we define these three objects in runtime, you'll see that 
1. The objects can all be added to a `List<Shape>` object.
2. When we call draw, it uses the draw methods defined in each of the classes.

```
var shapes = new List<Shape>
{
    new Rectangle(),
    new Triangle(),
    new Circle()
};

foreach (var shape in shapes)
{
    shape.Draw();
}
/* Output:
    Drawing a rectangle
    Performing base class drawing tasks
    Drawing a triangle
    Performing base class drawing tasks
    Drawing a circle
    Performing base class drawing tasks
*/
```

**Further Reading**
- [Source 1](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/polymorphism)
<hr>

### 16. is-a vs has-a relationship in OOP
`Is-A` relationship and `Has-a` relationships are a way of speaking about objects which indicates the way it is implemented.

`Is-A` relationship refers to the fact inheritance inheritance has been used e.g. 
> Class A is a derived class of Class B

> Class C extends Class D

`Has-A` relationships are used to refer to the fact composition has been used when defining a given object.

For example, if defining a class for a computer, speaking in `Has-A` relationships would mean that we say that the Computer class has a Motherboard and Hard Drive (i.e. it has references to other things/objects). Important to note here is that no inheritance is being used, Hard Drive doesn't inherit from a computer, it is part of the definition for a computer.

```
public class Computer {
    HardDrive drive = new HardDrive();
}
```
**Further Reading**
[text](https://stackoverflow.com/questions/36162714/what-is-the-difference-between-is-a-relationship-and-has-a-relationship-in#:~:text=An%20IS-A%20relationship%20is%20inheritance.%20The%20classes%20which,child%20class%20is%20a%20type%20of%20parent%20class.)
<hr>

### Wrapping up
Thanks for sticking around this far! The next article will be the last in the series where we wrap up and look at a final set of crucial ideas.