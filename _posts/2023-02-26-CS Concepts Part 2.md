---
layout: post
title:  CS Concepts (Part 2)
date:   2023-02-26
description: OOP Basics
tags: Concepts
categories: Concepts
---
Hi and welcome back to part 2 of my OOP basics series. In this series you're joining me as I re-explore OOP concepts. The topics we'll be looking at originate (in part) from a list I picked up in this article [article](https://sites.google.com/site/steveyegge2/five-essential-phone-screen-questions).

If you've not already done so, take a look at part 1 where I cover some of the basics.

<hr>

### 5.	Methods vs Functions
The primary different between methods and functions is in their association and scope. 

**Similarity**
- Both allow code to be written which can be executed one or more times.

**Differences**
- Methods are associated with an object e.g. if you've got a class `Person`, the class may have several methods belonging to it.
- Functions are standalone pieces of functionality which are called by name. They don't belong to a class and (in many scenarios) may be global in nature.

**Further reading**
- [Source 1](https://stackoverflow.com/questions/155609/whats-the-difference-between-a-method-and-a-function)
- [Soure 2](https://www.geeksforgeeks.org/methods-vs-functions-in-c-with-examples/)

<hr>

### 6. Inheritance

If you've ever started development work on something and say to yourself `hey ... object X and Y are kind of in the same family of "things" but differ in one or two aspects` you'll quickly stumble on to the idea of inheritance.

Inheritance is based on the idea that "things" in the world typically belong to a common family. For example, my dog and your dog are different breeds, but they are both dogs with features bearing some similarity. Similarly, the knife I have in my kitchen may differ from yours in style/shape/intended use case, but both still "belong" to the knife family.

Inheritance allows us to do is model the common features for a given family of "things" and give classes that shared functionality/properties.

Continuing with the knife example. I may have a class `Knife`
```
public class Knife{
    protected string Name { get; }
    protected float Length { get; }
}
```
you can declare another class which inherits the property from `Knife` i.e. if I have a Japanese knife class with an additional property `Family` then the use of inheritance may look as follows.

```
public class JapaneseKnife : Knife
{
    string Family;
    public JapaneseKnife(string knifeFamily)
    {
        Family = this.Name + knifeFamily;
    }
}
```
Notice how `Knife` is referenced at the top of the class as well as how this `Name` is referenced even though it isn't defined in the class itself. Note that the class `Knife` is commonly referred to as a `base` class.

**Further Reading**
- [Source 1](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/inheritance)

<hr>

### 7. Multiple Inheritance

We can take the idea of inheritance one step further with multiple inheritance. It's the idea a class can inherit from two or more parent classes. Languages like C# don't natively support this, but if it did you may find your `Knife` class inheriting from classes, such as `Blade` and `Handle`. Have a look at C++ if you want to know more.

<hr>

### 8. Abstract Classes

There may be scenarios in your application where it doesn't make sense to instantiate a base class. Using the Knife class from (6) as an example - you might not want programmers to instantiate an instance of `Knife` because no knife on your website can have just the name and length.

To enforce this rule, classes can be marked as `abstract`. In `Knife` class example, the class definition would be tweak to

```
public abstract class Knife
....
```

<hr>

### 9. Virtual vs Abstract Methods

It's best to start here by mentioning that both virtual and abstract methods are both methods belonging to a class. Abstract methods are methods belonging to a class with **no implementation** provided. This means that any class inheriting from it needs to provides an **override** for it in order to use it. For example, our `Knife` class from earlier can contain an abstract method `GetShippingCosts`. The logic for this method needs to be implemented by any child class.

```
public abstract class Knife{
    protected string Name { get; }
    protected float Length { get; }
    public abstract float GetShippingCosts();
}
```
It's worth noting that abstract methods can only belong to an abstract class in C#.

By contrast, virtual methods allow us to provide some kind of "default" implementation for a method that sub-classes are free to overwrite. For example, our `Knife` class can be changed as follows.

```
public class Knife{
    protected string Name { get; }
    protected float Length { get; }

    public virtual float GetShippingCosts()
    {
        return Length * 2;
    }
}
```

Note here that virtual methods don't need to be used only in abstract classes. What virtual methods do for us is to provide some kind of default implementation which can be changed by sub-classes if it makes sense to do so.


**Further Reading**
- [Source 1](https://stackoverflow.com/questions/391483/what-is-the-difference-between-an-abstract-method-and-a-virtual-method)
- [Source 2](https://stackoverflow.com/questions/14728761/difference-between-virtual-and-abstract-methods)

<hr>

### 10. Static methods
While we're on a mission to describe all the different types of methods, one important one is the static method. Static methods are using to provide functionality belonging to a class which **doesn't** depend on the class being instantiated or any instance of the class. Using the knife example, we can define a static method to calculate the shipping costs for a supplied knife length.

```
public class Knife{
    protected string Name { get; }
    protected float Length { get; }

    public static float GetShippingCosts(float length)
    {
        return length * 2;
    }
}
```

Important to note here is that static methods **cannot** be used in conjunction with properties which are not static. For example, we cannot use `Length` in our method as it's not static (is created when we instantiate our class)

<hr>

### Wrapping up
That's all for now! We're making our way nicely through the list. In the next article we'll keep digging into more CS concepts, such as Interfaces, Finalizers/Destructors and method overloading!