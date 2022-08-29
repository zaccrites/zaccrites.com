---
title: On Error Resume Somewhere
date: 2022-08-22T00:30:45-06:00
draft: true
tags: [programming]
# slug: magical-world-of-sat-solvers-part-2
series: []
description: "Examining the strengths and weaknesses of error handling strategies"
icon: "fa-solid fa-bomb"
---

{{< quote source="The Zen of Python" style="em" link="https://peps.python.org/pep-0020/">}}
Errors should never pass silently.\
Unless explicitly silenced.
{{< / quote >}}



What are some of the ways we can handle errors?

## Panic

If there is an error, don't even try to handle it.
Just crash the program.

In certain circumstances, this may be reasonable.
For an error which is truly beyond our powers to mitigate,
it may be the only option.

Go, Rust, Python, C/C++, others.

### Assertions

`assert` falls into this category.

Assertions are a useful defensive programming technique.
They make assumptions about the program state explicit and verifiable at runtime,
so problems that might otherwise go unnoticed become obvious.
Assertions have two main problems, however.
The first is that they are often removed from a release build,
so they will not do their job in that configuration and errors in program state may not be reported.
The second is that, like a `panic`, they crash the program.
They are unrecoverable.
It is technically possible to catch an `AssertionError` in Python,
but this is considered bad style.
If it is possible to handle an error locally or in a higher context, throw an exception instead.
If it is not possible to handle the error at all,
and checking it in a release build is not required, use an assertion.


## Return Null

Bad idea. It is the [billion dollar mistake](https://en.wikipedia.org/wiki/Null_pointer#History)
of computer science and has been a thorn in the side of this profession for many years.
In a language like Java, where everything is a reference,
it is *not safe* to call a method on a class unless you check that it is not null first.
The compiler allows this unsafe situation to be the norm,
even though it is completely possible to statically analyze the code at compile time
to determine whether a null reference should be allowed or not.
Therefore these checks end up happening at runtime.
Or they don't happen at all, in which case the program will work until
one day someone passes a null reference to a function that didn't expect it.

C# existed for nearly 20 years before nullability checking was added to the language.

In C++ a `Foo&` is safe to work with. A `Foo*`? Not so much. Better compare it to `nullptr` first!


Runtime null reference checks are a waste of resources and clutter up code.
Leaving them out is a risk, unless programmers are vigilant about never passing nullable
references to the function.
Human vigilance is not reliable.
The solution is for machines to do the checking,
but at that point why not just integrate it into the compiler and do away with nullable references
in the first place?

Instead, use *opt-in* nullability. That is, use **optional** types which explicitly opt-in
to the possibility of a variable having no value.
This extends even to traditionally non-nullable types like `int`,
where `0` and `None` may have distinct semantic meaning.



It is inconvenient to have to check for nulls all the time.
If this idea seems appealing, consider using an Optional type instead:

* C++ `std::optional`
* Rust `Option`
* Python `Optional` (which is really just a safety net around `None`, but aren't these all, really?), mypy required
* Haskell `Maybe`
* C# (since which version?) `?` suffix

## Return an Error Code

Either global ones like `errno` (gross), or returning them as "out" parameters.


## Throw an Exception

{{< quote >}}
In C++ a function can return anything. Just `throw`!
{{< / quote >}}

I can't remember where I heard this joke, but there's a bit of truth to it.
Exceptions are sort of like returning from a function,
but they don't "return" to where you came from.
It's really a `goto` to a `catch` block somewhere that the function throwing the exception
can't possibly know about.

For this reason, exceptions get a lot of criticism for being an unpredictable
error handling scheme that is difficult to reason about.
It is and it isn't.
The purpose of an exception is for a function, upon encountering an *exceptional* situation,
to throw up its hands and say
"I can't deal with this fatal situation, so I'm deferring responsibility".

That responsibility falls on the code which called the function in the first place.
It's not unreasonable for a function that opens a file
to give up if that file does not exist.
What else can we ask of it? It doesn't know our intentions.
But we as the developers can wrap that function call in a `try/catch` block
to handle the situation correctly.

The issue is *scope*. The scope of the error handling should be small.
In Python this small-scale exception handling is so common that it is considered idiomatic.
The "it's easier to ask forgiveness than permission" (EAFP) appears everywhere:

{{< highlight python >}}
try:
    value = int(some_string)
except ValueError:
    value = some_default_value
{{< / highlight >}}

Compare this to the "look before you leap" (LBYL) strategy:

{{< highlight python >}}
if some_string.isdigit():
    value = int(some_string)
else:
    value = some_default_value
{{< / highlight >}}

This isn't nearly as elegant. And there is a price to pay:
checking that `some_string` is an integer is something that the `int` function
must do anyway, so we're doubling the work.
There is also a price to pay to handle the exception when it does occur,
but if the conversion is likely to succeed then the EAFP version performs better.

If an exception is bubbling up the stack so far that it's difficult to keep track
of who should be handling the error and why, that's indicative of a design problem.
It should be clear who has the responsibility for handling an exception.
It may not be the immediate caller of the throwing function,
but it should be in the vicinity.

Sometimes the responsibility for "handling" the error is to wrap it in another
error for presentation at a different level of the API.
It would be inappropriate to allow a naked `ValueError` for a specific entry of a string
given to a JSON parser library to bubble all the way up to the application code.
Instead we would expect the parser library to catch the `ValueError` and throw a
`JsonSyntaxError` with more detailed information.

### On Error Resume Next

The title is a reference to
[a style of error handling](https://docs.microsoft.com/en-us/dotnet/visual-basic/language-reference/statements/on-error-statement)
available in Visual Basic.
The `On Error` statement allows the user to select a line to jump to if an exception is thrown.
In VB.NET this is discouraged in favor of the more structured `Try/Catch`,
but exists because that's how error handling was done in Visual Basic 6.

{{< highlight vb.net >}}
Function Foo(a As Double, b As Double) As Double
    On Error GoTo ErrorHandler
    Return A / B
ErrorHandler:
    Return 0.0
Function Sub
{{< / highlight >}}

What the `On Error` statement is best known for these days, however,
is `On Error Resume Next` which essentially disables error handling completely
by instructing the program to go to the next statement on error.
This is obviously not recommended at all.



## Error Objects

### Result Types

* Rust `Result<T, E>`
* Haskell `Either`
* Go `error`

