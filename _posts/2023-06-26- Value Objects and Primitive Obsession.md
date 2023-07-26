---
layout: post
title:  Value objects
date:   2023-06-26
description: 
tags: C# til PrimitiveObsession ValueObjects
categories: C#
---

In software development, it all to easy to fall into the trap of a phenomenon called "primitive obsession". The term refers to a coding anti-pattern where we as developers lean too heavily on primitive data types throughout our code base. For those not familiar, examples of primitives include integers, strings, or booleans. This can leading to various issues such as lack of clarity, reduced maintainability, and increased chances of bugs. However, by utilizing value objects we can effectively mitigate primitive obsession and enhance the overall quality of our code. This article explores what primitive obsession is and how value objects can be leveraged in C# to overcome this.

## Understanding Primitive Obsession

As mentioned previously, primitive obsession occurs when programmers rely too heavily on primitive types, neglecting the creation of meaningful abstractions for domain-specific concepts. Instead of creating dedicated classes or structures, they store and manipulate data using simple types like integers or strings. While these types are essential, they lack the expressive power and encapsulation capabilities offered by custom value objects.

## The Limitations of Primitive Types

Using primitive types extensively can lead to several issues. Firstly, primitive types lack semantic meaning. For instance, consider a  parameter called `int numberOfDays`. It's unclear what the purpose of this integer is without further context. Additionally, there's no built-in validation or behavior associated with primitive types, making it easier for invalid or inconsistent data to propagate through the system. For example, if you use a string to capture an email address there are no rules in the string primitive which stop you assigning an empty string which is obviously an invalid email. Moreover, primitive types don't encapsulate related operations and behavior, leading to scattered logic throughout the codebase.

## Introducing Value Objects

Value objects provide a powerful tool for combating primitive obsession. In C#, value objects are implemented using classes or structures, which encapsulate data and behavior related to a specific concept. Unlike primitive types, value objects:
- enforce invariants i.e. the integrity of the data encapsulated within a value object is guaranteed as the data must always follow the rules or conditions specified by the object
- allow us to build an object with structural equality
- build a data type provide meaningful behavior i.e. if you can't create an object you'll know why because we can provide meaningful errors.
- making the code more expressive and maintainable since a developers will focus less on catching edge cases in code and more on solving problems.

## Creating Value Objects in C#:

The example below is a simple way to define a value object. I've created a [more detailed example of this on my github repo](https://github.com/thatstatsguy/til/tree/main/ValueObjects) which showcases how you can achieve structural equality with value objects.

To create a value object in C#, we can define a class or structure that represents the concept we want to model. Let's consider an example of a value object representing a DateOfBirth:

```
public class DateOfBirth
{
    private readonly DateTime _value;

    public DateOfBirth(DateTime value)
    {
        // Add validation logic here if necessary
        _value = value;
    }

    public int CalculateAge()
    {
        // Calculate the age based on the date of birth
    }

    // Add other relevant methods and properties here
}
```
In this example, the DateOfBirth class encapsulates the concept of a person's date of birth. It ensures that only valid dates are accepted during construction, and it provides a method to calculate the person's age.

## Benefits of Value Objects

Using value objects offers several benefits. Firstly, the code enhances code clarity by providing self-descriptive types that represent domain concepts. Think back to the email address example, if you work with an email address type (in a code base you trust) there's a good chance there's at least an `@` check in the code. In terms of domain-driven design, value objects also creating a shared language between developers and domain experts.

## Conclusion

Primitive obsession can be a hinderance to code quality and maintainability, but value objects provide an interesting solution to the problem. By utilizing value objects in C#, we can create expressive, encapsulated, and domain-driven abstractions to solve our day to day problems while mitigating the drawbacks of primitive types. Next time you encounter primitive obsession in your C# code, consider using value objects to bring balance to the force.

Until next time :)

## Further Resources
- [Nick Chapsas Video on Result Types](https://www.youtube.com/watch?v=YbuSuSpzee4)
- [Milan Jovanović](https://www.youtube.com/watch?v=P5CRea21R2E)

## Additional Notes
- This article was inspired by the videos mentioned in the additional resources. A sprinkling of Chat GPT was used here an there to spice things up ;)