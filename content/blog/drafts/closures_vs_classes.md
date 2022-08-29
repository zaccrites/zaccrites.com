---
title: Closures vs. Classes
date: 2022-08-22
draft: true
tags: [software-design, python]
# slug:
---


# A Common Interface

I've
[written previously]({{< ref "blog/drafts/interfaces.md#its-not-just-about-classes" >}}) on how, when you need a common interface with multiple implementations, a class explicitly implementing that interface isn't always required. All that is required is a function with the same signature. The example in C uses a `char* (*Greeter)(void*)`, which is just [C gibberish](https://cdecl.org/?q=char*+%28*Greeter%29%28void*%29) for a function with a single parameter of type `void*` returning `char*`. The use of `void` pointers is very common in C in these situations because for the interface to be common, it has to allow for any use case, and those use cases will have different configuration and state. The only way to get that state into a common function pointer is through type erasure, and it's on the programmer to ensure that the type of the object passed through the `void` pointer interface is what the other side expects.

Object-oriented languages offer a solution-- just use a class. The required context is pre-packaged into the class itself, so providing a common function signature interface is easy. All the configuration occurs at initialization time.

{{< highlight csharp >}}

public interface IGreeter
{
    string Greet();
}

public class NameGreeter : IGreeter
{
    private string Name { get; init; }

    public NameGreeter(string name)
    {
        Name = name;
    }

    public string Greet()
    {
        return $"Hello from a class named {Name}";
    }
}

public class IntGreeter : IGreeter
{
    private int Count { get; init; }

    public NameGreeter(int count)
    {
        Count = count;
    }

    public int Greet()
    {
        return $"{Count} greetings from a class"
    }
}

{{< / highlight >}}


## Functions Returning Functions

Functional programming offers an alternative.
Programming languages in this category have *first-class functions*, which means that functions are objects in their own right.
Instead of a class, return a new function with the state embedded within it.

{{< highlight python >}}
def make_name_greeter(name: str) -> Callable[[], str]:
    return greeter() -> str:
        return f"Hello from a function named {name}"

def make_count_greeter(count: int) -> Callable[[], str]:
    return greeter() -> str:
        return f"{count} greetings from a function"

greeters = [make_name_greeter("Bob"), make_count_greeter("1000")]
for greeter in greeters:
    print(greeter())

# "Hello from a function named Bob"
# "1000 greetings from a function"

{{< / highlight >}}

A function with embedded state like this is called a **closure**.
