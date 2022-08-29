---
title: Designing Software Interfaces
date: 2022-08-22
draft: true
tags: [software-design, go, python]
# slug:
---



## Go Interfaces

[Go](https://go.dev/) is an extremely polarizing programming language.
One of its many controversial features is its use of *structural*, rather than *nominal*, interfaces.
That is, rather than explicitly declaring that a class implements an interface
[as you might do in C#](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/types/interfaces),
Go instead *infers* whether a type implements an interface by [examining its method set](https://go.dev/tour/methods/10).[^1]

{{< highlight csharp >}}
// Implementing an interface in C#

public interface IReader
{
    byte[] Read();
}

public class ArrayReader : IReader
{
    public byte[] Read()
    {
        ...
    }
}
{{< / highlight >}}

{{< highlight go >}}
// Implementing an interface in Go

type Reader interface {
    Read(b []byte) (n int, err error)
}

struct ArrayReader {
    ...
}

func (r *ArrayReader) Read(b []byte) (n int, err error) {
    ...
}

{{< / highlight >}}

The implicit nature of implementing interfaces in this way often surprises people.
The compiler is supposed to verify that all classes which claim to implement an
interface actually follow through on that promise.
Removing the `implements` keyword or an equivalent means that the compiler will not perform that check[^2], so what's the value?

For Go, at least, it's worth remembering that it's just just `struct`s
that can implement interfaces.
Functions, channels, and even built-in types can all implement an interface
implicitly by defining new functions which take them as a receiver (sometimes indirectly, through a type alias).

{{< highlight go >}}
package main

import "fmt"

type Greeter interface {
	Greet() string
}

type StructGreeter struct {
	name string
}

func (g *StructGreeter) Greet() string {
	return fmt.Sprint("Hello from a struct named ", g.name)
}

func MyFuncGreeter() string {
	return "Hello from a function"
}

type GreeterFunc func() string

func (f GreeterFunc) Greet() string {
	return f()
}

type GreetCount int

func (c GreetCount) Greet() string {
	return fmt.Sprint(c, " greetings from an integer")
}

func greet(g Greeter) {
	greeting := g.Greet()
	fmt.Println(greeting)
}

func main() {
	structGreeter := &StructGreeter{
		name: "Bob",
	}

	greet(structGreeter)  // "Hello from a struct named Bob"
	greet(GreeterFunc(MyFuncGreeter))  // "Hello from a function"
	greet(GreetCount(1000))  // "1000 greetings from an integer"
}

{{< / highlight >}}

It would be awkward to force an `implements` syntax on these use cases because the implementation isn't about the *structure* of the data type, it's about the *operations* it can perform.
There isn't anything special about the `GreetCount` type.
It's just an integer.
The special interface-implementing sauce comes from the
functions that can take it as a receiver.


## It's Not Just About Classes

I fear that the popularity of object-oriented programming
has created a perception that everything must be a class.
How can you implement an interface without `class Foo implements Bar`?

The truth is that C was doing this many, *many* years ago
using function pointers.

{{< highlight c "hl_lines=26 32 39 56-58">}}
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

// Function pointers are C's interfaces
typedef char* (*Greeter)(void* context);


struct StructGreeter {
    char* name;
};

void init_struct_greeter(struct StructGreeter* greeter, const char* name) {
    greeter->name = malloc(128);
    strncpy(greeter->name, name, 128);
}

void destroy_struct_greeter(struct StructGreeter* greeter) {
    free(greeter->name);
}


char* struct_greeter_greet(void* context) {
    struct StructGreeter* greeter = context;
    char* buffer = malloc(128);
    sprintf(buffer, "Hello from a struct named %s", greeter->name);
    return buffer;
}

char* func_greeter_greet(void* context) {
    char* buffer = malloc(128);
    sprintf(buffer, "Hello from a function");
    return buffer;
}

char* int_greeter_greet(void* context) {
    int* value = context;
    char* buffer = malloc(128);
    sprintf(buffer, "%d greetings from an integer", *value);
    return buffer;
}


void greet(Greeter g, void* context) {
    char* greeting = g(context);
    printf("%s \n", greeting);
    free(greeting);
}

int main() {
    struct StructGreeter struct_greeter;
    init_struct_greeter(&struct_greeter, "Bob");

    int greet_count = 1000;

    greet(struct_greeter_greet, &struct_greeter);  // "Hello from a struct named Bob"
    greet(func_greeter_greet, NULL);  // "Hello from a function"
    greet(int_greeter_greet, &greet_count);  // "1000 greetings from an integer"

    destroy_struct_greeter(&struct_greeter);
}
{{< / highlight >}}

Arguably things have improved somewhat over the years
(`void` pointers throw type safety out the window),
but the idea is the same:
an interface doesn't have to mean the keyword `interface`, it means exactly what it says-- it's an *interface* that other code *interfaces with*. All that is required is way to plug one function into another.

<!-- A plain function signature is just as valid,
and all we need is a way to pass objects of different
shapes and sizes to a receiving interface expecting -->
<!-- the arriving object to act a certain way. -->
The Go compiler obfuscates the `context` pointer,
but it's right there in the function receiver.



[^1]: [TypeScript](https://www.typescriptlang.org/docs/handbook/type-compatibility.html) also uses structural subtyping, and also checks a type's method and attribute set to determine if it satisfies an interface. TypeScript classes do support an `implements` keyword, however, to [explicitly check that a class implements one or more interfaces](https://www.typescriptlang.org/docs/handbook/2/classes.html#implements-clauses).

[^2]: It is actually possible to force the Go compiler to check that a type implements an interface by casting `nil` to the interface type and assigning to a throwaway value at file scope, like this: `var _ Reader = (*ArrayReader)(nil)`. If `ArrayReader` or `Reader` ever change such that the interface is no longer implemented, the Go compiler will produce an error since `ArrayReader` is no longer usable as a `Reader`.
