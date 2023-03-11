---
layout: post
title:  CS Concepts (Part 4)
date:   2023-03-11
description: OOP Basics
tags: Concepts
categories: Concepts
---
Hi and welcome back to the last article in my series on OOP basics. Join me as I re-explore OOP concepts. 

If you've not already done so, take a look at other articles in the series([1](https://thatstatsguy.github.io/blog/2023/CS-Concepts-Part-1/), [2](https://thatstatsguy.github.io/blog/2023/CS-Concepts-Part-2/), [3](https://thatstatsguy.github.io/blog/2023/CS-Concepts-Part-3/)).

<hr>

### 17.	Coupling
One goal of coupling is to measure how independent two pieces of code are. For example,

```
class Area {
    public static void main(String[] args) {
        Square square = new Square(5, 10);
        System.out.println(square.Area);
    }
}

class Square {
    public double Area;

    public Square(float length, float width) {
        Area = length * width;
    }
}
```

In this code example, the two classes are highly coupled to one another. This is bad for several reasons including:
- Any change in Square directly impacts `Area` (e.g. if we add in a new parameter our code will suddenly not compile)


**Further reading**
- [Source 1](https://shouts.dev/articles/understanding-oop-concepts-coupling)
- [Source 2](https://stackoverflow.com/questions/3085285/difference-between-cohesion-and-coupling)

<hr>

### 18. Cohesion

Cohesion refers to the scope of what an object can do. If a class is like a swiss army knife, it's likely it can do a **lot** of different unrelated things. This is known as code with low cohesion i.e. there's no one **single** thing this class is trying to achieve.

The idea of cohesion and coupling ties in to the single responsibility principle which is foundational to the SOLID development principles, specifically the idea of single responsibility.
<hr>

### 19. Association vs Composition vs Aggregation

The three topics mentioned in the title are specify different ways in which objects are related to one another. Note that in UML diagrams these relationships are noted differently. I'm going to use the example mentioned in the source as inspiration.

**Association**

This is the most generic relationship - in essence this defines a one to many relationship, many to one, or many to many relationship. As an example:

> For example Managers and Employees, multiple employees may be associated with a single manager and a single employee may be associated with multiple managers.

The idea here is that an many objects of type A may be associated to a single object of type B and vice versa. In UML diagrams, this is noted by an arrow.
{% include figure.html path="assets/img/umlarrow.png" class="img-fluid rounded z-depth-1" %}

**Aggregation**
We narrow down the association into a specific sub type called composition where one object is comprised of several other objects.

> For example, departments and employees, a department has many employees but a single employee is not associated with multiple departments.

It's clear here that for a given object A will contain a collection of Object B. In UML modelling this is noted with a "white diamond".
{% include figure.html path="assets/img/UMLWhiteDiamond.png" class="img-fluid rounded z-depth-1" %}


**Composition**

We take Aggregation even further with Composition and specify that an object can't exist outside of it's "parent". For example

> For example, the company and company location, a single company has multiple locations. If we delete the company then all the company locations are automatically deleted. The company location does not have their independent life cycle, it depends on the company object's life (parent object).

In UML modelling, this would be represented with a black diamond.
{% include figure.html path="assets/img/UMLBlackDiamond.png" class="img-fluid rounded z-depth-1" %}

**Further Reading**
- [Source 1](https://www.c-sharpcorner.com/UploadFile/ff2f08/association-aggregation-and-composition/#:~:text=Aggregation%20is%20a%20weak%20Association.%20Composition%20is%20a,is%20a%20requirement%20in%20both%20Composition%20and%20Aggregation.)
<hr>

### 20. Down vs Upcasting

Polymorphism showed that, at run time, we're able to move between object types (e.g. derived class to bass class as mentioned in the previous article). Casting follows in the same vein. For example, suppose we have the following classes.

```
public class Knife
{
    // implementation details omitted.
}

public class SantokuKnife: Knife
{
    // implementation details omitted.
}
```

**Downcasting** is the process by which we try and make our object specification more specific i.e. if we had an object of type `Knife`, downcasting would be (trying) to make the current object a `SantokuKnife` object. This may work if the current object can indeed be casted as this object. The `As` statement comes in handy to do this check (see sources).

**Upcasting** is going in the other direction, we're trying to convert a derived class into it's base class i.e. going from a `SantokuKnife` to `Knife`. This is a lot easier for the compiler to do since we already know it's possible to move back to the base class because the class inherits it!

**Further Reading**
- [Source 1](https://stackoverflow.com/questions/1524197/downcast-and-upcast)
<hr>

### Wrapping up
We've covered a lot of ground in this series. If you're anything like me who tends to forget this stuff, feel free to use this as a cheat sheet (I know I will). If you think there's something obvious I've missed, feel free to file an issue on my github repo with what I should have covered. Until next time, cheers! 