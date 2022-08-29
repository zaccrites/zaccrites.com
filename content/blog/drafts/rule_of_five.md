---
title: The Rule of Five
date: 2022-07-20
draft: true
tags: [c++]
# slug: magical-world-of-sat-solvers-part-2
---


https://en.cppreference.com/w/cpp/language/rule_of_three



## The Rule of Three


### The Destructor


### The Copy Constructor


## The Rule of Five

### The Move Constructor


### Move Semantics


## The Rule of Zero

Most classes do not deal with resource ownership.
These classes do not need to define custom destructors,
copy constructors, copy-assignment operators,
move constructors, or move-assignment operators.
These classes can leverage the automatic memory management features
of other classes like `std::string` and `std::vector`.
