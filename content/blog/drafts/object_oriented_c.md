---
title: "Object Oriented C"
date: 2022-08-22
draft: true
tags: [programming, object-oriented-programming]
# slug: magical-world-of-sat-solvers-part-2
---

In [a previous article]({{< ref "blog/drafts/oop.md" >}})
I discussed the way a C++ compiler might implement C++'s
object-oriented features.
That is, what a C++ compiler might have to add to C to get
the foundational OOP features of inheritance, polymorphism,
and encapsulation.

That got me thinking-- what would it take to replicate
these features in plain-old C itself?



## DIY C++ OOP in C

C++ is often described as the object-oriented counterpart
to the classic procedural C language that came before it.
But what does C++ give us that C doesn't?
Can we replicate C++ in plain old C?

To say that C++ is a feature-rich language is much like saying
that the Pacific ocean has some water in it.
Therefore we will restrict our focus to the three main OOP pillars:
inheritance, polymorphism, and encapsulation.

To that end, these are our desired outcomes:

* Single inheritance, including data members and "methods"
* Runtime polymorphism through base-class pointers
* Encapsulation of private data members


## DIY Inheritance

A `struct` may not inherit from another `struct` in C,
so "inheritance" looks more like composition.

{{< highlight c >}}
struct Base {
    int a;
    int b;
};

struct Derived {
    Base base;
    int c;
};
{{< / highlight >}}

Furthermore, functions declared to work on `Base` pointers
will not accept pointers to `Derived`,
so "methods" are not derived either.

TODO: type punning
https://stackoverflow.com/questions/25664848/unions-and-type-punning


### DIY Polymorphism

Create our own `vtable` of function pointers on the base struct.


### DIY Encapsulation

https://stackoverflow.com/questions/2672015/hiding-members-in-a-c-struct

C `struct` fields are all public,
so they are all fair game for users to read or modify at any time.
The best you can do is prefix field names with an underscore,
as is commonly done in Python.

Or is it?

C++ provides the `private` keyword to enforce encapsulation rules.
Accessing a private member from outside the context of the class
(or a `friend` function) is forbidden at compile time.
However the private fields must still be declared in the class
declaration in the public header file, because while users of the class
are not allowed to access its private member variables,
they must still know the *size* of the class in order to allocate
space for it in memory.
This requirement "leaks" private information about the class to clients,
which may not be desirable. In particular, it forces clients C++
source files to transitively include other header files which declare
the classes of internal private members, which can inflate compile times.

{{< highlight cpp >}}
// foo.h
#include <sqlite3.h>
class Database {
public:
    bool open(const std::string& path);
private:
    sqlite3* p_db;
};
{{< / highlight >}}

Enter the "[PImpl](https://en.cppreference.com/w/cpp/language/pimpl)"
or "pointer-to-impl" pattern.
In this scheme a class's private members are truly hidden from client code
by obscuring them through a pointer to a private class.

{{< highlight cpp >}}
// database.hpp
class Database {
public:
    bool open(const std::string& path);
private:
    class Impl;
    Impl* pImpl;
};

// database.cpp
#include "database.hpp"

class Database::Impl {
public:
    sqlite3* db;
};
{{< / highlight >}}

Under this scheme, client code doesn't see the internal members of
the class at all. There is a slight performance penalty because the
internal members of the class
