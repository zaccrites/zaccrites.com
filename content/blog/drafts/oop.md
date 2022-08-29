---
title: "Programming Paradigms: Object-oriented"
date: 2022-08-22
draft: true
tags: [programming, object-oriented-programming]
# slug: magical-world-of-sat-solvers-part-2
---

Series explaining different programming paradigms.
Start with the most popular, object-oriented,
and how it differs from the procedural style that preceded it.




Scala has been called a "pure" OOP language.

https://stackoverflow.com/questions/46822956/how-is-it-possible-to-have-a-purely-object-oriented-language

## The Three Pillars of OOP

In classic object oriented programming,
there are three foundational pillars.

### Inheritance

A class can inherit from another class. The inheriting class, called the subclass, gains from its parent class all its methods and attributes. The subclass can then add additional methods and attributes, or override the parent class’s methods to change behavior. The overriding method can still call the parent’s original version via the super function.

### Polymorphism

Polymorphism is Greek for “many-shaped” or “multiple-forms”. A polymorphic interface is one which provides multiple implementations of an interface behind that single interface. For example, a function might expect a parameter of class Base, but will also accept instances of DerivedA and DerivedB if those classes inherit from Base. A method of Base overridden in DerivedA and DerivedB is said to be invoked polymorphically through a base class reference to the Base class.

### Encapsulation

Classes and modules encapsulate private state and implementation details. This internal state and internal API is complexity hidden from the user.

Python has no private keyword like C++ or Java do. The convention is to add a leading underscore to private methods and attributes, like this: _my_private_variable.

The interpreter won’t stop you from accessing a private member of a class or module, but doing so is dangerous. An internal API can change at any time without warning, and classes may behave unexpectedly if their internal state is modified in way that it does not expect.



## Implementation Details: C++

C++ is often described as the object-oriented counterpart
to the classic procedural C language that came before it.
But what does C++ give us that C doesn't?
Can we replicate C++ in plain old C?

To say that C++ is a feature-rich language is much like saying
that the Pacific ocean has some water in it.
Therefore we will restrict our focus to the three main OOP pillars:
inheritance, polymorphism, and encapsulation.

TODO: single and multiple inheritance data members, `vtable` polymorphism,
compile-time encapsulation (the easiest by far)
