---
title: "RAII: The Killer Feature of C++"
date: 2022-06-11
draft: true
tags: [c++]
# slug: magical-world-of-sat-solvers-part-2
---

Above all others, I believe RAII is the most important C++ feature.
More important than exceptions, classes, and template put together
(not that those aren't awesome features too).

## "Resource Acquisition is Initialization"

RAII is a bit of an awkward acronym.
The idea is that, by acquiring a "resource" during an object's initialization
(i.e., in it's constructor), we can control using the object's lifetime.



TODO: talk about what RAII is, how it works


## Mutex Locks


## Managing C Resources

e.g. closing sqlite database, finalizing `sqlite3_stmt`, etc.


## Smart Pointers

C++11 added many features to C++ and its standard library to bring it into the modern era.
Arguably the most important of these was
[move semantics]({{< ref "blog/drafts/rule_of_five.md#move-semantics" >}}),
which enabled several important performance and safety improvements.

One of these safety improvements was the **smart pointer**.
A smart pointer is like a normal pointer, but it manages the lifecycle of the object it points to automatically.
That is, a smart pointer can delete the object it points to when it falls out of scope.
Smart pointers prevent memory leaks by leveraging RAII.

### Unique Pointers

TODO

[**unique pointer**](https://en.cppreference.com/w/cpp/memory/unique_ptr)

TODO: explain custom deleter

Using a custom deleter can remove much of the boilerplate around creating
one-off RAII wrapper classes.

{{< highlight cpp >}}
template <typename T>
using raii_ptr = std::unique_ptr<T, void(T*)>

raii_ptr<sqlite3> open_database(const std::string& path) {
    sqlite3* db = sqlite3_open(path.c_str());
    return {db, [](sqlite3* db) { sqlite3_close(db); }};
}
{{< / highlight >}}




### Shared Pointers

[**shared pointer**](https://en.cppreference.com/w/cpp/memory/shared_ptr)

A shared pointer can also distribute
[**weak pointers**](https://en.cppreference.com/w/cpp/memory/weak_ptr)
to its contents, which do not increase the reference count.
Users of a weak pointer must be careful when dereferencing it, however,
as it can return `nullptr` if the object it points to is deleted
(i.e., its reference count goes to zero) while the weak pointer is still held.


## Similar Features in Other Languages

https://news.ycombinator.com/item?id=24648406

https://swiftcoder.wordpress.com/2009/02/18/raii-why-is-it-unique-to-c/

### Python

`with` statement is inferior

comment on `__del__` and C#'s `Finalize()` and how they are more for when the object is
reclaimed by the garbage collector (or maybe if explicitly using `del` in Python),
not when the object goes out of scope

What are finializers from the `weakref` package?
https://docs.python.org/3/library/weakref.html

### C#

`using` statement is inferior

How do C# "destructors" work? They're only for dealing with "unmanaged" resources.


### Go

Go's `defer` mechanism is close, but not quite.
It only works at the function's scope, and is opt-in.
It schedules a function call for when the current function exits.

TODO: come up with a good example of locking and then unlocking a mutex

{{< highlight go >}}
func example() {
    defer lock.Release()
}
{{< / highlight >}}


### D

apparently, through a `scope` keyword

### Rust

`Drop` trait

https://doc.rust-lang.org/rust-by-example/trait/drop.html
https://doc.rust-lang.org/std/ops/trait.Drop.html

### Visual Basic 6

WTF is this real?

https://stackoverflow.com/a/166461
