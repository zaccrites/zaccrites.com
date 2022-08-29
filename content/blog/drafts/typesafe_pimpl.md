---
title: PImpl Type Safety
date: 2022-08-22
draft: true
tags: [c++]
# slug: magical-world-of-sat-solvers-part-2
---

The PImpl pattern is a popular way to hide the implementation details
of a C++ class.

{{< highlight cpp >}}
// database.hpp
class Database {
public:

private:
    struct Impl;
    std::unique_ptr<Impl> m_pImpl;
}

// database.cpp
#include <sqlite3.h>
#include "database.hpp"

struct Database::Impl {
    sqlite3* db;
}

{{< / highlight >}}

However, this implementation has a problem. Did you catch it?
Here is a more obvious example:

{{< highlight cpp >}}

#include <memory>
#include <iostream>

struct MyClass {
    MyClass(int value) : m_pData {std::make_unique<int>(value)} {}

    int get() const {
        return *m_pData;
    }

    void set(int value) {
        *m_pData = value;
    }

    void set_anyway(int value) const {
        *m_pData = value;
    }

private:
    std::unique_ptr<int> m_pData;
};


int main() {
    const MyClass foo (5);
    std::cout << foo.get() << "\n";  // "5"

    // does not compile:
    //   "passing 'const MyClass' as 'this' argument discards qualifiers"
    // foo.set(10);

    // but this *does* compile
    foo.set_anyway(10);
    std::cout << foo.get() << "\n";  // "10"
}
{{< / highlight >}}

The compiler correctly rejects the attempt to call a non-`const` method
of a `const` object, but lets us modify `m_pData` in the `const` method
for some reason. Why?

The problem is that it isn't the data `m_pData` points to that is protected
by `const MyClass`, it is `m_pData` itself.
It's like the difference between `const int*` and `int const*`--
the former is a mutable pointer to `const int`,
while the second is a `const pointer` to a mutable int.

What this means for the PImpl pattern is that calling a `const`
method on the public-facing class and forwarding them to the private `Impl`
object can call *non*-`const` method overloads on `Impl`.

{{< highlight cpp >}}
int PublicClass::Impl::privateMethod() {
    m_counter += 1;
    return m_counter;
}

int PublicClass::Impl::privateMethod() const {
    return m_counter;
}

int PublicClass::get() const {
    return m_pImpl->privateMethod();
}
{{< / highlight >}}

In this example, calling `PublicClass::get() const` on a `const PublicClass` is legal.
However `UniquePtr<Impl>::operator->() const` returns `Impl*`, not `const Impl*`.
Therefore the compiler is forced to call `PublicClass::Impl::privateMethod()`
instead of the intended `PublicClass::Impl::privateMethod() const`.

This is one of the worst kinds of bugs, because it is both totally unexpected
and totally silent, and in a real situation it may not be obvious that the
wrong method is being called.


## Fixing `operator->()` and `operator*()`

The solution is to create a wrapper class that adds overloads
for `operator->()` and `operator*()` which return a `const` pointer
and `const` reference, respectively.

{{< highlight cpp >}}
#pragma once
// "Because when const member function calls a function through
//  a non-const member pointer, the non-const overload of the
//  implementation function is called, the pointer has to be wrapped in
//  std::experimental::propagate_const or equivalent."
//
//  https://en.cppreference.com/w/cpp/language/pimpl
//
// Until propagate_const is more widely available, this wrapper class
// can provides a const-correct wrapper around a subset of unique_ptr's
// interface that is useful for the PIMPL idiom.
//
#include <memory>
#include <type_traits>
// Note that the public class will need to define a constructor and destructor
// in the source file where the Impl is defined,
// even if the default behavior is desired,
// because the compiler-generated constructor and destructor need a complete
// Impl type definition to create and delete the underlying unique_ptr.
template <class Impl>
class Pimpl
{
public:

    template <typename... Args>
    Pimpl(Args&&... args) :
        m_pImpl {std::make_unique<Impl>(std::forward<Args>(args)...)}
    {
        static_assert( ! std::is_array_v<Impl>, "Impl cannot be an array");
    }

    ~Pimpl() = default;

    // Pimpl classes should not be implicitly copyable.
    // If they need to be copied then the user can define an explicit
    // copy constructor which clones the underlying impl:
    //
    // Impl clone {*pImpl};
    //
    Pimpl(const Pimpl&) = delete;
    Pimpl& operator=(const Pimpl&) = delete;

    Pimpl(Pimpl&& other) : m_pImpl {other.m_pImpl.release()}
    {
    }

    Pimpl& operator=(Pimpl&& other)
    {
        m_pImpl.reset(other.m_pImpl.release());
        return *this;
    }

    Impl& operator*()
    {
        return *m_pImpl;
    }

    const Impl& operator*() const
    {
        return *m_pImpl;
    }

    Impl* operator->()
    {
        return m_pImpl.get();
    }

    const Impl* operator->() const
    {
        return m_pImpl.get();
    }

    operator bool() const
    {
        return static_cast<bool>(m_pImpl);
    }

private:
    std::unique_ptr<Impl> m_pImpl;

};

{{< / highlight >}}

This implementation adds a few other convenience features:

* It forwards its constructor arguments to `std::make_unique`,
  so instantiating it is a little prettier:

  {{< highlight cpp >}}
  // Before
  PublicClass() : m_pImpl(std::make_unique<Impl>(1, 2, 3)) {}

  // After
  PublicClass() : m_pImpl(1, 2, 3) {}
  {{< / highlight >}}

* It deletes the default copy constructor and copy-assignment operator,
  as these are rarely useful for a class using the PImpl pattern.
  If the user needs to clone the PImpl they can do it explicitly:
  `Impl clone {*pImpl}`.


# TODO: section name

One thing to keep in mind is that the caller *must* define a constructor
and destructor in the C++ source file where the `Impl` class is defined.
This is because the constructor and destructor need a complete type definition
for `Impl`, and the implicit compiler-generated ones do not have that information
if they are "defined" in the header file alone.
The default implementations are acceptable:

{{< highlight cpp >}}
// Example.hpp
class Example {
public:
    Example();
    ~Example();
private:
    struct Impl;
    Pimpl<Impl> m_pImpl;
};


// PublicClass.cpp
struct Example::Impl
{
    int x;
};

// Without a complete definition for Impl the compiler has no way
// of knowing how much memory to allocate or free for it.
Example::Example() = default;
Example::~Example() = ~default;

{{< / highlight >}}

This restriction isn't unique to the `Pimpl` class described above.
The same is true when using a plain `std::unique_ptr` as well.
