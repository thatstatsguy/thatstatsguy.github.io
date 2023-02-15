---
layout: post
title:  CS Concepts (Part 1)
date:   2023-02-15
description: We all have to start somewhere
tags: Concepts
categories: sample-posts
---
Hi there and welcome to my first blog post. In this blog post we're starting a journey to re-explore CS concepts. Having worked in a functional language (F#) for almost three years, it's easy to forget some specifiecs OOP principles. 

In this series I'm hitting the refresh button, to re-learn things I've forgotten and keep my mind sharp. I learn best when I'm explaining something to others, so if you're reading this I hope you find some value here! 


The topics I'll be covering are based on and article I read about in this [article](https://sites.google.com/site/steveyegge2/five-essential-phone-screen-questions).

Without further ado, let's jump into the list. I'll try my best to build the ideas incrementally, but no promises!

<hr>

### 1.	Classes and Object (and the difference between them)
Classes lay the groundwork for almost everything in OOP... it's in the name after all. To explain the difference between them, let's talk in examples. Say I'm building a program which takes your name and surname and does something with it (the what part isn't important here).

A class is nothing more than a theoretical representation of the thing you are trying to represent/model. In our example, we're trying to represent people in code so our class could be of 'type" Person and this type could contain properties like Name and Surname. 

In C#, the codified representation of a person can look like this.

```
public class Person
{
    public string Name;
    public string Surname;
}
```
If you're anything like me when I started programming, I would have got caught up qith questions like
> what the heck is the word public doing there

and not got much further. Try ignore that for now for this discussion. It's important, but not important for you to understand classes.

Objects are examples of a class (commonly referred to as instances) - if the goal of the class was to theoretically represent a person then an instance of a class (an obejct) would be a `Person` object with `Name = Chris` and `Surname = Dunderdale`.

<hr>

### 2. Constructors
Following on from point 1, you might be wondering

> How does one go about creating an instance of a class in code?

What we need is a way to "construct" an example of a new `Person`. Constructors allow us to do this. An example of this is given below.

```
public class Person
{
    public string Name;
    public string Surname;

    public Person(string name, string surname)
    {
        Name = name;
        Surname = surname;
    }
}
```

Now admitedly this constructor isn't doing much, it takes in two parameters (name and surname) and assigns it to `Name` and `Surname`. By definining the constructor in this way it allows us to do all sorts of things like checks before the value is assigned. 

For example, if we included an additional property cellphone number, we would be able to use our constructor to validate that number given to us is indeed valid.

Going back to the original objective, creating instances of a class, we can now use the constructor to create two new objects.

```
var chris = new Person("Chris", "Dunderdale")
var nelson = new Person("Nelson", "Mandela")
var name = chris.Name // will return Chris
```

<hr>

### 3. Inheritance
In the context of OOP programming, you start to notice that a lot of objects share the same properties over and over. Inheritance allows us to standardize what those shared properties are.

For example, if we're trying to represent various kinds of food, we can use inheritance to represent it by defining what's known as a "base class" from which everything else inherits.

```
public class Food
{
    public string Name;
    public string Calories;
}
```

this allows us to incrementally build up more complex structures without duplicating definitions of properties. For example, if we were to define a class for snacks and include a property for whether or not it contains nuts, the definition would be as follows.
```
public class Snack : Food
{
    public bool ContainsNuts;
}
```
notice the lack of duplication. When being used in code, every snack will still have the Name and Calories properties available (assuming they've got the public meethod - more on this in the next point)

<hr>

### 4. Encapsulation
In the Point 2, we saw that we could access the properties of objects using the "dot" notation in `var name = chris.Name`. These properties are called `public` properties since they could be accessed **outside** of the class.

Encapsulation allows us as programmers to control who can see and access properties of a class. This is also called data hiding.

Borrowing from dotnet [dotnet tutorials](https://dotnettutorials.net/lesson/encapsulation-csharp/), let's using a bank account example.
```
class BankAccount
{
    public int AccountNumber;
    public int Balance;
    
    public void GetBalance()
    {
        // some code here
    }
    public void WithdrawAmount()
    {
        // some code here
    }
    public void Deposit()
    {
        // some code here
    }
}
```

In this example, we want to ensure that we've got good control over who can see the bank balances! Encapsulation allows us to control who can see our bank balance value. In order for us to control this, we can use the `private` keyword.

`private int Balance`

what this means is that the value can only be accessed and or modified from **within** the class and not outside the class. Trying to access balance from outside the class would produce an error. e.g.

```
// lets assume we have a constructor takes in the account number and automatically sets a balance of zero.
var bankAccount = new BankAccount(1234);
var balance = bankAccount.Balance //the compiler is going to shout at you because it can't find Balance.
```
Doing this allows us to have fine grained control over who can access properties in a class. Beyond using `private`, there's many other ways one can encapsulate properties/methods.

To borrow from [Microsoft](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/access-modifiers)
- public: Members of a class can be accessed by any other code in the same assembly or another assembly that references it.
- private: Members can be accessed only by code in the same class.
- protected: Members are available within the same class as well as to the classes that are derived from that class via inheritance.
- internal: Members can be accessed by any code in the same assembly, but not from another assembly. I.e. if two projects are compiled down to DLLs, you can't access an internal class from a different project dll.
- file: Member is only visible in the current file.
- protected internal: Members can be accessed by any code in the assembly in which its declared, or from within a derived class in another assembly.

e.g if you have a class `ABC` with property `protected internal xyz` you can access the property in the same assembly/dll and any class which derives from `ABC` in a different dll.

- private protected: The private protected members can be accessed by types derived from the class that is declared within its containing assembly.
